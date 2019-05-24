---
layout: post
title: FESCAR：阿里重磅开源分布式事务解决方案
category: springcloud
tags: [springcloud]
keywords: 架构
---

## FESCAR名字的由来:

Fast & EaSy Commit And Rollback
## FESCAR是啥？

被用在微服务架构中的高性能分布式事务解决方案。

## 微服务中的分布式事务问题:

让我们想象一个传统的应用，由3个模块构成，并且这三个模块使用同一个数据源。很明显，数据一致性由数据库提供的本地事务就能搞定。
![](https://ziyekudeng.github.io/assets/images/2019/0111/FESCAR/1.webp)

然而，一切美好都被在微服务架构中被打破。3个模块变成了3个服务，每个服务有独立的数据源（参考https://microservices.io/patterns/data/database-per-service.html）。每个服务的数据一致性由本地事务保证，但是跨服务的业务呢？如下图所示，某个业务既需要操作库存（Storage），又需要操作订单（Order），还需要操作账户（Account）。
![](https://ziyekudeng.github.io/assets/images/2019/0111/FESCAR/2.webp)

## FESCAR怎么做？**
![](https://ziyekudeng.github.io/assets/images/2019/0111/FESCAR/3.webp)

首先，如果定义分布式事务？我们认为一个分布式事务是由多个分支事务组成的全局事务，通常来说，分支事务就是本地事务。
![](https://ziyekudeng.github.io/assets/images/2019/0111/FESCAR/4.webp)

## FESCAR有三个基本组件：

Transaction Coordinator(TC)：事务协调器，维护着全局事务和分支事务的状态， 它来决定全局的提交还是回滚。

Transaction Manager(TM)： 事务管理器，定义全局事务的范围：开始一个全局事务，提交或者回滚一个全局事务。

Resource Manager(RM)： 资源管理器，管理分支事务处理的资源，与TC通信以注册分支事务并报告分支事务的状态，并驱动分支事务提交或回滚.
![](https://ziyekudeng.github.io/assets/images/2019/0111/FESCAR/5.webp)

## 一个典型的FESCAR维护的分布式事务的生命周期如下:

TM向TC请求开启一个全局事务，TC生成一个XID，一个表示全局事务的唯一ID；

通过微服务的调用链，XID被广播出去（图中绿色线路）；

RM向TC注册一个属于XID表示的分布式事务下的本地事务（红色箭头）；

TM向TC询问是提交还是回滚XID表示的全局事务；

TC驱动XID表示的全局事务下的所有分支事务，完成提交或者回滚动作。

如下图所示：
![](https://ziyekudeng.github.io/assets/images/2019/0111/FESCAR/6.webp)
