# AI SDK

推荐赛事方在赛事的 README 中附上 AI SDK 的 Git仓库 链接，让选手们能通过 Git 轻松获取最新的 AI SDK 代码、文档 等。我们认为这比在平台上提供「下载 AI SDK」的功能更方便选手使用。

## How it works

HipAI 使用 Docker 构建、运行 AI。

SDK 程序目录会被绑定到 `/sdk` 目录，AI 程序目录会被绑定到 `/app` 目录。

### build AI

源码文件命名为 `src`，后缀名与上传时的后缀名相同。

构建容器请将构建产物保存在 AI 程序目录中。

### run AI

游戏逻辑容器命名为 `game_logic`。
