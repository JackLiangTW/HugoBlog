+++
date = "2026-04-21T10:30:00+05:30"
title = "ClickHouse 列式資料庫入門:索引、歸檔與 Docker 部署"
draft = false
showonlyimage = false
weight = 1
showmoretile = false
tags = "ClickHouse,資料庫,Docker"
+++

## 定位與選用時機

- **列式儲存**的 OLAP 資料庫,針對「少量欄位、大量列、批次寫」場景極快。
- 適合:訂單歷史歸檔、log 聚合、報表。
- 不適合:OLTP 式頻繁 `UPDATE` / 單列 `DELETE`,行級鎖不存在,刪改都是寫新 part + 後台 merge。
- 用 MySQL 寫、定期把舊資料丟到 ClickHouse,是常見架構。

## 連線

| 介面         | Port | 用途                                 |
|--------------|------|--------------------------------------|
| HTTP         | 8123 | Web UI `/play`、curl、多數 MCP client |
| Native TCP   | 9000 | 程式 driver(Octonica、DBeaver Native) |

```
Host=127.0.0.1;Port=9000;Database=mars_archive;User=default;Password=__CH_PASSWORD__
```

Web UI:`http://<host>:8123/play`。注意 Play 每次只能執行一條 SQL,不支援多行一起跑。

## 索引結構(和一般 RDBMS 不一樣!)

- **稀疏索引**:每 8192 筆(= 1 個 granule)才記一個索引項,索引本身幾 MB 就能涵蓋 TB 資料。
- `ORDER BY` 就是索引的建立依據;若沒指定 `PRIMARY KEY`,`PRIMARY KEY` 自動等於 `ORDER BY`。
- 要用複合索引,`PRIMARY KEY` 欄位必須是 `ORDER BY` 欄位的**前綴**。

```sql
CREATE TABLE mars_archive.orders
(
    OrderNo      String,
    MerchantId   UInt32,
    Status       UInt8,
    CreatedAt    DateTime,
    Amount       Decimal(18, 4)
)
ENGINE = MergeTree
PARTITION BY toYYYYMM(CreatedAt)
ORDER BY (MerchantId, CreatedAt, OrderNo);
```

## Data Skipping Index(二級索引)

選對類型很重要,選錯根本不會幫到查詢:

| 類型                  | 適合欄位特性                      | 範例 |
|-----------------------|-----------------------------------|------|
| `bloom_filter(0.01)`  | 高基數、精確等值查詢              | `OrderNo` |
| `minmax`              | 時間、數值範圍查詢                | `CreatedAt` |
| `set(N)`              | 低基數列舉(幾個到幾十個值)      | `Status`(6~10 個狀態) |

```sql
ALTER TABLE orders ADD INDEX idx_orderno OrderNo TYPE bloom_filter(0.01) GRANULARITY 4;
ALTER TABLE orders ADD INDEX idx_status  Status  TYPE set(16)             GRANULARITY 4;
```

`GRANULARITY N` = 每 N 個 granule 匯總一次索引摘要。大量重複資料要調大,不然 index 反而變肥。

## 常用建議

- **Nullable 欄位會拖慢查詢**:多一個 NULL bitmap 要掃。盡量用特殊值代替(`Int8` 用 `-1`,`String` 用 `'null'`)。
- **不要用多維陣列**。`Array(Array(...))` 在 MergeTree 上有諸多限制。
- **批次寫入**要夠大(每次至少數千筆)才有效率;單筆 insert 會產生超多 part。
- C# 用 `Octonica.ClickHouseClient` + `ColumnWriter` 做批次匯入最快。

## 歸檔 workflow 建議

1. 90 天以前的資料才歸檔(live 資料留在 MySQL)。
2. 批次大小 5000–200 筆看環境調整,太大會撐住 merge、太小 part 碎。
3. 遷移前**先查 ClickHouse 有沒有這筆**(用 bloom index 或主鍵查),才插入;刪 MySQL 前**再驗一次**。
4. 累計刪除超過 50 萬筆時執行 `OPTIMIZE TABLE ... FINAL` 回收磁碟(但這是重操作,夜間或分區跑)。

## Docker 部署要點

- 官方 `clickhouse/clickhouse-server` 有 alpine 版。
- 開機初始化 SQL 放 `/docker-entrypoint-initdb.d/`,或 compose 裡 mount 進去。
- 預設 user 是 `default`,無密碼。**自動 init 時一定要設密碼**,否則 8123 直接被外部打爆。

```yaml
clickhouse:
  image: clickhouse/clickhouse-server:23-alpine
  ports: ["8123:8123", "9000:9000"]
  volumes:
    - ch-data:/var/lib/clickhouse
    - ./init.sql:/docker-entrypoint-initdb.d/init.sql
  environment:
    CLICKHOUSE_PASSWORD: __CH_PASSWORD__
```

## DBeaver 連線

- HTTP Port 8123(UI 用 jdbc-http driver)
- Native Port 9000(jdbc-native driver,速度較快)

兩個 driver 都能寫查詢,差別在大量結果集時 native 省資源。
