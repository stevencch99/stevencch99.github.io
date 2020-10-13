---
layout: post
title: "Redis Notes - Cache and Memory management"
description: "Redis note - Cache and Memory management"
crawlertitle: "Redis note - Cache and Memory management"
date: 2020-10-12 19:10:52 +0800
categories: 'Redis'
tags: 'Redis'
comments: true
---

- toc
{:toc}

![paragraph break](https://order-brother.s3-ap-northeast-1.amazonaws.com/paragraph+break/separator-1.png)

## Using Redis as an LRU cache

> LRU (Least Recently Used): Discards the least recently used items first.

When Redis is used as a cache, often it is handy to let it automatically evict old data as you add new data.

This behavior is very well known in the community of developers since it is the default behavior of the popular _Memcached_ system.

## The new LFU cache mode

> LFU (Least Frequently Used): Discards the least frequently used items first.

Starting with Redis 4.0, a new [Least Frequently Used eviction mode](http://antirez.com/news/109) is available.

When using LFU mode, Redis will try to track the frequency of access of items, so that the one used often have a higher chance of remaining in memory.

## Memory Management

Redis `maxmemory` directive is used to limit the memory usage, when the memory limit is reached, Redis will try to remove keys according to the eviction policy selected.

By default, Redis uses unlimited memories of the host system (`maxmemory 0`), yet it is possible to set the configuration directive through the `redis.conf` file, or later using the [CONFIG SET](https://redis.io/commands/config-set) command at runtime.

If Redis can't remove keys according to the policy, or if the policy is set to `noeviction`, Redis will start to reply with errors to commands that would use more memory, like `SET`, `LPUSH`, and so on.

To configure a memory limit of 100 megabytes, for example, the following directive can be used inside the redis.conf file.

```bash
maxmemory 100mb
```

> Setting `maxmemory` to zero results in no memory limits.

### Replica will ignore its maxmemory setting

Starting from Redis 5, by default a **replica will ignore its maxmemory setting** unless it is promoted to master after a failover or manually.

It means that the eviction of keys will be just handled by the master, sending the `DEL` command to the replica as keys evict on the master side.

This behavior ensures that masters and replicas stay consistent, however, if your replica is writable, or you want the replica to have a different memory setting, you may change this directive to no (default is yes):

`replica-ignore-maxmemory no`

It is suggested that you set a **lower limit for maxmemory** so that there is some free RAM on the system for replica output buffers.

### Maxmemory policy

Specify how Redis will select what to remove when `maxmemory` is reached.

The default policy is `maxmemory-policy noeviction`, you can also select one from the following behaviors:

- **volatile-lru**    -> Evict using approximated LRU, only keys with an expire set.
- **allkeys-lru**     -> Evict any key using approximated LRU.
- **volatile-lfu**    -> Evict using approximated LFU, only keys with an expire set.
- **allkeys-lfu**     -> Evict any key using approximated LFU.
- **volatile-random** -> Remove a random key having an expire set.
- **allkeys-random**  -> Remove a random key, any key.
- **volatile-ttl**    -> Remove the key with the nearest expiry time (minor TTL, Time To Live)
- **noeviction**      -> Don't evict anything, just return an error on write operations.

> Both LRU, LFU and volatile-ttl are implemented using approximated randomized algorithms.

Redis will return an error on write operations when there are no suitable keys for eviction.

At the date of writing these commands are:
[`append`, `decr`, `decrby`, `exec`, `getset`, `hincrby`, `hmset`, `hset`, `hsetnx`, `incr`, `incrby`, `linsert`, `lpush`, `lpushx`, `lset`, `mset`, `msetnx`, `rpoplpush`, `rpush`, `rpushx`, `sadd`, `sdiff`, `sdiffstore`, `set`, `setex`, `setnx`, `sinter`, `sinterstore`, `sort`, `sunion`, `sunionstore`, `zadd`, `zincrby`, `zinterstore`, `zunionstore`]

## Tune algorithms for speed or accuracy

LRU, LFU and minimal TTL algorithms are not precise algorithms but approximated algorithms (in order to save memory), so you can tune it for speed or accuracy.

By default Redis will check five keys and pick the one that was used less recently, you can change the sample size using the following configuration directive.

The default of 5 produces good enough results. 10 Approximates very closely true LRU but costs more CPU. 3 is faster but not very accurate.

```bash
maxmemory-samples 5
```

The following is a graphical comparison of how the LRU approximation used by Redis compares with ture LRU.

![graphical comparison of how the LRU approximation used by Redis](https://i.imgur.com/xOVMZbr.png)
  > image [source](https://redis.io/topics/lru-cache#approximated-lru-algorithm)

  > The light gray band are objects that were evicted.
  > The gray band are objects that were not evicted.
  > The green band are objects that were added.

As you can see, when using a sample size of 10 in Redis 3.0, the approximation is very close to the theoretical performance of ture LRU algorithm.

## Reclaim expired keys

Redis reclaims expired keys in two ways: upon access when those keys are found to be expired, and also in background, in what is called the "active expire key".

The key space is slowly and interactively scanned looking for expired keys to reclaim, so that it is possible to free the memory of keys that are expired and will not be accessed again in a short time.

The default effort of the expire cycle will try to avoid having more than ten percent of expired keys still in memory and will try to avoid consuming more than 25% of total memory.

It is possible to increase the expire "effort" that is normally set to "1", up to "10". At its maximum value, the system will use more CPU, longer cycles and may introduce more latency, and will tolerate less already expired keys still present in the system.

`active-expire-effort 1`

It's a tradeoff between memory, CPU and latency.

> Ref: Default [redis.conf](https://github.com/redis/redis/blob/unstable/redis.conf)

### How Redis expires keys

Normally Redis keys are created without an associated time to live. The key will simply live forever, unless it is explicitly removed by the user, for instance using the `DEL` command.

Redis keys are expired in two ways: a passive way, and an active way.

A key is passively expired simply when some client tries to access it, and the key is found to be timed out.

For the keys that will never be accessed again, Redis tests a few keys at random among keys with an expire set periodically.

Specifically, this is what Redis does 10 times per second:

1. Test 20 random keys from the set of keys with an associated expire.
2. Delete all the keys found expired.
3. If more than 25% of keys were expired, start again from step 1.

![flow diagram of the passive way to remove expired keys](https://i.imgur.com/ByTWAZk.png)

> Ref: <https://redis.io/commands/expire#how-redis-expires-keys>

### How the eviction process works

The eviction process works like this:

- A client runs a new command, resulting in more data added.
- Redis checks the memory usage, and if it is greater than the `maxmemory` limit, it evicts keys according to the policy.
- A new command is executed, and so forth.

We continuously cross the boundaries of the memory limit, by going over it, and then by evicting keys to return under the limits.

If a command results in a lot of memory being used (like a big set intersection stored into a new key) for some time the memory limit can be surpassed by a noticeable amount.

> See: <https://redis.io/topics/lru-cache#how-the-eviction-process-works>

#### Write Amplification issue occurs during Redis reclaiming memory

When `maxmemory` limit has been set and `maxmemory-policy` is not `noeviction`, Redis memory reclaim process will be triggered every time client runs a new command.

If your Redis server working in a memory overflow state (used_memory > maxmemory) all the time, it will frequently trigger the mechanism and affects the performance of the server.

Note that if you have replicas attached, the related key remove operations will be synchronized to slave nodes, causes the write amplification issue.

Therefore, it is recommended to configure the Redis server to always run in the state that maxmemory > used_memory.

![execute command trigger memory reclaiming](https://i.imgur.com/jmzev3I.png)

### Lazy Freeing

Redis has two primitives to delete keys.

One is called `DEL` and is a blocking deletion of the object. It means that the server stops processing new commands in order to reclaim all the memory associated with an object synchronously.

The server could block for a long time to complete such the operation when the value containing millions of elements.

For the reason above, it is recommended to use the non-blocking deletion primitives such as `UNLINK`, `FLUSHALL ASYNC` and `FLUSHDB ASYNC`.

Those commands will remove keys in BIO (Background I/O) thread to avoid server blocking.

Redis deletes objects independently of a user call in the following scenarios:

1. On eviction

   Because of the maxmemory and maxmemory policy configurations, make room for new data without going over the specified memory limit.

2. Key Expired

   When the timeout of a key has expired, the key will automatically be deleted. A key with an associated timeout is often said to be **volatile** in Redis terminology.

   In this scenario, it is better to add randomness in the value to avoid massive keys expired at the same time.

   Ref: [EXPIRE command](https://redis.io/commands/expire)

3. Side effects of a command that stores data on a key that may already exist.

   For example, the `RENAME` command may delete the old key content when it is replaced with another one. Similarly `SUNIONSTORE`
   or `SORT` with `STORE` option may delete existing keys. The `SET` command itself removes any old content of the specified key to replace it with the specified string.

   > If the target key is a big key, this side effect could lead to a blocking issue.

4. During replication

   When a replica performs a full resynchronization with its master, the content of the whole database is removed to load the RDB file just transferred.

In all the above cases the default is to delete objects in a blocking way, like if DEL was called.

Also, you can configure each case specifically to release memory in a non-blocking way like if `UNLINK` was called, using the following configuration directives (default is disabled).

```bash
lazyfree-lazy-eviction no
# Note: turn on lazy eviction is possible to cause the memory usage exceeding the limit due to Redis fail to release memory in time.

lazyfree-lazy-expire no
lazyfree-lazy-server-del no
replica-lazy-flush no
```

## References

- [Redis.io](https://redis.io/topics/lru-cache)
