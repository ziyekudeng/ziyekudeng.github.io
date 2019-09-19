---
layout: post
title: Spring Cloud Alibaba与Spring Boot、Spring Cloud之间不得不说的版本关系
category: springcloud
tags: [springcloud]
keywords: 微服务,Spring Cloud,中小企业,架构
---

## 现阶段版本的特殊性

现在的Spring Cloud Alibaba由于没有纳入到Spring Cloud的主版本管理中，所以我们需要自己去引入其版本信息，比如之前教程中的例子：

    1.  `<dependencyManagement>`
    
    2.  `<dependencies>`
    
    3.  `<dependency>`
    
    4.  `<groupId>org.springframework.cloud</groupId>`
    
    5.  `<artifactId>spring-cloud-dependencies</artifactId>`
    
    6.  `<version>Finchley.SR1</version>`
    
    7.  `<type>pom</type>`
    
    8.  `<scope>import</scope>`
    
    9.  `</dependency>`
    
    10.  `<dependency>`
    
    11.  `<groupId>org.springframework.cloud</groupId>`
    
    12.  `<artifactId>spring-cloud-alibaba-dependencies</artifactId>`
    
    13.  `<version>0.2.1.RELEASE</version>`
    
    14.  `<type>pom</type>`
    
    15.  `<scope>import</scope>`
    
    16.  `</dependency>`
    
    17.  `</dependencies>`
    
    18.  `</dependencyManagement>`

而不是像以往使用Spring Cloud的时候，直接引入Spring Cloud的主版本（Dalston、Edgware、Finchley、Greenwich这些）就可以的。我们需要像上面的例子那样，单独的引入 `spring-cloud-alibaba-dependencies`来管理Spring Cloud Alibaba下的组件版本。

由于Spring Cloud基于Spring Boot构建，而Spring Cloud Alibaba又基于Spring Cloud Common的规范实现，所以当我们使用Spring Cloud Alibaba来构建微服务应用的时候，需要知道这三者之间的版本关系。

下表整理了目前Spring Cloud Alibaba的版本与Spring Boot、Spring Cloud版本的兼容关系：

| Spring Boot | Spring Cloud | Spring Cloud Alibaba |
| --- | --- | --- |
| 2.1.x | Greenwich | 0.2.2（还未RELEASE） |
| 2.0.x | Finchley | 0.2.1 |
| 1.5.x | Edgware | 0.1.1 |
| 1.5.x | Dalston | 0.1.1 |

所以，不论您是在读我的《Spring Boot基础教程》、《Spring Cloud基础教程》还是正在连载的《Spring Cloud Alibaba系列教程》。当您照着博子的顺序，一步步做下来，但是没有调试成功的时候，强烈建议检查一下，您使用的版本是否符合上表的关系。

## 推荐：Spring Cloud Alibaba基础教程

*   [《使用Nacos实现服务注册与发现》](https://mp.weixin.qq.com/s?__biz=MzAxODcyNjEzNQ==&mid=2247486665&idx=1&sn=bf733e1b1d1ff0a10edc221537fbebd7&chksm=9bd0a151aca72847b76e78759107c3db5c58efc25f0f8d5a7c82d179b5bab7acd2986e415f45&scene=21#wechat_redirect)

*   [《支持的几种服务消费方式》](https://mp.weixin.qq.com/s?__biz=MzAxODcyNjEzNQ==&mid=2247486746&idx=1&sn=b332f8a2301f78264760fd0ca0a0f474&chksm=9bd0a082aca72994dd5fe17c9e9ec5eda75515ddd9c3f2321f060446634b2b7e57a26e7fe08a&scene=21#wechat_redirect)

*   [《使用Nacos作为配置中心》](https://mp.weixin.qq.com/s?__biz=MzAxODcyNjEzNQ==&mid=2247486766&idx=1&sn=af405af3564e4d72cb45e1adb63034ac&chksm=9bd0a0b6aca729a0488ce0b94e246268397f300bac39f1db3c4a5224216afe96befb7610deb7&scene=21#wechat_redirect)

*   [《Nacos配置的加载规则详解》](https://mp.weixin.qq.com/s?__biz=MzAxODcyNjEzNQ==&mid=2247486791&idx=1&sn=638e5a7875a263832c717f69d6aebfd8&chksm=9bd0a0dfaca729c99724b40c5ae49829cd0b5995e4ca91559264683afdaaaff08210d32c2478&scene=21#wechat_redirect)

*   [《Nacos配置的多环境管理》](https://mp.weixin.qq.com/s?__biz=MzAxODcyNjEzNQ==&mid=2247486796&idx=2&sn=7080accddd960104852460ef0ab65751&chksm=9bd0a0d4aca729c26ce1f222a49d11a094bb7fbdcdf528e609a2b309baf9fef0b4b626315247&scene=21#wechat_redirect)

*   [《Nacos配置的多文件加载与共享配置》](https://mp.weixin.qq.com/s?__biz=MzAxODcyNjEzNQ==&mid=2247486815&idx=1&sn=27361e6362a8e6b7bec65936957c4fb9&chksm=9bd0a0c7aca729d18e185ab3ab8cf2ee73e35786cc9dbd28e53cee3897bf3a5f2066795af0b4&scene=21#wechat_redirect)

*   [《Nacos的数据持久化》](https://mp.weixin.qq.com/s?__biz=MzAxODcyNjEzNQ==&mid=2247486860&idx=1&sn=d2574bd3305c50e3ad1fac1fab92ce74&chksm=9bd0a014aca729020be790f1eaff02eb7948b4f1c9e86b85588f51300ee61fe9f7ee346e5070&scene=21#wechat_redirect)

*   《[Nacos的集群部署](https://mp.weixin.qq.com/s?__biz=MzAxODcyNjEzNQ==&mid=2247486880&idx=1&sn=f67cbcc9e5dab717fd5a43063a79e548&chksm=9bd0a038aca7292e43f133262242b3825d2fcc68d8dbeba3cb79ddf1f547efa815780528bfc0&scene=21#wechat_redirect)》

该系列教程的代码示例：

*   Github：https://github.com/dyc87112/SpringCloud-Learning/

*   Gitee：https://gitee.com/didispace/SpringCloud-Learning/

