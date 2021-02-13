---
layout: post
title: "Linux - crontab command"
description: "Linux - crontab command"
crawlertitle: "Linux - crontab command"
date: 2021-02-13 17:35:13 +0800
categories: Linux
tags: Linux
comments: true

---

- toc
{:toc}

![paragraph break](https://order-brother.s3-ap-northeast-1.amazonaws.com/paragraph+break/separator-1.png)

The Cron daemon is one of the most useful built-in utility in the Unix-like operating system. It is used to execute desired tasks at a specific time we want, the scheduled tasks also known as `cron jobs`.

Cron reads the `crontab` (cron tables) for predefined commands and scripts.

By using the `crontab` command we can configure some useful cron jobs to run automatically.

## Basic commands

- `crontab -l`           : To check crontab content.
- `crontab -u <USER> -l` : To check crontab from specific USER as an administrator.
- `crontab -r`           : Delete current all crontab.

### Add/Edit crontab for the current user

```bash
crontab -e
```

![editing crontab for current user](https://stevenchang.s3-ap-northeast-1.amazonaws.com/pics/blog/crontab-edit.jpg "editing crontab for current user")

### To change the default crontab editor

> Ref: <https://www.howtogeek.com/410995/how-to-change-the-default-crontab-editor/>

The very first time we execute the `crontab` command with the `-e` (edit) option in a terminal, will be asked to pick one of the editors we'd like to use.

After selecting the editor from the menu, the same editor will be used every time executing the `crontab -e` command. 

If we later change our mind, the command to use is `select-editor`.

If your OS distribution do not provide `select-editor`, type this command:

```bash
export VISUAL='vim'
```

- The crontab syntax format:

  ```bash
  # ┌───────────── minute ( 0 - 59)
  # │ ┌─────────── hour   ( 0 - 23)
  # │ │ ┌───────── day    ( 1 - 31)
  # │ │ │ ┌─────── month  ( 1 - 12)
  # │ │ │ │ ┌───── week   ( 0 - 7，0 for Sunday，6 for Saturday)
  # │ │ │ │ │
  # * * * * * /path/to/command
  ```

- Examples

  ```bash
  # 08:30 every morning
  30 08 * * * /home/steven/script.sh --your --parameter

  # 18:30 every Sunday
  30 18 * * 0 /home/steven/script.sh --your --parameter

  # 08:30 every year on April 7, logging result into specific file.
  30 08 07 04 * /bin/zsh -c 'echo "Happy Birthday!"' >/tmp/my_crontab.log 2>/tmp/my_crontab_err.log

  # 22:30 on 1, 15, 29 every month
  30 22 1,15,29 * * /home/steven/script.sh --your --parameter

  # every 10 minutes
  */10 * * * * /home/steven/script.sh --your --parameter

  # every hour from 09:00 to 18:00
  00 09-18 * * * /home/steven/script.sh --your --parameter
  ```

## Restrict user from using crontab

Add user in `/etc/cron.deny` so the user will be restricted from using `crontab -e`.

## Email notification

`crontab` will send mail to the user who had executed the job, we can change the recipient by change the `MAILTO` variable:

```crontab
# set the recipient to steve
MAILTO=steve

# 08:30 every morning
30 08 * * * /home/steven/script.sh --your --parameter
```

Or if you want to disable the mail notification:

```crontab
# do not send mail
MAILTO=""

# 08:30 every morning
30 08 * * * /home/steven/script.sh --your --parameter
```

---

## References

- [How to Change the Default crontab Editor](https://www.howtogeek.com/410995/how-to-change-the-default-crontab-editor/)
- [Linux 設定 crontab 例行性工作排程教學與範例](https://blog.steven.org/linux/linux-crontab-cron-job-tutorial-and-examples/)
- [鳥哥的 Linux 私房菜](http://linux.vbird.org/linux_basic/0430cron.php)

