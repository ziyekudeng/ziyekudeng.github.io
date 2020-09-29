---
layout: post
title: 一个基于springboot的快速集成多数据源的启动器
category: tools
tags: [tools]
---
 


文章来源于 https://baomidou.gitee.io/

wiki地址：[https://baomidou.gitee.io/dynamic-datasource-doc/guide/#%E7%89%B9%E6%80%A7](https://baomidou.gitee.io/dynamic-datasource-doc/guide/#%E7%89%B9%E6%80%A7)

github地址：[https://github.com/baomidou/dynamic-datasource-spring-boot-starter](https://github.com/baomidou/dynamic-datasource-spring-boot-starter)

# 约定

1.本框架只做 切换数据源 这件核心的事情，并不限制你的具体操作，切换了数据源可以做任何CRUD。

2.配置文件所有以下划线 _ 分割的数据源 首部 即为组的名称，相同组名称的数据源会放在一个组下。

3.切换数据源可以是组名，也可以是具体数据源名称。组名则切换时采用负载均衡算法切换。

4.默认的数据源名称为 master ，你可以通过 spring.datasource.dynamic.primary 修改。

5.方法上的注解优先于类上注解。

6.强烈建议只在service的类和方法上添加注解，不建议在mapper上添加注解。

#使用方法

**END**



