+++
date = "2026-04-21T10:40:00+05:30"
title = "Docker / Docker Compose 常用指令與 volumes 筆記"
draft = false
showonlyimage = false
weight = 1
showmoretile = false
tags = "Docker,DevOps"
+++

## 基礎指令

```bash
# 進入執行中的 container
docker exec -it <container_id|name> bash
docker exec -it apk-version-crawler bash -c "pm2 list"

# 進入尚未啟動的 image(偵錯用)
docker run -it --entrypoint /bin/bash <image_name>

# 刪除 image
docker rmi --force <image_name_or_id>

# 停止並清理未使用的 container / image / network
docker system prune -a
```

## Dockerfile 小技巧

- `COPY` 會尊重 `.dockerignore`,想排除檔案寫入該檔。
- 多階段 build:把 build stage 與 runtime stage 分開,只把 artifact 複製到 runtime image,大幅減少 image 大小。

## docker-compose volumes

### Bind mount(掛本機路徑)

```yaml
services:
  app:
    volumes:
      - type: bind
        source: ./data
        target: /app/data
      # 或簡短語法
      - ./data:/app/data
```

使用情境:**SQLite 檔案必須用 volumes 映射才會持久化**,否則容器刪除後資料跟著消失。

### Named volume(自動管理)

```yaml
services:
  db:
    volumes:
      - db-data:/var/lib/mysql

volumes:
  db-data:                  # 不指定 device → 存在 docker-managed 位置
  db-data-bind:             # 指定 device → 綁到專案目錄
    driver_opts:
      type: none
      o: bind
      device: ./db-data
```

## 生命週期指令

```bash
docker-compose up -d            # 背景啟動
docker-compose up --build       # 已存在的 image 也會強制重 build
docker-compose down             # 停止+移除 container;image 保留
docker-compose down --rmi all   # 連 image 也一起清
docker-compose logs -f <svc>    # tail 某個 service 日誌
```

## 本機 dev stack 範例(MySQL + phpMyAdmin + Redis)

```yaml
version: '3.8'
services:
  mysql:
    image: mysql:8.0.29
    environment:
      MYSQL_ROOT_PASSWORD: __MYSQL_PASSWORD__
      TZ: Asia/Taipei
    ports: ["3306:3306"]
    volumes: ["mysql-data:/var/lib/mysql"]

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    environment:
      PMA_HOST: mysql              # ← 用 service 名稱,不是 localhost
      PMA_USER: root
      PMA_PASSWORD: __MYSQL_PASSWORD__
      UPLOAD_LIMIT: 300M
      MEMORY_LIMIT: 512M
    ports: ["8083:80"]
    depends_on: [mysql]

  redis:
    image: redis:7
    ports: ["6379:6379"]

volumes:
  mysql-data:
```

重點:

- 容器間互連用 **service 名稱**(`mysql:3306`),不是 `localhost`。
- `depends_on` 只控制**啟動順序**,不等待 MySQL ready;若 app 啟動時要等 DB,加 healthcheck 或在 app 內做 retry。
- Redis 預設無認證,dev 可直連 `redis-cli -h 127.0.0.1`。

## 清理 / 重部署片段

```bash
docker-compose down \
  && docker rmi --force <image_name> \
  && docker-compose up -d --build
```

常搭配 CI/CD 部署腳本使用。若只要重啟內部 process,不需重 build,可用:

```bash
docker exec <container> bash -c "pm2 reload all"
```
