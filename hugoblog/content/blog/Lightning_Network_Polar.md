+++
date = "2026-04-21T11:15:00+05:30"
title = "Lightning Network 本地測試:Polar + lncli 速查"
draft = false
showonlyimage = false
weight = 1
showmoretile = false
tags = "區塊鏈,Lightning,LND"
+++

## Polar 是什麼

Polar(<https://lightningpolar.com/>)本機跑 Docker 的 LN 測試環境,一鍵起出 `bitcoind (regtest) + 多個 LND / Core Lightning node`。常用來開頻道、測發票、發送付款。

## lncli 連線(指定 node、TLS、macaroon)

```bash
lncli -n regtest \
  --rpcserver=127.0.0.1:10001 \
  --macaroonpath=./alice/data/chain/bitcoin/regtest/admin.macaroon \
  --tlscertpath=./alice/tls.cert \
  getinfo
```

參數解讀:

- `-n regtest`:網路名(`mainnet` / `testnet` / `regtest`)。
- `--rpcserver`:node gRPC port(Polar 對每個 node 開不同 port)。
- `--macaroonpath`:權限憑證;`admin.macaroon` 最大權限,呼叫哪個 api 決定要哪個 macaroon。
- `--tlscertpath`:gRPC TLS cert,Polar 每個 node 自己簽一張。

## 常用指令流程(兩個 node 開 channel → 付款)

```bash
# 1) 各自拿 pubkey
ALICE=$(lncli ... getinfo | jq -r .identity_pubkey)
BOB=$(lncli ... --rpcserver=127.0.0.1:10002 getinfo | jq -r .identity_pubkey)

# 2) Alice → Bob 建 peer
lncli ... connect $BOB@127.0.0.1:9735

# 3) 開 channel(單位 satoshis)
lncli ... openchannel $BOB 1000000
# 回傳 funding_txid;regtest 要挖幾個區塊才會 confirm:
bitcoin-cli -regtest generatetoaddress 6 $(bitcoin-cli -regtest getnewaddress)

# 4) Bob 產生發票
lncli ... --rpcserver=127.0.0.1:10002 addinvoice --amt 1000
# → payment_request: lnbcrt10...

# 5) Alice 付款
lncli ... sendpayment --pay_req lnbcrt10...
```

## 在 .NET / Node 應用整合

- 把 `tls.cert`、`admin.macaroon` 複製到 app 專案目錄。
- `appsettings.json` / `.env` 設定:

```json
"Lnd": {
  "Host": "127.0.0.1:10001",
  "TlsCertPath": "./ln-credentials/alice/tls.cert",
  "MacaroonPath": "./ln-credentials/alice/admin.macaroon"
}
```

- 啟動 app 若沒 error log → LN client 已成功 handshake;此時背景會嘗試 subscribe invoice stream。

## 安全提醒

- `admin.macaroon` = LND 全權,**等同私鑰**。別上 git,`.gitignore` 要列上整個 `ln-credentials/` 目錄。
- Prod 連線強烈建議用 **read-only macaroon**(`invoice.macaroon` 只能讀發票,`readonly.macaroon` 只查詢);需要付款再用 `admin` 且另存硬體保險。
- `regtest` 僅本地;上 `testnet` 前記得先在 faucet 拿測試幣,別拿 main 的幣來燒。
