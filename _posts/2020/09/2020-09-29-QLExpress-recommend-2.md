---
layout: post
title: java脚本引擎的设计原理浅析
category: QLExpress
tags: [QLExpress]
---


文章来源:阿里云开发者社区

作者：闪电侠天樵：[https://developer.aliyun.com/article/621200](https://developer.aliyun.com/article/621200)



# java脚本引擎的设计原理浅析


 **简介：**本人在阿里巴巴长期担任和负责规则引擎、流程引擎相关的技术开发，
 另外还负责开发和维护开源项目：https://github.com/alibaba/QLExpress QLExpress是一个脚本引擎工具，类似Groovy，JRuby等，是为了解决当时电商规则动态编译、表达式高精度计算、复杂布尔运算、自定义函数和操作符号、语法树生成等需求而设计的。

本人在阿里巴巴长期担任和负责规则引擎、流程引擎相关的技术开发，另外还负责开发和维护开源项目：
[https://github.com/alibaba/QLExpress](https://github.com/alibaba/QLExpress)

QLExpress是一个脚本引擎工具，类似Groovy，JRuby等，是为了解决当时电商规则动态编译、表达式高精度计算、复杂布尔运算、自定义函数和操作符号、语法树生成等需求而设计的。QLExpress项目开源自2012年，截至目前已经迭代了60多个版本，（在阿里的专有开源社区 index - Taocode ）后在社区和集团内部却意外获得非常好的反响。

然而，我要很不客气的讲，在众多优秀的脚本引擎工具list中，QLExpress还很不完美。本篇结合自己的工作和技术研究，列举其中一些比较有意思的。

## 一、分类标准

### 1、编译型 vs 解析性

如果能够产生一个独立的class文件则属于前者，例如：fel，simpleEl，groovy
否则通过编译成自定义的内存指令就属于后者，例如：QLExpress，aviator，JEXL

### 2、java语法 vs 表达式语言(EL expression language) vs 脚本(script)

如果语法和java保持一致，不做任何扩展，就是属于第一种：
如果语法大量简化（比如去掉显示类、方法、变量声明，异常处理，逻辑跳转循环等等），只支持简单的数学公式、对象方法成员变量调用， 就属于第二种：fel，simpleEl，aviator
介于两者之间，即提供很好的语法糖，又支持大部分java语法：for循环，if判断，函数定义，就属于第三种：groovy，QLExpress

### 3、静态类型 vs 动态类型 vs 强类型 vs 弱类型

区别看这里： 弱类型、强类型、动态类型、静态类型语言的区别是什么？
假设两个都很成熟的脚本引擎，强类型可以比弱类型提高一个数量级的运算效率，但是带来业务的灵活性也就差很多。比如fel、simpleEl比Groovy、QLExpress效率肯定要高一个量级，因此，Groovy也提供了静态类型的模式牺牲部分灵活性来提高性能。

## 二、设计思路

思路一：通过文本替换成java源码文件。

编译时，使用 jdk工具（ javax.tools.JavaCompiler ）或者 eclipse工具 动态编译成class文件
运行时，取出class文件放入jvm直接运行

思路二：通过 antlr 等 语法解析器 把文本流解析成抽象语法树（AST）。

编译时，把AST的节点重新编排成class文件或者指令集。
运行时，把其中的操作符映射成自定义的执行器。

## 三、看一些案例

### 1、simpleEL

作者：温绍锦(高铁@alibaba) [https://github.com/alibaba/simpleel](https://github.com/alibaba/simpleel)
demo代码：

    DefaultExpressEvalService service = new DefaultExpressEvalService();
    service.regsiterVariant(Integer.class,”a”,"b");//注册数据类型
    service.setAllowMultiStatement(true); //设置支持多行语句
    Map<String, Object> ctx = new HashMap();
    ctx.put("a", 2);
    ctx.put("b", 1);
    String dsl = "if (@a > @b) { return @a - @b; } else {return @b - @a; }";
    Object result = service.eval(ctx, dsl);
    //return 1

内部原理剖析：

    if (@a > @b) 
    {
      return @a - @b; 
    } else 
    {
      return @b - @a;
    }

通过@标识变量，直接转化为一下的java源码，然后再转化为class文件
生成的class文件反编译后的源码：
![](https://ziyekudeng.github.io/assets/images/2020/09/0929/QLExpress2/1.png)

再比如：

    DefaultExpressEvalService service = new DefaultExpressEvalService();
    service.regsiterVariant(MyBean.class,"bean");
    Map<String, Object> ctx = new HashMap();
    ctx.put("bean", new com.taobao.el.MyBean());
    String dsl = "@bean.getId() + ':' + @bean.getName()"
    Object result = service.eval(ctx, dsl);

生成的class文件反编译后的源码：
![](https://ziyekudeng.github.io/assets/images/2020/09/0929/QLExpress2/2.png)

特点:
简单如其名，最最simple的EL，源码实现也极其简单。
依赖变量标识和强类型注册，不借助任何语法解析工具，就实现了非常高效简单的EL计算。
因为太简单，所以后续功能开发停滞了，2014-9-17后停止更新。

### 2、fel

作者：未知 [https://code.google.com/archive/p/fast-el/](https://code.google.com/archive/p/fast-el/)
demo代码

    FelEngine fel = new FelEngineImpl();
    FelContext ctx = fel.getContext();
    ctx.set("首重价格", 10.5); ctx.set("续重价格", 2.5);
    ctx.set(“首重",5); ctx.set("总重量", 15);
    String dsl = "首重价格*首重+续重价格*(总重量-首重)";
    Expression exp = fel.compile(dsl, ctx);//必须变量全部注入后，才能编译通过
    Object result =  exp.eval(ctx);
    System.out.println(result);

内部原理剖析：

    首重价格*数量+续费价格*(首重-数量)

使用antlr 对文本流进行语法分析，转化为AST之后生成简单的java源码

生成的class文件反编译后的源码：
![](https://ziyekudeng.github.io/assets/images/2020/09/0929/QLExpress2/3.png)

特点：
和simpleEL相比，这是一个非常追求极致效率的作品，它能识别重复变量，开提供function注册、SourceBuilder等编译后期优化，基本可以保持到和java源码静态编译后只低一个数量级的效率。
静态类型，不支持多行表达式，new对象等，2013-3-26后停止更新

### 3、Groovy

大名鼎鼎的开源项目： The Apache Groovy programming language
demo:

    GroovyClassLoader groovyLoader = new GroovyClassLoader(); 
    Class<Script> groovyClass = (Class<Script>) groovyLoader.parseClass("a+b*c");
     Script script = groovyClass.newInstance(); 
    Binding bind = new Binding(); 
    bind.setVariable("a", 1);
    bind.setVariable("b", 2);
    bind.setVariable("c", 3); 
    script.setBinding(bind); 
    Object result =  script.run();
    //return 7

特点：
由于大规模的使用和发展历史悠久，groovy功能强大到无与伦比，本身和java源码文件无缝兼容，又提供一大堆的语法糖和功能库。

4、QLExpress
作者:强辉、包行杰 alibaba/QLExpress

    ExpressRunner runner = new ExpressRunner(); 
    DefaultContext<String, Object> context = new DefaultContext<String, Object>(); 
    context.put("a",1);
    context.put("b",2);
    context.put("c",3); 
    Object result =  runner.execute("a+b*c",context,null,true,false);

特点：
从阿里巴巴的电商业务土壤中开始发展，融合了众多奇葩却又非常普遍的业务特性。
从2012年2.0版本开始开源，到现在一直在不断的迭代功能和新特性。
弱类型动态类型，高精度数值计算，复杂逻辑运算定制，函数、操作符的重载、覆盖，对象方法的拦截重定义，语法树、变量、函数定义输出等等。
脚本本身具备线程安全，高性能高并发的特性。
和同类型的groovy相比：

    1、groovy比qlExpress更兼容java语法

相对功能复杂处理过程的大段java脚本，groovy可以直接拷贝过来运行，
而qlexpress的语法很轻量，对原始的java代码有一定的兼容性问题，一般需要把数据的类型声明全部去掉，同时不支持异常处理、for循环的集合写法 等

    2、qlExpress比groovy更强调功能扩展

因为qlExpress就是诞生于阿里的电商系统，定制了很多特别的常用功能需求（宏定义，语法解析，公式计算，布尔逻辑处理，操作符函数的内置替换），可读性和功能更贴合业务需要，详细看qlExpress的扩展能力部分

    3、qlExpress比groovy实现更简洁，操作系统以及应用领域更广

目前我们的qlExpress因为所有的语法解析、运算执行都自己编码实现，依赖实现极其简单，在阿里的新零售领域，在大量的低版本android系统的pos机上都有大规模使用，稳定性非常好。

     4、qlExpress和groovy性能相当

qlExpress和groovy同属弱类型语法，比如a+b，可以在运行时支持字符串，数字等多种计算模式，相比fel，simpleExpress 等强类型语言性能会差一个数量级。
他们都支持编译期做了缓存功能，qlExpress转化为InstructSet，groovy转化为一个特殊的groovyclass子类

## 文章总结

限于篇幅，本文没有再枚举更多的脚本引擎，对groovy和QLExpress也没有充分展开原理介绍，最后只表述本人的对这个行业的一些看法。

### 1、如何自己造轮子：基于jvm的java语言的脚本引擎本身门槛并不高。

我见过好几个case：开发能力比较强的团队在了解 jdk字节码生成工具，antlr语法生成工具，深入阅读一些编译原理文章之后，都可以在1-2月内开发出一个脚本引擎，满足业务述求。
但是，往往会因为与业务系统耦合、功能太少、通用性太差、不具备开放性等原因逐渐消亡，无法大范围推广。

### 2、如何选轮子：性能、稳定性、语法灵活性综合考虑

某种角度来说，脚本引擎因为可以动态解析执行，往往成为业务配置化的一部分。
业务控制逻辑越复杂，越需要流程引擎，规则引擎，脚本引擎一类的东西来编排业务。
作为业务架构师，底层一定要选择一个靠谱好用的脚本引擎工具。
性能上:

     大众化工具>小众化工具(内部工具实现上有很多可以优化的点)
      强类型 > 弱类型（在两者都很成熟稳定的情况下）

稳定性上:

     简单的设计原理>强大的开源社区>小众化工具引擎

语法的灵活性:

     代码越多、越复杂的实现方式功能会涵盖越多，语法糖、容错性、兼容性



**END**



