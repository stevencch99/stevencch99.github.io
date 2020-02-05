---
layout: post
title: "Easy Fixing File Encoding (in Vim)"
description: "Easy Fixing File Encoding (in Vim)"
crawlertitle: "Easy Fixing File Encoding (in Vim)"
date: 2020-02-05 23:59:59 +0800
categories: Editor
tags: Editor Vim Linux
comments: true
---

I have received some files from Japanese clients and have troble with read it through Vim, it looks like a mess:
![The Shift-JIS encoded file looks like a mess in my Vim](https://stevenchang.s3-ap-northeast-1.amazonaws.com/pics/blog/encoding_vim_sjis.png)

Check the file by `$ file -bi` command, and it returns:
![file -bi command and return result](https://stevenchang.s3-ap-northeast-1.amazonaws.com/pics/blog/file_command_sjis.png)

```shell
text/plain; charset=unknown-8bit
```

Fortunately, I've been informed that the file is encoded by Shift JIS, saved a hell lot of my time to figure out the encoding.

**Shift JIS** (*Shift Japanese Industrial Standards, also SJIS, Shift_JIS*) is a character encoding for the Japanese language, it has been replaced by CP932 in some case for better compatibility.

According to [Vim Tips Wiki](https://vim.fandom.com/wiki/Reloading_a_file_using_a_different_encoding), one can reload a file using a different encoding if Vim was not able to detect the correct encoding.

So, here is the simple solution in Vim, just use `:e ++enc=sjis` or `:e ++enc=cp932`:

![After reload the file by this command, it become readable](https://stevenchang.s3-ap-northeast-1.amazonaws.com/pics/blog/encoding_vim_sjis_after.png)

## References
- [Shift JIS](https://en.wikipedia.org/wiki/Shift_JIS) and [Code page 932](https://en.wikipedia.org/wiki/Code_page_932_(Microsoft_Windows)) on Wikipedia.
- [Vim Tips Wiki](https://vim.fandom.com/wiki/Reloading_a_file_using_a_different_encoding)
