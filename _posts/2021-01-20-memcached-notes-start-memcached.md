---
layout: post
title: "Memcached notes - Start Memcached"
description: "Memcached notes - Start Memcached"
crawlertitle: "Memcached notes - Start Memcached"
date: 2021-01-20 23:40:28 +0800
categories: CachingSystem
tags: Memcached CachingSystem
comments: true
---

- toc
{:toc}

![paragraph break](https://order-brother.s3-ap-northeast-1.amazonaws.com/paragraph+break/separator-1.png)

## Running Memcached on Ubuntu

Install Memcached via `apt`

```bash
sudo apt install memcached
```

Start a Memcached server

```bash
memcached
```

- Start options
  - `-c`: max simultaneous connections (default: 1024)
  - `-d`: run as a daemon
  - `-h`: print out help manual
  - `-l`: interface to listen on
  - `-m`: item memory in megabytes (default: 64)
  - `-u`: assume identity of `<username>` (only when run as root)
  - `-p`: TCP port to listen on (default: 11211)
  - `-P`: save PID in `<file>`, only used with `-d` option
  - `-v`: verbose (print errors/warnings while in event loop)
  - `-vv`: very verbose (print client commands/responses)
- Stop a Memcached server by `ctrl + c` or `kill <MemcachedPID>`
- Test connection
  Start another terminal session, use `telnet` to connect the Memcached server.

  ```bash
  # connect to memcached default port
  telnet localhost 11211
  ```

### Memcached basic commands

- `stats`: to list Memcached service information
- `add <key> <flags> <exptime> <bytes>\n<value>\n`: add a key and return **STORED** if value stored successfully
  - `<key>`: name of the cached item
  - `<flags>`:
  - `<exptime>`: expired time for the cache, use 0 for the maximum allowed time
  - `<byte>`: the size of the cache data
  - `<value>`: the value of the cache
- `replace`: Store this data, but only if the data already exists.
- `set <key> <flags> <exptime> <bytes>\n<value>\n`: combination of `add` and `replace`, to `ADD` or `UPDATE` a key
- `get <key>`: Command for retrieving data. Takes one or more keys and returns all found items.

- Example:

  ```bash
  # To add/update a key "hello" with value "world"
  set hello 1 0 5
  world
  # return STORED

  get hello
  # VALUE hello 1 5
  # world
  # END
  ```

---

### Run Memcached as Docker container

Below is an example of pull a Memcached docker image and run Memcached server as a daemon process, bind localhost port 11212 to the container's port 11211

```bash
docker run -d -p 11212:11211 --name memcached --rm memcached
```

Test Memcached through `telnet` command

```bash
telnet 127.0.0.1 11212
```

The output will be like:

```bash
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.

```

And it is ready for your input command, test the connection by type in `stats`:

```bash
stats
# should list the stats of Memcached server
```

> Exit `telnet` interface by `quit` command.

Stop the container:

```bash
docker container stop memcached
```

![paragraph break](https://order-brother.s3-ap-northeast-1.amazonaws.com/paragraph+break/separator-1.png)

## References

- [Memcached official](https://memcached.org/about)
