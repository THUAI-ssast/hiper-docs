# Worker

Worker 负责构建、运行用户程序，确保它们能正常构建、运行，并处理好它们的输出产物。实例：编译 AI、运行对局。

Hiper 中使用 Docker 运行用户程序。可用的编译环境、运行时都由用户提交 `Dockerfile` 构建，Worker 会根据用户提交的 `Dockerfile` 构建镜像。

worker 的工作流程：

1. 获取任务
2. 修改状态信息，表示正在执行
3. 执行任务
    1. 获取任务所需信息
    2. 起容器，执行任务
    3. 等待任务完成，获取任务输出，保存与修改相关信息
4. 修改状态信息，表示执行完成
5. ACK 消息，准备获取下一个任务

## 获取任务

Worker 从消息队列中获取任务。

### 任务调度

??? note "问题分析"

    简单的“先来先执行”策略，可能会导致 某些任务长时间占用 worker，导致其他任务无法执行。具体有两类情况需要考虑：

    1. 一大批任务 占满了队列，导致其他任务排在其后、一直等不到这批任务执行完让自己排到队列开头。
    2. 耗时长的任务 长时间占用 worker，导致其他耗时短的任务在外面、一直等不到 worker 被释放出来。

    ---

    考虑改进方案时，「优先级」是一个重要的考虑因素。
    
    首先，「用户手动触发的任务」优先于「赛事脚本自动触发的任务」。赛事脚本触发的对局应当对延时极不敏感，不需要立即执行，只需在空闲时执行即可。
    
    而「用户手动触发的任务」中，「构建」又应当优先于「创建对局」。

    而考虑得再细些的话，「赛事」内的任务又应当优先于「游戏」内的任务。

    「优先级」方案可将 问题1 局限在「同一优先级」的任务之间；无法解决 问题2。

    具体实现方面，可让不同优先级的消息对应不同的消息队列。worker 实例 按照优先级依次尝试获取这些消息队列中的消息，直到获取到消息为止。

    ---

    问题1，讨论后认为，全部任务本就都得执行，优先级相同的任务之间的顺序并不重要。可认为「优先级」方案已解决了 问题1。

    ---

    问题2 只会出现在需运行耗时极长的对局时，实际上不算是必须解决的问题。

    一些可能的解决方案：

    1. 保留部分 worker 专用于跑「构建」这种优先级极高的任务。可使用环境变量控制 worker 监听哪些消息队列。

    但代价是资源浪费。

    2. 允许抢占。例如使用 时间片轮转调度。
    
    被抢占的对局，要么重跑，要么暂停以伺之后恢复。重跑，则目前跑的进度全没了，已经跑了较长时间的话也很浪费，并且会导致无法处理确实需要运行很长时间的对局。暂停、恢复，资源利用充分，但实现起来复杂得多、有很多要考虑的坑（如 游戏逻辑/AI 中如果用到了计时……）。

    暂停、恢复 可能可以使用 Docker 的 pause/unpause 功能来实现。

    ---

    不过，问题1 和 问题2 在每个任务都执行得足够快的情况其实都不是问题。

分为 3 个优先级：用户触发的构建 > 用户触发的对局 > 赛事脚本触发的对局。它们分别对应 1 个消息队列。

## Docker in Docker

本项目需要两层 Docker：第一层 Docker 运行在 host，用于部署本项目；第二层 Docker 运行在本项目内，用于运行用户程序。相关术语叫 "Docker in Docker"(dind)。

??? note "Docker in Docker 的解决方案分析"
    可能的在 worker 实例中使用 Docker 的方案：

    1. 每个 worker 实例都启动一个 Docker
    2. 在 Hiper 项目内部、worker 实例之外启动一个 Docker，供所有 worker 实例共用。
    3. 直接用 host 的 Docker。

    分别探讨这三种方案的可行性。参考：[Using Docker-in-Docker for your CI or testing environment? Think twice.](https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/)

    ---

    > ... a restriction imposed by the Docker daemon, which does not allow its image cache to be shared concurrently among multiple daemon instances.

    因此在每个 worker 实例内部启动一个 Docker 的话，只能靠 Docker Registry 来共享镜像缓存。这基本意味着运行一个 本地的 Docker Registry。
    
    ---

    单独启动一个 Docker 供所有 worker 实例共用。同时跑多场对局时给容器命名可能不太方便；不过除此外似乎没什么问题。

    ---

    直接用 host 的 Docker 的话，主要是隔离性的问题，会污染 host 的 Docker 环境。

考虑到科协可能需要用多台性能较差的机器分布式部署 worker，倾向于使用方案 1。不过要迁移到其他方案也是很容易的。

> [Introductory demo for the Sysbox container runtime](https://asciinema.org/a/kkTmOxl8DhEZiM2fLZNFlYzbo)
>
> [Sysbox Quick Start Guide](https://github.com/nestybox/sysbox/blob/master/docs/quickstart/README.md)

## docker build

构建好的镜像需能持久化，以减少重复构建的开销。

docker image 命名为 `<name>:<tag>`，其中 `<name>` 为 `<domain>-<id>-<operation>`，`<tag>` 为 `<hash>`。

`<domain>` 为 `game_logic` / `ai`，`<id>` 为它们在数据库中的 ID，`<operation>` 为 `build` / `run`。

`<hash>` 为构建镜像时使用的 Dockerfile 的 hash，用于判断是否需要重新构建镜像。

## docker run

在运行前先检查镜像的 hash 是否与当前 Dockerfile 的 hash 相同，若不同则重新构建镜像。

使用 `latest` 镜像。

需要限制以下几个方面，以确保安全性：

- CPU、内存占用
- 运行时间
- 日志长度

使用 `--rm` 参数。

其他参数：-v, --name, --network, ...

## 任务：构建

### 构建游戏逻辑

输入：游戏逻辑 + 构建游戏逻辑的镜像。

输出：构建产物，保存在游戏逻辑的目录中。

### 构建 AI

输入：AI + 构建 AI 的镜像。

输出：构建产物，保存在 AI 的目录中。

## 任务：对局（评测机）

遵从惯例，俗称「评测机」、「judger」。

输入：

- 游戏逻辑 + 运行游戏逻辑的镜像。
- 参与对局的AI列表（内为 AI 的 id） + 运行 AI 的镜像。
- 对局定制信息（JSON），将被直接传递给游戏逻辑。如随机种子，乃至更直接的定制。

输出：

- 比分，按顺序给出参与对局的每个AI的分数，与输入的 AI列表 对应。
- 回放数据。
- 日志。

由 游戏逻辑容器 来决定如何将 AI程序 运行为实例的「玩家」；因为可能出现 THUAI6电子系赛道 那样要打上下两半场才能决定一局胜负的。一份 AI 程序在一场对局中可能会运行多个实例，这种实例称为玩家。

Hiper 允许有不管怎样花里胡哨的游戏流程，只要最终的输出和输入匹配即可。

### 运行流程

1. 准备工作：建立本场对局的 docker network，游戏逻辑容器 与 玩家容器 均加入该 network；创建命名管道，供游戏逻辑容器与评测机通信。
2. 启动游戏逻辑容器（传入「AI列表」与「对局定制信息」）。
3. 对局进行中，监听命名管道来接收 [游戏逻辑容器的指令](/hiper-docs/user/local-admin/game-logic)。
3. 对局结束后的清理工作。
4. 处理容器的输出。

AI 的日志是与输入的 AI列表 对应的，即第 i 个（列表中的下标，从 0 开始） AI 的日志保存在 `player_<i>.log` 中，基于该位置的 AI 先后运行的多个玩家实例的日志会被追加到同一个文件中。AI 通过 `stderr` 输出日志。

游戏逻辑容器命名为 `game_logic`。

> 目前一个 Docker 一次只会运行一个对局，故不用担心同时运行多个对局时的命名冲突。
