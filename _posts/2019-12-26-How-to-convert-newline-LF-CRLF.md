---
layout: post
title: "How to convert newlines between Unix(LF) and DOS/Windows(CRLF)"
description: "How to convert newlines between Unix(LF) and DOS/Windows(CRLF)"
crawlertitle: "How to convert newlines between Unix(LF) and DOS/Windows(CRLF)"
date: 2019-12-26 23:59:59 +0800
categories: Linux
tags: ['Linux']
comments: true
---

## DOS/Windows newline(CRLF) and Unix format(LF)

The term CRLF refers to Carriage Return (ASCII 13, \r) Line Feed(ASCII 10, \n).
They're used to note the termination of a line,
however, dealt with differently in today's popular Operation Systems.

For example: in Windows both a CR adn LF are required to note the end of a line, whereas in Linux/UNIX a LF is only required.

In the HTTP protocol, the CR-LF sequence is always used to terminate a line.

> See [CRLF Injection](https://www.owasp.org/index.php/CRLF_Injection)

## `sed` command

SED command in UNIX stands for stream editor and it can perform lots of functions on file, like searching, find and replace, insertion or deletion.

Given that the conversion programs `unix2dos` and `dos2unix` are not available to every system, using `sed` might be a good idea.

- Replace or substitute string:

The most common use is for substitute or find and replace. The commands below replace "cat" with "dog" in the `pet.txt`:

```shell
$ sed 's/cat/dog/' pet.txt
$ sed -i 's/cat/dog/' pet.txt # replace the original file, "-i" option means edit files in place.
```

Here the "s" specifies substitute, there are some useful flags for this operation:
```shell
$ sed 's/cat/dog/2' # replace second pattern
$ sed 's/cat/dog/g' # apply the replacement to all matches to the regexp, not just the first.
$ sed 's/cat/dog/i' # case-insensitive
```

### Replace newline

If you know how to enter the carriage return character in bash(`Ctrl-V` then `Ctrl-M`):

```shell
$ sed 's/^M$//g' # CRLF to LF
$ sed 's/$/^M/g' # LF to CRLF

```
> Notice that "^M" represents a carriage return character, is not just "^" + character "M".

```shell
$ sed 's/.$//g'  # CRLF to LF, assumes that all lines end with CRLF
$ sed 's/$/\r/g' # LF to CRLF
```

## Reference

- [CRLF Injection](https://www.owasp.org/index.php/CRLF_Injection)
- [Linux sed command](https://www.computerhope.com/unix/used.htm)
