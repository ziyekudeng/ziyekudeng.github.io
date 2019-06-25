---
layout: post
title: Spring Boot下的Tomcat，你真的会用吗？
category: springboot
tags: [springboot]
---

 



来源：锅外的大佬

* * *


## 1.概述

Spring Boot Web应用程序默认包含预配置的嵌入式Web服务器。但在某些情况下，我们要**修改默认配置**以满足自定义要求。

在本教程中，我们将介绍通过application.properties文件配置Tomcat嵌入式服务器的一些常见用例。

## 2.常见的嵌入式Tomcat配置

### 2.1\. 服务器地址和端口

**我们希望更改的最常见配置是端口号**：

    server.port = 80

如果我们不提供server.port 参数，则默认设置为8080。

在某些情况下，我们可能希望设置服务器应绑定的网络地址。换句话说，我们定义一个**服务器将监听的IP地址**：

    server.address = my_custom_ip

默认情况下，该值设置为0.0.0.0，允许通过所有IPv4地址进行连接。设置另一个值，例如localhost - 127.0.0.1 - 将使服务器更具选择性。

### 2.2\. 错误处理

**默认情况下，Spring Boot提供标准错误网页**。此页面称为Whitelabel。它默认启用，但如果我们不想显示任何错误信息，我们可以禁用它：

    server.error.whitelabel.enabled = false

*   Whitelabel的默认路径是/error。可以通过设置server.error.path参数来自定义它：


    server.error.path = /user-error

还可以设置属性，以确定显示有关错误的信息。例如，我们可以包含错误消息和堆栈跟踪：

    server.error.include-exception= true

    server.error.include-stacktrace= always

我们的教程Exception Message Handling for REST和Customize Whitelabel Error Page详细解释有关Spring Boot中处理错误的更多信息。

### 2.3\. 服务器连接

当在低资源容器上运行时，我们可能希望**减少CPU和内存负载**。一种方法是限制应用程序可以同时处理的请求数量。相反，我们可以增加此值以使用更多可用资源来获得更好的性能。

在 SpringBoot中，我们可以定义 Tomcat工作线程的最大数量：

    server.tomcat.max-threads= 200

配置Web服务器时，**设置服务器连接超时**也可能很有用。这表示服务器在连接关闭之前等待客户端发出请求的最长时间：

    server.connection-timeout= 5s

我们还可以定义请求头的最大大小：

    server.max-http-header-size= 8KB

请求正文的最大大小：

    server.tomcat.max-swallow-size= 2MB

或者整个POST请求的最大大小：

    server.tomcat.max-http-post-size= 2MB

### 2.4\. SSL

**要在我们的Spring Boot应用程序中启用SSL支持**，我们需要将server.ssl.enabled属性设置为true，并定义SSL协议：

    server.ssl.enabled = true

    server.ssl.protocol = TLS

我们要配置保存证书密钥库的密码，类型和路径：

    server.ssl.key-store-password=my_password

    server.ssl.key-store-type=keystore_type

    server.ssl.key-store=keystore-path

我们还必须定义标识密钥库中密钥的别名：

    server.ssl.key-alias=tomcat

有关SSL配置的更多信息，请访问:HTTPS using self-signed certificate in Spring Boot。

### 2.5\. Tomcat服务器访问日志

在尝试统计页面命中数，用户会话活动等时，Tomcat访问日志非常有用。

**要启用访问日志**，只需设置：

server.tomcat.accesslog.enabled = true

我们还应该配置其他参数，例如附加到日志文件的目录名，前缀，后缀和日期格式：

server.tomcat.accesslog.directory=logs

    server.tomcat.accesslog.file-date-format=yyyy-MM-dd

    server.tomcat.accesslog.prefix=access_log

    server.tomcat.accesslog.suffix=.log

## 3\. 结论

在本教程中，我们学习了一些常见的Tomcat嵌入式服务器配置。要查看更多可能的配置，请访问官方页面: Spring Boot application properties docs。

与往常一样，这些示例的源代码可以在GitHub上找到。



**作者：程序猿DD**  
**版权所有，欢迎保留原文链接进行转载**
