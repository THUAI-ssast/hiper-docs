# 架构

运行时整体架构：

```mermaid
graph TD
  subgraph Frontend
    Brower
  end
  Frontend --> B
  subgraph Backend
    subgraph Web
    B[Web API] --> C[Service]
    D[Contest Script] --> C
    end
    C --> MQ[Message Queue]
    subgraph Worker
      
    end
    MQ --> Worker
    subgraph Data
    C --> Cache
    C --> Database
    C --> File[File System]
    end
    Worker --> Database
    Worker --> File
  end
```

## 技术选型

仅为整体的技术选型，更多细节请参考各个模块的文档。

### 前端

- Language：`JavaScript`
- Web Framework: `Solid.js`
- UI: `Hope UI`
- Build：`Vite`

### 后端

Web:

- Language：`Go`
- Web Framework：`Gin`
- Run JavaScript in Go: `Goja`

Database：`MySQL`

Cache：`Redis`

Message Queue：`Redis` (`Redis Stream`)

> Redis Stream 支持「阻塞式」拉取消息。在读取消息时增加 BLOCK 参数即可。
> 消息是有自己的ID的。
> 消费者处理完消息后，需要执行 XACK 命令告知 Redis，这时 Redis 就会把这条消息标记为「处理完成」。如果消费者异常宕机，肯定不会发送 XACK，那么 Redis 就会依旧保留这条消息。待消费者重新上线后，Redis 就会把之前没有处理成功的数据，重新发给这个消费者。这样一来，即使消费者异常，也不会丢失数据了。
> Redis 可配置持久化，尽量减少 Redis 意外宕机的损失。

Deploy：
- `Docker`
- Containers：
  - web
  - mysql
  - redis
  - worker

File System：`Docker Volume`

## 整体架构细节

需要实现「优先级」，如 手动触发的操作 比 赛事脚本自动触发的 优先级更高；故弄两个消息队列，一个高优先级一个低优先级。
