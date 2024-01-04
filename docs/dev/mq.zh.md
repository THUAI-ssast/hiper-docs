# 消息队列

技术选型：`Redis Stream`

## Web to Worker

Startup:

```bash
# 若相应的 stream 不存在，会自动创建
XGROUP CREATE build worker_group 0 MKSTREAM
XGROUP CREATE manual_match worker_group 0 MKSTREAM
XGROUP CREATE auto_match worker_group 0 MKSTREAM
```

Send message:

```bash
# 若相应的 stream 不存在，会自动创建
# type: game_logic / ai
XADD build * type <type> id <id>
XADD manual_match * id <match_id>
XADD auto_match * id <match_id>
```

Read message:

```bash
# 若相应名字的 consumer 不存在，会自动在 group 中创建
# BLOCK 0 表示永久阻塞，直到有消息到达
# 使用 Docker 部署时，可考虑使用 $HOSTNAME 作为 consumer_name 以避免重名
XREADGROUP GROUP worker_group <consumer_name> COUNT 1 BLOCK 0 STREAMS build manual_match auto_match > > >
```

Ack message:

```bash
# stream: build / manual_match / auto_match
XACK <stream> worker_group <message_id>
```

## Worker to Web

Send message:

```bash
XADD match_result * id <match_id> replay <replay>
```

Read message:

```bash
XREAD BLOCK 0 STREAMS match_result $
```
