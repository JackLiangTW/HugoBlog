+++
date = "2026-04-21T11:10:00+05:30"
title = "Kibana Discover 查詢:KQL 語法與踩過的坑"
draft = false
showonlyimage = false
weight = 1
showmoretile = false
tags = "Kibana,Log,DevOps"
+++

## Discover 基本流程

1. Kibana 左側 → **Discover**。
2. 右上選時間範圍(`now-15m`、`now-7d/d` 等)。
3. 上方搜尋框輸入 KQL;左側用 **Add filter** 建結構化條件。

## KQL 語法重點

KQL 與 Lucene 不同;常見 pipeline 語法(Observability / Security 端):

```kql
AppLogs
| where request_body has '3a117b9e-6c38-10db-8003-5c092426e0ef'
| where path : "POST /api/orders"
| sort @timestamp desc
| limit 50
```

- `has` 做部分字串比對;`==` / `:` 做完全匹配。
- 多個條件用 `|` 串 `where`,比一整行用 `and` 可讀性高。

## 兩個踩過的坑

- **查 `request_body` 要帶前綴**:有時 log 是 JSON-escape 過後的字串,原本的 `34717482768` 會被 tokenize 成 `"x2234717482768"` 之類的格式,得用 `x2234717482768` 才能命中。看 raw document 確認 tokenizer 後的實際值。
- **先 Add filter,再細化**:常用欄位(`env`, `service`, `level`)用 filter UI 固定住比較快,搜尋框只放本次特殊條件,避免一行塞到底又長又錯。

## 分享 / 保存

- URL 本身會把查詢與時間範圍 encode 在 query string → 直接複製丟給同事即可。
- 需要固定的視圖做成 Dashboard 或 Saved Search,比每次手打 KQL 好。
