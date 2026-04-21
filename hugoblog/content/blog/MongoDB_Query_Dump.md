+++
date = "2026-04-21T11:25:00+05:30"
title = "MongoDB:find / aggregation / mongodump 速查"
draft = false
showonlyimage = false
weight = 1
showmoretile = false
tags = "MongoDB,資料庫"
+++

## find() 的 sort / skip / limit

即使寫成 `find().limit(10).skip(100).sort(...)`,MongoDB **一律**按照 `sort → skip → limit` 的順序執行(這是效能最佳順序)。想改順序只能改用 aggregation:

```js
db.orders.aggregate([
  { $match: { status: "paid" } },
  { $limit: 100 },          // 先 limit
  { $skip: 10 },
  { $sort: { createdAt: -1 } }
])
```

## Aggregation 常用 stage

```js
db.orders.aggregate([
  { $match: { createdAt: { $gte: ISODate("2024-01-01") } } },

  // 計算每筆的兩個時間差(秒)
  { $project: {
      lottery: "$lotteryId",
      elapsedSec: { $divide: [ { $subtract: ["$betAt", "$createdAt"] }, 1000 ] }
  }},

  // 依 lottery 分組取平均
  { $group: {
      _id: "$lottery",
      avgElapsed: { $avg: "$elapsedSec" },
      cnt: { $sum: 1 }
  }},

  { $sort: { avgElapsed: -1 } }
])
```

常用表達式:`$subtract`、`$divide`、`$sum`、`$avg`、`$count`。

## mongosh 互動查詢

```js
use myDB
db.auth("admin", "__MONGO_PASSWORD__")

db.open_code_240116.find().sort({ expect: 1 }).limit(5)
db.open_code_240116.find({ lotteryId: "251" }).sort({ expect: -1 }).limit(5)

db.users.updateOne(
  { _id: ObjectId("...") },
  { $set: { "profile.nickname": "jack" } }
)

db.users.deleteOne({ _id: ObjectId("...") })
```

## mongodump / mongorestore

```bash
# 在容器內 dump(stdin 保持開著:-i)
docker exec -i <mongo-container> \
  mongodump --username admin --password __MONGO_PASSWORD__ \
            --authenticationDatabase admin \
            --db myDB --out /dump

# 把容器內 /dump 複製出來並打包
docker cp <mongo-container>:/dump ./dump
tar -czvf backup.tar.gz ./dump

# 還原到新 DB,用 nsFrom/nsTo 映射避免覆寫線上
mongorestore \
  --uri="mongodb://admin:__MONGO_PASSWORD__@localhost:27017/" \
  --nsFrom="myDB.*" --nsTo="myDB_test.*" \
  ./dump/
```

重點:

- `mongorestore` 不要直接對線上 DB 跑,先用 `--nsTo` 映射到 `_test` 確認資料再切換。
- `--authenticationDatabase admin` 幾乎永遠要加,否則 auth 會失敗。

## bson → json 批次轉檔

```bash
for f in dump/myDB/*.bson; do
  bsondump "$f" > "${f%.bson}.json"
done
```
