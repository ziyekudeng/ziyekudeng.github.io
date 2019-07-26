---
layout: post
title: 分布式 Session 解决方案
category: arch
tags: [arch]
keywords: 架构
---

 

## 分布式 Session 解决方案

来源：https://www.cnblogs.com/SimpleWu/p/10118674.html

*   分布式Session存在的问题？

*   目前项目中存在的问题

*   如何解决这两个服务之间的共享问题呢？

* * *

**分布式Session一致性？**

说白了就是服务器集群Session共享的问题

**Session的作用？**

Session 是客户端与服务器通讯会话跟踪技术，服务器与客户端保持整个通讯的会话基本信息。

客户端在第一次访问服务端的时候，服务端会响应一个sessionId并且将它存入到本地cookie中，在之后的访问会将cookie中的sessionId放入到请求头中去访问服务器，如果通过这个sessionid没有找到对应的数据那么服务器会创建一个新的sessionid并且响应给客户端。

# 分布式Session存在的问题？

假设第一次访问服务A生成一个sessionid并且存入cookie中，第二次却访问服务B客户端会在cookie中读取sessionid加入到请求头中，如果在服务B通过sessionid没有找到对应的数据那么它创建一个新的并且将sessionid返回给客户端,这样并不能共享我们的Session无法达到我们想要的目的。

**解决方案：**

*   使用cookie来完成（很明显这种不安全的操作并不可靠）

*   使用Nginx中的ip绑定策略，同一个ip只能在指定的同一个机器访问（不支持负载均衡）

*   利用数据库同步session（效率不高）

*   使用tomcat内置的session同步（同步可能会产生延迟）

*   使用token代替session

*   我们使用spring-session以及集成好的解决方案，存放在redis中

# 目前项目中存在的问题

启动两个项目端口号分别为8080,8081。

**依赖：**

    
    <parent
            <groupIdorg.springframework.boot</groupId
            <artifactIdspring-boot-starter-parent</artifactId
            <version2.1.1.RELEASE</version
            <relativePath/ <!-- lookup parent from repository --
    </parent
    <dependencies
            <!--web依赖--
            <dependency
                <groupIdorg.springframework.boot</groupId
                <artifactIdspring-boot-starter-web</artifactId
            </dependency
    </dependencies

**创建测试类：**
    
    /**
     * Author: SimpleWu
     * date: 2018/12/14
     */
    @RestController
    public class TestSessionController {
    
        @Value("${server.port}")
        private Integer projectPort;// 项目端口
    
        @RequestMapping("/createSession")
        public String createSession(HttpSession session, String name) {
            session.setAttribute("name", name);
            return "当前项目端口：" + projectPort + " 当前sessionId :" + session.getId() + "在Session中存入成功！";
        }
    
        @RequestMapping("/getSession")
        public String getSession(HttpSession session) {
            return "当前项目端口：" + projectPort + " 当前sessionId :" + session.getId() + "  获取的姓名:" + session.getAttribute("name");
        }
    
    }

**yml配置：**
    
    server:
     port: 8080

**修改映射文件**

#将本机ip映射到www.hello.com上
    127.0.0.1 www.hello.com

**在这里我们开启nginx集群，修改配置：**
    
    #加入
    #默认使用轮询,
    upstream backserver{
            server 127.0.0.1:8080;
            server 127.0.0.1:8081;
    }
    #修改server中的local
    location / {
                proxy_pass  http://backserver;
                index  index.html index.htm;
            }

我们直接通过轮询机制来访问首先向Session中存入一个姓名，http://www.hello.com/createSession?name=SimpleWu

    当前项目端口：8081 当前sessionId :0F20F73170AE6780B1EC06D9B06210DB在Session中存入成功！

因为我们使用的是默认的轮询机制那么下次肯定访问的是8080端口，我们直接获取以下刚才存入的值http://www.hello.com/getSession

    当前项目端口：8080 当前sessionId :C6663EA93572FB8DAE27736A553EAB89 获取的姓名:null

这个时候发现8080端口中并没有我们存入的值，并且sessionId也是与8081端口中的不同。

别急继续访问，因为轮询机制这个时候我们是8081端口的服务器，那么之前我们是在8081中存入了一个姓名。那么我们现在来访问以下看看是否能够获取到我们存入的姓名：SimpleWu,继续访问：http://www.hello.com/getSession

    当前项目端口：8081 当前sessionId :005EE6198C30D7CD32FBD8B073531347 获取的姓名:null

为什么8080端口我们没有存入连8081端口存入的都没有了呢？

我们仔细观察一下第三次访问8081的端口sessionid都不一样了，是因为我们在第二次去访问的时候访问的是8080端口这个时候客户端在cookie中获取8081的端口去8080服务器上去找，没有找到后重新创建了一个session并且将sessionid响应给客户端，客户端又保持到cookid中替换了之前8081的sessionid，那么第三次访问的时候拿着第二次访问的sessionid去找又找不到然后又创建。一直反复循环。

# 如何解决这两个服务之间的共享问题呢？

spring已经给我们想好了问题并且已经提供出解决方案：spring-session 不了解的可以去百度了解下。

**我们首先打开redis并且在pom.xml中添加依赖：**


            <groupIdcom.alibaba</groupId
            <artifactIdfastjson</artifactId
            <version1.2.47</version
        </dependency
        <dependency
            <groupIdorg.springframework.boot</groupId
            <artifactIdspring-boot-starter-data-redis</artifactId
        </dependency
        <!--spring session 与redis应用基本环境配置,需要开启redis后才可以使用，不然启动Spring boot会报错 --
        <dependency
            <groupIdorg.springframework.session</groupId
            <artifactIdspring-session-data-redis</artifactId
        </dependency
        <dependency
            <groupIdorg.apache.commons</groupId
            <artifactIdcommons-pool2</artifactId
        </dependency
        <dependency
            <groupIdredis.clients</groupId
            <artifactIdjedis</artifactId
        </dependency

**修改yml配置文件:**
    
    server:
     port: 8081
    spring:
     redis:
     database: 0
     host: localhost
     port: 6379
     jedis:
     pool:
     max-active: 8
     max-wait: -1
     max-idle: 8
     min-idle: 0
     timeout: 10000
    redis:
     hostname: localhost
     port: 6379
     #password: 123456

**添加Session配置类**
    
    /**
     * Author: SimpleWu
     * date: 2018/12/14
     */
    //这个类用配置redis服务器的连接
    //maxInactiveIntervalInSeconds为SpringSession的过期时间（单位：秒）
    @EnableRedisHttpSession(maxInactiveIntervalInSeconds = 1800)
    public class SessionConfig {
    
        // 冒号后的值为没有配置文件时，制动装载的默认值
        @Value("${redis.hostname:localhost}")
        private String hostName;
        @Value("${redis.port:6379}")
        private int port;
       // @Value("${redis.password}")
       // private String password;
    
        @Bean
        public JedisConnectionFactory connectionFactory() {
            JedisConnectionFactory connection = new JedisConnectionFactory();
            connection.setPort(port);
            connection.setHostName(hostName);
            //connection.setPassword(password);
            // connection.setDatabase(0);
            return connection;
        }
    }

**初始化Session配置**
    
    /**
     * Author: SimpleWu
     * date: 2018/12/14
     */
    //初始化Session配置
    public class SessionInitializer extends AbstractHttpSessionApplicationInitializer {
        public SessionInitializer() {
            super(SessionConfig.class);
        }
    }

**然后我们继续启动8080,8081来进行测试：**

首先存入一个姓名http://www.hello.com/createSession?name=SimpleWu：

    当前项目端口：8080 当前sessionId :cf5c029a-2f90-4b7e-8345-bf61e0279254在Session中存入成功！

应该轮询机制那么下次一定是8081，竟然已经解决session共享问题了那么肯定能够获取到了，竟然这样那么我们直接来获取下姓名http://www.hello.com/getSession：

    当前项目端口：8081 当前sessionId :cf5c029a-2f90-4b7e-8345-bf61e0279254 获取的姓名:SimpleWu

这个时候我们发现不仅能够获取到值而且连sessionid都一致了。

**实现原理：**

就是当Web服务器接收到http请求后，当请求进入对应的Filter进行过滤，将原本需要由web服务器创建会话的过程转交给Spring-Session进行创建，本来创建的会话保存在Web服务器内存中，通过Spring-Session创建的会话信息可以保存第三方的服务中，如：redis,mysql等。Web服务器之间通过连接第三方服务来共享数据，实现Session共享！

（完） 

