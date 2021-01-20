---
layout: post
title: "Memcached notes - Basic understanding"
description: "Memcached notes - Basic understanding"
crawlertitle: "Memcached notes - Basic understanding"
date: 2021-01-20 18:03:33 +0800
categories: Memcached
tags: Memcached CachingSystem
comments: true
visible: true

---

- toc
{:toc}

![paragraph break](https://order-brother.s3-ap-northeast-1.amazonaws.com/paragraph+break/separator-1.png)

## About

> **Memcached** pronounced variously _mem-cash-dee_ or _mem-cashed_

Memcached is a high-performance, in-memory data store, distributed memory object caching system.

It offers a mature, scalable, open-source solution for delivering sub-millisecond response times making it useful as a **cache** or **session store**.

Memcached is an in-memory key-value store for small chunks of arbitrary data (strings, objects) from results of database calls, API calls, or page rendering.

It is often used to speed up dynamic database-driven websites by caching data and objects in RAM to reduce the number of times an external data source (such as a database or API) must be read.

## Rationale

- Store data in RAM
  By eliminating the need to access the disk, in-memory key-value stores such as Memcached avoid seek time delays and can access data in microseconds.

- Discards the oldest values
  Memcached's API provides a very large hash table distributed across multiple machines.  
  When the table is full, subsequent inserts cause older data to be purged in **least recently used (LRU)** order.

### Software architecture

The system uses a client-server architecture.

Each client knows all servers; the servers do no communication with each other.

- Server

  - The servers understand how to store and fetch items. They also manage when to evict or reuse memory.
  - The servers maintain a key-value associative array; the clients populate this array and query it by key.  
    Keys are up to 250 bytes long and values can be at most 1 Mb in size.
  - Servers are Disconnected From Each Other
    Memcached servers are unaware of each other. There is no **crosstalk**, no **synchronization**, no **broadcasting**, no **replication**.

- Client
  - Clients use client-side libraries to contact servers.
  - Clients understand how to choose which server to read/write, and what to do when it cannot contact a server.

A typical deployment has several servers and many clients. However, it is possible to use Memcached on a single computer, acting simultaneously as a client and server.

## Benefits of Memcached

- Sub-millisecond latency  
  No "pauses" waiting for a garbage collector ensures low latency, and free space is lazily reclaimed.
- Easy to use
- Scalability  
  Memcached's **distributed** and **multithreaded** architecture makes it easy to scale.  
  ![Memcached node structure](https://i.imgur.com/tHpDmqY.png)

  > [image source](https://memcached.org/about)

  Memcached allows you to make better use of your memory. Please refer to the diagram above, you can see two deployment scenarios:

  1. Each node is completely independent (top).
  2. Each node can make use of memory from other nodes (bottom).
     With Memcached, you can see that all of the servers are looking into the same virtual pool of memory. This means that a given item is always stored and always retrieved from the same location in your entire web cluster.

- Multithreaded architecture  
  Since Memcached is multithreaded, it can make use of multiple processing cores. This means that you can handle more operations by scaling up compute capacity.

- Cluster mode support  
  Memcached servers do not communicate with each other and in fact, a Memcached server is completely blind to which objects are stored on it and other servers.

  There is no master node, all nodes are equal, there is no replication, and node selection is done by the client hashing algorithm.
  ![memcached cluster topology](https://i.imgur.com/Lm6ubDC.png)

## Cons of Memcached

- Memcached has no internal mechanism to track misses.
- Lack of native certification and security mechanism. Therefore most deployments of Memcached are within trusted networks.

---

## References

- [Introduction from Amazon](https://aws.amazon.com/memcached)

