---
layout: post
title: "Reload a file with different encoding in Vim?"
description: "Reload a file with different encoding in Vim?"
crawlertitle: "Reload a file with different encoding in Vim?"
date: 2020-01-15 23:59:59 +0800
categories: Editor
tags: Editor Vim
comments: true
---

I have a file received from a Japanese client, and with troble to read it through Vim, check this file by `file -bi` command, and it returns: `text/plain; charset=unknown-8bit`

**Shift JIS** (Shift Japanese Industrial Standards, also SJIS, Shift_JIS) is a character encoding for the Japanese language, but som

Under Vim `:e ++enc=sjis` or `:e ++enc=cp932`

## Reference
- [Shift JIS](https://en.wikipedia.org/wiki/Shift_JIS) and [Code page 932](https://en.wikipedia.org/wiki/Code_page_932_(Microsoft_Windows)) on Wikipedia.
