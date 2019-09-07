---
layout: post
title: Spring Cloud开发人员如何解决服务冲突和实例乱窜？
category: springcloud
tags: [springcloud]
keywords: 微服务,Spring Cloud,中小企业,架构
---
参考
https://github.com/Nepxion/Discovery

## 一、背景

在我们开发微服务架构系统时，虽然说每个微服务都是孤立的可以单独开发，但实际上并非如此，

要调试和测试你的服务不仅需要您的微服务运行，还需要它的上下文服务、依赖的基础服务等都要运行；

但如果你的系统服务数和依赖比较多呢，那就是一个比较棘手的问题！有没有办法能提高开发效率呢？

![](https://ziyekudeng.github.io/assets/images/2019/0907/spring-cloud-debug/1.webp) 

如上图所示，我们能不能用服务器把所有的服务都部署起来，然后开发只在本地运行自己所负责开发的服务，

因为需要依赖其他服务所以本地启动的服务也需要注册到公共的注册中心里；

> 例子中`业务服务B`有3台实例注册到注册中心里
> 
> 分别是：服务上的、开发A与开发B自己本机启动的

但是这样做又会出现新的问题：服务会冲突乱窜，

意思就是`开发A`在debug自己的`业务服务B`服务的时候可能请求会跳转到其他人的实例上(服务器、开发B)

## 二、解决思路

解决这个服务乱窜问题有一个比较优雅的方式就是`自定义负载均衡规则`，主要实现以下目标：

1.  普通用户访问服务器上的页面时，请求的所有路由只调用`服务器上的实例`

2.  开发A访问时，请求的所有路由优先调用`开发A本机启动的实例`，如果没有则调用`服务器上的实例`

3.  开发B访问时同上，请求的所有路由优先调用`开发B本机启动的实例`，如果没有则调用`服务器上的实例`

## 三、具体实现

要实现上面的目标有两个比较关键的问题需要解决

1.  区分`不同用户的服务实例`

2.  实现`自定义负载均衡规则`

### 3.1\. 区分不同用户的服务实例

直接使用注册中心的元数据(metadata)来区分就可以了

> 主流的注册中心都带有元数据管理

以`Nacos`为例，只需要在配置文件下添加

    spring:
      cloud:
        nacos:
          discovery:
            server-addr: localhost:8848
            metadata:
              version: zlt

> metadata下的`version`就是我添加的元数据`key`为version，`value`为zlt

启动服务后元数据就会注册上去，如下图

![](https://ziyekudeng.github.io/assets/images/2019/0907/spring-cloud-debug/2.webp) 

经过元数据区分后，目前是下面这个情况，服务器的实例`version`为空，而开发人员自己本地启动的实例`version`为唯一标识(自己的名字)

![](https://ziyekudeng.github.io/assets/images/2019/0907/spring-cloud-debug/3.webp) 

### 3.2\. 自定义负载均衡规则

首先在`Spring Cloud`微服务框架里实例的负载均衡是由`Ribbon`负责。
**CustomIsolationRule类**

    public class CustomIsolationRule extends RoundRobinRule {
        /**
         * 优先根据版本号取实例
         */
        @Override
        public Server choose(ILoadBalancer lb, Object key) {
            if (lb == null) {
                return null;
            }
            String version = LbIsolationContextHolder.getVersion();
            List<Server> targetList = null;
            List<Server> upList = lb.getReachableServers();
            if (StrUtil.isNotEmpty(version)) {
                //取指定版本号的实例
                targetList = upList.stream().filter(
                        server -> version.equals(
                                ((NacosServer) server).getMetadata().get(CommonConstant.METADATA_VERSION)
                        )
                ).collect(Collectors.toList());
            }
    
            if (CollUtil.isEmpty(targetList)) {
                //只取无版本号的实例
                targetList = upList.stream().filter(
                        server -> {
                            String metadataVersion = ((NacosServer) server).getMetadata().get(CommonConstant.METADATA_VERSION);
                            return StrUtil.isEmpty(metadataVersion);
                        }
                ).collect(Collectors.toList());
            }
    
            if (CollUtil.isNotEmpty(targetList)) {
                return getServer(targetList);
            }
            return super.choose(lb, key);
        }
    
        /**
         * 随机取一个实例
         */
        private Server getServer(List<Server> upList) {
            int nextInt = RandomUtil.randomInt(upList.size());
            return upList.get(nextInt);
        }
    }

> 集成轮询规则`RoundRobinRule`来实现，主要的逻辑为
> 
> 1.  根据上游输入的版本号`version`，有值的话则取`服务元信息`中`version`值一样的实例
>     
>     
> 2.  上游的版本号`version`没值或者该版本号匹配不到任何服务，则只取`服务元信息`中`version`值为空的实

并通过配置开关控制是否开启自定义负载规则

    @Configuration
    @ConditionalOnProperty(value = "zlt.ribbon.isolation.enabled", havingValue = "true")
    @RibbonClients(defaultConfiguration = {RuleConfigure.class})
    public class LbIsolationConfig {
    
    }

相关源码

https://gitee.com/zlt2000/microservices-platform/blob/master/zlt-commons/zlt-ribbon-spring-boot-starter/src/main/java/com/central/common/ribbon/rule/CustomIsolationRule.java

## 四、总结

上面提到的区分服务实例和自定义负载规则为整个解决思路的核心点，剩下要做的就是上游的`version`怎样指定呢?，下面我提供两个思路

*   开发人员自己启动前端工程，通过配置参数，统一在前端工程传递`version`

*   通过`postman`调用接口的时候在header参数中添加

![](https://ziyekudeng.github.io/assets/images/2019/0907/spring-cloud-debug/4.webp) 



