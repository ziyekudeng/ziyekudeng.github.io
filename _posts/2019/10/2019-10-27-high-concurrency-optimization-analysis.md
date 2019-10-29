---
layout: post
title: Java高并发秒杀分析与解决
category: tools
tags: [tools]
---
 本文链接：[https://blog.csdn.net/yd201430320529/article/details/70544203](https://blog.csdn.net/yd201430320529/article/details/70544203)


#### 先来分析一下java 控制事务行为

**如下图可知，java事务是串联发生的，即当一个事务还没有执行完，其他事务都是要等待而被阻塞。**
![](https://img-blog.csdn.net/20170423201635593?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWQyMDE0MzAzMjA1Mjk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**那一个事务具体要干什么东西呢？或者说高并发的瓶颈在哪里，看下图**
![](https://img-blog.csdn.net/20170423202419209?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWQyMDE0MzAzMjA1Mjk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
**分析：**
**1、一个Java事务可能有多条sql语句的操作，此时算上sql语句操作时间。**
**2、网络会出现延迟。**
**3、GC（java回收机制）处理时，会消耗一定的时间**
**综上，Java事务处理时间=sql消耗+网络延迟+GC消耗，一个java事务处理时间大约在2ms，也就是说1秒只能有500个事务串行发生，这是非常不高效的。**

* * *

#### 而之前说的行级锁(保证java事务之间可以串行进行，从而保证数据的一致性)，则在commit之后就会释放，所以我们的优化方向，就是减少行级锁的持有时间

**我们先分析如何判断update语句成功更新**
**1、update语句自身没有报错**
**2、客户端确认update影响记录数**
**所以我们在这个方面的优化方案时，把客户端逻辑放到mysql服务端，避免网络延迟和GC**
**即为，使用存储过程：整个事务在Mysql端进行，因为这样可以将网络延迟和GC消耗降至最低，大大减少了整个事务的运行时间，同时，Mysql一秒中可以做出几万次的事务，是非常高效快速的。**

* * *

### 下面开始秒杀的实际优化

**首先优化的应该是秒杀地址暴露接口，这里说下为啥要优化它**
**1.因为秒杀地址暴露接口是根据后台判断秒杀开始时间，结束时间，以及当前时间来给客户端返回秒杀地址的，所以无法放进CDN中。**
**2.秒杀地址暴露是在秒杀执行之前需要进行的动作，因此伴有大量的并发操作，同时需要请求数据库，因此必须要优化。**
**3.针对这些情况，我们可以用redis，优化秒杀地址暴露接口。**

**首先建立一个redisDao类，专门进行与Redis有关的业务逻辑**
![](https://img-blog.csdn.net/20170427111247199?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWQyMDE0MzAzMjA1Mjk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

* * *

**这里需要声明一下，如果从redis中获得数据，是获得byte的数组，而不是具体的类，因为redis无法帮你反序列化，因此，反序列化的操作应该由你自己完成。**

**因为java内置的序列化接口Serializable，效率没有protostuff自定义序列化高，protostuff的反序列化效率比Serializable高了很多倍，而且压缩后的空间比Serializable小了10倍左右，因此，使用protostuff进行自定义反序列化操作。**

**这里生成protostuff的一个schem,其原理是将Seckill.class的字符码传过去，然后利用反射的原理，获得Seckill中的属性值等。**
![](https://img-blog.csdn.net/20170427152245359?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWQyMDE0MzAzMjA1Mjk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**接下来，通过redis获得缓存中的Seckill对象，以及一些protostuff的反序列化操作**
![](https://img-blog.csdn.net/20170427152922942?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWQyMDE0MzAzMjA1Mjk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![](https://img-blog.csdn.net/20170427153258916?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWQyMDE0MzAzMjA1Mjk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**接下来是将Seckill对象放进redis缓存中，这里同样利用protostuff来序列化Seckill，LinkedfBuffer为缓冲器，保证在大型类序列化时能够起到缓冲的作用，timeout设置缓存时间，这里设置一小时之后redis中的Seckill数据就会消失。result为redis返回的结果，可以依据此判断缓存是否成功。**
![](https://img-blog.csdn.net/20170427155124284?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWQyMDE0MzAzMjA1Mjk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**这两个方法写好后，我们回到SeckillServiceImpl中，将之前Seckill直接从数据库中获取的流程，改成先判断redis里面有没有Seckill缓存，如果有，直接从redis中获取，如果没有，从数据库中取出Seckill，将其放入redis中。这里运用在超时的基础上维护redis数据的一致性，是最基础的维护一致性的方式。**
![](https://img-blog.csdn.net/20170427160333208?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWQyMDE0MzAzMjA1Mjk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**以上就是通过redis来缓存信息，下面我们来看执行秒杀操作时的并发优化**

**上面我们总结过，行级锁的主要发生地，主要是库存update操作这里，之前的业务逻辑如下图**
**由下图可知，无论这个商品是否可以被秒杀，都会执行update语句，获得行级锁，再返回是否update成功的结果，并通过这个update操作结果，来插入购买明细，这是非常错误的。**
**我们知道，insert操作也可以判断是否插入成功（返回值0代表失败，1代表成功），但是，insert操作是不用通过行级锁来阻塞其他事务的，因此，我们可以将update语句和insert语句调换位置（先过滤不符合update操作条件的事务，即挡住一部分重复秒杀），通过insert语句的结果，来判断是否需要update操作，这样，阻塞时间将会大大减少，增加了系统的性能。**
![](https://img-blog.csdn.net/20170427164457622?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWQyMDE0MzAzMjA1Mjk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**下图是调换位置后的事务执行**
![](https://img-blog.csdn.net/20170427165514297?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWQyMDE0MzAzMjA1Mjk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**下面来看看具体代码的修改**
![](https://img-blog.csdn.net/20170427170216770?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWQyMDE0MzAzMjA1Mjk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

* * *

**接下来是深度优化，即将事务SQL在Mysql端进行（存储过程），减少网络延迟和GC带来的时间消耗**

**下面将插入购买明细和更新库存操作，放在存储过程里面执行**
![](https://img-blog.csdn.net/20170427173436443?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWQyMDE0MzAzMjA1Mjk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![](https://img-blog.csdn.net/20170427173004809?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWQyMDE0MzAzMjA1Mjk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**这里简单说明一下存储过程：**
**1.存储过程主要优化事务行级锁持有时间**
**2.不要过度依赖存储过程**
**3.简单的逻辑可以应用存储过程**
**4.一秒可以接受4000个请求，这是基本足够的。**

**下面说说在java客户端是如何调用存储过程的**

**在SeckillDao中，使用存储过程执行秒杀**
![](https://img-blog.csdn.net/20170427180200104?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWQyMDE0MzAzMjA1Mjk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**然后在seckillDao.xml中实现它**
![](https://img-blog.csdn.net/20170427180607474?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWQyMDE0MzAzMjA1Mjk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**在SeckillService接口中定义一个方法，作为利用存储过程来实现事务的方法，其参数和executeSeckill方法一样**
![](https://img-blog.csdn.net/20170427175443891?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWQyMDE0MzAzMjA1Mjk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**然后编写其实现类**
![](https://img-blog.csdn.net/20170427181557870?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWQyMDE0MzAzMjA1Mjk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![](https://img-blog.csdn.net/20170427181728687?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWQyMDE0MzAzMjA1Mjk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 以上，就是高并发优化分析的全部内容了。



### 相关文章

*   [JAVA高并发秒杀系统构建之——Web层](https://blog.csdn.net/yd201430320529/article/details/70226619)
*   [JAVA高并发秒杀系统构建之——Service层](https://blog.csdn.net/yd201430320529/article/details/70196432)

*   [JAVA高并发秒杀系统构建之——Dao层](https://blog.csdn.net/lewky_liu/article/details/78159983)
*   [JAVA高并发秒杀系统构建之——高并发优化](https://blog.csdn.net/lewky_liu/article/details/78166080)


