---
layout: post
title: "免密碼！透過 SSH 公開金鑰認證登入 Linux server"
description: "免密碼！透過 SSH 公開金鑰認證登入 Linux server"
crawlertitle: "免密碼！透過 SSH 公開金鑰認證登入 Linux server"
date: 2021-01-15 18:16:23 +0800
categories: Linux
tags: Linux
comments: true
---

- toc
{:toc}

![paragraph break](https://order-brother.s3-ap-northeast-1.amazonaws.com/paragraph+break/separator-1.png)

這篇文章紀錄如何使用 SSH 金鑰認證 ssh 連線，省去輸入密碼的動作。

## Step 1. 建立 SSH key (如果已經有想用的 key 可以跳過此步驟)

如果還沒有 SSH key，可透過 `ssh-keygen` 工具快速建立一組。

```bash
ssh-keygen -t rsa -N "" -f ~/.ssh/my_key
```

  > - `-t` 使用 [RSA 加密演算法](https://zh.wikipedia.org/wiki/RSA%E5%8A%A0%E5%AF%86%E6%BC%94%E7%AE%97%E6%B3%95) 加密。
  > - `-f` 指定金鑰的檔名。
  > - `-N` 指派空的 passphrase 以後使用此金鑰進行認證的時候就不必輸入密碼。

這個指令會產生兩個檔案:

- `my_key`: 私密金鑰（private key）,相當於密碼本人，需要妥善保管它。
- `my_key.pub`: 公開金鑰（public key），這是可以對外公開的資訊，接下來要把它放在遠端的 Linux server 供認證之用。

## Step 2. 將公開金鑰加入到 server 端的信任清單

這邊要把公開金鑰的內容放到 server 上的 `~/.ssh/authorized_keys` 檔案內。

- Option 1. 使用 `ssh-copy-id` 複製指定的公鑰到 server:

  ```bash
  ssh-copy-id -i ~/.ssh/id_rsa.pub user@host-address
  ```

  - `-i`: 僅複製指定的公鑰，如果不使用 `-i` 選項，預設會將 `~/.ssh` 資料夾內所有的公鑰都複製過去。

- Option 2. 使用 `cat` 指令複製公鑰:

  ```bash
  pub_key=$(cat ~/.ssh/my_key.pub)
  target='~/.ssh/authorized_keys'
  ssh user@host-address "mkdir -p ~/.ssh; grep -w $target -e $pub_key || cat >> $target < $pub_key"
  ```

  - 如果 server 沒有 `~/.ssh` 資料夾的話，`mkdir -p ~/.ssh;` 會先建一個。
  - 為避免加入重複的金鑰，我用 `grep` 做正則匹配 server `~/.ssh/authorized_keys` 的內容，如果有更好的作法還請留言讓我知道一下，大感謝!

最後測試一下 ssh 連線, 從此以後就不用再打密碼登入 server 囉！

---

## References

- BSD General Commands Manual
- GNU User Commands manual
- [RSA 加密演算法](https://zh.wikipedia.org/wiki/RSA%E5%8A%A0%E5%AF%86%E6%BC%94%E7%AE%97%E6%B3%95) - Wikipedia
- [Automating “enter” keypresses for bash script generating ssh keys](https://stackoverflow.com/questions/3659602/automating-enter-keypresses-for-bash-script-generating-ssh-keys) - Stack Overflow
