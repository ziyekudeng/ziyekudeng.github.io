---
layout: post
title: 分布式协调神器ZooKeeper之整体概述
category: tools
tags: [tools]
---




# 分布式协调神器ZooKeeper之整体概述

![](../data/img/img-index/head1.png) 陈涛原创 ![](../data/img/img-index/eye.png) 750 ![](../data/img/img-index/clock.png) 2019-02-26

今天主要想给大家介绍分布式协调神器：Zookeeper。

Zookeeper 最早起源于雅虎研究院的一个研究小组。当时，雅虎内部很多大型系统基本都需要依赖一个类似的系统来进行分布式协调，但是这些系统往往都存在分布式单点问题。所以，雅虎的开发人员就试图开发一个通用的无单点问题的分布式协调框架，以便让开发人员将精力集中在处理业务逻辑上。

立项初期，考虑到之前内部很多项目都是使用动物的名字来命名的（例如著名的Pig项目)，雅虎的工程师希望给这个项目也取一个动物的名字。当时研究院的首席科学家 RaghuRamakrishnan 开玩笑说：“再这样下去，我们这儿就变成动物园了！”是不是很有趣，顺势大家就表示既然已经是动物园了，它就叫动物园管理员吧！各个以动物命名的分布式组件放在一起，雅虎的整个分布式系统看上去就像一个大型的动物园了，而 Zookeeper 正好要用来进行分布式环境的协调一一于是，Zookeeper 的名字也就由此诞生了！

**ZooKeeper概述**

ZooKeeper是一种用于分布式应用程序的分布式开源协调服务。它公开了一组简单的原语，分布式应用程序可以构建这些原语，以实现更高级别的服务，以实现同步，配置维护以及组和命名。它被设计为易于编程，并使用在熟悉的文件系统目录树结构之后设计的数据模型。它在Java中运行，并且具有Java和C的绑定。

众所周知，协调服务很难做到。他们特别容易出现诸如竞争条件和死锁等错误。ZooKeeper背后的动机是减轻分布式应用程序从头开始实施协调服务的责任。

**集群模型**

![](https://p.qpic.cn/pic_wework/3435926283/6066b36f8190da431f9a3ac171f4957ecfc646b7f23d8a9b/0)

Leader服务器是整个ZooKeeper集群工作机制中的核心，其主要工作有以下两个：

1. 事务请求的唯一调度和处理者，保证集群事务处理的顺序性。

2. 集群内部各服务器的调度者。

从角色名字上可以看出，Follewer服务器是ZooKeeper集群状态的跟随者，其主要工作有以下三个：

1. 处理客户端非事务请求，转发事务请求给Leader服务器。

2. 参与事务请求Proposal的投票。

3. 参与Leader选举投票。

Observer充当了一个观察者的角色，在工作原理上基本和Follower一致，唯一的区别在于，它不参与任何形式的投票。

**数据结构**

![](https://p.qpic.cn/pic_wework/3435926283/5553865a5d8a986804b7f4e90c1c9a584aa761d897a0bce1/0)

**树形结构**

首先我们来看上述数据节点示意图，从而对ZooKeeper上的数据节点有一个大体上的认识，在ZooKeeper中，每一个节点都被称为一个ZNode，所有ZNode按层次化机构进行组织，形成一棵树。ZNode节点路径标识方式和Unix文件系统路径非常相似，都是由一系列使用斜杠（/）进行分割的路径表示，开发人员可以向这个节点中写入数据，也可以在节点下面创建子节点。

**节点操作流程**

![](https://p.qpic.cn/pic_wework/3435926283/4a4c99451b770be53824802fa7aec7051fa60cb721d56d34/0)

1. 在Client向Follower发出一个写请求。

2. Follower把请求转发给Leader。

3. Leader接收到以后开始发起投票并通知Follower进行投票。

4. Follower把投票结果发送给Leader。

5. Leader将结果汇总后，如果需要写入，则开始写入，同时把写入操作通知给Follower，然后commit。

6. Follower把请求结果返回给Client。

**设计目标**

① 顺序一致性

来自任意特定客户端的更新都会按其发送顺序被提交。也就是说，如果一个客户端将Znode z的值更新为a，在之后的操作中，它又将z的值更新为b，则没有客户端能够在看到z的值是b之后再看到值a（如果没有其他对z的更新）。

② 原子性

每个更新要么成功，要么失败。这意味着如果一个更新失败，则不会有客户端会看到这个更新的结果。

③ 单一系统映像

一个客户端无论连接到哪一台服务器，它看到的都是同样的系统视图。这意味着，如果一个客户端在同一个会话中连接到一台新的服务器，它所看到的系统状态不会比 在之前服务器上所看到的更老。当一台服务器出现故障，导致它的一个客户端需要尝试连接集合体中其他的服务器时，所有滞后于故障服务器的服务器都不会接受该 连接请求，除非这些服务器赶上故障服务器。

④ 持久性

一个更新一旦成功，其结果就会持久存在并且不会被撤销。这表明更新不会受到服务器故障的影响。

**整体架构**

![](https://p.qpic.cn/pic_wework/3435926283/3f39ec306ac84a7c3ef25158a9e3380be29fd164b37fb5a7/0)

**ServerCnxnFactory**

ZooKeeper服务端网络连接工厂。在早期版本中，ZooKeeper都是自己实现NIO框架，从3.4.0版本开始，引入了Netty。可以通过zookeeper.serverCnxnFactory来指定使用具体的实现。

**SessionTracker**

ZooKeeper服务端会话管理器。创建时，会初始化expirationInterval、nextExpirationTime、sessionsWithTimeout(用于保存每个会话的超时时间)，同时还会计算出一个初始化的sessionID。

**RequestProcessor**

ZooKeeper的请求处理方式是典型的责任链模式，在服务端，会有多个请求处理器依次来处理一个客户的请求。在服务器启动的时候，会将这些请求处理器串联起来形成一个请求处理链。基本的请求处理链如下：

![](https://p.qpic.cn/pic_wework/3435926283/6563b66a320d9a223717592447d2373236125cfd9ba6f2ba/0)

**LearnerCnxAcceptor**

Learner服务器(等于Follower服务器)连接请求接收器。负责Leader服务器和Follower服务器保持连接，以确定集群机器存活情况，并处理连接请求。

**LearnerHandler**

Leader接收来自其他机器的连接创建请求后，会创建一个LearnerHandler实例。每个LearnerHandler实例都对应了一个Leader和Learner服务器之间的连接，其负责Leader和Learner服务器之间几乎所有的消息通信和数据同步。

**ZKDatabase**

ZooKeeper内存数据库，负责管理ZooKeeper的所有会话记录以及DataTree和事务日志的存储。

**FileTxnSnapLog**

ZooKeeper上层服务和底层数据存储之间的对接层，提供了一系列的操作数据文件的接口，包括事务文件和快照数据文件。ZooKeeper根据zoo.cfg文件中解析出的快照数据目录dataDir和事务日志目录dataLogDir来创建FileTxnSnapLog。

**LeaderElection**

ZooKeeper会根据zoo.cfg中的配置，创建相应的Leader选举算法实现。在ZooKeeper中，默认提供了三种Leader选举算法的实现，分别是LeaderElection、AuthFastLeaderElection、FastLeaderElection，可以通过配置文件中electionAlg属性来指定，分别用0~3来表示。从3.4.0版本开始，ZooKeeper废弃了前两种算法，只支持FastLeaderEletion选举算法。

