+++
date = "2026-04-21T11:20:00+05:30"
title = "掛機專案架構參考:Processor Factory、多層快取與 Worker 狀態機"
draft = false
showonlyimage = false
weight = 1
showmoretile = false
tags = "架構,NestJS,Redis"
+++

> 記錄這個專案是因為裡面幾個 **pattern 可以套到其他 Node / NestJS 後端**,不是要復刻整個業務邏輯。

## 整體分層

```
┌───────────┐
│ adminSys  │   後台 (AdminJS) ─ 管理方案、賠率、使用者
├───────────┤
│ processor │   第三方 API 封裝 (Factory Pattern)
├───────────┤
│ schedule  │   定時任務 + Redis Queue Worker
├───────────┤
│ modules   │   核心 domain(方案、meter、user-odds、bet-scheme…)
├───────────┤
│ config    │   DB + Redis 載入玩法設定
└───────────┘
```

## Processor Factory 模式

多家第三方平台(fbao、ds、ls、sd…)API 形狀不同,但業務內部只關心「下注 / 查結果 / 拿賠率」。

```ts
interface IProcessor {
  bet(params: BetParams): Promise<BetResult>;
  queryOdds(userId: string, lottery: string): Promise<Odds>;
}

// factory
export function createProcessor(platformCode: string): IProcessor {
  switch (platformCode) {
    case 'fbao': return new FbaoProcessor(...);
    case 'ds':   return new DsProcessor(...);
    case 'ls':   return new LsProcessor(...);
    // ...
  }
  throw new Error(`Unknown platform: ${platformCode}`);
}
```

**優點**:呼叫端 `createProcessor(code).bet(p)`,新增平台只動 factory;測試時可注入 mock processor。

## Schedule(Cron + Redis Queue)

| 模組                  | 工作 |
|-----------------------|------|
| `bet-worker`          | 從 Redis queue RPOP,分流虛擬投注 / 真實投注 |
| `bet-result`          | 監聽 `BetResult` queue,算獎金寫回 |
| `platform-setting`    | `@Cron('*/12 * * * * *')` 每 12 秒拉取採種狀態 |
| `open-code`           | 定時爬開獎 |

失敗重試走 `@OnQueueFailed`,用 retry count 配合 `moveNext()` 管狀態機。

## user-odds 快取(降第三方 API 壓力)

第三方賠率 API 常 timeout,每個請求現拉等於把自家服務拖死。解法:

1. **Entity `user-odds`**:`(userId, lotteryId, gameTypeId)` 複合索引。
2. `@Cron` 每小時預刷賠率;生命週期綁到 refresh token 過期時間(token 更新就強制重刷)。
3. `@MethodCache` 把記憶體快取包住 `getUserOdds()`,TTL 1 小時。
4. 查詢時:**記憶體 → DB → 第三方 API**,拿到後把 DB 寫回並清記憶體快取(讓下次讀到新值)。

效果:高峰時期 95%+ 命中率,第三方 API 呼叫降一個數量級。

## Bet-scheme 狀態機

- `scheme_draft` — 使用者自建方案的草稿。
- `scheme_meter` — 啟動後的「掛機任務」,有狀態欄位(waiting / running / paused / completed)。
- `BetWorkerState` 轉換都經過 `moveNext()`,retry 計數存在 entity 上;失敗超過 N 次直接進 `failed`。

## State-job 服務

集中做:

- Redis queue 註冊與消費。
- 分散式鎖(避免多個 node 同時撈同一個 meter)。
- 啟動時 `start()` 把所有必要 worker 掛起。

## 技術 take-away

- 第三方介接要用 **Factory + Interface**,資源不對稱時是救命稻草。
- 多層快取 + 預刷 + 主動失效,是對付「慢且貴的外部 API」的三件套。
- 任務型態寫成 **狀態機**,比零散 if/else 容易收斂;狀態欄位本身就是 audit log。
- `@Cron` + `@Process` + `@OnQueueFailed` 三個裝飾器配起來就是一個輕量 worker 框架。
