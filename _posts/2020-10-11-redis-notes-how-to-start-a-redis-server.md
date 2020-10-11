---
layout: post
title: "Redis Notes - How to start a Redis server"
description: "Redis note - How to start a Redis server"
crawlertitle: "Redis note - How to start a Redis server"
date: 2020-10-11 21:45:52 +0800
categories: 'Redis'
tags: 'Redis'
comments: true
---

- toc
{:toc}

![paragraph break](https://order-brother.s3-ap-northeast-1.amazonaws.com/paragraph+break/separator-1.png)

## Running Redis on Ubuntu 20.04

- Install Redis via `apt` tool.

  ```bash
  sudo apt update
  sudo apt install redis-server
  ```

- Checking that the Redis service is running:

  ```bash
  sudo systemctl status redis
  ```

  If it is running without errors, the output will be like:

  ```bash
  ● redis-server.service - Advanced key-value store
      Loaded: loaded (/lib/systemd/system/redis-server.service; enabled; vendor preset: enable>
      Active: active (running) since Fri 2020-08-07 16:02:26 CST; 6min ago
        Docs: http://redis.io/documentation,
              man:redis-server(1)
      Process: 8982 ExecStart=/usr/bin/redis-server /etc/redis/redis.conf (code=exited, status=>
    Main PID: 8983 (redis-server)
        Tasks: 4 (limit: 3713)
      Memory: 2.9M
      CGroup: /system.slice/redis-server.service
              └─8983 /usr/bin/redis-server 127.0.0.1:6379
  ```

- Connect to the server using `redis-cli`, the Redis command line interface:

  ```bash
  redis-cli
  ```

- Test connection with the `ping` command:

  ```bash
  127.0.0.1:6379> ping
  PONG

  127.0.0.1:6379> set hello world
  OK

  127.0.0.1:6379> get hello
  "world"
  ```

- To shutdown the Redis server

  Using `redis-cli` command to shutdown a Redis server is a relatively elegant way, this will save the data into **RDB snapshot** before the server shutdown,
  so you can get data back after the server rebooted.

  ```bash
  redis-cli shutdown [save|nosave]
  ```

  You will see the log of Redis outputs:

  ```bash
  $ redis-cli shutdown

  # the Redis server log output
  74902:M 11 Oct 2020 19:10:06.457 # User requested shutdown...
  74902:M 11 Oct 2020 19:10:06.458 * Saving the final RDB snapshot before exiting.
  74902:M 11 Oct 2020 19:10:06.460 * DB saved on disk
  74902:M 11 Oct 2020 19:10:06.460 # Redis is now ready to exit, bye bye...
  ```

### Start, Stop and disable a system service in Ubuntu system

```bash
$ systemctl start redis
$ systemctl stop redis
$ systemctl restart redis

# Enable/disable the redis service at boot
$ systemctl enable redis
$ systemctl disable redis
```

## Running Redis on MacOS

- Install Redis via `brew`: `brew install redis`
- To start a Redis server:
  - Option 1. Run Redis server as service process:

    ```bash
    brew services start redis
    brew services stop redis
    brew services restart redis
    ```

  - Option 2. Run a Redis server and let output shows on terminal: 

    ```bash
    redis-server
    ```

    To shutdown a Redis server with `redis-cli`:

    ```bash
    redis-cli shutdown
    ```

## Deploy Redis in Docker

Pull the Redis image from [Docker Hub](https://hub.docker.com/_/redis), start a Redis instance based on Alpine Linux in background, and start listening TCP port 6379:

```bash
docker run -d --rm --name redis -p 6379:6379 redis:alpine
```

Test Redis through `telnet` command:

```bash
telnet 127.0.0.1 6379
```

The outputs will be like:

```bash
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.

```

> Note: Exit `telnet` interface by type in `quit` command.

And the Redis server is ready to go, test the connection by Redis `ping` command:

```bash
ping
+PONG
```

Stop the instance:

```bash
docker container stop redis
```

## References

- [Redis.io](https://redis.io)
