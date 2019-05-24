---
layout: post
title: 缓存架构设计 - 缓存与数据库不一致，咋办？
category: springcloud
tags: [springcloud]
keywords: 架构
---

![](https://ziyekudeng.github.io/assets/images/2019/0212/agreement/1.png)

![](https://ziyekudeng.github.io/assets/images/2019/0212/agreement/2.png)

![](https://ziyekudeng.github.io/assets/images/2019/0212/agreement/3.png)

![](https://ziyekudeng.github.io/assets/images/2019/0212/agreement/4.png)

五、结尾

问：如何完全避免，主从同步时间差，数据的一致性？

![](https://ziyekudeng.github.io/assets/images/2019/0212/agreement/5.png)

![](https://ziyekudeng.github.io/assets/images/2019/0212/agreement/6.png)

![](https://ziyekudeng.github.io/assets/images/2019/0212/agreement/7.png)

![](https://ziyekudeng.github.io/assets/images/2019/0212/agreement/8.png)

![](https://ziyekudeng.github.io/assets/images/2019/0212/agreement/9.png)





 

问：该方案，只能优化，并发读写情况下，缓存与数据库一致性问题。如果，缓存与数据库两次操作，原子性被破坏（例如：修改数据库成功，淘汰缓存失败，导致的数据不一致），如何优化数据的一致性呢？

![](https://ziyekudeng.github.io/assets/images/2019/0212/agreement/10.png)

![](https://ziyekudeng.github.io/assets/images/2019/0212/agreement/11.png)

![](https://ziyekudeng.github.io/assets/images/2019/0212/agreement/12.png)

![](https://ziyekudeng.github.io/assets/images/2019/0212/agreement/13.png)

![](https://ziyekudeng.github.io/assets/images/2019/0212/agreement/14.png)



