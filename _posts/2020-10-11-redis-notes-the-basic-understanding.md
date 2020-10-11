---
layout: post
title: "Redis Notes - The basic understanding"
description: "Redis note - The basic understanding"
crawlertitle: "Redis note - The basic understanding"
date: 2020-10-11 19:40:52 +0800
categories: 'Redis'
tags: 'Redis'
comments: true
---

- toc
{:toc}

![paragraph break](https://order-brother.s3-ap-northeast-1.amazonaws.com/paragraph+break/separator-1.png)

Redis is an open source, in-memory **data structure store**, used as a database, cache and message broker.

It supports data structures such as strings, hashes, lists, sets, sorted sets with range queries, bitmaps, hyperloglog (_probabilistic data structure_), geospatial indexes with radius queries and streams.

- Data persisting  
  In order to achieve its performance, Redis works with an in-memory dataset. Depending on your use case, you can persist it either by _dumping the dataset to disk_ every once in a while or by _appending each command to a log_.

- Common use case
  - Cache
  - Analytics
  - Leaderboard
  - Queues
  - Cookie storage
  - Expiring Data
  - Distributing Locks
  - High IO Workloads
  - Messaging
  - Pub/Sub (service that broadcast messages for the subscriber)

## Benefits of Redis

- Fast, Sub-millisecond latency
- Atomic operations: It is possible to group commands together so that they are executed as a single transaction.
- Easy to use
- Different levels of on-disk persistence  
  Persistence can be optionally disabled if you just need a feature-rich networked, in-memory cache.

  - Replication  
    Redis lets you create multiple replicas of a Redis primary instance. This allows you to scale database reads and to have highly available clusters.

    ![replication system drawing](https://i.imgur.com/6roUtW3.png)

  - Snapshots  
    Keep data on disk with a point in time snapshot which can be used for archiving or recovery.

  - AOF (Append Only File) persistence log:  
     The AOF persistence logs every write operation received by the server, that will be played again at server startup, reconstructing the original dataset. Commands are logged using the same format as the Redis protocol itself, in an append-only fashion. Redis can rewrite the log in the background when it gets too big.
    > Ref: <https://redis.io/topics/persistence>

- Automatic partitioning with Redis Cluster  
  Which allows you to distribute your data among multiple nodes. This allows you to scale out to better handle more data when demand grows.

- Flexible data structures  
  Redis data types include:
  - Strings – text or binary data up to 512MB in size
  - Lists – a collection of Strings in the order they were added
  - Sets – an unordered collection of strings with the ability to intersect, union, and diff other Set types
  - Sorted Sets – Sets ordered by a value
  - Hashes – a data structure for storing a list of fields and values
  - Bitmaps – a data type that offers bit-level operations
  - HyperLogLogs – a probabilistic data structure to estimate the unique items in a data set

## Cons of Redis

- Transaction is not recommended
  - Redis doesn't support rollback during a transaction. ([Ref](https://redis.io/topics/transactions#why-redis-does-not-support-roll-backs))
  - Multi-key operations (transaction) must involve a single slot, otherwise, you’d encounter a **CROSSSLOT** error.
- Redis Cluster does not support multiple databases like the stand-alone version of Redis. There is just database 0 and the SELECT command is not allowed. ([Ref](https://redis.io/topics/cluster-spec#redis-cluster-specification))

## Using Redis as an LRU cache

> LRU (least recently used): Discards the least recently used items first.

When Redis is used as a cache, often it is handy to let it automatically evict old data as you add new data.

This behavior is very well known in the community of developers since it is the default behavior of the popular _Memcached_ system.

Starting with Redis version 4.0, a new LFU (Least Frequently Used) eviction policy was introduced.

> Ref: <https://redis.io/topics/lru-cache>

## References

- [Redis.io](https://redis.io)
