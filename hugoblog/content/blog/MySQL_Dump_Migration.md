+++
date = "2026-04-21T11:30:00+05:30"
title = "MySQL 備份還原、外鍵級聯與 Unique Index 變更 migration"
draft = false
showonlyimage = false
weight = 1
showmoretile = false
tags = "MySQL,資料庫"
+++

## 備份 / 還原 / 複製

```bash
# Dump
mysqldump -h 127.0.0.1 -P 3306 -u root -p__MYSQL_PASSWORD__ apollo > apolloDump.sql

# 砍掉舊 DB(小心!)
mysql -h 127.0.0.1 -P 3306 -u root -p__MYSQL_PASSWORD__ -e "DROP DATABASE IF EXISTS mars;"

# 建新 DB 並匯入
mysql -h 127.0.0.1 -u root -p__MYSQL_PASSWORD__ -e \
  "CREATE DATABASE mars CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
mysql -h 127.0.0.1 -u root -p__MYSQL_PASSWORD__ mars < apolloDump.sql
```

注意:

- `-p` 與密碼**中間不能有空格**(`-p123456`);有空格會解讀成「用密碼空字串、再指定一個 database」。
- 生產環境不要在 cmdline 帶密碼(`ps` 看得到),改用 `--defaults-extra-file=~/.my.cnf`。
- `utf8mb4` 才支援 emoji 與中日韓全字元;`utf8` 是 MySQL 的舊別名(只到 3 bytes)。

## 外鍵 / 級聯刪除(EF Core 範例)

```csharp
b.HasOne<PaymentChannel>()
 .WithMany()
 .HasForeignKey(x => x.PaymentChannelId)
 .OnDelete(DeleteBehavior.Cascade);   // 父刪 → 子自動刪
// 反之 DeleteBehavior.Restrict 保留子表
```

## Unique Index 變更的 migration 陷阱

**情境**:原本 `UNIQUE(MerchantId)`,要改成 `UNIQUE(MerchantId, Currency)`。

直接 drop 再 add 會撞到「刪舊 index 時下面還沒有新 index,資料可能暫時出現重複 merchant」。正確順序:

1. `ALTER TABLE ADD COLUMN Currency ...`(先把新欄位加好,暫時允許 NULL 或給 default)。
2. `CREATE UNIQUE INDEX IX_m_c ON ...(MerchantId, Currency);`
3. `DROP INDEX IX_m ON ...;`

Down migration **要完全反向**(先重建舊 index,再 drop 新 index,再 drop column),否則 rollback 也會失敗。

## percona-toolkit

正式環境做 schema 變更時值得裝:

- `pt-online-schema-change` — 不鎖表做 DDL(蓋影子表、用觸發器同步)。
- `pt-archiver` — 把舊資料分批歸檔到其他表 / 檔案,避免一次 DELETE 炸掉主從。
- `pt-query-digest` — 分析 slow log,找熱點 query。
