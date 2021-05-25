---
layout: post
title: "[Solve] Failed to install erlang 24.0 via asdf on macOS 11.3.1"
description: "[Solve] Failed to install erlang 24.0 via asdf on macOS 11.3.1"
crawlertitle: "[Solve] Failed to install erlang 24.0 via asdf on macOS 11.3.1"
date: 2021-05-25 14:43:52 +0800
categories: Erlang
tags: Erlang
comments: true

---

Got following error when installing erlang 24.0 via asdf (`$ asdf install erlang 24.0`):

```bash
ERROR: /Users/user_name/.asdf/plugins/erlang/kerl-home/builds/asdf_24.0/otp_src_24.0/lib/crypto/configure failed!
```

- Solution: Using version 2.69 of `autoconf` on MacOs BigSur.

    ```bash
    brew install cjntaylor/personal/autoconf@2.69
    brew unlink autoconf
    brew link autoconf@2.69
    ```

Now the installation command should be able to work.

```bash
asdf install erlang 24.0
```

---

## References

- [Erlang OTP-24.0.1 released!](https://elixirforum.com/t/erlang-otp-24-0-1-released/39855/14)
