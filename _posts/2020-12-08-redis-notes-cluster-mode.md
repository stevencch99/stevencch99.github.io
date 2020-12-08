---
layout: post
title: "Redis Notes - Cluster mode"
description: "Redis Notes - Cluster mode"
crawlertitle: "Redis Notes - Cluster mode"
date: 2020-12-08 20:52:26 +0800
categories: Redis
tags: Redis
comments: true
---

- toc
{:toc}

![paragraph break](https://order-brother.s3-ap-northeast-1.amazonaws.com/paragraph+break/separator-1.png)

## Redis Cluster mode

A Redis cluster is simply a data sharding strategy. Sharding is also known as partitioning, it splits the data up by key.

Redis Cluster provides a way to run a Redis installation where data is **automatically sharded across multiple Redis nodes**.

By running Redis Cluster, we could get:

- The ability to **automatically split dataset among multiple nodes**.
- The ability to **continue operations when a subset of the nodes are experiencing failures** or are unable to communicate with the rest of the cluster.

---

### Creating and using a Redis Cluster

To create a cluster, we need to have a few Redis instances running in **cluster mode**.
The minimal cluster that works as expected requires to contain **at least 3 master nodes**.

For the first test, we start a 6 nodes cluster with three masters and slaves.

- Create the minimal Redis cluster configuration file: `redis.conf`.

  ```bash
    touch redis.conf
  ```

  In the `redis.conf`:

  ```conf
  port 7000 # port number needs to be replaced accordingly
  cluster-enabled yes
  cluster-config-file nodes.conf
  cluster-node-timeout 5000
  appendonly yes
  ```

- Make diretories for cluster nodes.

  ```bash
  mkdir cluster-test
  cd cluster-test

  for i in {0..5}
  do
  mkdir 700$i && cp ../redis.conf ./700$i/redis-700$i.conf
  done
  ```

- Replace the port number in configurations for every nodes.

  ```bash
  for i in {0..5}
  do
  sed -i.bak 's/port 7000/port 700$i/' ./700$i/redis-700$i.conf
  done
  ```

- Copy the redis-server executable, **complied from the latest sources on the Redis official site**.

  > Redis official site: <https://redis.io/download>

  - Download, extract and compile Redis (check official site for latest version) in my case _v6.0.7_:

    ```bash
    wget http://download.redis.io/releases/redis-6.0.7.tar.gz
    tar xzf redis-6.0.7.tar.gz
    cd redis-6.0.7
    make
    ```

  - The binaries that are now compiled and available in the `src` directory.

    ```bash
    src/redis-server
    ```

  - Copy the binaries to `cluster-test` folder.

    ```bash
    cp <path_to_redis-6.0.7/src/redis-server> <path_to_cluster-test>
    ```

    ![directory tree of cluster-test](https://i.imgur.com/arIZ4XS.png)

- Start Redis server as background service

  ```bash
  for i in {0..5}
  do
  nohup ./redis-server 700$i/redis-700$i.conf &
  done
  ```

  Check the Redis server is working: `ps a | grep redis`

  ![check the Redis server is working](https://i.imgur.com/lHHxrUO.png)

- Create Redis cluster by redis-cli

  Use `redis-cli --cluster create` command to create a new cluster:

  ```bash
  redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 \
  127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 \
  --cluster-replicas 1
  ```

  > The option `--cluster-replicas 1` means that we want a slave for every master created.

  The command shows **Node ID** and **covered slots** for all the nodes, enter **yes** to accept the proposed configuration by redis-cli.

  At the end of the message shows "**[OK] All 16384 slots covered**" means the cluster created successfully and ready to accept connections.

  ![creating Redis cluster by redis-cli](https://i.imgur.com/DLoahwr.png)

- Test Redis cluster with redis-cli

  Use `redis-cli` with `-c` option to enable cluster mode, `-p` to specify which master node that you attempt to connect.

  ```bash
  # connect the node which listening port 7000
  redis-cli -c -p 7000

  # list cluster info
  redis-cli -c -p 7000 cluster info

  # list all nodes in the cluster
  redis-cli -c -p 7000 cluster nodes
  ```

  Interact with Redis by setting keys, Redis cluster will return which slot and node used to store the data.

  ![testing Redis cluster with redis-cli](https://i.imgur.com/tCYJ619.png)

- Stop Redis cluster

  ```bash
  ps -ef | grep redis | awk '{print $2}' | xargs kill -9
  ```

### Creating a Redis Cluster using the create-cluster script

There is a much simpler system to create a Redis cluster instead of executing instances manually as above.

You can find this script in `redis-6.0.7/utils/create-cluster` directory in the Redis distribution.

```bash
create-cluster start
create-cluster create
create-cluster stop
```

### Suggestions of using Redis Cluster

- Use rdb instead of aof when dealing colossal network traffic loading.
- Do not us `KEYS`, `FLUSH`
- Do not use `MGET` on Redis Cluster


### Redis Cluster TCP ports

Every Redis Cluster node requires 2 TCP connections open. The normal Redis TCP port used to serve clients (6379), plus the port obtained by adding 10000 to the data port, for example 16379 (6379 + 10000).

The second port (16379) is used for the Cluster bus, that is a node-to-node communication channel using a binary protocol for failure detection, configuration update, failover authorization and so forth.

Clients should always communicate with the normal Redis command port (6379). Furthermore, both these two ports must be opened in firewall and be reachable from all the other cluster nodes.

### About Redis Cluster and Docker

Currently Redis Cluster does not support NATted environments and in general environments where IP address or TCP ports are remapped.

Docker uses a technique called _port mapping_: Programs running inside Docker containers may be exposed with a different port against to the one that program believes to be using.

In order to make Docker compatible with Redis Cluster, using the **host networking mode** of Docker is required. For instance, if you run a container which binds to port 6379 and you use host networking, the container’s application is available on port 6379 on the host’s IP address.

Please check the `--net=host` option in the [Docker documentation](https://docs.docker.com/network/) for more information.

### Redis Cluster data sharding

Redis Cluster does not use consistent hashing (for example Memcached), but a different form of sharding where every key is conceptually part of a **hash slot**.

There are 16384 hash slots in Redis cluster, and to compute what is the hash slot of a given key, it take the CRC16 of the key modulo 16384.

Every node in a Redis Cluster is responsible for a subset of the hash slots, so for example you may have a cluster with 3 nodes, where:

- Node A contains hash slots from 0 to 5500.
- Node B contains hash slots from 5501 to 11000.
- Node C contains hash slots from 11001 to 16383.

Redis Cluster supports multiple key operations as long as all the keys involved into a single command execution (or whole transaction, or Lua script execution) all belong to the same hash slot.

The user can force multiple keys to be part of the same hash slot by using a concept called _hash tags_.

### Hash tags

While it is possible for many keys to be in the same hash slot, we could check the slot with [CLUSTER KEYSLOT](https://redis.io/commands/cluster-keyslot) command when naming keys.

Hash tags are documented in the Redis Cluster specification, the gist is that only the string between `{}` curly brackets in a key is hashed, e.g. `this{foo}key` and `another{foo}key` are guaranteed to be in the same hash slot, and can be used together in a command with multiple keys as arguments.

Example:

| Key                 | Hashing Pseudocode                   | Hash Slot |
| ------------------- | ------------------------------------ | --------- |
| user-profile:1234   | CRC16(‘user-profile:1234’) mod 16384 | 15990     |
| user-session:1234   | CRC16(‘user-session:1234’) mod 16384 | 2963      |
| user-profile:5678   | CRC16(‘user-profile:5678’) mod 16384 | 9487      |
| user-session:5678   | CRC16(‘user-session:5678’) mod 16384 | 4330      |
| user-profile:{1234} | CRC16(‘1234’) mod 16384              | 6025      |
| user-session:{1234} | CRC16(‘1234’) mod 16384              | 6025      |
| user-profile:{5678} | CRC16(‘5678’) mod 16384              | 3312      |
| user-profile:{5678} | CRC16(‘5678’) mod 16384              | 3312      |

---

### Redis Cluster master-slave model

Redis Cluster uses a master-slave model where every hash slot has from 1 (the master node) to N replicas (N - 1 additional slaves nodes).

In the example cluster with nodes A, B, C, if node B fails, the cluster is not able to continue, since the hash slots in the range 5501-11000 has no way to serve anymore.

However, when the cluster is created, we add slave node to every master, so that the final cluster is composed of A, B, C that are masters nodes, and A1, B1, C1 are slaves nodes, the system is able to continue if node B fails.

Node B1 replicates B, and B fails, the cluster will promote node B1 as the new master and will continue to operate correctly.

> Note: If nodes B and B1 fail at the same time Redis Cluster is not able to continue to operate.

### Redis Cluster consistency guarantees

Redis Cluster is not able to guarantee **strong consistency**.

In practical terms, this means that under certain conditions it is possible that Redis Cluster will lose writes that were already acknowledged client by the system.

The first reason why Redis Cluster can lose writes is because asynchronous replication.

The following happens during writes:

- The client writes to the master B.
- The master B replies OK to the client.
- The master B propagates the writes to its slaves B1, B2 and B3.

The master node does not wait for the acknowledgement from slave nodes before replying to the client, since this would be a prohibitive latency penalty for Redis.

So if the client writes something, but crashes before being able to send the write to its slaves, one of the slaves (that did not receive the write) can be promoted to master, losing the write forever.

![Redis Cluster consistency issue](https://i.imgur.com/auG8g3j.png)

## References

- [Redis.io - Cluster tutorial](https://redis.io/topics/cluster-tutorial)
- [Redis.io - CLUSTER KEYSLOT key](https://redis.io/commands/cluster-keyslot)
- [Redis Clustering Best Practices with Keys](https://redislabs.com/blog/redis-clustering-best-practices-with-keys/)
- [Docker network](https://docs.docker.com/network/)
