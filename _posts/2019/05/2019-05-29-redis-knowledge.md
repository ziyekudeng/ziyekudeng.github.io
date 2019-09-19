---
layout: post
title: redis的高级特性一览
category: redis
tags: [redis]
---

 


# Redis的高级特性一览


# 应用场景

*   缓存系统：用于缓解数据库的高并发压力
*   计数器：使用Redis原子操作，用于社交网络的转发数，评论数，粉丝数，关注数等
*   排行榜：使用zset数据结构，进行排行榜计算
*   实时系统：使用Redis位图的功能实现布隆过滤器，进而实现垃圾邮件处理系统
*   消息队列：使用list数据结构，消息发布者push数据，多个消息订阅者通过阻塞线程pop数据，以此提供简单的消息队列能力

> 之所以说简单，是因为Redis官方不提供可靠消费/发布的机制；需要自行实现故障转移、队列持久化、队列监控和流量控制等mq具备的功能；


![](https://ziyekudeng.github.io/assets/images/2019/0529/2.webp)

# 高级功能

## 慢查询

慢查询只记录Redis在处理存储的时间计数（图中的3步骤），并不包含网络通信时间和排队时间，所以客户端超时分析时要综合每个因素。

![](https://ziyekudeng.github.io/assets/images/2019/0529/3.webp)

注意：

*   慢查询保存数量参数不要设置过小，同时最好能定期持久化慢查询记录，方便历史问题排查。

## pipline

pipline用于异步处理大量Redis请求。

注意：

*   大量任务需要划分出多个pipline进行操作(否则，网络和等待时间都承受压力)。
*   pipline每次只能作用在一个Redis节点上。
*   M操作(mget,mset类似的指令)相比pipline，前者是原子操作，后者并不是。Redis会把一个携带很多命令的pipeline拆分成几个子命令。

![](https://ziyekudeng.github.io/assets/images/2019/0529/4.webp)

## 发布订阅

Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。

注意：

*   Redis无法做消息堆积（新订阅者无法获取历史订阅消息）

![](https://ziyekudeng.github.io/assets/images/2019/0529/5.webp)

## bitmap

字符串`big`对应的二进制（ASCII码）如图所示， bitmap可以直接操控位。使用每个数位代表一个用户或者状态，相比int数据结构保存，节省了32倍的内存空间。

注意：

*   bitmap并不是适合所有场景去替换常规的数据存储
*   bit是string类型，最大只能存512MB
*   注意setbit函数会自动补位，所以生产环境要注意setbit的偏移量，可能会造成较大的耗时

![](https://ziyekudeng.github.io/assets/images/2019/0529/6.webp)

## Hyperloglog

基于HyperLogLog算法，实现用极小空间完成独立数量的统计，类型本质是string。

注意：

*   无法保证数据完全正确。官网说明错误率为0.81%
*   无法取到单条数据

## GEO

GEO(地理信息定位)是Redis3.2版本发布的功能，存储经纬度，计算两地距离，范围计算等，类型本质是zset。

## Redis-sentinel

Redis哨兵是Redis2.8版本发布的功能，解决Redis集群的故障转移等痛点，支持高可用。

## Redis-cluster

Redis集群是Redis3.0版本发布的功能，支持分布式



