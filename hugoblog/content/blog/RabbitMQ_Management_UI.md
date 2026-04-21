+++
date = "2026-04-21T11:45:00+05:30"
title = "RabbitMQ Management UI 重點與新 user 建立"
draft = false
showonlyimage = false
weight = 1
showmoretile = false
tags = "RabbitMQ,Queue,Docker"
+++

## Docker 部署

快速起一個帶 Management UI 的 RabbitMQ(image 用 `rabbitmq:3-management-alpine`):

```yaml
services:
  rabbitmq:
    image: rabbitmq:3-management-alpine
    ports:
      - "5672:5672"     # AMQP,程式連線用
      - "15672:15672"   # Management UI
    environment:
      RABBITMQ_DEFAULT_USER: admin
      RABBITMQ_DEFAULT_PASS: __MQ_PASSWORD__
    volumes:
      - rabbitmq-data:/var/lib/rabbitmq

volumes:
  rabbitmq-data:
```

- 如果單純用 `rabbitmq:3`(無 management)也可以跑,但沒 UI 時排查幾乎是黑箱。
- **Volume 一定要掛**:容器刪掉時 queue、exchange、user 都會跟著消失。
- `RABBITMQ_DEFAULT_USER/PASS` 只會在**資料目錄為空**時生效,重啟 container 不會覆蓋既有設定;日後要改密碼走 `rabbitmqctl change_password`。
- Management UI 第一次就用上面設的 `admin:__MQ_PASSWORD__` 登入。

## Management UI

- URL:`http://<host>:15672`
- 預設帳密:`guest / guest`(**只能從 localhost 登入**;想從其他機器連要額外建 user)

## 主要頁面用途

| 頁面          | 看什麼 |
|---------------|--------|
| Overview      | 訊息 rate、node 狀態、memory / disk 水位 |
| Connections   | 連上的 client;逐個看 channel 數、idle 時間 |
| Channels      | 每個 channel 在 ack、publish 速率;卡住通常在這看得出 |
| Queues        | 每個 queue 的 ready/unacked/rate;判斷堆積 |
| Exchanges     | binding、route key,檢查 routing 是否正確 |
| Users / Vhosts / Policies | 權限與 dead-letter / TTL 策略 |

## 除錯常見觸發點

- `unacked` 一直上升 → consumer 沒 ack、或 prefetch 太高卡 worker。
- `ready` 堆積 → consumer 處理不過來或沒啟動。
- 連線數狂漲 → client 沒重用 connection / channel。

## 建立新 user(最小範例)

```bash
# 進入 container 或 rabbitmqctl 所在 host
rabbitmqctl add_user myapp __MQ_PASSWORD__
rabbitmqctl set_user_tags myapp management
rabbitmqctl set_permissions -p / myapp ".*" ".*" ".*"
```
