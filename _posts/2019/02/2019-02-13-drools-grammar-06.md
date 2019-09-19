---
layout: post
title: drools语法介绍(六)
category: drools
tags: [drools]
---



1、基本的匹配规则

1.1变量

**drools使用匹配的方式对Fact进行比对**，

比如

account : Account(balance > 100)

这个规则的含义就是在Fact中找到类型为Account，且balance属性值大于100的所有Account实例。

**可以指定变量来描述一个类型或者一个映射一个类的属性**，

比如

$account : Account($type : type)

使用$Variable来定义一个变量，这里定义了两个变量，$account表示定义一个类型为Account的变量，而$type映射Account类型中的type属性。定义变量是为了在后续的规则中使用。

$account : Account(balance >100) 
Customer(account == $account)

这个就是说要找到一些Custom类型的Fact，且其account属性必须满足前面定义的balance>100的条件。

1.2类型

drools支持各种java数据类型

**String：字符串**

Customer(name == "john")

**正则表达式：**

Customer(name matches "[A-Z][a-z]+")

表示Customer类型的name属性必须满足首字母为大写A-Z，且第二位以后有一个或者多个小写的a-z组成。

**Date：日期类型**

Account(dateCreate > "01-Feb-2009")

日期的格式默认是"dd-mmmm-yyyy"，可以更改。

**Boolean：布尔类型**

Transaction(isApprove == true)

**Enum：枚举类型**

Account(type == Account.Type.STUDENT)

1.3注释

单行注释：//或者#

多行注释： /* */

1.4包

package com.kingsun.drools.rules

声明该规则文件所属的包，是一种namespace组织方式，和java的包类似，物理上不需要存在相应的目录结构，它只是逻辑上的划分。

1.5导入

import可以导入在规则中使用到的类，也可以导入在规则中使用到的外部定义的functiong

import java.util.Map;

import com.kingsun.drools.service.LegacyBankService.extractData;

当**导入方法时，这个方法必须是static**的，因为利用的是jdk1.5的static import特性。

1.6全局变量global

**声明一个global变量**

global ReportFactory reportFactory;

**给全局变量赋值**

session.setGlobal("reportFactory", reportFactory);

或者

    List<Command> commands = new ArrayList<Command>();
    
    commmands.add(CommandFactory.newSetGlobal("reportFactory", reportFactory));

**使用全局变量**

session.getGlobals();

1.7函数Functions

function可以在规则文件中定义，但更多的是使用外部类中定义的static方法，这样只要java中可以实现的逻辑，在规则中都可以做为function调用。

调用外部类的functiong需要注意的是方法必须是静态的，static，而且这个类可以在Help辅助类中定义。

外部类需要import，此时在function中用到的参数类型也需要import。

如： 在外部的ValidationHelper辅助类中定一个一个static方法

    public static double calculateAccount(Account account) {
    
        return 100 + account.balance * 1.2;
    
    }

在规则drl文件中可以这么使用：

    import com.kingsun.drools.domain.Account;
    
    import function com.kingsun.drools.util.ValidationHelper.calculateAccount;
    
    rule "validation account"
    
        when
    
            $account : Account(balance > 100)
    
        then
    
            Account(balance == calculateAccount($account));
    
    end

1.8方言dialect

在规则表达式中可以使用方言来简化表达式，使之更加具有可读性。

package com.kingsun.drools.rules;

dialect "mvel"

方言默认的是java，drools也支持mvel，在package的后面声明该规则文件使用的方言

**mvel**

mvel是一种基于java应用程序的表达式语言，它支持属性和方法的直接访问

**简单属性表达式：**

($customer.name == "john") && (balance > 100) 

满足姓名为“john”，且balance必须大于100的customer

**支持属性导航：**

**Bean属性导航**

$customer.address.postalCode = "123" 等同于 $customer.getAddress().setPostalCode("123")

**访问List数据结构**

$customer.accounts[3] 等同于 $customer.getAccounts(3)

**访问Map数据结构**

$customerMap["123"] 等同于$customerMap.get["123"]

**内置的List、Map和数组arrays**

**Map**：创建一个AccountMap实例-->

 ["001", new Account("001"), "002", new Account("002")]

等同于创建了一个Map，并向Map中put了两个entry。

**List**：创建一个List实例-->

["001", "002", "003"] 

等同于创建了一个List，并向List中add了三个对象。

**Arrays**：创建一个数组-->

 {"001", "002", "003"}

等同于创建了一个Array，并初始化了三个成员。

**嵌套**

使用这个可以方便的访问复杂对象中包括的集合类型的对象。

 listOfPostCode = (postCode in (addresses in $customerS))

这个得到一个postCode列表，它是customers集合中的每一个custemer对象的地址属性中包含的postCode信息的汇集。

**强制转换**

当使用array = {1, 2, 3}时，mvel会自动将元素转换成integer类型。

**返回值**

保存变量的值等于最后一次赋予的值。

mvel同时还支持方法调用、控制流、赋值、动态类型等等，使用mvel的性能很好，不过要小心使用。在drools中有一些核心特性就是通过mvel来实现的。

1.9规则的条件部分

**And 与**

$customer :　Customer(name == "john", age > 20)

在condition中使用换行来表示与

**OR 或**

$customer :　Customer(name == "john" || age > 20)

**Not 非**

not Account(type == Account.Type.STUDENT) 

表示所有账户类型不是STUDENT的账户

**exists 存在**

exists Account(type == Account.Type.STUDENT)

**Eval**

捕获异常，只要eval表达式中的结果为true或者false就可以

$account : Account()

eval(accountService.isUniqueAccountNumber($account))

**返回值约束**

$customer1 : Customer()

Customer(age == ($customer.getAge + 10))

**内置eval**

$customer1 : Customer()

Customer(eval(age == $customer1.getAge() + 10))

和上个例子一样，只是使用了内置的eval来写的。

内置eval不能使用this，比如：

$customer1 : Customer()

$customer2 : Customer(this != $customer1)

customer1和customer2不是同一个对象实例

使用内置的eval表达式来描述：

$customer1 : Customer()

$customer2 : Customer(eval($customer2 != $customer1))

注意，在内置的eval中不能使用this来指代自己。

**嵌套访问**

$customer : Customer()

Account(this == $customer.accounts[0])

得到所有customer对象中的第一个账号

如果一个嵌套访问的对象属性值更改了，我们使用modify来提示drools属性改变了。

**This**

this指向当前的Fact

**和集合相关的运算**

**contains**

$account : Account()

$customer :　Customer(accounts contains $account)

$customer : Customer(accounts not contains $account)

**member of**

$customer : Customer($accounts : accounts)

Account(this member of $accounts)

或

Account(this member of $customer.accounts)

**From**

$customer : Customer()

Account() from $customer.accounts

from可以接受service的方法调用后的结果集，也可以指向一个对象或者一个集合

1.10规则的推理部分

规则引擎的作用就是在于根据预先制定的规则去和事实匹配，对符合激发条件的规则，执行规则中定义的推理，作出相应的处理。

有时候，规则和规则之间存在冲突，drools使用冲突解决策略来解决矛盾。有两种办法：一个是设置规则的优先级，优先级高的先触发。另一个是

在推理部分不推荐使用if then这样的条件判断语句，它应该是明确的行为，因为条件判断应该在LHS中就已经明确区分出来了，如果推论部分存在条件判断的话，应该增加新的规则以满足要求。这样做符合最佳实践。

当一个规则被激活，并且推理部分被执行，它可能会对Fact做一些修改或者更新，这样的修改和更新可能会激活更多的其他规则，因此，必须在推论部分明确指出这个规则对Fact作出的更新和修改，Drools能够及时对Fact进行更新，这样其他规则才能及时应用到最新的Fact数据上来。

推论部分经常使用的几个函数

**modify 修改**

更新session中存在的Fact

    rule "interst calculation"
    
    no-loop
    
        when 
    
            $account : Account()
    
        then
    
            modify($account) {
    
                setBalance((double)($account.getBalance() * 1.2))
    
            };
    
    end

**insert 插入**

向session的Fact中新增一个事实

insert(new Account())

**retract 收回**

从session的Fact中移除一个事实

retract(0)

1.11规则的属性部分

规则属性出现在rule和when关键字之间，用于修改和增强标准的规则行为。

    rule "rule attribute"
    
    salience 100
    
    dialect "mvel"
    
    no-loop
    
        when     
    
            //conditons条件部分
    
        then
    
            //consquence推论部分
    
    end

**salience**

salience是优先级的意思，它可以为负数，salience越高，表明规则的优先级越高，越先被激发。

默认值是0。

**no-loop**

表明对于每一个Fact实例，该规则只能被激活一次。

**dialect**

指定方言。

dialect "mvel"

