+++
date = "2026-04-21T10:25:00+05:30"
title = "Claude Code CLI:Plugin、MCP Server 設定速查"
draft = false
showonlyimage = false
weight = 1
showmoretile = false
tags = "ClaudeCode,CLI,AI"
+++

## Plugin Marketplace & 安裝

```bash
# 透過 CLI 介面(在 Claude Code session 內)
/plugin marketplace add anthropics/claude-plugins-official
/plugin marketplace add thedotmack/claude-mem
/plugin marketplace add obra/superpowers-marketplace

/plugin install superpowers@claude-plugins-official
/plugin install claude-mem@thedotmack
```

### 各 Plugin / Marketplace 功用

| 名稱 | 類別 | 用途 |
|---|---|---|
| `anthropics/claude-plugins-official` | Marketplace | Anthropic 官方 marketplace,`superpowers` 等官方套件都從這邊安裝 |
| `thedotmack/claude-mem` | Marketplace | 第三方記憶系統作者維護的 marketplace |
| `obra/superpowers-marketplace` | Marketplace | Obra 維護的 skill / 工作流集合(code review、執行 plan、subagent-driven dev 等進階用法) |
| `superpowers@claude-plugins-official` | Plugin | 核心強化套件,安裝後出現 `/superpowers:*` 指令,含 brainstorming / TDD / writing-plans / git-worktrees / receiving-code-review 等 skill |
| `claude-mem@thedotmack` | Plugin | 跨 session 持久化記憶:把你偏好的工作方式、專案慣例、過往決策存進 local DB,下次啟動 session 自動載入相關片段 |

## Superpowers 常用 skill

- `/superpowers:brainstorming` — 創意/需求設計前的腦力激盪。
- `/superpowers:using-git-worktrees` — 把實作隔離到 worktree,跟主要 repo 解耦。
- `/superpowers:writing-plans` — 把任務切成可逐步驗證的 plan。
- `/superpowers:test-driven-development` — TDD 工作流,強制先寫測試。
- 工作流常見順序:brainstorm → worktree → plan → TDD → code review。

## Status Line(context 使用率)

安裝:把下列加進 `~/.claude/settings.json`:

```json
{
  "statusLine": {
    "type": "command",
    "command": "npx -y ccstatusline@latest",
    "padding": 0
  }
}
```

顯示 context 使用百分比,快滿時提醒自己 `/compact` 或分段操作。

## MCP Server 設定

### GitLab(kf-gitlab)

```bash
claude mcp add-json kf-gitlab '{
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "@zereight/mcp-gitlab"],
  "env": {
    "GITLAB_PERSONAL_ACCESS_TOKEN": "__GITLAB_PAT__",
    "GITLAB_API_URL": "https://gitlab.example.com/api/v4",
    "NODE_TLS_REJECT_UNAUTHORIZED": "0"
  }
}'
```

- PAT 要的 scope:至少 `api`、`read_repository`、`write_repository`。
- 內網 GitLab 若 cert 是自簽,加 `NODE_TLS_REJECT_UNAUTHORIZED=0` 跳過驗證(或把 CA cert 加到 Node 信任鏈)。

### MySQL / ClickHouse(可跑多個,分別對應不同 DB)

```bash
claude mcp add-json mysql '{
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "@benborla29/mcp-server-mysql"],
  "env": {
    "MYSQL_HOST": "127.0.0.1",
    "MYSQL_PORT": "3306",
    "MYSQL_USER": "root",
    "MYSQL_PASS": "__MYSQL_PASSWORD__",
    "ALLOW_INSERT_OPERATION": "true",
    "ALLOW_UPDATE_OPERATION": "true",
    "ALLOW_DELETE_OPERATION": "true",
    "MULTI_DB_WRITE_MODE": "true"
  }
}'

claude mcp add-json clickhouse '{
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "@arvoretech/clickhouse-mcp"],
  "env": {
    "CLICKHOUSE_HOST": "127.0.0.1",
    "CLICKHOUSE_PORT": "8123",
    "CLICKHOUSE_USER": "default",
    "CLICKHOUSE_PASSWORD": "__CH_PASSWORD__"
  }
}'
```

想接遠端測試機:把 `HOST` 改成內網 IP(需先打開公司 VPN)。

### Playwright

```bash
claude mcp add-json playwright '{
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "@playwright/mcp@latest"]
}'
```

### 檢視 / 管理

```bash
claude mcp list                  # 連線狀態總覽
claude mcp list -s user          # 只看 user scope
claude mcp list -s project       # 只看當前專案 scope
claude mcp remove <name>         # 移除
```

在 session 內用 `/mcp` 也可以互動式檢查 + OAuth 流程(Gmail / Google Drive / Google Calendar 等官方 remote MCP)。

## 設定檔位置

| 路徑                              | 角色 |
|-----------------------------------|------|
| `~/.claude/settings.json`         | 全域設定(model、statusLine、enabledPlugins…) |
| `~/.claude.json`                  | MCP servers、plugins install 歷史、專案 state |
| `~/.claude/plugins/`              | plugin 快取 + installed_plugins.json |
| `<project>/.claude/settings.local.json` | 專案層 permission / skills / hooks |
