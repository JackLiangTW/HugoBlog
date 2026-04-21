+++
date = "2026-04-21T10:35:00+05:30"
title = "部署腳本範例:rsync + ssh + pm2 / docker-compose"
draft = false
showonlyimage = false
weight = 1
showmoretile = false
tags = "DevOps,部署,Docker,Nodejs"
+++

## 後端(node / pm2 或 docker-compose)

```bash
#!/usr/bin/env bash
set -e
WORKSPACE=/home/app/apk-version-crawler
MACHINE_IP=__INTERNAL_IP__
DOCKER_FLAG=true           # true: docker-compose;false: 宿主 pm2

cd /local/repo
yarn install && yarn build              # 本機編譯
cp -r -f ./ops/prod/. ./                # 複製 prod 專用 Dockerfile / config

# 同步到遠端(排除 logs 與 sqlite)
rsync -azh \
  -e "ssh -o StrictHostKeyChecking=no" \
  --delete \
  --exclude logs --exclude 'sqlite.data.db' \
  ./ root@$MACHINE_IP:$WORKSPACE

# 首次部署才做:建空 sqlite 檔讓 container volume 綁
ssh -o StrictHostKeyChecking=no root@$MACHINE_IP "touch $WORKSPACE/sqlite.data.db"

if [ "$DOCKER_FLAG" = "true" ]; then
  ssh root@$MACHINE_IP "cd $WORKSPACE \
    && docker-compose down \
    && docker rmi --force apk-version-crawler \
    && docker-compose up -d --build"
else
  ssh root@$MACHINE_IP "docker exec apk-version-crawler bash -c 'pm2 reload all'"
fi

# Smoke test
curl -fsS http://$MACHINE_IP:5001/apk-version-crawler/health
```

重點:

- `StrictHostKeyChecking=no` 在 CI 裡可跳過 known_hosts 提示;手動部署還是建議先加 host 認證。
- `rsync --delete` 會把遠端多餘檔案刪除 → 確定 `--exclude` 清單(`logs`、DB 檔)有列齊,否則會毀資料。
- **reload 比 restart 平滑**:`pm2 reload all` 用新 worker 逐個替換,不中斷連線;`restart` 會瞬斷。

## 前端(monorepo 多包 → 打包 mobile → 部署)

```bash
#!/usr/bin/env bash
set -e
WORKSPACE=/home/data/lottery-bot-frontend/beta
MACHINE_IP=__PUBLIC_IP__

# 各子 package 先裝 + build
for pkg in lottery-unit scheme-unit number-tool; do
  ( cd packages/$pkg && yarn ib )
done

# 最終 mobile 前端
cd apps/cloud-bet
yarn build:mobile:beta                  # 結果在 dist/mobile

rsync -azh -e "ssh -o StrictHostKeyChecking=no" --delete \
  ./dist/mobile/ root@$MACHINE_IP:$WORKSPACE/
```

## 除錯片段

```bash
ssh root@$MACHINE_IP "docker ps \
  && docker logs --tail 200 apk-version-crawler \
  && docker exec apk-version-crawler bash -c 'pm2 list && pm2 logs --lines 50 --nostream'"
```

## CI/CD 安全提醒

- `MACHINE_IP` / 密碼 / PAT 放環境變數或 secret store,腳本不要 hardcode。
- 部署後跑一個 health endpoint,失敗就 `exit 1` 讓 CI 標紅——不要只看 `docker-compose up` exit code。
- 回滾策略:保留最近幾個 image tag,部署失敗時 `docker tag prev latest && docker-compose up -d`。
