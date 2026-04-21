+++
date = "2026-04-21T11:00:00+05:30"
title = "GraphQL 前後端開發流程與 Codegen 常見錯誤"
draft = false
showonlyimage = false
weight = 1
showmoretile = false
tags = "GraphQL,Web"
+++

## 前後端開發流程

1. **後端**:定義 resolver、type、enum module;啟動服務(例如 `http://localhost:18055/graphql`)。
2. **前端**:寫 `.graphql`(query / mutation / subscription 檔)。
3. 執行 `npm run codegen` / `yarn codegen`,依 codegen config 向後端拉 schema,產生 `graphql.ts`(含 TS 型別、React hooks 或 Apollo 操作函式)。
4. 前端程式 import `graphql.ts` 使用強型別呼叫。

## Codegen 常見錯誤

- **後端未啟動**:codegen 抓不到 schema → 顯示 `Failed to load schema`。先確認對應 port 有起來。
- **Schema 需要 auth**:codegen config 要加 headers(例 `Authorization: Bearer <token>`),否則被擋。
- **兩端 enum 不一致**:後端 enum 名稱 / 值動過卻沒重跑 codegen,前端會拿到舊型別。codegen 放到 pre-commit / CI 避免漏跑。

## GraphQL Playground / Apollo Sandbox

- 啟動後打 `http://<host>/graphql` 多半會自動進入互動介面。
- 左側 **Docs / Schema** 欄可看所有 query、mutation、type 定義,是寫 resolver 或補 query 時的入口。
- 支援直接在 UI 試跑 query;配合 `HTTP Headers` 帶 auth token 測登入後才能用的 endpoint。

## GraphQL Playground 獨立 App

除了內嵌在後端 endpoint 的 web UI,GraphQL Playground 也有**獨立桌面 App**(Electron 打包),即使後端關掉 `/graphql` 路由也能用:

- 下載:<https://github.com/graphql/graphql-playground/releases>
- 適用情境:
  - Production 關閉 Playground 路由,但臨時想用 GUI 驗 schema / 測 query
  - 同時管理多組 endpoint(dev / staging / prod 或不同微服務)的 query tab
  - 將常用 query 存成 collection,像 Postman 管 REST 一樣管 GraphQL
- 新增 tab 時填 endpoint URL + HTTP Headers(`Authorization: Bearer ...`),體驗與 web 版一致。

> 注意:GraphQL Playground 已進入 maintenance mode,Apollo 官方推薦改用 [Apollo Sandbox](https://studio.apollographql.com/sandbox/explorer)(功能相似,仍持續更新)。新專案可直接從 Sandbox 開始。
