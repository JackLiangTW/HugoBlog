+++
date = "2026-04-21T12:15:00+05:30"
title = "Node.js Cluster / PM2 模式與 wrk 併發壓測"
draft = false
showonlyimage = false
weight = 1
showmoretile = false
tags = "Nodejs,PM2,Cluster,壓測"
+++

## PM2 的兩種模式

- **Cluster mode**:`pm2 start app.js -i max`,多 worker 共享 TCP 連線;缺點是不能用 Node debug inspector(同一個 port 被多 process 搶)。
- **Fork mode**:`pm2 start app.js`,單一 process;想跨 process 共享快取要搭配 `cluster-shared-memory`(LRU,需設定 `max` 與 `maxAge`)。
- Debug cluster 時解法:暫時停用 cluster,用 `nodemon -r tsconfig-paths/register src/main.ts` 跑單 process 配合 `--inspect`。

## 自製 ClusterService(NestJS 範例)

```ts
// ClusterService.clusterize(bootstrap, 3)
import cluster from 'cluster';
import * as os from 'os';

export class ClusterService {
  static clusterize(bootstrap: () => Promise<void>, workers: number = os.cpus().length) {
    if (cluster.isPrimary) {
      for (let i = 0; i < workers; i++) cluster.fork();
      cluster.on('exit', (worker) => {
        console.log(`worker ${worker.process.pid} died, respawn`);
        cluster.fork();
      });
    } else {
      bootstrap();
    }
  }
}
```

- master 判斷 `cluster.isPrimary` fork N 個 worker,監聽 `exit` 自動重啟。
- worker 執行真正的 `NestFactory.create(AppModule)` 流程。
- 想共享 memory(例如 LRU cache)要另外塞 `cluster-shared-memory`;直接宣告 `new Map()` 會各跑各的、不同 worker 看到不同資料。

## 併發壓測 `wrk`

```bash
brew install wrk
wrk -t12 -c400 -d30s --latency http://localhost:3000/api/xxx
#  -t: 執行緒數   -c: 同時連線數   -d: 持續時間   --latency: 印分位數
```

幾個實務建議:

- `-t` **不要超過 CPU 核心數**,否則 wrk 自己在搶資源,壓測數字失真。
- `-c` 從 100 往上爬,觀察 p99 latency 在哪個值開始飆,就是當前瓶頸。
- 一輪跑至少 30 秒才有意義;太短的話 warmup 會吃掉整段結果。
- `--latency` 印 p50/p90/p99/p99.9,比單看平均值準確得多。

## 相關工具

- **`autocannon`**(純 Node)— 可寫成 npm script,方便塞進 CI。
- **`k6`**(Go)— 用 JS 腳本定義情境,比 wrk 彈性高,適合複雜流程壓測。
- **`ab`**(Apache Bench)— 老牌,單機能跑但受 HTTP/1.0 限制多,不建議新專案使用。
