---
layout: post
title: "[PHP] Setup Xdebug with VS Code for macOS"
description: "[PHP] Setup Xdebug with VS Code for macOS"
crawlertitle: "[PHP] Setup Xdebug with VS Code for macOS"
date: 2021-07-03 02:09:34 +0800
categories: PHP
tags: PHP
comments: true

---

> Ref: <https://xdebug.org/docs/install>

- On macOS, you should have PHP installed with Homebrew, by this you will have pecl installed as well.

## Installing with PECL

```bash
pecl install xdebug
```

- `pecl` will change the `php.ini` file to add a configuration line to load Xdebug, you can check it by running `$ php -v` on terminal, you should see information like below:

    ```bash
    PHP 7.4.20 (cli) (built: Jun  4 2021 03:32:07) ( NTS )
    Copyright (c) The PHP Group
    Zend Engine v3.4.0, Copyright (c) Zend Technologies
        with Xdebug v3.0.4, Copyright (c) 2002-2021, by Derick Rethans
        with Zend OPcache v7.4.20, Copyright (c), by Zend Technologies
    ```

## Xdebug Installation Wizard

- Create a simple `test.php` to print out `phpinfo()` and next step test.

  ```php
  <?php 
    echo phpinfo() 
  ?>
  ```

Go this website: <https://xdebug.org/wizard> , paste the **full** output of  `test.php` or by php command `$ php -i`, and then click "Analyse my phpinfo() output".

> The output might be very long, there's 1,052 lines in my case

You will see the Summary and the line says "You're already running the latest Xdebug version". 

Follow the instruction, we need to compile this PHP extensions by download [xdebug-3.0.4.tgz](http://xdebug.org/files/xdebug-3.0.4.tgz).

- Check you have these prerequisites tool `php` and `autoconf` installed or not.

  ```bash
  # install if you don't have these
  brew list | grep -E 'php|autoconf'
  ``` 

- Unpack the downloaded file: `tar -xvzf xdebug-3.0.4.tgz`
- Run: `cd xdebug-3.0.4`
- Run: `phpize`

  As part of its output it should show:

  ```bash
  Configuring for:
  ...
  Zend Module Api No:      20190902
  Zend Extension Api No:   320190902
  ```

- Run: `./configure`
- Run: `make`
- Run: `cp modules/xdebug.so /usr/local/lib/php/pecl/20190902`
- Update `/usr/local/etc/php/7.4/php.ini`
  > You can locate your configuration file (`php.ini`) by run `$ php -i | grep php.ini`

  Append these configuration in the buttom of `php.ini`:

  ```ini
  ; Make sure this is the only line which assign `zend_extension` to `xdebug.so`
  zend_extension = /usr/local/lib/php/pecl/20190902/xdebug.so
  [xdebug]
  xdebug.start_with_request=yes
  xdebug.mode=debug
  xdebug.client_host = 127.0.0.1
  xdebug.client_port = 9003

  xdebug.show_exception_trace = On
  xdebug.remote_handler = dbgp
  ```

## VSCode setup

- Install the adapter extension: [PHP Debug](https://marketplace.visualstudio.com/items?itemName=felixfbecker.php-debug)

- Configure launch.json, setup listening default port of Xdebug: 9003
  ![config launch.json](https://i.imgur.com/tRkdtyd.png)

- Click launch and the debug toolbar will indicate (Listen for Xdebug) 
- Place breakpoint on wherever you want, and run the php code, the cursor focus will be automatically switched back to the line of code, then you can play with the code on "DEBUG CONSOLE".
- Disconnect debugger by the button on **Debug toolbar** (`⇧`+`F5`), the display settings of debug toolbar could be found in vscode settings.

![launching debugger](https://i.imgur.com/Oeig9XE.png)
