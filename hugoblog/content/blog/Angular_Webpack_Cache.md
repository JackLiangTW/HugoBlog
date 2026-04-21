+++
date = "2026-04-21T10:15:00+05:30"
title = "Angular CLI webpack 快取造成改動沒效的排除"
draft = false
showonlyimage = false
weight = 1
showmoretile = false
tags = "Angular,Web"
+++

## CLI webpack 快取造成改了沒效

症狀:改動某個 library 的 source 後,`ng serve` 沒反映出來,或 build 拿到舊版本。

原因:Angular CLI 會在 **`~/.angular/cache`** 以及 `node_modules/.cache/angular` 儲存 webpack 增量快取。

解法:

```bash
# 1) 直接砍全域快取
rm -rf ~/.angular/cache
rm -rf node_modules/.cache

# 2) 或在專案 angular.json 關掉 cache
```

```json
{
  "cli": {
    "cache": { "enabled": false }
  }
}
```

推薦:平時保留快取(build 快很多),遇到懷疑是快取問題時再動手清。

## 常用重建指令

```bash
ng build --configuration production --aot
ng serve --host 0.0.0.0 --port 4200 --disable-host-check
```

大型 monorepo 的 workspace lib 改動沒被 HMR 吃到 → 檢查 `tsconfig.paths`、`angular.json` 的 `projects.*.architect.build.options.preserveSymlinks` 設定。
