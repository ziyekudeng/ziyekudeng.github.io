---
layout: post
title: Drools6新特性
category: drools
tags: [drools]
---



其实Drools6有挺多优秀的入门学习资料了，按理说没必要在重复别人的内容了。不过由于项目的需要，需要对Drools有个透彻的了解甚至说现有的中文资料都没法支持我把需求做出来，所以还是把基础打扎实把。
所以什么是规则引擎，什么是Drools，就都先参考下以下资料吧。

Drools5官方中文文档（供参考）：[http://pan.baidu.com/s/1sj6uoQp](http://pan.baidu.com/s/1sj6uoQp)
Drools6的入门资料：[http://blog.xiongzhijun.com/?cat=48](http://blog.xiongzhijun.com/?cat=48)
Drools-6.1.0.Final 入门文档：[http://pan.baidu.com/s/1bnuB4fl](http://pan.baidu.com/s/1bnuB4fl)
Drools6官方文档：[http://docs.jboss.org/drools/release/6.1.0.Final/drools-docs/html_single/index.html](http://docs.jboss.org/drools/release/6.1.0.Final/drools-docs/html_single/index.html)

个人认为Drools6与Drools5使用上的差距还是很大的，很多用法都已经被统一与简化了，所以Drools5的资料可以作为参考但是相应的例子估计都跑不起来了。
Drools6的官方文档很强大，不过个人认为作为进阶学习的资料不错。

* * *

## Drools6 新特性

关于为什么要介绍Drools6 新特性，是因为在我刚开始接触Drools的时候，在网上的中文资料不多，所以看到有介绍它的文章就会点进去看。很多文章往往就是这么个标题

> 规则引擎 Drools 使用解析

然后靠谱的文章会告诉你下当前使用Drools环境是Drools6还是其他，但是大部分人基本没有写，所以当你照着文章敲例子的时候就发现什么KnowledgeBase 、KnowledgeBuilder 为什么没有，然后接下来就是纠结到底是这文章问题还是包没有引进来！所以有必要介绍Drools5之前的版本与Drools6的用法区别！

废话不说了，接下来介绍Drools6新特性！
官方文档中有专门的第二章节来介绍 **Chapter 2\. Release Notes**。

#### **Kie API**

新版本与之前版本最大的区别就在于推出了一套新的基于KIE概念的API。
通过 Kie 的 API 统一了旗下的OptaPlanner、Drools、JBPM、UBerFire多个工程的使用。

##### **Configuration and convention based projects**

kmodule.xml

    <code class=" hljs xml"><kmodule xmlns="http://jboss.org/kie/6.0.0/kmodule">
      <kbase name="kbase1" packages="org.mypackages>
        <ksession name="ksession1"/>
      </kbase>
    </kmodule></code>

Kmodule.xml是Kie API中比较重要的一部分，之后会介绍相应的用法，在这里只是告诉你这时新特性，别傻傻的在旧版本里面找它了。

##### **KieModules, KieContainer and KIE-CI**

    <code class=" hljs avrasm">KieServices ks = KieServices.Factory.get();
    KieContainer kContainer = ks.newKieContainer(
            ks.newReleaseId("org.mygroup", "myartefact", "1.0") );
    KieSession kSession = kContainer.newKieSession("ksession1");
    kSession.insert(new Message("Dave", "Hello, HAL. Do you read me, HAL?"));
    kSession.fireAllRules();</code>

这里引用Drools-6.1.0.Final 入门文档 中的介绍来介绍这个特性：

> Drools推出了一套新的基于KIE概念的API，其目的是将之前版本中对规则引擎繁琐的调用和加载过程加以简化。
> Drools6给我的最大不同就是把rules打包成jar，使用端通过kie-ci来动态从maven repo中获取指定rules jar版本，虽然和maven有紧耦合，简化以及清晰了rules的使用和动态升级：系统建立2个项目：一个Drools项目来实现规则，验收规则，生成jar包，另外一个就是真正要用规则的项目，直接通过引入不同版本的jar包实习规则动态升级。

#### **What is New and Noteworthy in Drools 6.0.0**

1.  引入新的算法PHREAK，官方文档的意思是能让Drools处理大量的规则以及事实。我没有找到太多关于PHREAK算法的资料，有空再去研究下吧。
2.  Automatically firing timed rule in passive mode.这什么意思，大致指的是Drools默认情况下是惰性的执行规则，即除非你调用了fireAllRules（），不然他不会执行规则的。而现在有一个Drools加了个新特性，能让你改变默认的行为，他会自动执行规则。

    <code class=" hljs avrasm">KieSessionConfiguration ksconf = KieServices.Factory.get().newKieSessionConfiguration();
    ksconf.setOption( TimedRuleExectionOption.YES );
    KSession ksession = kbase.newKieSession(ksconf, null);</code>

这个新特性什么时候用，我还真没数。
3\. Expression Timers.

    <code class=" hljs ruby">declare Bean
        delay   : String = "30s"
        period  : long = 60000
    end
    rule "Expression timer"
        timer( expr: $d, $p )
    when
        Bean( $d : delay, $p : period )
    then
    end</code><code class=" hljs vbscript">timer (int: 30s 10s; start=3-JAN-2010, end=5-JAN-2010)</code>

感觉就像个定时器，在从2010年1月3日开始，延迟30秒时间。之后每隔10秒发生一次，直到1月5日。

#### **New and Noteworthy in Integration 6.0.0**

关于整合的新特性，我觉得不错的估计就是Spring以及CDI，之前版本就能与Spring整合了，而现在是可以用Spring的配置文件来代替kmodule.xml。在我看源码的时候发现，Drools在运行的时候会到META-INF下面查找名为kmodule-spring.xml的配置文件。

##### **CDI**

    <code class=" hljs java">@Inject
    @KSession("ksession1") 
    @KReleaseId( groupId = "jar1", rtifactId = "art1", version = "1.0")
    private KieSession ksessionv10;
    
    @Inject
    @KSession("ksession1") 
    @KReleaseId( groupId = "jar1", rtifactId = "art1", version = "1.1")
    private KieSession ksessionv11;</code>

注入相应版本的KieBase和KieSession。


