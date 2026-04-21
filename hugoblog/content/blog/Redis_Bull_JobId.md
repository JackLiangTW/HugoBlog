+++
date = "2026-04-21T11:55:00+05:30"
title = "Redis + Bull Queue:jobId 去重機制與事件觸發規則"
draft = false
showonlyimage = false
weight = 1
showmoretile = false
tags = "Redis,Bull,Queue,Nodejs"
+++

## jobId 去重機制

```ts
await queue.add(QueueName, data, { jobId: data.meterId });
```

- 同一個 `jobId` 同時間只會存在一個 active job。
- 若 job **還在執行中**(active / delayed / waiting / failed-未處理),再次 add 同 jobId **不會建立新 job**——Bull 把 key 當作 Redis lock。
- 等 `@OnQueueCompleted` 或 `@OnQueueFailed` 觸發、job 從 Redis 被清掉後,下一次 `add` 才會成功。

## 事件觸發規則

| Processor 狀況                          | 觸發事件                  |
|-----------------------------------------|---------------------------|
| `@Process` 內 `throw` / reject          | `@OnQueueFailed`          |
| `@Process` 正常 return(含 return undefined) | `@OnQueueCompleted` |

所以想自己把 job 標記為失敗就 `throw`,想標記完成就正常 return——不要兩個都做。

## 用 redis-cli 查 Bull queue 狀態

Bull 預設用 Redis DB 1(不是 DB 0 的一般應用資料):

```bash
# 進入 redis container
docker exec -it <redis-container> redis-cli

127.0.0.1:6379> SELECT 1
127.0.0.1:6379[1]> KEYS "bull:*"              # 列出所有 queue 相關 key
127.0.0.1:6379[1]> KEYS "bull:BetQueue:*"     # 只看 BetQueue
127.0.0.1:6379[1]> HGETALL bull:BetQueue:425  # 看 jobId=425 的內容
```

常見 key 結構:

- `bull:<QueueName>:id` — 自增 id
- `bull:<QueueName>:<jobId>` — hash,儲存 `data`, `opts`, `progress`, `returnvalue`, `failedReason`…
- `bull:<QueueName>:waiting` / `:active` / `:delayed` / `:failed` — 各狀態的 list/zset

## 踩坑提醒

- 生產環境想防重複觸發(如帳單、通知),務必傳入 `jobId`,不要靠 `removeOnComplete` 自己防。
- `@OnQueueFailed` handler 要能處理**無限 retry**情境;Bull 預設會 retry 直到 `attempts` 用完。
- Scheduler 起的 repeat job 刪除要用 `queue.removeRepeatable(name, opts)`,`removeJob` 只會移除當下這次。
