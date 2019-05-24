---
layout: post
title: 群消息，究竟存1份还是多份？
category: springcloud
tags: [springcloud]
keywords: 架构
---


群信息，用户信息，群成员关系都是基础数据：

    group_info(gid, group_info);
    
    user_info(uid, user_info);
    
    group_members(gid, uid);



假设一个群(gid)里有4个成员，其中三个在线(A, uid1, uid2)，一个不在线(uid3)。



A发送了一条消息，很容易想到，对于不同的群友消息存多份，每个群友一个队列来存储。但由于在线的用户会实时的收到消息，所以暂定只为离线的用户存储。

 

用户收到的群消息，也是基础数据：

    user_msgs(uid,msgid,gid,sender_uid,time,content);

![](https://ziyekudeng.github.io/assets/images/2019/0202/group-news/1.webp)



很容易想到，整个群消息的发送流程如上图1-4：

（1）发送消息

（2）查询状态

（3）不在线的存储离线

（4）在线的实时推送

 

“在线的群友不存储，离线的群友才存储”会带来的问题是，如果第四步发生异常，群友会丢失消息。



消息的可达性是聊天系统中最重要的要素（没有之一），故这个方案是不行的，需要优化为“不管是否在线，都要先存储”。

![](https://ziyekudeng.github.io/assets/images/2019/0202/group-news/2.webp)


发送群消息的流程优化为，如上图1-4：

（1）发送消息

（2）所有人都存一份

（3）查询状态

（4）在线的实时推送

 

先将消息落地，能够保证消息可达性，那何时才能删除已经落地的群消息呢？

![](https://ziyekudeng.github.io/assets/images/2019/0202/group-news/3.webp)


对于在线的群友，收到群消息后，给个ack确认，才能删除。

画外音：逻辑删除，还是物理删除，根据业务是否有消息漫游决定。


![](https://ziyekudeng.github.io/assets/images/2019/0202/group-news/4.webp)


对于离线的群友，在下次登陆后，拉取完离线消息，再给ack确认，才能删除。

 

总之，为了保证消息的可达性，不管是在线消息，还是离线消息，必须接收方给ack确认，才能删除消息。

 

“不管是否在线，都冗余一份群消息”带来的问题是，同一条消息存储了很多次，对磁盘和带宽造成了很大的浪费。很容易想到的优化是：群消息实体存储一份，用户只冗余消息ID。

![](https://ziyekudeng.github.io/assets/images/2019/0202/group-news/5.webp)


故基础数据可以由：

user_msgs(uid,msgid,gid,sender_uid,time,content);

优化为：

group_msgs(msgid,gid,sender_uid,time,content);

user_msgs(uid, msgid, gid);

 

这个优化，对于消息投递，以及消息删除的核心流程没有影响，几个实践为：

在线用户投递消息实体，ack消息ID

离线用户先拉取消息ID，再拉取消息实体，再ack消息ID

 

如此这般，假如在某个群友A期间，群里陆续发送了N条消息，则user_msgs(uid, msgid, gid)里，会有 uidA -> mid1,mid2, mid3, … midN 等N条离线记录，拉取离线消息时，可以把这N条消息一次性拉取出来，然后再删除：

delete from user_msgs 

where msgid in($mid1,$mid2…, $midN) and gid=$gid

 

然而，群消息具备“偏序”特性，上面的一次性删除完全可以优化为：

delete from user_msgs 

where msgid >= $mid1 and gid=$gid

 

这就意味着，每个用户只需要记录“最近一次收到的消息ID”，而不用记录“所有未收到的消息ID集合”，每当收在线消息ack，以及拉离线消息ack时，只需要更新这个“最近一次收到的消息ID”即可。

 

于是乎，基础数据可以由：

group_members(gid, uid);

group_msgs(msgid,gid,sender_uid,time,content);

user_msgs(uid, msgid, gid);

优化为：

group_members(gid, uid, last_ack_msgid);

group_msgs(msgid,gid,sender_uid,time,content);

user_msgs(uid, msgid, gid); // 不再需要


![](https://ziyekudeng.github.io/assets/images/2019/0202/group-news/6.webp)



即，群消息只存储一份，群友无需冗余任何消息实体，或者消息ID了。


![](https://ziyekudeng.github.io/assets/images/2019/0202/group-news/7.png)


对于在线的群友，收到群消息后，修改这个last_ack_msgid。


![](https://ziyekudeng.github.io/assets/images/2019/0202/group-news/8.webp)


对于离线的群友，拉取群消息后，也修改这个last_ack_msgid。

画外音：这里的讨论，仅限于接收方收到了哪些消息，和发送方的已读回执没有关系。

 

总结

任何架构方案都不是灵光一现，而是逐步迭代优化产生的：

存多份，只存在线，消息容易丢

存多份，所有群友都存储，消息冗余多

存多份，只存ID，未利用偏序

存一份，只存last_ack_msgid

 

架构不（只）是设计出来的，更是演进出来的。



