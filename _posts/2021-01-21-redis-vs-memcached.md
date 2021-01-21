---
layout: post
title: "Caching system - Redis vs. Memcached"
description: "Caching system - Redis vs. Memcached"
crawlertitle: "Caching system - Redis vs. Memcached"
date: 2021-01-21 12:06:55 +0800
categories: CachingSystem
tags: Redis Memcached CachingSystem
comments: true
---

- toc
{:toc}

![paragraph break](https://order-brother.s3-ap-northeast-1.amazonaws.com/paragraph+break/separator-1.png)

Memcached is designed for simplicity while Redis offers a rich set of features that make it effective for a wide range of use cases.

- The popularity compare with Redis and Memcached:
  ![The popularity compare with Redis and Memcached](https://i.imgur.com/BqU5ZxC.png)
  > Source: [Google Trends](https://trends.google.com/trends/explore?date=all&q=redis,memcached)

## Similarities between Redis and Memcached

There are many similarities between Memcached and Redis.

- **Easy to use**  
  Both Redis and Memcached are syntactically easy to use and require a minimal amount of code to integrate into your application.
- In-memory store, fast response, sub-millisecond latency
- Support for concurrent manipulation of data
<!-- - **Data partitioning**   -->
  <!-- Both Redis and Memcached allow you to distribute your data among multiple nodes. This allows you to scale out to better handle more data when demand grows. -->

  > Ref: <https://aws.amazon.com/elasticache/redis-vs-memcached/>

## Differences

- **Atomicity**

  - Redis supports the concept of the transaction through commands like `MULTI`, `WATCH`, `EXEC`, which could execute a batch of commands atomically.
    > Ref: <https://redis.io/topics/transactions>
  - Memcached has only `incr` and `decr` commands.

- **Memory usage**: Only Redis is capable of reclaiming memory usage.
  - Redis: Setting a max size is up to you. Redis will never use more than it has to and will give you back memory it is no longer using.
  - Memcached: you could flush the database, but it would still use the full chunk of RAM you configured it with.

- **Data persistence** (Disk I/O dumping)
  - Redis has very configurable persistence.
    - Snapshot
    - AOF persistence log
  - Memcached:
    - Memcached has no mechanisms for dumping to disk without 3rd party tools.
    - RAM only. Everything Memcached does is an attempt to guarantee latency and speed. If you have to sometimes hit the disk, that's no longer true ([__ref__](https://github.com/memcached/memcached/wiki/ProgrammingFAQ#why-only-ram)).

- **Memcached is multithreaded architecture**
  - Memcached is multithreaded, it can make use of multiple processing cores.
  - Redis has lots of features and is very fast, but completely limited to one core as it is based on an event loop.

  > Ref: <https://stackoverflow.com/questions/10558465/memcached-vs-redis>

### The use case suits for Memcached

- Simple model
- Use multiple threads to execute multi-node programs for large cache size
- Aim for scalability of nodes
- Mean to distribute data to different nodes (using memory amount servers more efficiently)

### The use case suits for Redis

- Dealing with complicated data types (Supported data types are strings, hashes, lists, sets and sorted sets, bit arrays, hyperloglogs and geospatial indexes)
- Sorting data in-memory
- Separate data into multiple points of reading and writing (Replication)
- Automated error repair
- Backup and restore data
- Advanced data structures

  In addition to strings, Redis supports list, sets, sorted sets, hashes, bit arrays, and hyperloglogs. Applications can use their more advanced data structures to support a variety of use cases. (e.g. use Redis Sorted Sets to implement real-time rank sorting)

## Comparison Chart

| Name                                                                                              | Memcached                                                      | Redis                                                                                                       |
| ---                                                                                               | ---                                                            | ---                                                                                                         |
| Description                                                                                       | In-memory key-value store, originally intended for caching     | In-memory data structure store, used as database, cache and message broker                                  |
| Primary database model                                                                            | key-value store                                                | Key-value store                                                                                             |
| Secondary database models                                                                         |                                                                | Document store<br>Graph DBMS<br>Search engine<br>Time Series DBMS                                           |
| Partitioning methods<br>(Methods for store different data on different nodes)                     | Yes                                                            | Cluster mode                                                                                                |
| Transaction concepts<br>(Support to ensure data integrity after non-atomic manipulations of data) | No                                                             | Optimistic locking, atomic execution of commands blocks and scripts                                         |
| Access control                                                                                    | Using SASL (Simple Authentication and Security Layer) protocol | Simple password-based access control (Access control lists and SSL are available in the commercial version) |
| Predefined data type                                                                              | No                                                             | strings, hashes, lists, sets and sorted sets, bit arrays, hyperloglogs and geospatial indexes               |
| Multithreaded architecture                                                                        | Yes                                                            | No                                                                                                          |
| Data persistence                                                                                  | No                                                             | Snapshot<br>AOF persistence log                                                                             |
| Backup and restore                                                                                | No                                                             | Yes                                                                                                         |
| Reclaim memory                                                                                    | No                                                             | Yes                                                                                                         |
| High availability<br>(replication)                                                                | No                                                             | Yes                                                                                                         |

> Ref: <https://docs.aws.amazon.com/AmazonElastiCache/latest/mem-ug/SelectEngine.html>

![paragraph break](https://order-brother.s3-ap-northeast-1.amazonaws.com/paragraph+break/separator-1.png)

## References

- [Memcached official site](https://memcached.org/about)
- [Memcached ProgrammingFAQ](https://github.com/memcached/memcached/wiki/ProgrammingFAQ)
- [System Properties Comparison Memcached vs. Redis](https://db-engines.com/en/system/Memcached%3BRedis)
- [Comparing Redis and Memcached](https://aws.amazon.com/elasticache/redis-vs-memcached/) - AWS official site
- [Memcached vs Redis, 挑选哪一个？](https://blog.csdn.net/itguangit/article/details/80019179) - [PostTruth](https://itguang.blog.csdn.net/)

