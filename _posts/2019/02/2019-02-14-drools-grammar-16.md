---
layout: post
title: Jboss rules规则引擎 Drools 6.4.0 Final 教程(5)
category: drools
tags: [drools]
---


# 6、Drools6.4版本关于session的不同方法


KIE = Knowledge is Everything

在 JBoss 定义的 knowledge 除了规则引擎 Drools 之外，还有工作流引擎 jBPM

## KieServices

KieServices 是一个线程安全的单例：

<code style="font-family: Inconsolata, monospace, sans-serif;">KieServices kieServices = KieServices.Factory.get();</code> 

KieServices 是访问规则引擎其它服务的中心。

以创建 KieContainer 实例为例：

<code class="language-html">KieContainer kieContainer = kieServices.newKieClasspathContainer();</code> 
## KieModule.xml

KieModule 是一个标准的 Java Maven 工程，包含了 pom.xml、kmodule.xml 和规则等必要资源。

KieModule 可以包含子 KieModule。

kmodule.xml内容分析

![](https://img-blog.csdn.net/20160723122857000?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center) 

kmodule.xml 位于 src/main/resources/META-INF 目录下，定义了规则引擎如何加载工程中定义的规则。

一个空的 kmodule.xml 文件如下所示：

<code class="language-html"><?xml version="1.0" encoding="UTF-8"?>  
<kmodule xmlns="http://www.drools.org/xsd/kmodule"/></code> 
## 定义 kbase

*   name KIEBase 名称
*   includes 子 KIEBase 名称，多个使用逗号分隔
*   packages 规则文件所在包路径，多个使用逗号分隔

示例：

<code class="language-html"><?xml version="1.0" encoding="UTF-8"?>  
<kmodule xmlns="http://jboss.org/kie/6.0.0/kmodule">  
    <kbase name="rules" packages="rules.xxx">
    </kbase>
</kmodule></code> 
## 定义 ksession

*   name KIESession 名称
*   default 是否为默认 session，可选值：true false，默认值：false
*   type 会话类型，可选值：stateful 有状态会话 stateless 无状态会话，默认值：stateful 有状态会话

示例：

<code class="language-html"><?xml version="1.0" encoding="UTF-8"?>  
<kmodule xmlns="http://jboss.org/kie/6.0.0/kmodule">  
    <kbase name="rules" packages="rules.xxx">
        <ksession default="true"/>
    </kbase>
</kmodule></code> 
## KieBase

知识仓库。

## KieContainer

KieModule 及子 KieModule 所有 KieBase 的容器。

获取 KieContainer 的两种方式：

*   通过 classpath 加载规则
*   通过 Maven GAV 加载规则（使用 KIE CI）

## KieSession

用于与规则引擎进行交互的会话。

会话分为两类：

*   有状态的 KieSession
*   无状态的 StatelessKieSession

 KIESession

 KIESession 会在多次与规则引擎进行交互中，维护会话的状态。

 定义 KieSession，在 kmodule.xml 文件中定义 type 为 stateful 的 session：

 <code style="font-family: Inconsolata, monospace, sans-serif;"><ksession name="stateful_session" type="stateful"></ksession></code> 

Tip：stateful 是 type 属性的默认值。

获取 KIESession 实例：

<code style="font-family: Inconsolata, monospace, sans-serif;">KieSession statefulSession = kieContainer.newKieSession("stateful_session");</code> 

接下来，可以在 KIESession 执行一些操作。

最后，如果需要清理 KIESession 维护的状态，调用 dispose() 方法。

## StatelessKIESession

与 KIESession 相反，StatelessKIESession 隔离了每次与规则引擎的交互，不会维护会话的状态。

如果将 session 比作编程语言中的函数，StatelessKIESession 就是无副作用的函数。

StatelessKIESession 适用场景：

*   数据校验
*   运算
*   数据过滤
*   消息路由
*   任何能被描述成函数或公式的规则

定义 StatelessKIESession，在 kmodule.xml 文件中定义 type 为 stateless 的 session：

<code style="font-family: Inconsolata, monospace, sans-serif;"><ksession name="stateless_session" type="stateless"></ksession></code> 

获取 StatelessKIESession 实例：

<code style="font-family: Inconsolata, monospace, sans-serif;">StatelessKieSession statelessKieSession = kieContainer.newStatelessKieSession("stateless_session");</code> 

通过 KieServices 获取 command 工厂类 KieCommands：

<code style="font-family: Inconsolata, monospace, sans-serif;">KieCommands commandFactory = kieServices.getCommands();</code> 

可以使用工程类 KieCommands 调用 newXXXCommand 开头的方法创建 command 实例。

会话执行 command：

<code style="font-family: Inconsolata, monospace, sans-serif;">statelessKieSession.execute(command);</code>
