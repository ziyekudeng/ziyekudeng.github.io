---
layout: post
title: Apache Dubbo实际应用总结
category: dubbo
tags: [dubbo]
---

## Dubbo实际应用总结

一方面是SpringCoud微服务框架如火如荼的发展，另一方面随着Dubbo的重启，接着又捐献给Apache社区，Dubbo在国内技术市场上又重新攻城略地，随着孵化即将毕业，以后正式称为Apache Dubbo，相应会应用的更加广泛。

下面罗列几点Dubbo在项目的应用，没有特别复杂的场景，简单做个总结，Dubbo上手容易，但用的好还是有难度的。

### 1.超时

Timeout设置依据就近原则，服务提供者的优先级低于调用者，类级别的优先级低于方法级别的

### 2.异步调用

使用async='true'，即可完成异步调用。初级应用时未能仔细查看API，异步实现时，自己实现多线程来进行，这完全是忽略了dubbo本来的特性。

### 3.spring-boot-starter

随着SpringBoot的普及，Dubbo 的starter一键引入版也是呼欲出。一个是alibaba/dubbo-spring-boot-starter，1.4k+Star，Dubbo在移交后Apache孵化期间，该项目已经停止维护，进行封存阶段。另一个是apache/incubator-dubbo-spring-boot-project，目前2.6k+Star，应该说未来的官方版，活跃度很高。

### 4.缓存目录

dubbo应用期间，默认存储在/root/.dubbo目录，会发现有两类文件，一个application_name-ip-port.cacahe和lock文件，cache文件中保存着该应用所有注册的服务。当有些时候发布的时候无法调用到时，可以清理此文件，重新注册服务。

### 5.同接口不同实现，分组

经常会遇到相同接口不同实现的情况，可结合Spring配置@Qualifier注解，再给接口定义时增加group参数即可。

### 6.非幂等接口设置不重试

请求超时时，默认值是2。如果是非幂等性接口，一定要此参数设置为0。dubbo-spring-boot-starter的0.2.0版本中必须设置为-1才能不重试。

### 7.独立开发不走dubbo

新产品独立开发时（如果有外部依赖可采用mock方式），可完全采用jar内部依赖的方式进行，到正式测试时，再分解后不同的服务启动，可以提高开发阶段的效率。

### 8.增加版本

尽量给每个接口加个版本号，便于后期接口变更时进行兼容性升级。

### 9.Mock处理

在接口不完善的情况下，可直接通过Mock形式为接口调用方返回结果，保证接口可用，不影响调用测试。采用mock配置即可。

### 10.HTTP支持。

Dubbo在大家的印象中，只做内部服务调用，在Dubbo重启维护后的2.6.0版本中，将Dubbox的分支合并到主干，以此可以对外提供语言无关的HTTP接口服务。Apache&nbsp;Dubbo已不再局限于Java语言

### 11.异常自定义处理

使用Dubbo后，发现抛出来的异常都是RuntimeException，不能很友好提示给用户，这时需要自定义异常。扩展Dubbo的一个Filter，将自定义的异常写进去。同时定义好com.alibaba.dubbo.rpc.Filter文件，以便能正确寻址到自定义的异常类。

### 12.链路追踪

虽然Dubbo内置了brave实现，但使用起来不是很方便，可以采用官方推荐的两种方式：Pinpoint和Skywalking，有更好的UI界面方便跟踪查询，再结合日志查询系统，如ELK Stack进行异常定位。

### 13.监控平台

原来的dubbo-admin子项目，迁移到dubbo-ops项目中来独立发展。在dubbo-admin中可进行接口的负载、容错、权重的配置等等。

### 14.分布式事务

近基于开源的FESCAR，对Dubbo进行了完美的支持，分布式事务开源解决方案——FESCAR。





