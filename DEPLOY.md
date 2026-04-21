# 發文與部署流程

## 整體架構提醒

本專案有**兩個 git repo** 要推送:

| Repo | 位置 | 內容 | Remote |
|---|---|---|---|
| **外層** (source) | repo root | 原始 markdown、config、theme | `HugoBlog` |
| **內層 submodule** (production) | `hugoblog/public/` | Hugo 產出的 HTML | `jackliangtw.github.io` ← 這才是 GitHub Pages 實際吃的 |

> **只 push 外層不會上線**,要 push `public/` 這個 submodule,GitHub Pages 才會更新。

---

## 完整流程

### 步驟 0 — 只有第一次需要:確保 submodule 在 main branch 上

`git submodule update` 完成後,`public/` 預設會是 **detached HEAD** 狀態,直接 commit 會造成 commit 飄在空中。先切到 main:

```bash
cd hugoblog/public
git checkout main
git pull origin main
cd ../..
```

### 步驟 1 — 新增/編輯內容

在 `hugoblog/content/blog/` 或 `hugoblog/content/portfolio/` 加 `.md`,依照現有檔案的 TOML `+++` front matter 格式。

### 步驟 2 — 本地預覽(可選但建議)

```bash
cd hugoblog
hugo server -D
# 瀏覽器開 http://localhost:1313,確認沒問題後 Ctrl+C 結束
```

### 步驟 3 — Build,產出到 public/

```bash
# 仍在 hugoblog/ 底下
hugo -t hugo-creative-portfolio-theme
```

這會把 HTML 寫進 `hugoblog/public/`(也就是 submodule 內)。

### 步驟 4 — 推 submodule(這步才會真的上線 GitHub Pages)

```bash
cd public
git add -A
git commit -m "deploy: <這次更新的摘要>"
git push origin main
cd ..
```

推完後,等 1~2 分鐘 GitHub Pages 就會更新。

### 步驟 5 — 推外層 repo(保存原始碼 + 更新 submodule 指標)

```bash
cd ..    # 回到 repo root
git add hugoblog/content/...      # 你新增/修改的 markdown
git add hugoblog/public           # 這會更新 submodule 指向的 commit
git commit -m "blog: <這次更新的摘要>"
git push origin master
```

> 注意外層 repo 的預設分支是 **master**,submodule 是 **main**,不要搞混。

---

## 快速複習用一行懶人版

```bash
cd hugoblog && hugo -t hugo-creative-portfolio-theme && \
cd public && git add -A && git commit -m "deploy" && git push origin main && \
cd ../.. && git add -A && git commit -m "update content" && git push origin master
```

(實際使用時建議分步,commit message 認真寫,方便日後追溯。)

---

## 常見踩雷

1. **忘記 push submodule** → source 改了但網站沒更新。症狀:外層 push 成功但 `jackliangtw.github.io` 沒動。
2. **Submodule 在 detached HEAD 上 commit** → push 不出去,或推到奇怪的地方。→ 步驟 0 要做。
3. **只改 markdown 沒 rebuild** → `public/` 不會自己更新。每次改都要重跑 `hugo`。
4. **外層忘記 `git add hugoblog/public`** → submodule 指標沒更新,以後 clone 下來會對不上。
