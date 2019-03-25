---
layout: post
title: Redis的各项功能解决了哪些问题？
category: redis
tags: [redis]
---



### **先看一下Redis是一个什么东西。**

> 官方简介解释到：Redis是一个基于BSD开源的项目，是一个把结构化的数据放在内存中的一个存储系统，你可以把它作为数据库，缓存和消息中间件来使用。
> 
> 同时支持strings，lists，hashes，sets，sorted sets，bitmaps，hyperloglogs和geospatial indexes等数据类型。
> 
> 它还内建了复制，lua脚本，LRU，事务等功能，通过redis sentinel实现高可用，通过redis cluster实现了自动分片。以及事务，发布/订阅，自动故障转移等等。

综上所述，Redis提供了丰富的功能，初次见到可能会感觉眼花缭乱，这些功能都是干嘛用的？都解决了什么问题？什么情况下才会用到相应的功能？那么下面从零开始，一步一步的演进来粗略的解释下。

### **1 从零开始**

最初的需求非常简单，我们有一个提供热点新闻列表的api：http://api.xxx.com/hot-news，api的消费者抱怨说每次请求都要2秒左右才能返回结果。

随后我们就着手于如何提升一下api消费者感知的性能，很快最简单粗暴的第一个方案就出来了：为API的响应加上基于HTTP的缓存控制 cache-control:max-age=600 ，即让消费者可以缓存这个响应十分钟。

如果api消费者如果有效的利用了响应中的缓存控制信息，则可以有效的改善其感知的性能（10分钟以内）。但是还有2个弊端：第一个是在缓存生效的10分钟内，api消费者可能会得到旧的数据；第二个是如果api的客户端无视缓存直接访问API依然是需要2秒，治标不治本呐。

### **2 基于本机内存的缓存**

为了解决调用API依然需要2秒的问题，经过排查，其主要原因在于使用SQL获取热点新闻的过程中消耗了将近2秒的时间，于是乎，我们又想到了一个简单粗暴的解决方案，即把SQL查询的结果直接缓存在当前api服务器的内存中（设置缓存有效时间为1分钟）。

后续1分钟内的请求直接读缓存，不再花费2秒去执行SQL了。假如这个api每秒接收到的请求时100个，那么一分钟就是6000个，也就是只有前2秒拥挤过来的请求会耗时2秒，后续的58秒中的所有请求都可以做到即使响应，而无需再等2秒的时间。

其他API的小伙伴发现这是个好办法，于是很快我们就发现API服务器的内存要爆满了。。。

### **3 服务端的Redis**

在API服务器的内存都被缓存塞满的时候，我们发现不得不另想解决方案了。最直接的想法就是我们把这些缓存都丢到一个专门的服务器上吧，把它的内存配置的大大的。然后我们就盯上了redis。。。

至于如何配置部署redis这里不解释了，redis官方有详细的介绍。随后我们就用上了一台单独的服务器作为Redis的服务器，API服务器的内存压力得以解决。

#### 3.1 持久化（Persistence）

单台的Redis服务器一个月总有那么几天心情不好，心情不好就罢工了，导致所有的缓存都丢失了（redis的数据是存储在内存的嘛）。虽然可以把Redis服务器重新上线，但是由于内存的数据丢失，造成了缓存雪崩，API服务器和数据库的压力还是一下子就上来了。

所以这个时候Redis的持久化功能就派上用场了，可以缓解一下缓存雪崩带来的影响。redis的持久化指的是redis会把内存的中的数据写入到硬盘中，在redis重新启动的时候加载这些数据，从而最大限度的降低缓存丢失带来的影响。

#### 3.2 哨兵（Sentinel）和复制（Replication）

Redis服务器毫无征兆的罢工是个麻烦事。那么怎办办？答曰：备份一台，你挂了它上。那么如何得知某一台redis服务器挂了，如何切换，如何保证备份的机器是原始服务器的完整备份呢？这时候就需要Sentinel和Replication出场了。

Sentinel可以管理多个Redis服务器，它提供了监控，提醒以及自动的故障转移的功能；Replication则是负责让一个Redis服务器可以配备多个备份的服务器。Redis也是利用这两个功能来保证Redis的高可用的。此外，Sentinel功能则是对Redis的发布和订阅功能的一个利用。

#### 3.3 集群（Cluster）

单台服务器资源的总是有上限的，CPU资源和IO资源我们可以通过主从复制，进行读写分离，把一部分CPU和IO的压力转移到从服务器上。但是内存资源怎么办，主从模式做到的只是相同数据的备份，并不能横向扩充内存；单台机器的内存也只能进行加大处理，但是总有上限的。

所以我们就需要一种解决方案，可以让我们横向扩展。最终的目的既是把每台服务器只负责其中的一部分，让这些所有的服务器构成一个整体，对外界的消费者而言，这一组分布式的服务器就像是一个集中式的服务器一样（之前在解读REST的博客中解释过分布式于基于网络的差异：基于网络应用的架构）。

在Redis官方的分布式方案出来之前，有twemproxy和codis两种方案，这两个方案总体上来说都是依赖proxy来进行分布式的，也就是说redis本身并不关心分布式的事情，而是交由twemproxy和codis来负责。

而redis官方给出的cluster方案则是把分布式的这部分事情做到了每一个redis服务器中，使其不再需要其他的组件就可以独立的完成分布式的要求。我们这里不关心这些方案的优略，我们关注一下这里的分布式到底是要处理那些事情?也就是twemproxy和codis独立处理的处理分布式的这部分逻辑和cluster集成到redis服务的这部分逻辑到底在解决什么问题？

如我们前面所说的，一个分布式的服务在外界看来就像是一个集中式的服务一样。那么要做到这一点就面临着有一个问题需要解决：既是增加或减少分布式服务中的服务器的数量，对消费这个服务的客户端而言应该是无感的；那么也就意味着客户端不能穿透分布式服务，把自己绑死到某一个台的服务器上去，因为一旦如此，你就再也无法新增服务器，也无法进行故障替换。

**解决这个问题有两个路子：**

第一个路子最直接，那就是我加一个中间层来隔离这种具体的依赖，即twemproxy采用的方式，让所有的客户端只能通过它来消费redsi服务，通过它来隔离这种依赖（但是你会发现twermproxy会成为一个单点），这种情况下每台redis服务器都是独立的，它们之间彼此不知对方的存在；

第二个路子是让redis服务器知道彼此的存在，通过重定向的机制来引导客户端来完成自己所需要的操作，比如客户端链接到了某一个redis服务器，说我要执行这个操作，redis服务器发现自己无法完成这个操作，那么就把能完成这个操作的服务器的信息给到客户端，让客户端去请求另外的一个服务器，这时候你就会发现每一个redis服务器都需要保持一份完整的分布式服务器信息的一份资料，不然它怎么知道让客户端去找其他的哪个服务器来执行客户端想要的操作呢。

上面这一大段解释了这么多，不知有没有发现不管是第一个路子还是第二个路子，都有一个共同的东西存在，那就是分布式服务中所有服务器以及其能提供的服务的信息。这些信息无论如何也是要存在的，**区别在于第一个路子是把这部分信息单独来管理，用这些信息来协调后端的多个独立的redis服务器；第二个路子则是让每一个redis服务器都持有这份信息，彼此知道对方的存在，来达成和第一个路子一样的目的，优点是不再需要一个额外的组件来处理这部分事情。**

Redis Cluster的具体实现细节则是采用了Hash槽的概念，即预先分配出来16384个槽：在客户端通过对Key进行CRC16（key）% 16384运算得到对应的槽是哪一个；在redis服务端则是每个服务器负责一部分槽，当有新的服务器加入或者移除的时候，再来迁移这些槽以及其对应的数据，同时每个服务器都持有完整的槽和其对应的服务器的信息，这就使得服务器端可以进行对客户端的请求进行重定向处理。

### **4 客户端的Redis**

上面的第三小节主要介绍的是Redis服务端的演进步骤，解释了Redis如何从一个单机的服务，进化为一个高可用的、去中心化的、分布式的存储系统。这一小节则是关注下客户端可以消费的redis服务。

#### 4.1 数据类型

redis支持丰富的数据类型，从最基础的string到复杂的常用到的数据结构都有支持：

*   string：最基本的数据类型，二进制安全的字符串，最大512M。

*   list：按照添加顺序保持顺序的字符串列表。

*   set：无序的字符串集合，不存在重复的元素。

*   sorted set：已排序的字符串集合。

*   hash：key-value对的一种集合。

*   bitmap：更细化的一种操作，以bit为单位。

*   hyperloglog：基于概率的数据结构。

这些众多的数据类型，主要是为了支持各种场景的需要，当然每种类型都有不同的时间复杂度。其实这些复杂的数据结构相当于之前我在《解读REST》这个系列博客基于网络应用的架构风格中介绍到的远程数据访问（Remote Data Access = RDA）的具体实现，即通过在服务器上执行一组标准的操作命令，在服务端之间得到想要的缩小后的结果集，从而简化客户端的使用，也可以提高网络性能。比如如果没有list这种数据结构，你就只能把list存成一个string，客户端拿到完整的list，操作后再完整的提交给redis，会产生很大的浪费。

#### 4.2 事务

上述数据类型中，每一个数据类型都有独立的命令来进行操作，很多情况下我们需要一次执行不止一个命令，而且需要其同时成功或者失败。redis对事务的支持也是源自于这部分需求，即支持一次性按顺序执行多个命令的能力，并保证其原子性。

#### 4.3 Lua脚本

在事务的基础上，如果我们需要在服务端一次性的执行更复杂的操作（包含一些逻辑判断），则lua就可以排上用场了（比如在获取某一个缓存的时候，同时延长其过期时间）。redis保证lua脚本的原子性，一定的场景下，是可以代替redis提供的事务相关的命令的。相当于基于网络应用的架构风格中介绍到的远程求值（Remote Evluation = REV）的具体实现。

#### 4.4 管道

因为redis的客户端和服务器的连接时基于TCP的， 默认每次连接都时只能执行一个命令。管道则是允许利用一次连接来处理多条命令，从而可以节省一些tcp连接的开销。管道和事务的差异在于管道是为了节省通信的开销，但是并不会保证原子性。

#### 4.5 分布式锁

官方推荐采用Redlock算法，即使用string类型，加锁的时候给的一个具体的key，然后设置一个随机的值；取消锁的时候用使用lua脚本来先执行获取比较，然后再删除key。具体的命令如下：

    SET resource_name my_random_value NX PX 30000
    
    if redis.call("get",KEYS[1]) == ARGV[1] then
        return redis.call("del",KEYS[1])
    else
        return 0
    end

### **总结**

本篇着重从抽象层面来解释下redis的各项功能以及其存在的目的，而没有关心其具体的细节是什么。从而可以聚焦于其解决的问题，依据抽象层面的概念可以使得我们在特定的场景下选择更合适的方案，而非局限于其技术细节。

以上均是笔者个人的一些理解，如果不当之处，欢迎指正。




