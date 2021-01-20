---
layout: post
title: "Redis Notes - Configuration"
description: "Redis Notes - Configuration"
crawlertitle: "Redis Notes - Configuration"
date: 2020-10-21 22:57:32 +0800
categories: CachingSystem
tags: Redis CachingSystem
comments: true
---

- toc
{:toc}

![paragraph break](https://order-brother.s3-ap-northeast-1.amazonaws.com/paragraph+break/separator-1.png)

## Redis configuration file

Redis has built-in default configuration makes it ready to go right after installation, to run Redis with default setting just simply type: `redis-server`.

However, this setup is only recommandedf for testing purposes, the proper way to configure Redis is by providing a configuration file `redis.conf`.

If you want to run it with custom settings:

```bash
redis-server </path/to/your_redis.conf>
```

And it is possible to alter the Redis configuration by passing parameters as options, for example:

```bash
redis-server --port 9999 --memory 8g
redis-server /path/to/redis.conf --loglevel debug
```

The `redis.conf` file contains a number of directives with very simple format:

```conf
keyword argument1 argument2 ... argumentN

# for example:
salveof 127.0.0.1 6380
```

Example `redis.conf` file:

```conf
bind 127.0.0.1
protected-mode yes
port 6379
pidfile /var/run/redis_6379.pid

# using password 'foobar' to portect the server
requirepass foobar

# Specify the eviction policies
maxmemory-policy volatile-lru

# Specify the log file name.
logfile /var/log/redis_6379.log

# Close the connection after a client is idle for N seconds (0 to disable)
timeout 0

# The working directory.
#
# The DB will be written inside this directory, with the filename specified
# above using the 'dbfilename' configuration directive.
#
# The Append Only File will also be created inside this directory.
#
# Note that you must specify a directory here, not a file name.
dir /var/redis/6379
```
### About logging

To enable logging to the system logger, just set 'syslog-enabled' to yes, and optionally update the other syslog parameters to suit your needs.

```conf
syslog-enabled yes
```

Setup loglevel:

```conf
loglevel notice
```

- debug: To log rich information for debug purpose
- verbose: Provide plenty of information in consicely way, less than debug mode
- notice: The right amount of information is basically what you need in the production environment
- warning: Only records important messages

Specify the syslog identity:

```conf
syslog-ident reeeeeedis

# specify the filename
logfile my_logfile.log

# or, print log to standard output
# Note: if Redis runs as a daemon service, this config will send log into /dev/null
logfile stdout
```

## rename-command

We can rename some dangerous command in order to pretect production environment:

```conf
rename-command FLUSHALL ""
rename-command FLUSHDB  ""
rename-command CONFIG   ""
rename-command KEYS     ""
```

- `FLUSHALL`: Empty all records and database
- `FLUSHDB`: Empty database
- `CONFIG`: Client-side could modify the configs of server
- `KEYS`: Client-side could see all the keys

## References

- [Redis.io](https://redis.io/topics/config)
