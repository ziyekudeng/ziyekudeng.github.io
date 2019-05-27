---
layout: post
title: 从 0 开始手写一个 Mybatis 框架
category: arch
tags: [arch]
keywords: 架构
---


 

## 从 0 开始手写一个 Mybatis 框架，三步搞定！

来源：http://t.cn/E99ZCPo

> *   一、Mybatis框架流程简介
>     
>     
> *   二、梳理自己的Mybatis的设计思路
>     
>     
> *   三、实现自己的Mybatis

* * *

> 最近研究了一下Mybatis，给大家磕叨磕叨，MyBatis框架的核心功能其实不难，无非就是动态代理和jdbc的操作，难的是写出来可扩展，高内聚，低耦合的规范的代码。本文完成的Mybatis功能比较简单，代码还有许多需要改进的地方，大家可以结合Mybatis源码去动手完善。

# 一、Mybatis框架流程简介

![](https://ziyekudeng.github.io/assets/images/2019/0527/1.webp)

在手写自己的Mybatis框架之前，我们先来了解一下Mybatis，它的源码中使用了大量的设计模式，阅读源码并观察设计模式在其中的应用，才能够更深入的理解源码。我们对上图进行分析总结：

**mybatis的配置文件有2类**

mybatisconfig.xml，配置文件的名称不是固定的，配置了全局的参数的配置，全局只能有一个配置文件。

Mapper.xml 配置多个statemement，也就是多个sql，整个mybatis框架中可以有多个Mappe.xml配置文件。

通过mybatis配置文件得到SqlSessionFactory

通过SqlSessionFactory得到SqlSession，用SqlSession就可以操作数据了。

SqlSession通过底层的Executor（执行器），执行器有2类实现：

![](https://ziyekudeng.github.io/assets/images/2019/0527/2.webp)

基本实现

带有缓存功能的实现

MappedStatement是通过Mapper.xml中定义statement生成的对象。

参数输入执行并输出结果集，无需手动判断参数类型和参数下标位置，且自动将结果集映射为Java对象

HashMap，KV格式的数据类型

Java的基本数据类型

POJO，java的对象

# 二、梳理自己的Mybatis的设计思路

根据上文Mybatis流程，我简化了下，分为以下步骤：

![](https://ziyekudeng.github.io/assets/images/2019/0527/3.webp)

**1.读取xml文件，建立连接**

从图中可以看出，MyConfiguration负责与人交互。待读取xml后，将属性和连接数据库的操作封装在MyConfiguration对象中供后面的组件调用。本文将使用dom4j来读取xml文件，它具有性能优异和非常方便使用的特点。

**2.创建SqlSession，搭建Configuration和Executor之间的桥梁**

我们经常在使用框架时看到Session，Session到底是什么呢？一个Session仅拥有一个对应的数据库连接。类似于一个前段请求Request，它可以直接调用exec(SQL)来执行SQL语句。从流程图中的箭头可以看出，MySqlSession的成员变量中必须得有MyExecutor和MyConfiguration去集中做调配，箭头就像是一种关联关系。我们自己的MySqlSession将有一个getMapper方法，然后使用动态代理生成对象后，就可以做数据库的操作了。

**3.创建Executor，封装JDBC操作数据库**

Executor是一个执行器，负责SQL语句的生成和查询缓存（缓存还没完成）的维护，也就是jdbc的代码将在这里完成，不过本文只实现了单表，有兴趣的同学可以尝试完成多表。

**4.创建MapperProxy，使用动态代理生成Mapper对象**

我们只是希望对指定的接口生成一个对象，使得执行它的时候能运行一句sql罢了，而接口无法直接调用方法，所以这里使用动态代理生成对象，在执行时还是回到MySqlSession中调用查询，最终由MyExecutor做JDBC查询。这样设计是为了单一职责，可扩展性更强。

# 三、实现自己的Mybatis

工程文件及目录：

![](https://ziyekudeng.github.io/assets/images/2019/0527/4.webp)

首先，新建一个maven项目，在pom.xml中导入以下依赖：

![](https://ziyekudeng.github.io/assets/images/2019/0527/5.webp)

创建我们的数据库xml配置文件：

![](https://ziyekudeng.github.io/assets/images/2019/0527/6.webp)

然后在数据库创建test库，执行如下SQL语句：

![](https://ziyekudeng.github.io/assets/images/2019/0527/7.webp)

创建User实体类，和UserMapper接口和对应的xml文件:

![](https://ziyekudeng.github.io/assets/images/2019/0527/8.webp)

基本操作配置完成，接下来我们开始实现MyConfiguration：

![](https://ziyekudeng.github.io/assets/images/2019/0527/9.webp)

![](https://ziyekudeng.github.io/assets/images/2019/0527/10.webp)

![](https://ziyekudeng.github.io/assets/images/2019/0527/11.webp)

![](https://ziyekudeng.github.io/assets/images/2019/0527/12.webp)

用面向对象的思想设计读取xml配置后：

![](https://ziyekudeng.github.io/assets/images/2019/0527/13.webp)

接下来实现我们的MySqlSession,首先的成员变量里得有Excutor和MyConfiguration，代码的精髓就在getMapper的方法里。

![](https://ziyekudeng.github.io/assets/images/2019/0527/14.webp)

紧接着创建Executor和实现类：

![](https://ziyekudeng.github.io/assets/images/2019/0527/15.webp)

MyExecutor中封装了JDBC的操作：

![](https://ziyekudeng.github.io/assets/images/2019/0527/16.webp)

![](https://ziyekudeng.github.io/assets/images/2019/0527/17.webp)

MyMapperProxy代理类完成xml方法和真实方法对应，执行查询：

![](https://ziyekudeng.github.io/assets/images/2019/0527/18.webp)

到这里，就完成了自己的Mybatis框架，我们测试一下：

![](https://ziyekudeng.github.io/assets/images/2019/0527/19.webp)

**执行结果：**

![](https://ziyekudeng.github.io/assets/images/2019/0527/20.webp)

**查询一个不存在的用户试试：**

![](https://ziyekudeng.github.io/assets/images/2019/0527/21.webp)

到这里我们就大功告成了！

* * *

* * *
