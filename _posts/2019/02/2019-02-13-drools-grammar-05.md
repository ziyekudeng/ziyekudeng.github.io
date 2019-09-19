---
layout: post
title: drools语法介绍(五)
category: drools
tags: [drools]
---

编辑pom.xml 文件，添加依赖

![](https://img-blog.csdn.net/20160725231046468?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center) 

 与决策表（不一定需要电子表格）相关的是“规则模板”（ drools-templates 模块中）。它们使用任何表格式的数据源作为一个规则数据源——填入模板产生多数规则。 这可以允许两个更灵活的电子表格，而且实例在现有的数据库中管辖（代价是预先开发产生规则的模板）。

 利用规则模板，数据与规则分离，并且有关规则的数据驱动部分没有限制。所以，你同时可以做你在

 规则表中能够做的任何事情，你也可以做到以下几点：

*   存储你的数据在数据库中（或者任何其他格式）。
*   根据数据的值有条件地产生规则。
*   为你的规则的任何部分使用数据（例如，条件运算符，类名，属性名）。
*   在相同的数据上运行不同的模板。

 我们先来一下 模板文件是如何编写的：

 首先先要引入一个xls文件 文件名要与配置文件中保持一致，路径也是一样的

 后面会对这个xls文件说明

 ![](https://img-blog.csdn.net/20160725231503878?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

 编辑drt文件 文件名要与配置文件中保持一致，路径也是一样的 最好是将xls与drt 文件 放在同一下目录下

 <code class="language-html">template header         #注：template header 固定写法，必须在模板的最开始部分
age                     #注： age log 表示标题的是依次显示数据的列的列表。在这个案例中，我们命名第一列为"age"，第三列为"log"
log
                        #注：这里是一空行，表示定义结束 
package rules.testdrt;
import com.drools.api.rule.Person;
global java.util.List list;

template "cheesefans"  #注"template" 关键字通知一个规则模板开始。在一个模板文件中可能超过一个模板。模板应该有一个唯一的名字

rule "Cheese fans_@{row.rowNumber}"#注：@{row.rowNumber}， 它给每个数据行一个唯一的数字， 使你能产生唯一的规则名字
    when
        Person(age == @{age})
    then
        list.add("@{log}");
end
end template
#上面是一个标准的模板的写法   可能写的比较乱，那请参考下面完整的例子
#在xls中的列数据要与 规则模板中用来做操作的类型包括java中的类型要保持一致。否则会报错，但要注意的一点是 如果用的是String类型的  在模板中引用 必须是要加 “”的 @{age} 是数字类型的 所以可以省略不加</code>

 <code class="language-html">template header 
age
log

package rules.testdrt;
import com.drools.api.rule.Person;
global java.util.List list;

template "cheesefans"

rule "Cheese fans_@{row.rowNumber}"
    when
        Person(age == @{age})
    then
        list.add("@{log}");
end
end template</code>**完整的规则模板的说明**

 其实就是将xls中的数据填写到模板文件中对应的 **占位符**上。但在编写xls有几个细节要注意一下：

 **第一点：xls中的具体行数据是根据配置文件row值来确定的，如果设置为row="1"则表示从第一行开始取数据**

 **第二点：xls中的具体列数据是根据配置文件col值来确定的，如果设置为col="1"则表示从第一列开始取数据**

 具体的java调用，其实与执行规则时的是一样的：

![](https://img-blog.csdn.net/20160725231253783?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

 **还有一个有变化的地方就是配置文件**： 

 ![](https://img-blog.csdn.net/20160725231206360?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center) 

 ruleTemplate 规则模板，dtable引用xls文件，表示二维表 template是具体的模板文件

 row/col表示行/列，引用xls的话 该值是不能省略的。

还有一种不通过 xls引用的方式来实现 规则模板 请参考：https://blog.csdn.net/u013115157/article/details/52029258


