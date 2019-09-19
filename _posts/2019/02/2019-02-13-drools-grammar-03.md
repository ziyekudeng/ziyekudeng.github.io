---
layout: post
title: drools语法介绍(三)
category: drools
tags: [drools]
---


**1．** **概述：**

Drools 3 采用了原生的规则语言，那是一种非 XML 文本格式。在符号方面，这种格式是非常轻量的，并且通过“ expanders ”支持符合你问题域的 Domain Specific Language （ DSL ）。这一章把焦点放在了 Drools 原生的规则格式。如果你想从技术上了解规则语言的机制，可以参考“ drl.g ”源文件，这是用 Antlr3 语法来描述规则语言。如果你使用 Rule Workbench ，内容助手将会为你完成大量的规则结构，例如输入“ ru ”，然后按 ctrl ＋ space ，会为你建立规则结构。

**1.1** **规则文件**

一个规则文件通常是一个以 .drl 扩展名结尾的文件。在一个 drl 文件中，你可以有多条 rules ， functions 等等。尽管如此，你也可以将你的规则分布在多个文件中，这有利于管理大量的规则。一个 DRL 文件是一个简单的文本文件。

**1.2** **规则的结构**

一个规则结构大致如下：

 rule  " name "
    ATTRIBUTES
    when
        LHS
    then
        RHS
end

可以看到，这是非常简单的。通常的标点符号都是不需要的，甚至连“ name ”的双引号都是不需要的。 ATTRIBUTES 是简单的，也是可选的，来提示规则的行为方式。 LHS 是规则的条件部分，需要按照一定的语法来写。 RHS 基本上是一个允许执行 Java 语法的代码的块（以后将会支持 groovy 和 C ＃）。任何在 LHS 中使用的变量都可以在 RHS 中使用。

注意：每行开始的空格是不重要的，除非在 DSL （ Domain Specific Language ）语言中有特别的指明。

**1.3** **Domain Specific Language**

Domain Specific Language 是对原生规则语言的加强。它们使用“ expander ”机制。 Expander 机制是一种可扩展的 API 。你可以使用 .dsl 文件，来提供从域或自然语言到规则语言和你的域对象的映射。你可以将 .dsl 文件看成是对你的域模型的映射。 DSL 提供了更高的规则可读性，你可以选择使用你自己创建的 DSL ，或者是原生的规则语言。

**1.4** **保留字**

在规则语言中存在一些保留字。你应该避免使用这些保留字，来命名规则文本中的域对象，属性，方法，功能。保留字如下： when ， then ， rule ， end ， contains ， matches ， and ， or ， modify ， retract ， assert ， salience ， function ， query ， exists ， eval ， agenda-group ， no-loop ， duration ， -> ， not ， auto-focus 。

**2\.** **注释**

**2.1** **单行注释：**

 ![](https://image.360doc.com/DownloadImg/7147/596021_1.png)

**Figure 2.1. Single line comment**

**2.2 多行注释：** 
 **![](https://image.360doc.com/DownloadImg/7147/596021_2.png)** **Figure 2.2. Multi line comment** 

**3\.** **Package**

一个包是 rule 和其他相关结构，像 import 和 global 的集合。 Package 的成员之间通常都是相关联的。一个 Package 代表了一个命名空间（ namespace ），用来使给定的规则组之间保持唯一性。 Package 的名字本身就是命名空间，并且与文件或文件夹并无关联。

可以将来自不同规则源的规则装配在一起，前提是这些规则必须处在同一个命名空间中。尽管如此，一个通常的结构是将处于同一个命名空间中的所有规则都放在同一个相同的文件中。

下面的 rail-road 图显示了组成一个 Package 的所有组件。注意：一个 package 必须有一个命名空间，并且采用 Java 包名的约定。在一个规则文件中，各组件出现的位置是任意的，除了“ package ”和“ expander ”语句必须出现在任何一个规则之前，放在文件的顶部。在任何情况下，分号都是可选的。

 ![](https://image.360doc.com/DownloadImg/7147/596021_3.png)

**Figure 3.1. package** **3.1 import** 
 **![](https://image.360doc.com/DownloadImg/7147/596021_4.png)
Figure 3.2. import** 

Import 语句的使用很像 Java 中的 import 语句。你需要为你要在规则中使用的对象，指定完整的路径和类名。 Drools 自动从相同命名的 java 包中引入所需的类。

**3.2 expander**

**![](https://image.360doc.com/DownloadImg/7147/596021_5.png)** 

**Figure 3.3. expander**

expander 语句是可选的，是用来指定 Domain Specific Language 的配置（通常是一个 .dsl 文件）。这使得解析器可以理解用你自己的 DSL 语言所写的规则。

**3.3 global** 
 **![](https://image.360doc.com/DownloadImg/7147/596021_6.png)** 

 **Figure 3.4. global**

Global 就是全局变量。如果多个 package 声明了具有相同标识符的 global ，那么它们必需是相同的类型，并且所有的引用都是相同的。它们通常用来返回数据，比如 actions 的日志，或者为 rules 提供所需的数据或服务。 global 并不是通过 assert 动作放入 WorkingMemory 的，所有当 global 发生改变时，引擎将不会知道。所以， global 不能作为约束条件，除非它们的值是 final 的。将 global 错误的使用在约束条件中，会产生令人惊讶的错误结果。

注意： global 只是从你的 application 中传入 WorkingMemory 的对象的命名实例。这意味着你可以传入任何你想要的对象。你可以传入一个 service locator ，或者是一个 service 本身。

下面的例子中，有一个 EmailService 的实例。在你调用规则引擎的代码中，你有一个 EmailService 对象，然后把它放入 WorkingMemory 。在 DRL 文件中，你声明了一个类型为 EmailService 的 global ，然后将它命名为“ email ”，像这样： global EmailService email ；。然后在你的规则的 RHS 中，你可以使用它，像这样： email.sendSMS(number,message) 等等。

**4\. Function**

![](https://image.360doc.com/DownloadImg/7147/596021_7.png)
**Figure 4.1. function**

Function 是将代码放到你的规则源中的一种方法。它们只能做类似 Helper 类做的事（实际上编译器在背后帮你生成了 Helper 类）。在一个 rule 中使用 function 的主要优势是，你可以保持所有的逻辑都在一个地方，并且你可以根据需要来改变 function （这可能是好事也可能是坏事）。 Function 最有用的就是在规则的 RHS 调用 actions ，特别是当那个 action 需要反复调用的时候。

一个典型的 function 声明如下：

 function String calcSomething(String arg) {
return   " hola ! " ;
}

注意：“ function ”关键字的使用，它并不真正是 Java 的一部分。而 function 的参数就像是一个普通的 method （如果不需要参数就不用写）。返回类型也跟普通的 method 一样。在一条规则（在它的 RHS 中，或可能是一个 eval ）中调用 function ，就像调用一个 method 一样，只需要 function 的名字，并传给它参数。

function 的替代品，可以使用一个 Helper 类中的静态方法： Foo.doSomething() ，或者以 global 的方式传入一个 Helper 类或服务的实例： foo.doSomething() （ foo 是一个命名的 global 变量）。


