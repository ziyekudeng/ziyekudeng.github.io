---
layout: post
title: feed流拉取，读扩散，究竟是啥？
no-post-nav: true
category: life
tags: [life]
---




任何脱离业务的架构设计都是耍流氓。

 

哪些产品是feed流典型业务？

答：微博，微信朋友圈，Pinterest是典型的feed流业务，系统中的每一条消息就是一个feed。

 

这类业务的特点是：

有好友关系，例如关注，粉丝

我们的主页由别人发布的feed组成

 

这类业务的典型动作是：

关注，取关

发布feed

拉取自己的主页feed流

 

这类业务的核心元数据是：

关系数据

feed数据

 

feed流的“拉取”与“推送”实现，是个怎么回事？

答：feed流业务最大的特点是“我们的主页由别人发布的feed组成”，获得朋友圈消息feed流集合，从技术上说，主要有“拉取”与“推送”两种方式。feed流的推与拉主要指的是这里。

 

今天将简述拉模式（圈内说的较多的是“读扩散”）的核心数据结构，核心流程，优缺点。

 

例如：某feed系统里有ABCD四个用户，其中：

A关注了BC，D关注了B

![](https://ziyekudeng.github.io/assets/images/2019/0202/read-feed/1.webp)

其关系存储又包含关注关系与粉丝关系，“A关注了BC，D关注了B”的潜台词是“B有两个粉丝AD，C有一个粉丝A”。

 

B发布过四条feed：msg1, msg3, msg5, msg10

C发布过两条feed：msg2, msg8

![](https://ziyekudeng.github.io/assets/images/2019/0202/read-feed/2.webp)

每一个用户，都有一个feed队列，记录自己曾经发布的所有feed数据。

 

在拉模式中，发布一条feed的流程非常简单，例如C新发布了一条msg12：

![](https://ziyekudeng.github.io/assets/images/2019/0202/read-feed/3.webp)

此时只需往C的feed队列里加入一条feed即可。



在拉模式中，取消关注的流程也非常简单，例如A取消关注C：

![](https://ziyekudeng.github.io/assets/images/2019/0202/read-feed/4.webp)

此时只需要在A的关注列表里删除C，并在C的粉丝列表里删除A即可。

 

在拉模式中，用户A获取“由别人发布的feed组成的主页”的过程比较复杂，此时需要：

获取A的关注列表

    list<gz_uid> = select uid from GZ where uid=A

获取所关注的用户发布的feed

    list<msg> = NULL;
    
    for(uid in list<gz_uid>){
    
             list<some_msg> = 
    
                select * from F where uid=$uid offset | limit
    
             list<msg> += list<some_msg>;
    
    }

对消息进行rank排序（假设按照发布时间排序），分页取出对应的一页feeds

    sort_msg_by_time(list<msg>);
    
    get_one_page(list<msg>, page_num);

 

feed流的拉模式（“读扩散”）有什么优缺点？

优点：

存储结构简单，数据存储量较小，关系数据与feed数据都只存一份

取消关注，发布feed的业务流程非常简单

存储结构，业务流程都比较容易理解，非常适合项目早期用户量、数据量、并发量不大时的快速实现

 

缺点也显而易见：

拉取朋友圈feed流列表的业务流程非常复杂

有多次数据访问，并且要进行大量的内存计算，大量数据的网络传输，性能较低

 

在拉模式中，系统的瓶颈容易出现在“用户所发布feed列表”的读取上，而每个用户发布feed的频率其实是很低的，此时，架构优化的核心是通过缓存降低数据存储磁盘IO。

 

当用户量、数据量、并发量数据逐步增加之后，拉模式会慢慢扛不住了，需要升级优化，但对于“取消关注”与“发布feed”这两个写流程又会有冲击和影响，具体架构应该如何迭代，下一章和大家分享（额，今天笔记本没电了）。

 

架构，不只是设计出来的，更是演进而来的。



