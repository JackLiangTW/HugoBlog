+++
date = "2026-04-21T12:00:00+05:30"
title = "Telegram Bot 建立、Chat ID 查詢與 setMyCommands"
draft = false
showonlyimage = false
weight = 1
showmoretile = false
tags = "Telegram,Bot"
+++

## 建立 Bot

1. Telegram 找 **@BotFather** → `/start`。
2. `/newbot` → 設定 bot name(顯示名,可重複)。
3. 設定 bot **username**(唯一,結尾必須 `bot`,例如 `mycorp_ops_bot`)。
4. BotFather 回你一串 token:`__TG_BOT_TOKEN__`(例格式 `1234567890:AA...`)。

## 找 chat_id(群組 / 頻道)

最快的方法:

1. 手機:**Settings → Devices → Link Desktop Device**(掃 QR)。
2. 電腦瀏覽器開 <https://web.telegram.org/a>。
3. 點進目標群組 / 頻道 → URL 會變成 `#<chat_id>`,群組常是負數(例 `-4066074156`)→ `__TG_CHAT_ID__`。

另一種:bot 加入群組後 `GET https://api.telegram.org/bot<token>/getUpdates`,回傳裡找 `message.chat.id`。

## sendMessage

```http
POST https://api.telegram.org/bot__TG_BOT_TOKEN__/sendMessage
Content-Type: application/json

{
  "chat_id": "__TG_CHAT_ID__",
  "text": "Hello, world",
  "parse_mode": "MarkdownV2"        // 或 "HTML",支援粗體/連結
}
```

- MarkdownV2 有一堆字元要跳脫(`_ * [ ] ( ) ~ \` > # + - = | { } . !`),建議寫 util 自動 escape。
- 想寄檔案 / 圖片:用 `sendPhoto`、`sendDocument` endpoint(`multipart/form-data`)。

## 註冊 Bot Commands(聊天框左下的 Menu)

```http
POST https://api.telegram.org/bot__TG_BOT_TOKEN__/setMyCommands
Content-Type: application/json

{
  "commands": [
    { "command": "apply",   "description": "差勤" },
    { "command": "booking", "description": "會議室租借" },
    { "command": "help",    "description": "查看所有指令" }
  ]
}
```

- 設定後約 1–2 分鐘生效;客戶端也可能要重開 App。
- 群組 vs 私訊命令可分別設定(`scope` 參數)。

## 常見注意

- **Bot Token 等同密碼**:外洩者可冒充你發訊息、改 webhook。放進環境變數或 secret store,不要 commit。
- 群組訊息預設 bot 只能收到「被 @mention 或 command」的訊息;要讓 bot 收到所有訊息,需請管理員把 bot 的 *Group Privacy* 關掉(BotFather → `/mybots` → Bot Settings)。
- 寄大量訊息要注意 rate limit(每秒 30 則、每分鐘對同一 chat 約 20 則),超過會被暫時封鎖。
