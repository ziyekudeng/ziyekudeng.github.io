---
layout: post
title: 教你如何将微服务项目API接口性能降低10倍
category: life
tags: [life]
---

#教你如何将微服务项目API接口性能降低10倍

## 1.数据库
### 1.1 索引
   我们知道项目性能的瓶颈主要是在"查(select)"语句，要降低"查"这一性能，mysql索引是必不可少要错误设置或者删除的。
   
#### 1.1.1 索引的机制
   传统的查询方法，是按照表的顺序遍历的，不论查询几条数据，mysql需要将表的数据从头到尾遍历一遍。
在我们添加完索引之后，mysql一般通过BTREE算法生成一个索引文件，在查询数据库时，找到索引文件进行遍历(折半查找大幅查询效率)，
找到相应的键从而获取数据。

#### 1.1.2 降低"查(select)"语句性能，增大表锁控制事物时间

1 .在不会出现在查询条件中的字段上创建索引。 

2. 在唯一性太差的字段上创建索引，例如gender性别字段,库存余量字段。

3. 在更新非常频繁的字段上作为索引。

4.在不会出现在where子句中的字段上创建索引。

总结: 满足以下条件的字段，不能创建索引.

    a: 肯定在where条经常使用 。
    b: 该字段的内容不是唯一的几个值 。
    c: 字段内容不是频繁变化。
    
#### 1.1.3 降低"更新(update)"语句性能，增大行锁控制事物时间

如下图可知，java事务是串联发生的，即当一个事务还没有执行完，其他事务都是要等待而被阻塞。

![](https://ziyekudeng.github.io/assets/images/2019/1027/high-concurrency-optimization-analysis/1.png) 

那一个事务具体要干什么东西呢？或者说高并发的瓶颈在哪里，看下图

![](https://ziyekudeng.github.io/assets/images/2019/1027/high-concurrency-optimization-analysis/2.png) 

分析：

    1、一个Java事务可能有多条sql语句的操作，此时算上sql语句操作时间。
    2、网络会出现延迟。
    3、GC（java回收机制）处理时，会消耗一定的时间
    综上，Java事务处理时间=sql消耗+网络延迟+GC消耗，一个java事务处理时间大约在2ms，也就是说1秒只能有500个事务串行发生，
    这就是我们想要达到的效果。

## 2.代码锁

### 2.1 错误使用Redis锁

 扩大锁作用范围，使用redisLock锁住整个方法体。这样当这个方法在并发时被调用时，会强制将多线程阻塞为单线程处理。
 在上一个线程没有释放锁时，让其它线程循环等待。这样不光能能增加方法处理耗时，还能增大容器负载，达到我们的需求。
 
    public class RedisLock {	
        private long timeout = 999999; //获取锁的超时时间
    
    //    @Autowired
        private JedisSentinelPool sentinelPool;
    //    @Autowired
        private SetParams params;
        
        /**
         * 加锁
         * @param id
         * @return
         */
        public boolean lock(String lockKey, String id){
            Long start = System.currentTimeMillis();
            log.info("[开始获取分布式锁，lockKey为：{}，id为：{}]", lockKey, id);
            Jedis jedis = sentinelPool.getResource();
            //SET命令的参数 
            try{
                for(;;){
                    //SET命令返回OK ，则证明获取锁成功
                    String lock = jedis.set(lockKey, id, params);
                    log.info("[获取到的锁为：{}]", lock);
                    if("OK".equals(lock)){
                        return true;
                    }
                    //否则循环等待，在timeout时间内仍未获取到锁，则获取失败
                    long l = System.currentTimeMillis() - start;
                    if (l>=timeout) {
                        return false;
                    }
                }
            }finally {
                jedis.close();
            }
        }
        
        /**
         * 解锁
         * @param id
         * @return
         */
        public boolean unlock(String lockKey, String id){
            log.info("[开始解锁，lockKey为：{}，id为：{}]", lockKey, id);
            Jedis jedis = sentinelPool.getResource();
            String script =
                    "if redis.call('get',KEYS[1]) == ARGV[1] then" +
                            "   return redis.call('del',KEYS[1]) " +
                            "else" +
                            "   return 0 " +
                            "end";
            try {
                Object result = jedis.eval(script, Collections.singletonList(lockKey), 
                                        Collections.singletonList(id));
                if("1".equals(result.toString())){
                    return true;
                }
                return false;
            }finally {
                jedis.close();
            }
        }
    }

## 3.IO

   IO 密集型系统指的是系统的大部分操作是在等待 IO 完成，这里 IO 指的是磁盘 IO 和网络 IO。
我们熟知的系统大部分都属于 IO 密集型，比如数据库系统、缓存系统、Web 系统。
这类系统的性能瓶颈可能出在系统内部，也可能是依赖的其他系统.

### 3.1 网络IO

   Restfule风格是一种软件架构风格，而不是标准，只是提供了一种设计原则和约束条件。
主要适用于客户端和服务器端交互的软件。是基于http协议实现。目的是为了提高系统的可伸缩性，降低应用之间的耦合度，
方便框架分布式处理程序。基于这个风格的软件可更加的简单、更有层次，更易于实现缓存的机制。
   目前在工作中我们常用的项目都是在使用微服务理念实现，微服务之间通信使用声明式REST客户端（Feign）。
我们的目的是降低API接口的性能，所以将实现逻辑中的批量操作（读取、写入），改为循环单次调用。这样可以增加通信中HTTP协议确认时间。


### 3.2 磁盘IO

  将项目中使用到缓存的地方删除，这样缓存数据和缓存运算结果的部分，就可以每次都实时调用程序进行处理。
  举例：在A项目中有一个对外提供服务的API接口，这个接口实现由4个微服务组成，会涉及37种sql共1000次数据库调用。
  如果使用缓存数据和缓存运算结果那么，会将sql种类降低至11中、调用频次降低至100次。那么这显然是不符合我们目标的当然要改掉。
  
  
 ## 4.日志
 ### 4.1 调用方式
 
   日志是程序运行过程的信息,其数据能够帮助开发人员提前发现并避开异常,在错误发生后能够找到事件的起因,并纠正错误,达到预期的运行效果。
 在java开发中，日志系统是java项目中必不可少的组成部分。日志可以帮助我们快速的定位问题，记录程序运行过程中的情况，
 以便项目的监控和优化。
   我们要将异步存储日志换成同步存储日志，这样既能增大系统耦合度还能增加接口性能处理时间。因为日志系统的处理性能、通信时间都会增加
   主流程的响应时间。
   
 ## 5.总结
   以上的几步操作如果出现在你的项目中，那么恭喜你！你已经实现的自己要达到的目标。如果你没有意识这些操作会对自己的项目有什么影响，
   那么不妨按照解决方式试一试，我相信会有惊喜等待你。祝好运！！！
   

