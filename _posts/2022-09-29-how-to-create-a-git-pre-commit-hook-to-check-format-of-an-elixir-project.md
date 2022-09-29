---
layout: post
title: "如何設定 Git pre-commit hook 自動執行語法檢查"
description: "如何設定 Git pre-commit hook 自動執行語法檢查"
crawlertitle: "如何設定 Git pre-commit hook 自動執行語法檢查"
date: 2022-09-29 23:47:29 +0800
categories: Git
tags: Git
comments: true
---

將想要執行的 script 命名為 `pre-commit`，放在專案 `.git/hooks/` 目錄底下： `your_repo/.git/hooks/pre-commit`

> 記得把 `pre-commit` 設定成可執行的權限 (`$ chmod +x pre-commit`)。

例如希望在每次 commit 之前都執行一次 Elixir 的 format check： `$ mix format --check-formatted`

`pre-commit` 的內容如下：

```bash
#!/bin/bash
cd `git rev-parse --show-toplevel`
mix format --check-formatted
if [ $? == 1 ]; then
   echo "commit failed due to format issues..."
   exit 1
fi
```

只要 pre-commit script 的 exit code 不等於零就會終止 commit，若要跳過已設定好的 pre-commit hook 可在 commit 後面加上 `—no-verify` option：`$ git commit --no-verify`

## 參考

* [Elixir Git pre-commit hooks · Allan MacGregor](https://allanmacgregor.com/til/elixir-git-pre-commit-hooks)
* [Git - Git Hooks](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)
