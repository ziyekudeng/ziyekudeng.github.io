---
layout: post
title: Spring Cloud Alibaba基础教程：使用Nacos实现服务注册与发现
category: springcloud
tags: [springcloud]
---



什么是Nacos
Nacos致力于帮助您发现、配置和管理微服务。Nacos提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。Nacos帮助您更敏捷和容易地构建、交付和管理微服务平台。Nacos是构建以“服务”为中心的现代应用架构 (例如微服务范式、云原生范式) 的服务基础设施。

在接下里的教程中，将使用Nacos作为微服务架构中的注册中心（替代：eureka、consul等传统方案）以及配置中心（spring cloud config）来使用。

安装Nacos
下载地址：https://github.com/alibaba/nacos/releases
本文版本：0.7.0

下载完成之后，解压。根据不同平台，执行不同命令，启动单机版Nacos服务：

Linux/Unix/Mac： sh startup.sh-m standalone

Windows： cmd startup.cmd-m standalone

startup.sh脚本位于Nacos解压后的bin目录下。这里主要介绍Spring Cloud与Nacos的集成使用，对于Nacos的高级配置，后续再补充。所以，持续关注我的博客或者公众号吧！

启动完成之后，访问： http://127.0.0.1:8848/nacos/，可以进入Nacos的服务管理页面，具体如下；

![](https://ziyekudeng.github.io/assets/images/2019/0117/springcloudAlibabaNacos/1.webp.jpg)

构建应用接入Nacos注册中心
在完成了Nacos服务的安装和启动之后，下面我们就可以编写两个应用（服务提供者与服务消费者）来验证服务的注册与发现了。

服务提供者
第一步：创建一个Spring Boot应用，可以命名为： alibaba-nacos-discovery-server。如果您还不会或者不了解Spring Boot应用，建议先学习《Spring Boot基础教程》。

《Spring Boot基础教程》可以在我的博客在线阅读：didispace.com

也可以关注我的公众号，回复：001，获取PDF离线阅读



第二步：编辑 pom.xml，加入必要的依赖配置，比如：

<parent>

    
<groupId>
org.springframework.boot
</groupId>

    
<artifactId>
spring-boot-starter-parent
</artifactId>

    
<version>
2.0.5.RELEASE
</version>

    
<relativePath/>
 
<!-- lookup parent from repository -->

</parent>



<dependencyManagement>

    
<dependencies>

        
<dependency>

            
<groupId>
org.springframework.cloud
</groupId>

            
<artifactId>
spring-cloud-dependencies
</artifactId>

            
<version>
Finchley.SR1
</version>

            
<type>
pom
</type>

            
<scope>
import
</scope>

        
</dependency>

        
<dependency>

            
<groupId>
org.springframework.cloud
</groupId>

            
<artifactId>
spring-cloud-alibaba-dependencies
</artifactId>

            
<version>
0.2.1.RELEASE
</version>

            
<type>
pom
</type>

            
<scope>
import
</scope>

        
</dependency>

    
</dependencies>

</dependencyManagement>



<dependencies>

    
<dependency>

        
<groupId>
org.springframework.boot
</groupId>

        
<artifactId>
spring-boot-starter-web
</artifactId>

    
</dependency>

    
<dependency>

        
<groupId>
org.springframework.cloud
</groupId>

        
<artifactId>
spring-cloud-starter-alibaba-nacos-discovery
</artifactId>

    
</dependency>

    
<dependency>

        
<groupId>
org.projectlombok
</groupId>

        
<artifactId>
lombok
</artifactId>

        
<version>
1.18.2
</version>

        
<optional>
true
</optional>

    
</dependency>

</dependencies>

上述内容主要三部分：

parent：定义spring boot的版本

dependencyManagement：spring cloud的版本以及spring cloud alibaba的版本，由于spring cloud alibaba还未纳入spring cloud的主版本管理中，所以需要自己加入

dependencies：当前应用要使用的依赖内容。这里主要新加入了Nacos的服务注册与发现模块： spring-cloud-starter-alibaba-nacos-discovery。由于在 dependencyManagement中已经引入了版本，所以这里就不用指定具体版本了。

第三步：创建应用主类，并实现一个HTTP接口：

@EnableDiscoveryClient

@SpringBootApplication

public
 
class
 
TestApplication
 {



    
public
 
static
 
void
 main(
String
[] args) {

        
SpringApplication
.run(
TestApplication
.
class
, args);

    }



    
@Slf4j

    
@RestController

    
static
 
class
 
TestController
 {

        
@GetMapping
(
"/hello"
)

        
public
 
String
 hello(
@RequestParam
 
String
 name) {

            log.info(
"invoked name = "
 + name);

            
return
 
"hello "
 + name;

        }

    }

}

内容非常简单， @SpringBootApplication定义是个Spring Boot应用； @EnableDiscoveryClient开启Spring Cloud的服务注册与发现，由于这里引入了 spring-cloud-starter-alibaba-nacos-discovery模块，所以Spring Cloud Common中定义的那些与服务治理相关的接口将使用Nacos的实现。这点不论我们使用Eureka、Consul还是其他Spring Cloud整合的注册中心都一样，这也是Spring Cloud做了封装的好处所在，如果对Eureka、Consul这些注册中心的整合还不熟悉的可以看看以前的这篇：Spring Cloud构建微服务架构：服务注册与发现（Eureka、Consul）。

第四步：配置服务名称和Nacos地址

spring.application.name=alibaba-nacos-discovery-server

server.port=
8001



spring.cloud.nacos.discovery.server-addr=
127.0
.
0.1
:
8848

第五步：启动上面创建的应用。可以在启动时候增加 -Dserver.port=8001的形式在本机的不同端口上启动多个实例。

在应用启动好之后，我们可以在控制台或日志中看到如下内容，代表已经注册成功：

INFO 
10476
 --- [           main] o.s.c.a.n.registry.
NacosServiceRegistry
  : nacos registry, alibaba-nacos-discovery-server 
10.123
.
18.216
:
8001
 register finished

在启动都ok之后，我们可以访问Nacos的管理页面http://127.0.0.1:8848/nacos/来查看服务列表，此时可以看到如下内容：

![](https://ziyekudeng.github.io/assets/images/2019/0117/springcloudAlibabaNacos/2.webp.jpg)

这里会显示当前注册的所有服务，以及每个服务的集群数目、实例数、健康实例数。点击详情，我们还能看到每个服务具体的实例信息，如下图所示：

![](https://ziyekudeng.github.io/assets/images/2019/0117/springcloudAlibabaNacos/3.webp.jpg)

服务消费者
接下来，实现一个应用来消费上面已经注册到Nacos的服务。

第一步：创建一个Spring Boot应用，命名为： alibaba-nacos-discovery-client-common

第二步：编辑pom.xml中的依赖内容，与上面服务提供者的一样即可。

第三步：创建应用主类，并实现一个HTTP接口，在该接口中调用服务提供方的接口。

@EnableDiscoveryClient

@SpringBootApplication

public
 
class
 
TestApplication
 {



    
public
 
static
 
void
 main(
String
[] args) {

        
SpringApplication
.run(
TestApplication
.
class
, args);

    }



    
@Slf4j

    
@RestController

    
static
 
class
 
TestController
 {



        
@Autowired

        
LoadBalancerClient
 loadBalancerClient;



        
@GetMapping
(
"/test"
)

        
public
 
String
 test() {

            
// 通过spring cloud common中的负载均衡接口选取服务提供节点实现接口调用

            
ServiceInstance
 serviceInstance = loadBalancerClient.choose(
"alibaba-nacos-discovery-server"
);

            
String
 url = serviceInstance.getUri() + 
"/hello?name="
 + 
"didi"
;

            
RestTemplate
 restTemplate = 
new
 
RestTemplate
();

            
String
 result = restTemplate.getForObject(url, 
String
.
class
);

            
return
 
"Invoke : "
 + url + 
", return : "
 + result;

        }

    }

}

这里使用了Spring Cloud Common中的 LoadBalancerClient接口来挑选服务实例信息。然后从挑选出的实例信息中获取可访问的URI，拼接上服务提供方的接口规则来发起调用。

第四步：配置服务名称和Nacos地址，让服务消费者可以发现上面已经注册到Nacos的服务。

spring.application.name=alibaba-nacos-discovery-client-common

server.port=
9000



spring.cloud.nacos.discovery.server-addr=
127.0
.
0.1
:
8848

第五步：启动服务消费者，然后通过curl或者postman等工具发起访问，下面以curl为例：

$ curl localhost:
9000
/test

Invoke
 : http:
//10.123.18.216:8001/hello?name=didi, return : hello didi

$ curl localhost:
9000
/test

Invoke
 : http:
//10.123.18.216:8002/hello?name=didi, return : hello didi

可以看到，两次不同请求的时候，真正实际调用的服务提供者实例是不同的，也就是说，通过 LoadBalancerClient接口在获取服务实例的时候，已经实现了对服务提供方实例的负载均衡。但是很明显，这样的实现还是比较繁琐，预告下后面的几篇，关于服务消费的几种不同姿势。

参考资料
Nacos官方文档: https://nacos.io/zh-cn/docs/what-is-nacos.html

Nacos源码分析: http://www.iocoder.cn/Nacos/good-collection/?vip

代码示例
本文示例读者可以通过查看下面仓库的中的 alibaba-nacos-discovery-server和 alibaba-nacos-discovery-client-common项目：

Github：https://github.com/dyc87112/SpringCloud-Learning/tree/master/4-Finchley

Gitee：https://gitee.com/didispace/SpringCloud-Learning/tree/master/4-Finchley

