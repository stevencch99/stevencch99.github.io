---
layout: post
title: "Redis Notes - Sentinel (for High Availability)"
description: "Redis Notes - Sentinel (for High Availability)"
crawlertitle: "Redis Notes - Sentinel (for High Availability)"
date: 2020-12-06 11:03:30 +0800
categories: Redis
tags: Redis
comments: true
---

- toc
{:toc}

![paragraph break](https://order-brother.s3-ap-northeast-1.amazonaws.com/paragraph+break/separator-1.png)

## Redis Sentinel (for High Availability)

Redis Sentinel provides high availability for Redis.  

It also provides other collateral tasks such as monitoring, notifications and acts as a configuration provider for clients.

![Redis Replica and Sentinel structure](https://i.imgur.com/M3LE6Il.png)

- **Monitoring** Sentinel constantly checks if your master and replica instances are working as expected.
- **Notification** Sentinel can notify the system administrator, or other computer programs, via an API, that something is wrong with one of the monitored Redis instances.
- **Automatic failover** If a master is not working as expected, Sentinel can start a failover process where a replica is promoted to master, the other additional replicas are reconfigured to use the new master, and the applications using the Redis sever are informed about the new address to use when connecting.
- **Configuration provider** Sentinel acts as a source of authority for clients service discovery:  
  Clients connect to Sentinels in order to ask for the address of the current Redis master. If failover occurs, Sentinels will report the new address.

### Distributed nature of Sentinel

Redis Sentinel is a distributed system:

Sentinel itself is designed to run in a configuration where there are **multiple Sentinel processes cooperating together**.

The advantage of having multiple Sentinel processes cooperating are the following:

- Failure detection is performed when multiple Sentinels agree about the fact a given master is no longer available. This lowers the probability of false positives.
- Sentinel works even if not all the Sentinel processes are working, making the system robust against failures.

### Configuring Sentinel

The Redis source distribution contains a file called `sentinel.conf` that is self-documented example configuration file you can use to configure Sentinel.

A typical minimal configuration file looks like:

```conf
# sentinels is required to determine the 'mymaster' server is failed and start failover process
# * please refer to the section below for more details
sentinel monitor mymaster 127.0.0.1 6379 2

# the time in millisecons an instance does not response to be treated as down server
sentinel down-after-milliseconds mymaster 60000
sentinel failover-timeout mymaster 180000

# sets the number of replicas that can be reconfigured to use the new master after a failover at the same time
sentinel parallel-syncs mymaster 1

sentinel monitor resque 192.168.1.3 6380 4
sentinel down-after-milliseconds resque 10000
sentinel failover-timeout resque 180000
sentinel parallel-syncs resque 5
```

The example configuration above monitors 2 sets of Redis instances, each composed of a master and an undefined number of replicas. One set of instances is called mymaster, and the other resque.

Noitce that you only need to specify the masters to monitor, giving to each separated master a different name.

There is no need to specify replicas, which are auto-discovered. Sentinel will update the configuration automatically with additional information about replicas in order to retain the inforation in case of restart.

- The meaning of the `sentinel monitor` statements:

  ```bash
  sentinel monitor <master-group-name> <ip> <port> <quorum>
  # sentinel monitor mymaster 127.0.0.1 6379 2
  ```

  This line tell Redis to monitor a master called **mymaster**, that is at address 127.0.0.1 and port 6379, with a quorum of 2. 

  - The **quorum** is the number of Sentinels that need to agree about the fact the master is down and to start a failover procedure afterwards if possible.

  - The quorum is only used to detect the failure.

#### The failover procedure

So, in this case, if you have 5 Sentinel process, and the quorum for a given master set to the value of 2:  

- If 2 Sentinels agreee at the same time about the master being unreachable, 1 of them will try to start a failover.
- If there are at least 3 Sentinels are reachable, the failover will be autorized and will actually start.

This means during failures **Sentinel never starts a filover if the majority of Sentinel porcesses are unable to talk**.

### Running Sentinel

> It is **mandatory** to use a configuration file when running Sentinel.

- To run Sentinel service with `redis-server` executable:

```bash
$ redis-server /path/to/your/sentinel.conf --sentinel

# check the information of the Sentinel server
$ redis-cli -h localhost -p 26379 info
```

Sentinels by default run **listening for connections to TCP port 26379**, so port 26379 of your servers **must be open** to receive connections from the IP address of the other Sentinel instances.

Otherwise Sentinels could neither talk to each other nor make agreement about what to do, hence failover will never be performed.

### Fundamental things to know about Sentinel before deploying

1. There must have at least **3 Sentinel instances** for a robust deployment.
2. The 3 Sentinel instances should be placed into computers or virtual machines that are believed to fail in an independent way, e.g. different physical servers or Virtual Machines executed on different availability zones.
3. Sentinel + Redis distributed system does not guarantee that acknowledged writes are retained during failures, since Redis uses asynchronous replication.
4. Needed Sentinel support in client side.
5. There is no HA (High Availability) setup which is safe if you didn't test from time to time in development environments.
6. **Sentinel, Docker, or other forms of Network Address Translation or Port Mapping should be mixed with care**.

## References

- [Redis.io](https://redis.io/topics/sentinel)
