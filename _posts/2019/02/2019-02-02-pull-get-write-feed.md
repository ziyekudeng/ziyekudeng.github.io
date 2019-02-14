---
layout: post
title: 朋友圈微博feed流，推拉实践
no-post-nav: true
category: life
tags: [life]
---



为了对比“拉”“推”两种方案，同时也为了避免大伙在两篇文章切换，故先简单赘述背景与“拉”（读扩散）方案。

 

feed业务特点：有类似于“关注/粉丝”的好友关系，个人的主页由别人发布的feed消息组成。

 

feed业务典型动作：关注，取关，发布feed，拉取自己的主页feed流。

 

feed业务核心数据：关系数据，feed数据。

 

一、拉模式 “读扩散”方案简介

例如，某feed系统里有ABCD四个用户，其中：

A关注了BC，D关注了B

![](https://ziyekudeng.github.io/assets/images/2019/0202/write-feed/1.webp)

其关系存储又包含关注关系与粉丝关系，“A关注了BC，D关注了B”的潜台词是“B有两个粉丝AD，C有一个粉丝A”。

 

B发布过四条feed：msg1, msg3, msg5, msg10

C发布过两条feed：msg2, msg8

![](https://ziyekudeng.github.io/assets/images/2019/0202/write-feed/2.webp)

每一个用户，都有一个feed队列，记录自己曾经发布的所有feed数据。

 

在拉模式中，发布一条feed的流程非常简单，例如C新发布了一条msg12：

![](https://ziyekudeng.github.io/assets/images/2019/0202/write-feed/3.webp)

此时只需往C的feed队列里加入一条feed即可。

 

在拉模式中，取消关注的流程也非常简单，例如A取消关注C：

![](https://ziyekudeng.github.io/assets/images/2019/0202/write-feed/4.webp)

此时只需要在A的关注列表里删除C，并在C的粉丝列表里删除A即可。

 

在拉模式中，用户A获取“由别人发布的feed组成的主页”的过程及其复杂，此时需要：

获取A的关注列表

获取所关注列表中，所有用户发布的feed

对消息进行rank排序（假设按照发布时间排序），分页取出对应的一页feeds

 

feed流的拉模式（“读扩散”）的优点是：

存储结构简单，数据存储量较小，关系数据与feed数据都只存一份

关注，取关，发布feed的业务流程非常简单

存储结构，业务流程都比较容易理解，适合项目早期用户量、数据量、并发量不大时的快速实现

 

缺点也显而易见：

拉取朋友圈feed流列表的业务流程非常复杂

有多次数据访问，并且要进行大量的内存计算，网络传输，性能较低

 

二、推模式 “写扩散”方案简介

推模式（写扩散），关系数据的存储与拉模式（读扩散）完全一样。

![](https://ziyekudeng.github.io/assets/images/2019/0202/write-feed/5.webp)

 

feed数据，每个用户也存储自己发布的feed。

![](https://ziyekudeng.github.io/assets/images/2019/0202/write-feed/6.webp)

如上图：

B曾经发布过1，3，5，10

C曾经发布过2，8

画外音：不妨设，这里的msgid按照feed的发布时间偏序。

 

feed数据存储，与拉（读扩散）不同的是，每个用户还需要存储自己收到的feed流。

![](https://ziyekudeng.github.io/assets/images/2019/0202/write-feed/7.webp)

如上图：

A关注了BC，所以A的接收队列是1，2，3，5，8，10

D关注了B，所以D的接受队列是1，3，5，10

 

在推模式（写扩散）中，获取“由别人发布的feed组成的主页”会变得异常简单，假设一页消息为3条feed，A如果要看自己朋友圈的第二页消息，直接返回1，2，3即可。

画外音：第一页朋友圈是最新的消息，即5，8，10。

 

在推模式（写扩散）中，发布一条feed的流程会更复杂一点。

![](https://ziyekudeng.github.io/assets/images/2019/0202/write-feed/8.webp)

例如B新发布了一条msg12：

在B的发布feed存储里加入消息12

查询B全部粉丝AD

在粉丝AD的接收feed存储里也加入消息12

 

之所以该方案称为推模式（写扩散），就是因为，用户发布feed的时候：

直接将feed推到了粉丝的接收列表里，故称为“推模式”

不止写发布feed存储，而且要写多个粉丝的接收feed存储，故称为“写扩散”

 

在推模式（写扩散）中，添加关注的流程也会变得复杂。

![](https://ziyekudeng.github.io/assets/images/2019/0202/write-feed/9.webp)

例如D新增关注C：

在D的关注存储里添加C

在C的粉丝存储里添加D

在D的接收feed存储里加入C发布的feed

画外音：有些产品有这样的逻辑，“关注之后才能看到feed”，这样的话就不需要第三步，旧feed无需插入。

 

在推模式（写扩散）中，取消关注的流程也会变得复杂。

![](https://ziyekudeng.github.io/assets/images/2019/0202/write-feed/10.webp)

例如A取消关注C：

在A的关注存储里删除C

在C的粉丝存储里删除A

在A的接收feed存储里删除C发布的feed 



feed流的推模式（写扩散）的优点是：

消除了拉模式（读扩散）的IO集中点，每个用户都读自己的数据，高并发下锁竞争少

画外音：拉模式（读扩散）中，用户发布feed存储容易称为IO瓶颈。

拉取朋友圈feed流列表的业务流程异常简单，速度很快

拉取朋友圈feed流列表，不需要进行大量的内存计算，网络传输，性能很高

画外音：feed业务是典型的读多写少业务场景，读写比甚至高于100：1，即平均发布1条消息，有至少100次阅读。

 

其缺点是：

极大极大消耗存储资源，feed数据会存储很多份，例如杨幂5KW粉丝，她每次一发博文，消息会冗余5KW份

画外音：有朋友提出，可以存储一份消息实体，只冗余msgid，这样的话，拉取feed流列表时，还要再次拉取实体，网络时延会更长，所以很多公司选择直接冗余消息实体，当然，这是一个用户体验与存储量的折衷设计。

新增关注，取消关注，发布feed的业务流会更复杂

 

三、小结

feed流业务的推拉模式小结：

拉模式，读扩散，feed存一份，存储小，用户集中访问数据，性能差

推模式，写扩散，feed存多份，用冗余存储换锁冲突，性能高



