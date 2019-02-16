---
layout: post
title: Spring配置事务 service 异常捕获回滚问题
category: spring
tags: [spring]
---



**1.首先来看一Spring配置事务的传播种类：**

Spring在TransactionDefinition接口中规定了7种类型的事务传播行为，它们规定了事务方法和事务方法发生嵌套调用时事务如何进行传播：

　　PROPAGATION_REQUIRED

　　如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。这是最常见的选择。

　　PROPAGATION_SUPPORTS

　　支持当前事务，如果当前没有事务，就以非事务方式执行。

　　PROPAGATION_MANDATORY

　　使用当前的事务，如果当前没有事务，就抛出异常。

　　PROPAGATION_REQUIRES_NEW

　　新建事务，如果当前存在事务，把当前事务挂起。

　　PROPAGATION_NOT_SUPPORTED

　　以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。

　　PROPAGATION_NEVER

　　以非事务方式执行，如果当前存在事务，则抛出异常。

　　PROPAGATION_NESTED

　　如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作。

　　在使用过程中需要注意以下几点：

　　1) NESTED和NEW的区别

　　PROPAGATION_REQUIRES_NEW 和 PROPAGATION_NESTED是容易混淆的两个传播行为。PROPAGATION_REQUIRES_NEW 启动一个新的、和外层事务无关的“内部”事务。该事务拥有自己的独立隔离级别和锁，不依赖于外部事务，独立地提交和回滚。当内部事务开始执行时，外部事务将被挂起，内务事务结束时，外部事务才继续执行。由此可见， PROPAGATION_REQUIRES_NEW 和 PROPAGATION_NESTED 的最大区别在于：PROPAGATION_REQUIRES_NEW 将创建一个全新的事务，它和外层事务没有任何关系，而 PROPAGATION_NESTED 将创建一个依赖于外层事务的子事务，当外层事务提交或回滚时，子事务也会连带提交和回滚。嵌套事务不能够提交，它必须通过外层事务来完成提交的动作，外层事务的回滚也会造成内部事务的回滚。

　　2) 当方法被设置为PROPAGATION_NOT_SUPPORTED时，外层业务方法的事务会被挂起，当内部方法运行完成后，外层方法的事务重新运行。如果外层方法没有事务，直接运行，不需要做任何其它的事。

　　3) 当业务方法被设置为PROPAGATION_NEVER时，它将不能被拥有事务的其它业务方法调用。

　　4) 当业务方法被设置为PROPAGATION_MANDATORY时，它就不能被非事务的业务方法调用。

**2.Spirng默认抛出未受检异常才会进行回滚，捕获异常要注意事项**

throw new RuntimeException("xxxxxxxxxxxx"); 事物回滚
throw new Exception("xxxxxxxxxxxx"); 事物没有回滚

1).Spring的AOP即声明式事务管理默认是针对unchecked exception回滚。也就是默认对RuntimeException()异常或是其子类进行事务回滚；checked异常,即Exception可try{}捕获的不会回滚，如果使用try-catch捕获抛出的unchecked异常后没有在catch块中采用页面硬编码的方式使用spring api对事务做显式的回滚，则事务不会回滚， “将异常捕获,并且在catch块中不对事务做显式提交=生吞掉异常” ，要想捕获非运行时异常则需要如下配置：

解决办法：
1.在针对事务的类中抛出RuntimeException异常，而不是抛出Exception。
2.在txAdive中增加rollback-for，里面写自己的exception，例如自己写的exception：
<tx:advice id="txAdvice" transaction-manager="transactionManager">
　　<tx:attributes>
　　　　<tx:method name="*" rollback-for="com.cn.untils.exception.XyzException"/>
　　</tx:attributes>
</tx:advice>

或者
定义不会滚的异常
<tx:advice id="txAdvice">
<tx:attributes>
<tx:method name="update*" no-rollback-for="IOException"/>
<tx:method name="*"/>
</tx:attributes>
</tx:advice>

2).spring的事务边界是在调用业务方法之前开始的，业务方法执行完毕之后来执行commit or rollback(Spring默认取决于是否抛出runtime异常).
如果抛出runtime exception 并在你的业务方法中没有catch到的话，事务会回滚。
一般不需要在业务方法中catch异常，如果非要catch，在做完你想做的工作后（比如关闭文件等）一定要抛出runtime exception，否则spring会将你的操作commit,这样就会产生脏数据.所以你的catch代码是画蛇添足。

如：
try {
//bisiness logic code
} catch(Exception e) {
//handle the exception
}

由此可以推知，在spring中如果某个业务方法被一个 整个包裹起来，则这个业务方法也就等于脱离了spring事务的管理，因为没有任何异常会从业务方法中抛出！全被捕获并吞掉，导致spring异常抛出触发事务回滚策略失效。
不过，如果在catch代码块中采用页面硬编码的方式使用spring api对事务做显式的回滚，这样写也未尝不可。

3).基于注解的事物：
Transactional的异常控制，默认是Check Exception 不回滚，unCheck Exception回滚
如果配置了rollbackFor 和 noRollbackFor 且两个都是用同样的异常，那么遇到该异常，还是回滚
rollbackFor 和noRollbackFor 配置也许不会含盖所有异常，对于遗漏的按照Check Exception 不回滚，unCheck Exception回滚

 <a id="btn-readmore" class="btn article-footer-btn" data-track-view="{&quot;mod&quot;:&quot;popu_376&quot;,&quot;con&quot;:&quot;,https://blog.csdn.net/glory1234work2115/article/details/51866644,&quot;}" data-track-click="{&quot;mod&quot;:&quot;popu_376&quot;,&quot;con&quot;:&quot;,https://blog.csdn.net/glory1234work2115/article/details/51866644,&quot;}"></a>阅读更多 <a class="btn article-footer-btn article-footer-bookmark-btn">收藏</a>
 分享

#### 关于Spring 声明式_事务_处理时，throws exception不_回滚_的_问题_

11-28 21

[前一段时间，项目代码评审，发现有TX不使用Spring的事务处理，而直接封装方法，手动进行数据的回滚，得悉原因是：抛出异常以后事务不起作用，没有回滚。这理由顿时让人很无语，不过还算个聪明的TX，知晓另...](https://blog.csdn.net/fade8007/article/details/84912770 "关于Spring 声明式<em>事务</em>处理时，throws exception不<em>回滚</em>的<em>问题</em>") [来自： fade8007](https://blog.csdn.net/fade8007)

