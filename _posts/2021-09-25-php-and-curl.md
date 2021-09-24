---
layout: post
title: "[PHP] Exception: Call to undefined function curl_init()"
description: "[PHP] Exception: Call to undefined function curl_init()"
crawlertitle: "[PHP] Exception: Call to undefined function curl_init()"
date: 2021-09-25 00:17:21 +0800
categories: PHP
tags: PHP Ubuntu
comments: true

---

Today I ran into a trap while maintaining an old PHP Codeigniter project on an ubuntu server, the error message says:

```
Severity: error --> Exception: Call to undefined function curl_init()
```

Turns out this is very easy to fix, we just need to install the missing CURL extension for PHP.

## Solution

### (Optional) Check the running PHP version for Apache

To locate where the Apache's ROOT configuration directory

```bash
apache2ctl -V | grep HTTPD_ROOT
```

The result in my case is

```
-D HTTPD_ROOT="/etc/apache2"
```

Just so we know the ROOT directory is `/etc/apache2`, and let's check the `mods-enabled` directory in it:

```bash
ls /etc/apache2/mods-enabled/ | grep php
```

The result in my case is

```
php7.4.conf
php7.4.load
```

Now we confirm that we're running `php7.4` here.

### Install the CURL extension

```bash
sudo apt install php7.4-curl
```

To check the installation by running the command:

```bash
php7.4 -r "curl_init();"
```

It returns nothing if we're all good, otherwise, an error message will be shown like below:

```bash
PHP Fatal error:  Uncaught Error: Call to undefined function curl_init() in Command line code:1
```

Lastly, don't forget to restart apache2 service:

```bash
sudo service apache2 restart
```

