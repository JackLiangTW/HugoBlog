+++
date = "2026-04-21T11:35:00+05:30"
title = "NestJS 深入筆記:動態 Module、Scope 與雙層快取"
draft = false
showonlyimage = false
weight = 1
showmoretile = false
tags = "NestJS,Nodejs,Redis"
+++

## 在 bootstrap 中測試 service

```ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  const svc = await app.get(GcashCrawlerService);
  await svc.runDailyCrawler();        // 用於臨時驗證邏輯
  await app.listen(3000);
}
```

## Module 的幾個坑

- **`@Global()` module**:不需靠 import 順序,只要有人 import 過就全域可用。
- **全域 CacheModule 仍需在 root 掛 import**,否則被注入時找不到 provider。
- **`registerAsync()` / `forRootAsync()`**:等 config 準備好才 import 下一個 module。`ConfigModule.forRootAsync()` 先解析完,後面才載入依賴它的 module。

## 動態 Module(根據參數開關功能)

```ts
@Module({})
export class PlatformLineModule {
  static forRoot(includeJob = true): DynamicModule {
    const imports = [TypeOrmModule.forFeature([PlatformLine])];
    const providers: Provider[] = [PlatformLineService];
    if (includeJob) {
      imports.push(ScheduleModule.forRoot());
      providers.push(PlatformLineJob);
    }
    return {
      module: PlatformLineModule,
      imports,
      providers,
      exports: [PlatformLineService],
    };
  }
}
```

呼叫方 `PlatformLineModule.forRoot(false)` 即可關掉 cron。

## Scope / Provider

| Scope                 | 行為                                                 | 情境 |
|-----------------------|------------------------------------------------------|------|
| `Scope.DEFAULT`       | 整個 app 生命週期單一實例(singleton)                | 多數 service |
| `Scope.REQUEST`       | 每個 HTTP 請求新實例                                  | 需要 request 隔離(如 tenant context) |
| `Scope.TRANSIENT`     | 每次注入都新實例;不依賴 HTTP context                 | 背景 worker、排程任務 |

`useFactory` 搭配 `scope: Scope.TRANSIENT` → 每次注入都執行 factory;預設只執行一次(結果被 cache)。

## 兩層快取(Memory + Redis)

```ts
@Injectable()
export class MemoryCacheService {
  async get<T>(key: string): Promise<T | null> {
    const mem = this.memCache.get<T>(key);
    if (mem) return mem;                        // L1 hit
    const redis = await this.redisCache.get<T>(key);
    if (redis) this.memCache.set(key, redis, defaultMemoryTtl); // 把 Redis 值放回 L1
    return redis;
  }

  async set(key: string, value: unknown, ttlRedis = 60, ttlMem = 5) {
    if (value == null) return;                   // null 不寫
    this.memCache.set(key, value, ttlMem);
    await this.redisCache.set(key, value, ttlRedis);
  }

  async del(key: string) {
    this.memCache.del(key);
    await this.redisCache.del(key);
  }
}
```

重點:

- Memory TTL 通常遠短於 Redis TTL(例 5s vs 60s),避免節點間失步。
- `null` 不寫快取,否則下次會讀到快取住的 `null`。
- 寫/刪要**同時**維護兩層,否則 L1 會留污染值。
- 有循環依賴時用 `forwardRef(() => MEMORY_CACHE_TOKEN)`。

## `@MethodCache` 裝飾器快取(per-method)

```ts
@MethodCache({
  ttl: 60 * 60,                                           // 1 小時
  key: (target, [userId, lotteryId, gameTypeId]) =>
    `userOdds:${userId}:${lotteryId}:${gameTypeId}`,
})
async getUserOdds(userId: string, lotteryId: string, gameTypeId: string) {
  return this.thirdPartyApi.getGameOdds(...);
}
```

- 複合維度一律塞進 key,避免不同參數拿到同一筆快取。
- 搭配 `@RemoveMethodCache` 或在變更時主動 `this.cache.del(key)`,否則髒資料會留一整個 TTL。
- 使用場景:外部 API 貴或容易 timeout,資料變動不頻繁(賠率、匯率、設定表)。

## Cron 排程

```ts
@Cron('0 9 * * *')          // 每天早上 09:00 執行一次(不是每分鐘!)
@Cron('*/12 * * * * *')     // 每 12 秒一次(6 欄位含秒)
```

驗證工具:<https://tool.lu/crontab/>。
