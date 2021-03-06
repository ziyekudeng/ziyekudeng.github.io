---
layout: post
title: 如何设计一个完美的权限管理模块
category: arch
tags: [arch]
keywords: 架构
---

 
 

## 如何设计一个完美的权限管理模块



来 源：cnblogs.com/myindex/p/9116177.html

我们比较常见的就是基于角色的访问控制，用户通过角色与权限进行关联。简单地说，一个用户拥有多个角色，一个角色拥有多个权限。这样，就构造成“**用户-角色-权限**”的授权模型。在这种模型中，用户与角色之间、角色与权限之间，通常都是多对多的关系。

如下图： 

![](https://ziyekudeng.github.io/assets/images/2019/0523/1.webp)

基于这个，得先了解角色到底是什么？我们可以理解它为一定数量的权限的集合，是一个权限的载体。

例如：一个论坛的“管理员”、“版主”，它们都是角色。但是所能做的事情是不完全一样的，版主只能管理版内的贴子，用户等，而这些都是属于权限，如果想要给某个用户授予这些权限，不用直接将权限授予用户，只需将“版主”这个角色赋予该用户即可。

但是通过上面我们也发现问题了，**如果用户的数量非常大的时候，就需要给系统的每一个用户逐一授权**(分配角色)，这是件非常繁琐的事情，这时就可以增加一个用户组，每个用户组内有多个用户，除了给单个用户授权外，还可以给用户组授权，这样一来，通过一次授权，就可以同时给多个用户授予相同的权限，而这时用户的所有权限就是用户个人拥有的权限与该用户所在组所拥有的权限之和。用户组、用户与角色三者的关联关系。

如下图：

![](https://ziyekudeng.github.io/assets/images/2019/0523/2.webp)

通常在应用系统里面的权限我们把它表现为菜单的访问(页面级)、功能模块的操作(功能级)、文件上传的删改，甚至页面上某个按钮、图片是否可见等等都属于权限的范畴。有些权限设计，会把功能操作作为一类，而把文件、菜单、页面元素等作为另一类，这样构成“用户-角色-权限-资源”的授权模型。而**在做数据表建模时，可把功能操作和资源统一管理，也就是都直接与权限表进行关联，这样可能更具便捷性和易扩展性。

**如下图：

![](https://ziyekudeng.github.io/assets/images/2019/0523/3.webp)

> 这里特别需要注意以下权限表中有一列“PowerType(权限类型)”，我们根据它的取值来区分是哪一类权限，可以把它理解为一个枚举，如“MENU”表示菜单的访问权限、“OPERATION”表示功能模块的操作权限、“FILE”表示文件的修改权限、“ELEMENT”表示页面元素的可见性控制等。

**这样设计的好处有两个：**

一、**不需要区分哪些是权限操作，哪些是资源**，（实际上，有时候也不好区分，如菜单，把它理解为资源呢还是功能模块权限呢？）；

二、**方便扩展**，当系统要对新的东西进行权限控制时，我只需要建立一个新的关联表“权限XX关联表”，并确定这类权限的权限类型字符串即可。

需要注意的是，权限表与权限菜单关联表、权限菜单关联表与菜单表都是一对一的关系。（文件、页面权限点、功能操作等同理）。也就是每添加一个菜单，就得同时往这三个表中各插入一条记录。

这样，可以不需要权限菜单关联表，让权限表与菜单表直接关联，此时，须在权限表中新增一列用来保存菜单的ID，权限表通过“权限类型”和这个ID来区分是种类型下的哪条记录。最后扩展出来的模型完整设计。

如下图：


![](https://ziyekudeng.github.io/assets/images/2019/0523/4.webp)

> 注意上面我额外增加了一个操作日志表；

随着系统的日益庞大，为了方便管理，如果有需要可引入角色组对角色进行分类管理，跟用户组不同，角色组不参与授权。

例如：当遇到有多个子公司，每个子公司下有多个部门，这是我们就可以把部门理解为角色，子公司理解为角色组，角色组不参于权限分配。另外，为方便上面各主表自身的管理与查找，可采用树型结构，如菜单树、功能树等，当然这些可不需要参于权限分配。

## 数据字典

**1.用户表：**


![](https://ziyekudeng.github.io/assets/images/2019/0523/5.webp)

**2.角色表：**

![](https://ziyekudeng.github.io/assets/images/2019/0523/6.webp)

**3.用户与角色关联表**

![](https://ziyekudeng.github.io/assets/images/2019/0523/7.webp)

**4.用户组表**

![](https://ziyekudeng.github.io/assets/images/2019/0523/8.webp)

**5.用户组与用户信息关联表**

![](https://ziyekudeng.github.io/assets/images/2019/0523/9.webp)


**6.用户组与角色关联表**

![](https://ziyekudeng.github.io/assets/images/2019/0523/10.webp)


**7．菜单表**

![](https://ziyekudeng.github.io/assets/images/2019/0523/11.webp)


**8.页面元素表**

![](https://ziyekudeng.github.io/assets/images/2019/0523/12.webp)


**9.文件表**

![](https://ziyekudeng.github.io/assets/images/2019/0523/13.webp)


**10.权限表**

![](https://ziyekudeng.github.io/assets/images/2019/0523/14.webp)


**11.权限与菜单关联表**

![](https://ziyekudeng.github.io/assets/images/2019/0523/15.webp)


**12.权限与页面元素关联表**

![](https://ziyekudeng.github.io/assets/images/2019/0523/16.webp)


**13.权限与文件关联表**

![](https://ziyekudeng.github.io/assets/images/2019/0523/17.webp)


**14.功能操作表**

![](https://ziyekudeng.github.io/assets/images/2019/0523/18.webp)


**15.权限与功能操作关联表**

![](https://ziyekudeng.github.io/assets/images/2019/0523/19.webp)


**16.角色与权限关联表**

![](https://ziyekudeng.github.io/assets/images/2019/0523/20.webp)


**17.操作日志表**

![](https://ziyekudeng.github.io/assets/images/2019/0523/21.webp)


（完）
