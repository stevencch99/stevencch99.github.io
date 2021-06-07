---
layout: post
title: "[MySQL] Disable the ONLY_FULL_GROUP_BY restriction"
description: "[MySQL] Disable the ONLY_FULL_GROUP_BY restriction"
crawlertitle: "[MySQL] Disable the ONLY_FULL_GROUP_BY restriction"
date: 2021-06-07 14:50:02 +0800
categories: Database
tags: Database
comments: true

---

## Customize MySQL configuration

Example of disable the `ONLY_FULL_GROUP_BY` restriction:

- Check the current sql_mode setting in `mysql`: `SELECT @@GLOBAL.sql_mode` and copy the result.
- Stop mysql service: `$ sudo /etc/init.d/mysql stop` or `$ sudo systemctl stop mysql`
- List the configuration files: `$ mysql --help | grep 'my.cnf'`

- To disable ONLY FULL GROUP BY in MySQL 5.7

  Added this line in the bottom of `/etc/mysql/mysql.conf.d/mysqld.cnf`

  ```config
  sql_mode = 'STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION'
  ```

### For MySQL 8.0

- Create custom `*.cnf` file under `/etc/mysql/conf.d/` directory:

  ```bash
  $ sudo vim /etc/mysql/conf.d/disable_full_group_by.cnf
  ```
    - Content of `disable_full_group_by.cnf`

      ```config
      [mysqld]
      # Note: Origin sql_mode = 'ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION'
      sql_mode = 'STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION'
      ```

## Reboot mysql service to see if the modification works

- Reboot mysql service:  `$ sudo /etc/init.d/mysql start` or `$ sudo systemctl start mysql`
- Login `mysql` and check the result: `SELECT @@GLOBAL.sql_mode`

