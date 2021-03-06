---
layout: post
title: 数据库锁和Java中的锁
category: tools
tags: [tools]
---

本文来源：

https://mp.weixin.qq.com/s/r9Ach_AxuO2ErxCDz_TBBg


# 数据库锁

提高共享资源并发性的方式是让锁定的对象更加有选择性，尽量只锁定需要修改的部分数据，
而不是所有的资源，更理想的方式是只对会修改的数据片进行精确的锁定。加锁也需要消耗资源，锁的各种操作，包括获得锁，检查锁是否已经解除、释放锁都会增加系统的开销。

### 读锁
读锁是共享的，或者说是相互不阻塞的，多个客户在同一时刻可以读取同一资源，而不互相干扰。读锁不会产生死锁，有表级读锁和行级读锁。

### 写锁
则是排他的，也就是说一个写锁会阻塞其他的写锁和读锁，这是出于安全策略的考虑，只有这样，才能确保在给定的时间内，只有一个用户能执行写入。写锁不会产生死锁，有表级写锁和行级写锁。

### 意向锁
意向锁在表锁和行级锁同时存在时才会出现。

#### 举个我以前看到的例子：
两中类型的锁共存的问题考虑这个例子：事务A锁住了表中的一行，让这一行只能读，不能写。之后，事务B申请整个表的写锁。如果事务B申请成功，那么理论上它就能修改表中的任意一行，这与A持有的行锁是冲突的。数据库需要避免这种冲突，就是说要让B的申请被阻塞，直到A释放了行锁。

#### 数据库要怎么判断这个冲突呢？

step1：判断表是否已被其他事务用表锁锁表
step2：判断表中的每一行是否已被行锁锁住。
注意step2，这样的判断方法效率实在不高，因为需要遍历整个表。于是就有了意向锁。在意向锁存在的情况下，事务A必须先申请表的意向共享锁，成功后再申请一行的行锁。

#### 在意向锁存在的情况下，上面的判断可以改成

step1：不变
step2：发现表上有意向共享锁，说明表中有些行被共享行锁锁住了，因此，事务B申请表的写锁会被阻塞。
可以看出有意向锁的情况下，效率明显提高，意向锁是有程序自动提高的，不需要我们自己申明。

### 更新锁
当一个事务执行修改语句时，数据库系统会先为事务分配一把更新锁。当读取数据完毕，执行更新操作时，会把更新锁升级为独占锁。更新锁与共享锁是兼容的，也就是说，一个资源可以同时放置更新锁和共享锁，但是最多放置一把更新锁。这样，当多个事务更新相同的数据时，只有一个事务能获得更新锁，然后再把更新锁升级为独占锁，其他事务必须等到前一个事务结束后，才能获取得更新锁，这就避免了死锁。允许多个事务同时读锁定的资源，但不允许其他事务修改它。

### 表锁
表锁和行锁是从粒度来进行区分的，在MyISAM中只用到表锁，不会有死锁的问题，锁的开销也很小，但是相应的并发能力很差。

### 行锁
在innodb实现了行级锁和表锁，锁的粒度变小了，并发能力变强，但是相应的锁的开销变大，很有可能出现死锁。InnoDB行锁是通过给索引上的索引项加锁来实现的，只有通过索引条件检索数据，InnoDB才使用行级锁，否则，InnoDB将使用表锁。

### MVCC
是行级锁的一个变种，在很多情况下避免了加锁操作，具体的实现是通过保存数据在某个时间点的快照来实现的。实现方式有乐观锁和悲观锁两种方式。InnoDB的MVCC，是通过在每行记录后面保存两个隐藏的列来实现的。这两个列，一个保存了行的创建时间，一个保存了行的过期时间。存储的不是实际的是时间值，而是系统的版本号。每开始一个新的事务，系统版本号都会自动递增。

# Java中的锁

### 可重入锁
   可重入锁又名递归锁，是指在同一个线程在外层方法获取锁的时候，在进入内层方法会自动获取锁。说的有点抽象，下面会有一个代码的示例。
    对于Java ReetrantLock而言，从名字就可以看出是一个重入锁，其名字是Re entrant Lock 重新进入锁。
    对于Synchronized而言，也是一个可重入锁。可重入锁的一个好处是可一定程度避免死锁。

### 公平锁/非公平锁
   公平锁是指多个线程按照申请锁的顺序来获取锁。
    非公平锁是指多个线程获取锁的顺序并不是按照申请锁的顺序，有可能后申请的线程比先申请的线程优先获取锁。有可能，会造成优先级反转或者饥饿现象。
    对于Java ReetrantLock而言，通过构造函数指定该锁是否是公平锁，默认是非公平锁。非公平锁的优点在于吞吐量比公平锁大。
    对于Synchronized而言，也是一种非公平锁。由于其并不像ReentrantLock是通过AQS的来实现线程调度，所以并没有任何办法使其变成公平锁。

### 分段锁
   分段锁其实是一种锁的设计，并不是具体的一种锁，对于ConcurrentHashMap而言，其并发的实现就是通过分段锁的形式来实现高效的并发操作。
    我们以ConcurrentHashMap来说一下分段锁的含义以及设计思想，ConcurrentHashMap中的分段锁称为Segment，它即类似于HashMap（JDK7和JDK8中HashMap的实现）的结构，即内部拥有一个Entry数组，数组中的每个元素又是一个链表；同时又是一个ReentrantLock（Segment继承了ReentrantLock）。
    当需要put元素的时候，并不是对整个hashmap进行加锁，而是先通过hashcode来知道他要放在哪一个分段中，然后对这个分段进行加锁，所以当多线程put的时候，只要不是放在一个分段中，就实现了真正的并行的插入。
    但是，在统计size的时候，可就是获取hashmap全局信息的时候，就需要获取所有的分段锁才能统计。
    分段锁的设计目的是细化锁的粒度，当操作不需要更新整个数组的时候，就仅仅针对数组中的一项进行加锁操作。

### 偏向锁/轻量级锁/重量级锁
   这三种锁是指锁的状态，并且是针对Synchronized。在Java 5通过引入锁升级的机制来实现高效Synchronized。这三种锁的状态是通过对象监视器在对象头中的字段来表明的。
    偏向锁是指一段同步代码一直被一个线程所访问，那么该线程会自动获取锁。降低获取锁的代价。
    轻量级锁是指当锁是偏向锁的时候，被另一个线程所访问，偏向锁就会升级为轻量级锁，其他线程会通过自旋的形式尝试获取锁，不会阻塞，提高性能。
    重量级锁是指当锁为轻量级锁的时候，另一个线程虽然是自旋，但自旋不会一直持续下去，当自旋一定次数的时候，还没有获取到锁，就会进入阻塞，该锁膨胀为重量级锁。重量级锁会让他申请的线程进入阻塞，性能降低。

### 自旋锁
   在Java中，自旋锁是指尝试获取锁的线程不会立即阻塞，而是采用循环的方式去尝试获取锁，这样的好处是减少线程上下文切换的消耗，缺点是循环会消耗CPU。

 

