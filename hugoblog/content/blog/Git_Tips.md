+++
date = "2026-04-21T10:50:00+05:30"
title = "Git 日常技巧:分支同步、reset、rebase squash 與 reflog 救援"
draft = false
showonlyimage = false
weight = 1
showmoretile = false
tags = "Git"
+++

## 分支同步與管理

```bash
git fetch origin                                # 更新所有 remote ref(預設)
git fetch --all --prune                         # 連帶清掉 remote 刪掉的分支

# 從遠端分支建本地分支並切過去
git checkout -b feature/xx origin/feature/xx

# 本地分支推到遠端不同名稱
git push origin 22-decimal-precision:22-ux

# 刪遠端分支
git push origin --delete feature/xx
```

## 回退本次 commit

| 目的                       | 指令                        | 效果 |
|----------------------------|-----------------------------|------|
| 撤 commit、保留修改(回到 staged 前) | `git reset --soft HEAD^`    | 檔案內容保留,變「未 commit」 |
| 撤 commit、保留修改(回到 working dir) | `git reset --mixed HEAD^`   | 預設行為,unstage |
| 連修改都丟掉               | `git reset --hard HEAD^`    | **會損失未推送的改動** |

## reflog 救援

```bash
git reflog                     # 看所有 HEAD 移動紀錄(包括被 reset --hard 掉的 commit)
git reset --hard <sha>         # 把 HEAD 拉回那個 commit
```

規則:**只要 commit 過,多半能透過 reflog 救回**(預設保留 90 天)。沒 commit 過的檔案被 `git checkout .` 蓋掉就很難救。

## rebase -i 壓縮 commits

```bash
git rebase -i HEAD~5
```

編輯器裡把下面幾行的 `pick` 改成 `s`(squash,保留訊息一起合)或 `f`(fixup,丟掉訊息),存檔退出。

GUI(SourceTree):

1. 對起點 commit 按右鍵 → Rebase children of `<sha>` interactively。
2. 對想合併的 commit 按 **Squash with previous** 直到全塞進第一個。
3. Apply。

完成後若已推過,要 force push 蓋掉遠端:

```bash
git push --force-with-lease     # 比 --force 安全,會檢查遠端沒被別人動過
```

## 把「已追蹤」的檔案加進 .gitignore

```bash
# 1) 先加到 .gitignore
# 2) 從 index 移除(本機檔案保留)
git rm --cached path/to/file
# 對資料夾
git rm -r --cached path/to/dir

git add .gitignore
git commit -m "chore: untrack <file>"
```

未先 `rm --cached` 而光改 `.gitignore` 是沒用的——Git 只忽略**尚未追蹤**的檔案。
