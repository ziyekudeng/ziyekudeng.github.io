---
layout: post
title: drools语法介绍(四)
category: drools
tags: [drools]
---



 **Figure 5.1. rule** 

Rule 结构是最重要的结构。 Rule 使用了形如“ IF ” something “ THEN ” action （当然，我们的关键字是“ when ”和“ then ”）的形式。

一个规则在一个 package 中必须要有唯一的名字。如果一个名字中含有空格，那就需要将名字放在双引号中（最好总是使用双引号）。

Attribute 是可选的（最好是每行只有一个 Attribute ）。

规则的 LHS 跟在“ when ”关键字的后面（最好是另起一行），同样 RHS 要跟在“ then ”关键字后面（最好也另起一行）。规则以关键字“ end ”结束。规则不能嵌套。

**5.1 Left Hand Side**

Left Hand Side 其实就是规则的条件部分。 LHS 对应的 rail-road 图如下，我们在后面会做进一步解释：

![](http://image.360doc.com/DownloadImg/7147/596025_2.png)

 **Figure 5.2. Left Hand Side

![](http://image.360doc.com/DownloadImg/7147/596025_3.png)** 

 **Figure 5.3. pattern**

**5.2 Right Hand Side**

Right Hand Side （ RHS ）就是规则的结果（ consequence ）或者动作（ action ）部分。 RHS 的目的是 retract 或 add facts 到 WorkingMemory 中，还有针对你的 application 的动作。实际上， RHS 是当规则激发（ fire ）时执行的代码块。

在 RHS 中，你可以使用几个方便的 method 来改变 WorkingMemory ：

“ modify(obj) ”：告诉引擎一个对象已经发生变化，规则必须重新匹配（ obj 对象必须是出现在 LHS 中的对象）；

“ assert(new Something()) ”：将一个新的 Something 对象加入 WorkingMemory ；

“ assertLogical(new Something()) ”：与 assert 方法类似。但是，当没有 fact 支持当前激发规则的真实性的时候，这个新对象会自动被 retract ，

“ retract(obj) ”：从 WorkingMemory 中移除一个对象。

这些方法都是宏指令，提供了到 KnowledgeHelper 实例的快捷方式（参考 KnowledgeHelper 接口）。 KnowledgeHelper 接口可以在 RHS 代码块中调用，通过变量“ drools ”。如果你在 assert 进引擎的 JavaBean 中加入“ Property Change Listener ”，在对象发生变化的时候，你就不用调用“ modify ”方法。

**5.3 Rule Attributes**

 ![](http://image.360doc.com/DownloadImg/7147/596025_4.png)

 **Figure 5.4. rule attributes**

**5.3.1 no-loop** 

默认值： false

类型： boolean

当在 rule 的 RHS 中修改了一个 fact ，这可能引起这个 rule 再次被 activate ，引起递归。将 no-loop 设为 true ，就可以防止这个 rule 的 Activation 的再次被创建。

**5.3.2 salience** 

默认值： 0

类型： int

每个 rule 都可以设置一个 salience 整数值，默认为 0 ，可以设为正整数或负整数。 Salience 是优先级的一种形式。当处于 Activation 队列中时，拥有高 salience 值的 rule 将具有更高的优先级。

**5.3.3 agenda-group** 

默认值： MAIN

类型： String

Agenda group 允许用户对 Agenda 进行分组，以提供更多的执行控制。只有具有焦点的组中的 Activation 才会被激发（ fire ）。

**5.3.4 auto-focus** 

默认值： false

类型： boolean

当一个规则被 activate （即 Activation 被创建了），如果这个 rule 的 auto-focus 值为 true 并且这个 rule 的 agenda-group 没有焦点，此时这个 Activation 会被给予焦点，允许这个 Activation 有 fire 的潜在可能。

**5.3.5 activation-group** 

默认值： N/A

类型： String

当处于同一个 activation-group 中的第一个 Activation fire 后，这个 activation-group 中其余剩下的 Activation 都不会被 fire 。

**5.3.6 duration** 

默认值：没有默认值

类型： long

**5.4 Column**

 ![](http://image.360doc.com/DownloadImg/7147/596025_5.png)

 **Figure 5.5. Column**

**Example 5.1. Column** 
 Cheese( )
Cheese( type  ==   " stilton " , price  <   10  )

一个 Column 由一个类的一个或多个域约束构成。第一个例子没有约束，它将匹配 WorkingMemory 中所有的 Cheese 实例。第二个例子对于一个 Cheese 对象有两个字面约束（ Literal Constraints ），它们被用“，”号隔开，意味着“ and ”。

![](http://image.360doc.com/DownloadImg/7147/596025_6.png)

 **Figure 5.6. Bound Column**

**Example 5.2. Bound Column** 
 cheapStilton : Cheese( type  ==   " stilton " , price  <   10  )

这个例子同前一个例子有点类似。但是在这个例子中，我们将一个变量绑定到匹配规则引擎的 Cheese 实例上。这意味着，你可以在另一个条件中使用 cheapStilton ，或者在 rule 的 RHS 中。

**5.4.1 Field Constraints**

Field Constraints 使规则引擎可以从 WorkingMemory 中挑选出合适的 Fact 对象。一个 Fact 的“ Field ”必须符合 JavaBean 规范，提供了访问 field 的 getter 方法。你可以使用 field 的名字直接访问 field ，或者使用完整的方法名（省略括号）。

例如，以我们的 Chess 类为例，下面是等价的： Cheese(type = = …) 和 Cheese(getType = = …) 。这意味着，你可以使用不太严格遵守 JavaBean 规范对象。尽管如此，你要保证 accessor 方法是不带参数的，以保证它不会改变对象的状态。

注意：如果一个 field 使用原始类型（ primitive type ）， Drools 将会把它们自动装箱成相应的对象（即使你使用 java 1.4 ），但是在 java 1.4 下却不能自动拆箱。总的来说，尽量在 rule 所使用的类中，使用非原始类型的域。如果是使用 java 5 ，就可以比较随意了，因为编译器会帮你执行自动装拆箱。

**5.4.1.1 Operators** 
 **![](http://image.360doc.com/DownloadImg/7147/596025_7.png)** 

 **Figure 5.7. Operators**

有效的操作符是同域类型相关的。例如，对于日期域，“ < ”意味着“之前”。“ matches ”只适用于 String 域，“ contains ”和“ excludes ”只适用于 Collection 类型域。

**5.4.1.2 字面值约束（ Literal Constraints ）** 

最简单的域约束就是字面值约束，允许用户将一个 field 约束于一个已知值。

注意：你可以检查域是否为 null ，使用 = = 或 != 操作符和字面值‘ null ’关键字。如， Cheese(type != null) 。字面值约束，特别是“ = = ”操作符，提供了非常快的执行速度，因为可以使用散列法来提高性能。

![](http://image.360doc.com/DownloadImg/7147/596025_8.png)

 **Figure 5.8. Literal Constraints**

_**Numeric**_

所有标准的 Java 数字基本类型都可以用。

有效操作符：

·         ==

·         !=

·         >

·         <

·         >=

·         <=

**Example 5.3 . Numeric Literal Constraint** 
 Cheese( quantity  ==   5  )

_**Date**_ 

当前只对“ dd-mm-yyyy ”的日期格式提供默认支持。你可以通过指定 drools.dateformat 系统属性，来改变默认的日期格式。如果需要更多的控制，要用谓词约束（ Predicate Constraint ）。

有效操作符：

·         ==

·         !=

·         >

·         <

·         >=

·         <=

**Example 5.4. Date Literal Constraint** 
 Cheese( bestBefore  <   " 27-Oct-2007 "  )

 _**String**_ 

可以使用任何有效的 Java String 。

有效操作符：

·         ==

·         !=

**Example 5.5. String Literal Constraint** 
 Cheese( type  ==   " stilton "  )

 **_Boolean_** 

只能用 “ true ”或“ false ”。 0 和 1 不能被识别，而且 Cheese(smelly) 也是不被允许的。

有效操作符：

·         ==

·         !=

**Example 5.6 Boolean Literal Constraint** 
 Cheese( smelly  =   =   true  )

 **_Matches Operator_** 

Matches 操作符后面可以跟任何有效的 Java 正则表达式。

**Example 5.7. Regular Expression Constraint** 
 Cheese( type matches  " (Buffulo)?\\S*Mozerella "  )

 **_Contains Operator and Excludes Operator_** 

“ contains ”和“ excludes ”可以用来检查一个 Collection 域是否含有一个对象。

**Example 5.8. Literal Cosntraints with Collections**

 CheeseCounter( cheeses contains  " stilton "  )
CheeseCounter( cheeses excludes  " chedder "  )

**5.4.1.3 Bound Variable Constraint** 

可以将 Facts 和它们的 Fields 附给一个 Bound Variable ，然后在后续的 Field Constraints 中使用这些 Bound Variable 。一个 Bound Variable 被称为声明（ Declaration ）。 Declaration 并不能和“ matches ”操作符合用，但却可以和“ contains ”操作符合用。

**Example 5.9. Bound Field using ‘==‘ operator** 
 Person( likes : favouriteCheese )
Cheese( type  ==  likes )

在上面的例子中，“ likes ”就是我们的 Bound Variable ，即 Declaration 。它被绑定到了任何正在匹配的 Person 实例的 favouriteCheese 域上，并且用来在下一个 Column 中约束 Cheese 的 type 域。可以使用所有有效的 Java 变量名，包括字符“ $ ”。“ $ ”经常可以帮助你区分 Declaration 和 field 。下面的例子将一个 Declaration 绑定到匹配的实例上，并且使用了“ contains ”操作符。注意： Declaratino 的第一个字符用了“ $ ”：

**Example 5.10 Bound Fact using ‘contains‘ operator** 
 $stilton : Cheese( type  ==   " stilton "  )
Cheesery( cheeses contains $stilton )

 **5.4.1.4 Predicate Constraints** 
 **![](http://image.360doc.com/DownloadImg/7147/596025_9.png)** 

 **Figure 5.9. Predicate expression**

Predicate 表达式可以使用任何有效的 Java 逻辑表达式。先前的 Bound Declaration 可以用在表达式中。

下面的例子将会找出所有男性比女性大 2 岁的 pairs of male/femal people ：

**Example 5.11\. Predicate Constraints**
 Person( girlAge : age, sex  =   =   " F "  )
Person( boyAge : age  ->  ( girlAge.intValue()  +   2   ==  boyAge.intValue() ), sex  =   =   " M "  )

**5.4.1.5 Return Value Constraints** 
 **![](http://image.360doc.com/DownloadImg/7147/596025_9.png)** 

 **Figure 5.10. Return Value expression**

一个 Retrurn Value 表达式可以使用任何有效的 Java 表达式，只要它返回一个对象，不能返回原始数据类型。如果返回值是原始数据类型，要先进行装箱。先前的 Bound Declaration 也可以使用在表达式中。

下面的例子跟上一节的例子一样，也将会找出所有男性比女性大 2 岁的 pairs of male/femal people 。注意：这里我们不用绑定 boyAge ，增加了可读性：

**Example 5.12. Return Value Constraints**

 Person( girlAge : age, sex  =   =   " F "  )
Person( age  =   =  (  new  Integer(girlAge.intValue()  +   2 ) ), sex  =   =   " M "  )

**5.5 Conditional Elements**

Conditional Elements 用来连接一个或多个 Columns 。

**5.5.1 “ and ”** 
 **![](http://image.360doc.com/DownloadImg/7147/596025_11.png)** 

 **Figure 5.11. and**

**Example 5.13\.  And**

 Cheese( cheeseType : type )  &&  Person( favouriteCheese  ==  cheeseType )
Cheese( cheeseType : type ) and Person( favouriteCheese  ==  cheeseType )

**5.5.2 “ or ”** 
 **![](http://image.360doc.com/DownloadImg/7147/596025_12.png)** 

 **Figure 5.12. or**

**Example 5.14. or**

 Person( sex  ==   " f " , age  >   60  )  ||  Person( sex  ==   " m " , age  >   65  )
Person( sex  ==   " f " , age  >   60  ) or Person( sex  ==   " m " , age  >   65  )

 **![](http://image.360doc.com/DownloadImg/7147/596025_13.png)** 

 **Figure 5.13. or with binding**

**Example 5.15. or with binding**

 pensioner : Person( sex  ==   " f " , age  >   60  )  ||  pensioner : Person( sex  ==   " m " , age  >   65  ) 
pensioner : ( Person( sex  ==   " f " , age  >   60  ) or Person( sex  ==   " m " , age  >   65  ) )

“ or ” Conditional Element 的使用会导致多条 rule 的产生，称为 sub rules 。上面的例子将在内部产生两条规则。这两条规则会在 WorkingMemory 中各自独立的工作，也就是它们都能进行 match ， activate 和 fire 。当对一个“ or ” Conditional Element 使用变量绑定时，要特别小心，错误的使用将产生完全不可预期的结果。

可以将“ OR ” Conditional Element 理解成产生两条规则的快捷方式。因此可以很容易理解，当“ OR ” Conditional Element 两边都为真时，这样的一条规则将可能产生多个 activation 。

**5.5.3 “ eval ”** 
 **![](http://image.360doc.com/DownloadImg/7147/596025_14.png)** 

 **Figure 5.14 . eval**

Eval is essentially a catch all which allows any semantic code (that returns a primitive boolean) to be executed. 在表达式中可以引用在 LHS 中出现的变量，和在 rule package 中的 Functions 。一个 Eval 应该是 LHS 中的最后一个 Conditional Element 。在一个 rule 中，你可以有多个 eval 。

Eval 不能被索引，因此不能像 Field Constraints 那样被优化。尽管如此，当 Functions 的返回值一直在变化时，应该使用 Eval ，因为这在 Field Constraints 中时不允许的。如果规则中的其他条件都匹配，一个 eval 每次都要被检查。（现在还不理解到底 eval 要怎么用？）

**Example 5.16. eval**

 p1 : Parameter() 
p2 : Parameter()
eval( p1.getList().containsKey(p2.getItem()) )
eval( isValid(p1, p2) )  // this is how you call a function in the LHS - a function called  // "isValid"

**5.5.4 “ not ”** 
 **![](http://image.360doc.com/DownloadImg/7147/596025_15.png)** 

 **Figure 5.15. not**

“ not ”是一阶逻辑的存在量词（ first order logic’s Existential Quantifier ） , 用来检查 WorkingMemory 中某对象的非存在性。现在，只有 Columns 可以放在 not 中，但是将来的版本会支持“ and ”和“ or ”。

**Example 5.17. No Buses**

 not Bus()

**Example 5.18. No red Buses**

 not Bus(color  ==   " red " )
not ( Bus(color  ==   " red " , number  =   =   42 ) )  // brackets are optional

**5.5.5 “ exists ”** 
 **![](http://image.360doc.com/DownloadImg/7147/596025_16.png)** 

 **Figure 5.16. exists**

“ exists ” 是一阶逻辑的存在量词（ first order logic’s Existential Quantifier ），用来检查 WorkingMemory 中某对象的存在性。可以将“ exists ”理解为“至少有一个”（ at least one… ）。它的意义不同于只有 Column 本身，“ Column ”本身可以理解为“对于每一个 … ”（ for each of … ）。如果你对一个 Column 使用了“ exists ”，那么规则将只 activate 一次，而不管 WorkingMeomry 中有多少个数据匹配了那个条件。

现在，只有 Columns 可以放在“ exists ”中，但是将来的版本会支持“ and ”和“ or ”。

**Example 5.19. At least one Bus**

     exists Bus()

**Example 5.20. At least one red Bus**
    
     exists Bus(color  ==   " red " )

**5.5.6 “ group ”** 
 **![](http://image.360doc.com/DownloadImg/7147/596025_17.png)** 

 **Figure 5.17. group**

Group 的作用相当于代数学中的“（）”，显式的指明操作的顺序。

**5.6** **再谈自动装箱和原始类型**

Java 5 支持在原始类型与其对应包装类之间的装拆箱。尽管如此，因为要让 drools 能够在 J2SE 1.4 下运行，我们不能依靠 J2SE 。因此， drools 自己实现了自动装箱。被引用（即被 Bound Variable 绑定）的 Field 将被自动进行装箱（如果它们本身就是 object ，就不会有任何变化）。尽管如此，必须要注意的是，他们并不会被自动拆箱。

还有一点要注意的，就是对于 ReturnValue Constraints ，返回值的代码必须返回一个对象，而不能是一个原始类型。

**6** **．** **Query**

 ![](http://image.360doc.com/DownloadImg/7147/596025_18.png)

 **Figure 6.1 . query**

一个 query 只包含了一个 rule 的 LHS 结构（你不用指定“ when ”或“ then ”）。这是查询 WorkingMemory 中匹配条件的 Facts 的简单的方法。

要得到结果，要使用 WorkingMemory.getQueryResults(“name”) 方法，其中“ name ”就是 query 的名字。 Query 的名字在 RuleBase 中是全局的，所以， do not add queries of the same name to different packages for the same RuleBase 。

下面的例子创建了一个简单的 query 来查询所有年龄大于 30 的人：

**Example 6.1. Query People over the age of 30**  

     query "people over the age of 30" 
        person : Person( age > 30 )
    end

我们通过一个标准的循环来迭代一个返回的QueryResults对象。每一次的iterate将返回一个QueryResult对象。我们可以用QueryResult对象的get()方法来访问每个Column，通过传入Bound Declaration或index position。

**Example 6.2. Query People over the age of 30**

     QueryResults results = workingMemory.getQueryResults( "people over the age of 30" );
    System.out.println( "we have " + results.size() + " people over the age  of 30" );
    
    System.out.println( "These people are are over 30:" );
    
    for ( Iterator it = results.iterator; it.hasNext(); ) {
        QueryResult result = ( QueryResult ) it.next();
        Person person = ( Person ) result.get( "person" );
        System.out.println( person.getName() + "\n" );
    }


