---
layout: post
title: 关于Redis的一些新特性 ，使用建议和最佳实践
category: redis
tags: [redis]
---



redis从2009年诞生到现在已经走过将近10年，从最开始大家在讨论nosql和传统关系数据库孰优孰劣，到现在大家谈起分布式锁，缓存纷纷将Redis作为其第一选择，服务端面试中Redis也作为一项必备能力，而如今Redis 5.0已经发布，越来越多的新特性被加入，我完整的观察到并参与了一项新的开源产品从走入大家的视野到被接受，之后再流行的整个过程，也同时见证了memcache的日薄西山。
但是在工作中发现很多人只是了解一些Redis的基本使用，也并未完整的阅读过Redis的官方文档，对于一些命令不熟悉，不同场景下滥用不合理的数据结构，对一些新的特性似乎也不会去关注。鉴于自己对Redis的一些了解和实践经验，并收集了网络上一些资料，总结了一些使用建议。

## 特性

1.  SET key value [expiration EX seconds|PX milliseconds][NX|XX]

    Redis 2.6.12版本后的Set命令支持过期时间等参数，不必再像以前一样分为set和expire两个命令

2.  bitmap

    *   适用于大量数据的位图信息标记，例如如果要标记大量用户的某个状态值，可以考虑使用bitmap

    *   bitmap的另外一个应用是基于redis的[bloom filter](https://github.com/Baqend/Orestes-Bloomfilter)

3.  stream

    Redis4开始提供的新的数据结构，可以理解成轻量级的kafka steam，主要解决了pub/sub无法保证通知处理成功和blocked list无法多个client消费的问题，具体点击[此处](https://redis.io/topics/streams-intro)查看topic。

如果想实现一个简单的聊天室，可以尝试下steam。

## 使用建议

1.  合理分配过期时间
    不管是将Redis作为缓存，还是存储，如果不愿意看到内存被慢慢消耗殆尽，最后只能扩容或者人工介入，就给自己的key设置一个合理的过期时间。 当把Redis作为缓存时，更要预估自己的数据量和数据大小，选择一个合理的过期时间。

2.  多个操作使用pinepine

    *   这是Redis使用中的一项基本原则，同时需要知道，另外如果下一个命令的input基于上一个命令的output，就不可以放到一个pipeline里面执行了

    *   使用时考虑pipeline中一个命令执行失败的场景，后面的命令未执行是否因为一致性带来问题

3.  使用命名空间

    *   方便key的管理，我们开发中常用的redis-desktop客户端能够按照命名空间对key进行展示，另外，命名空间方便需要对某一类key进行统计和管理
    *   如果需要通过key进行分片，命名空间可以作为分片参数
4.  选用合适的数据结构

    *   理解每个数据结构的用途，和常用的命令，我曾经见过开发人员因为不知道scard命令可以获得set的size，而将所有的元素取出然后在程序中计算，所以需要平时多查看[Redis命令文档](https://redis.io/commands)；如果能够理解每种数据结构背后的原理，使用时会更加得心应手。
    *   不建议使用Redis缓存单个数据大小较大的对象，尤其是使用Set，Hash此类数据结构时候，考虑到Redis是单线程，过多的大对象访问增加了网络IO压力，对Redis性能有一定影响，另一方面Redis的虚拟内存page较小，如果内存碎片率较高，则分配/申请内存时在性能上有些影响。如果要缓存较大的对象，可以考虑memcache
5.  禁用keys
    很基本的Redis使用常识，可以通过rename-command来将一些类似的命令重命名，实现disable的效果

6.  选用lua script
    如果要保证多个操作的原子性，可以选择使用lua脚本

7.  config set parameter value
    redis 2.0后提供了config set 命令来动态修改一些运行参数而不必重启redis，目前已经支持动态修改maxmemory，可以通过CONFIG GET * 查看支持动态修改的参数列表

## 最佳实践

1.  key的命名
    合理的命名自己的key，不能在查看数据时可读性更强，也更便于统计和管理
2.  key name的长度
    预估key的存活数量，如果key的数量可能达到百万级别，就需要考虑key的名字过长而导致占用太多的存储空间，我在曾经参与过的一个消息系统中使用redis存储消息阅读量，但是后面由于消息量过多，导致name的占用空间达到几百M，如果精简name，可以节省大量的空间，减少不必要的困扰。 例如，保存用户的基本信息可以使用u:${id}
3.  不滥用Lua Script
    由于Redis是单线程，在QPS很高的情况下，过多的lua脚本执行，特别是内部包含较多业务逻辑处理的情况下，会对Redis性能产生很大的影响。曾经参与过的直播业务的生产环境中，我们在Lua脚本中对送礼物触发的的积分和活动信息的有较多的逻辑处理（20行左右），导致Redis负载100%，所以在排查时Lua脚本有可能是负载较高的元凶之一。
4.  关注内存和slowlog等统计数据

    *   通过info memory查看内存的分配和使用大小，碎片等情况
    *   slowlog get N 查看最近几条执行较慢的命令
    *   通过redis-cli --bigkeys 通过采样scan元素较多的key，不会一直阻塞redis执行
    *   更多好玩的redis-cli命令可以查看[此处](https://redis.io/topics/rediscli)

    > monitor命令不建议生产环境使用

