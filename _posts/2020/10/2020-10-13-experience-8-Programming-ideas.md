---
layout: post
title: 编程是一种思想，而不是敲代码
category: life
tags: [life]
keywords: 一线团队管理经验

---

## 文章来源

  **新亮笔记**  微信号 XinLiangTalk

 

编程是一个先思考再编码的过程，思考是优于编码技能的，在思考过程中我们会考虑代码的可重用性、可靠性、更容易被他人理解，
这时就会使用到设计模式让代码编写工程化，这篇文章整理了设计模式的六大原则。

## 单一职责原则

单一职责原则（Single Responsibility Principle）

    There should never be more than one reason for a class to change.（有且仅有一个原因可以引起类的变更）

不管让我干啥，我都只干一件事，你让我下楼取快递，顺便将垃圾带下去，对不起，我不干，我只取快递。

优点：

*   类的复杂性降低，实现什么职责都有清晰明确的定义；
*   可读性提高，复杂性降低，那当然可读性提高了；
*   可维护性提高，可读性提高，那当然更容易维护了；
*   变更引起的风险降低，变更是必不可少的，如果接口的单一职责做得好，一个接口修改只对相应的实现类有影响，对其他的接口无影响，这对系统的扩展性、维护性都有非常大的帮助。

## 接口隔离原则

接口隔离原则（Interface Segregation Principle）
    
    1、Clients should not be forced to depend upon interfaces that they don't use.（客户端不应该依赖它不需要的接口）
    
    2、The dependency of one class to another one should depend on the smallest possible interface.（类间的依赖关系应该建立在最小的接口上）

举个例子，类A 通过 Interface1 依赖类B，方法1，方法2，方法3；类B 通过 Interface1 依赖D，方法1，方法4，方法5，看下未使用接口隔离原则和使用了接口隔离原则发生了什么变化。

![](https://ziyekudeng.github.io/assets/images/2020/1013/Programming-ideas/1.png)

## 依赖倒置原则

依赖倒置原则（Dependence Inversion Principle）
    
    1、High level modules should not depend upon low level modules.Both should depend upon abstractions. （高层模块不应该依赖低层模块，两者都应该依赖其抽象）
    
    2、Abstractions should not depend upon details. （抽象不应该依赖细节）
    
    3、Details should depend upon abstractions. （细节应该依赖抽象）

举个例子，类A 直接依赖 类B，假如要将 类A 改为依赖 类C，则必须通过修改 类A 的代码来达成。这种场景下，类A一般是高层模块，负责复杂的业务逻辑；类B 和类C 是低层模块，负责基本的原子操作；假如修改类A，会给程序带来不必要的风险。

![](https://ziyekudeng.github.io/assets/images/2020/1013/Programming-ideas/2.png)

解决方案：将 类A 修改为依赖 接口Interface1，类B 和 类C 各自实现 接口Interface1，类A 通过 接口Interface1 间接与 类B 或者 类C 发生联系，则会大大降低修改 类A 的几率。

## 里氏替换原则

里氏替换原则（Liskov Substitution Principle）
    
    1、If for each object o1 of type S there is an object o2 of type T such that for all programs P defined in terms of T,the behavior of P is unchanged when o1 is substituted for o2 then S is a subtype of T.（如果对每一个类型为 S 的对象o1，都有类型为 T 的对象 o2，使得以 T 定义的所有程序 P 在所有的对象 o1 都代换成 o2 时，程序 P 的行为没有发生变化，那么类型 S 是类型 T 的子类型）
    
    2、Functions that use pointers or references to base classes must be able to use objects of derived classes without knowing it.（所有引用基类的地方必须能透明地使用其子类的对象）

举个例子，有一功能 P1，由 类A 完成，现需要将功能 P1 进行扩展，扩展后的功能为 P，其中 P 由原有功能 P1 与新功能 P2 组成，新功能 P 由 类A 的 子类B 来完成，则 子类B 在完成新功能 P2 的同时，有可能会导致原有功能 P1 发生故障。

解决方案：当使用继承时，遵循里氏替换原则。类B 继承 类A 时，除添加新的方法完成新增功能 P2 外，尽量不要重写 父类A 的方法，也尽量不要重载父类A的方法。

继承包含这样一层含义：父类中凡是已经实现好的方法，实际上是在设定一系列的规范和契约，虽然它不强制要求所有的子类必须遵从这些契约，但是如果子类对这些方法任意修改，就会对整个继承体系造成破坏，而里氏替换原则就是表达了这一层含义。

继承作为面向对象三大特性之一，在给程序设计带来巨大便利的同时，也带来了弊端。比如使用继承会给程序带来侵入性，程序的可移植性降低，增加了对象间的耦合性，如果一个类被其他的类所继承，则当这个类需要修改时，必须考虑到所有的子类，并且父类修改后，所有涉及到子类的功能都有可能会产生故障。

优点：

*   代码共享，减少创建类的工作量，每个子类都拥有父类的方法和属性；
*   提高代码的重用性，可扩展性。
*   提高产品或项目的开放性。

## 迪米特法则
    
    迪米特法则（Law of Demeter）
    
    1、Each unit should have only limited knowledge about other units: only units "closely" related to the current unit.（每个单元对于其他的单元只能拥有有限的知识：只是与当前单元紧密联系的单元）
    
    2、Each unit should only talk to its friends; don't talk to strangers.（每个单元只能和它的朋友交谈：不能和陌生单元交谈）
    
    3、Only talk to your immediate friends.（只和自己直接的朋友交谈）

举个例子，我们通过 `手机` 阅读 `微信读书 APP` 内的 `书籍`，如何设计类的编写？

![](https://ziyekudeng.github.io/assets/images/2020/1013/Programming-ideas/3.png)

手机类 和 书籍类，这两个不能直接发生调用关系，需要 手机类 和 微信读书 APP 类先发生调用关系，然后微信读书 APP 类 再和 书籍 类可以发生调用关系，这样才遵循迪米特法则。

## 开闭原则

开闭原则（Open Closed Principle）

    Software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification.（软件中的对象（类，模块，函数等等）应该对于扩展是开放的，但是对于修改是封闭的）

在软件的生命周期内，因为变化、升级和维护等原因需要对软件原有代码进行修改时，可能会给旧代码中引入错误，也可能会使我们不得不对整个功能进行重构，并且需要原有代码经过重新测试。

解决方案：当软件需要变化时，尽量通过扩展软件实体的行为来实现变化，而不是通过修改已有的代码来实现变化。

## 小结

*   单一职责原则告诉我们实现类要职责单一；
*   接口隔离原则告诉我们在设计接口的时候要精简单一；
*   依赖倒置原则告诉我们要面向接口编程；
*   里氏替换原则告诉我们不要破坏继承体系；
*   迪米特法则告诉我们要降低耦合；
*   开闭原则是总纲，告诉我们要对扩展开放，对修改关闭；

 
