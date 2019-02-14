---
layout: post
title: Drools规则引擎(九)-KIE之基础API详解
category: drools
tags: [drools]
---


在有些术语使用的时候，我有时候会用KIE项目、KIE引擎或者Drools项目、Drools引擎，大家应该理解KIE是Drools等项目的一个统称，所以在大多数情况下KIE或者特指Drools都是差不多的。

现在我们开始了解KIE的相关API，在这个helloworld例子中，我们接触过如下这些类和接口：

![kie-api-class-1](https://img2.tuicool.com/Eb2eae.png!web)

我们通过KieServices对象得到一个KieContainer，然后KieContainer根据session name来新建一个KieSession，最后通过KieSession来运行规则。

**KieServices:**

该接口提供了很多方法，可以通过这些方法访问KIE关于构建和运行的相关对象，比如说可以获取KieContainer，利用KieContainer来访问KBase和KSession等信息；可以获取KieRepository对象，利用KieRepository来管理KieModule等。

KieServices就是一个中心，通过它来获取的各种对象来完成规则构建、管理和执行等操作。

**KieContainer:**

可以理解KieContainer就是一个KieBase的容器，KieBase是什么呢？

**KieBase：**

KieBase就是一个知识仓库，包含了若干的规则、流程、方法等，在Drools中主要就是规则和方法，KieBase本身并不包含运行时的数据之类的，如果需要执行规则KieBase中的规则的话，就需要根据KieBase创建KieSession。

**KieSession:**

KieSession就是一个跟Drools引擎打交道的会话，其基于KieBase创建，它会包含运行时数据，包含“事实 Fact”，并对运行时数据事实进行规则运算。我们通过KieContainer创建KieSession是一种较为方便的做法，其实他本质上是从KieBase中创建出来。的。

KieSession就是应用程序跟规则引擎进行交互的会话通道。

创建KieBase是一个成本非常高的事情，KieBase会建立知识（规则、流程）仓库，而创建KieSession则是一个成本非常低的事情，所以KieBase会建立缓存，而KieSession则不必。

较为完善的类关系如下：

![kie-api-class-2](https://img0.tuicool.com/Z3eE7n.png!web)

**KieRepository：**

KieRepository是一个单例对象，它是一个存放KieModule的仓库，KieModule由kmodule.xml文件定义（当然不仅仅只是用它来定义）。

**KieProject：**

KieContainer通过KieProject来初始化、构造KieModule，并将KieModule存放到KieRepository中，然后KieContainer可以通过KieProject来查找KieModule定义的信息，并根据这些信息构造KieBase和KieSession。

**ClasspathKieProject:**

ClasspathKieProject实现了KieProject接口，它提供了根据类路径中的META-INF/kmodule.xml文件构造KieModule的能力，也就是我们能够基于Maven构造Drools组件的基本保障之一。

意味着只要我们按照前面提到过的Maven工程结构组织我们的规则文件或流程文件，我们就能够只用很少的代码完成模型的加载和构建。

现在我们知道了可以通过ClasspathKieProject来解析kmodule.xml文件来构建KieModule，那么整个过程是如何进行的呢？kmodule.xml里面的kbase、ksession和KieBase和KieSession又是什么关系呢？下一章节我们继续。

 
