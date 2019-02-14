---
layout: post
title: Jboss rules规则引擎 Drools 6.4.0 Final 教程(2)
category: drools
tags: [drools]
---

# 1-Drools语法详解

 ![](https://img-blog.csdn.net/20160618153917652?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

# -------rules规则文件-------

 ![](https://img-blog.csdn.net/20160618153921687?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

 2.package：

 对一个规则文件而言，package是必须定义的，必须放在规则文件第一行。特别的是，package的名字是随意的，不必必须对应物理路径，跟java的package的概念不同，这里只是逻辑上的一种区分。

   example: com.bw.drools;

 3.import：

 导入规则文件需要使用到的外部变量，这里的使用方法跟java相同，但是不同于java的是，这里的import导入的不仅仅可以是一个类，也可以是这个类中的某一个可访问的静态方法。

example: 类 com.bw.drools.Student;
example: 类 com.bw.drools.StudentController.studentList; 

 4.global：
定义global全局变量，通常用于返回数据和提供服务
全局变量与fact不一样，引擎不能知道全局变量的改变
必须要在插入fact之前，设置global变量 

 ![](https://img-blog.csdn.net/20160618154420100?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

 5.function。 
 规则中的代码块，封装业务操作，提高代码复用
函数以function开头，其它与JAVA方法类似
业务代码书写采用的是标准JAVA语法 

 <code class="language-java">function void SysName(String name){

System.out.println("I am"+name)

}</code>6.rule定义一个规则。

 rule "ruleName"。一个规则可以包含三个部分：

属性部分：定义当前规则执行的一些属性等，比如是否可被重复执行、过期时间、生效时间等。

条件部分，即LHS，定义当前规则的条件，如 when Message(); 判断当前workingMemory中是否存在Message对象。

结果部分，即RHS，这里可以写普通java代码，即当前规则条件满足后执行的操作，可以直接调用Fact对象的方法来操作应用。

 7.属性。

 activation-group、agenda-group、auto-focus、date-effective、date-expires、dialect、duration、duration-value、enabled、lock-on-active、no-loop、ruleflow-group、salience

salience 该属性的作用我们在前面的内容也有涉及，它的作用是用来设置规则执行的优先级，salience 属性的值是一个数字，数字越大执行优先级越高，同时它的值可以是一个负数。默认情况下，规则的salience 默认值为0，所以如果我们不手动设置规则的salience 属性，那么它的执行顺序是随机的。

no-loop 在一个规则当中如果条件满足就对Working Memory 当中的某个Fact 对象进行了修改，比如使用update 将其更新到当前的Working Memory 当中，这时引擎会再次检查所有的规则是否满足条件，如果满足会再次执行，

date-effective 该属性是用来控制规则只有在到达后才会触发，在规则运行时，引擎会自动拿当前操作系统的时候与date-effective 设置的时间值进行比对，只有当系统时间>=date-effective 设置的时间值时，规则才会触发执行，否则执行将不执行。在没有设置该属性的情况下，规则随时可以触发，没有这种限制。 date-effective 的值为一个日期型的字符串，默认情况下，date-effective 可接受的日期格式为“dd-MMM-yyyy”，例如2009 年9 月25 日在设置为date-effective 的值时，如果您的操作系统为英文的，那么应该写成“25-Sep-2009”；如果是英文操作系统“25-九月-2009”，

date-expires 该属性的作用与date-effective 属性恰恰相反， date-expires 的作用是用来设置规则的有效期，引擎在执行规则的时候，会检查规则有没有date-expires 属性，如果有的话，那么会将这个属性的值与当前系统时间进行比对，如果大于系统时间，那么规则就执行，否则就不执行。该属性的值同样也是一个日期类型，默认格式也是“dd-MMM-yyyy”，具体用法与date-effective 属性相同。

enabled enabled 属性比较简单，它是用来定义一个规则是否可用的。该属性的值是一个布尔值，默认该属性的值为true，表示规则是可用的，如果手工为一个规则添加一个enabled 属性，并且设置其enabled 属性值为false，那么引擎就不会执行该规则。

dialect 该属性用来定义规则当中要使用的语言类型，目前Drools5 版本当中支持两种类型的语言：mvel 和java，默认情况下，如果没有手工设置规则的dialect，那么使用的java 语言。

duration 对于一个规则来说，如果设置了该属性，那么规则将在该属性指定的值之后在另外一个线程里触发。该属性对应的值为一个长整型，单位是毫秒，代码清单2-37 里的规则rule1 添加了duration 属性，它的值为3000，表示该规则将在3000 毫秒之后在另外一个线程里触发。

lock-on-active 当在规则上使用ruleflow-group 属性或agenda-group 属性的时候，将lock-on-action 属性的值设置为true，可能避免因某些Fact 对象被修改而使已经执行过的规则再次被激活执行。可以看出该属性与no-loop 属性有相似之处，no-loop 属性是为了避免Fact 修改或调用了insert、retract、update 之类而导致规则再次激活执行，这里的lock-on-action 属性也是起这个作用，lock-on-active 是no-loop 的增强版属性，它主要作用在使用ruleflow-group 属性或agenda-group 属性的时候。lock-on-active 属性默认值为false。

activation-group 该属性的作用是将若干个规则划分成一个组，用一个字符串来给这个组命名，这样在执行的时候，具有相同activation-group 属性的规则中只要有一个会被执行，其它的规则都将不再执行。也就是说，在一组具有相同activation-group 属性的规则当中，只有一个规则会被执行，其它规则都将不会被执行。当然对于具有相同activation-group 属性的规则当中究竟哪一个会先执行，则可以用类似salience 之类属性来实现。

agenda-group 规则的调用与执行是通过StatelessSession 或StatefulSession 来实现的，一般的顺序是创建一个StatelessSession 或StatefulSession，将各种经过编译的规则的package 添加到session当中，接下来将规则当中可能用到的Global 对象和Fact 对象插入到Session 当中，最后调用fireAllRules 方法来触发、执行规则。在没有调用最后一步fireAllRules 方法之前，所有的规则及插入的Fact 对象都存放在一个名叫Agenda 表的对象当中，这个Agenda 表中每一个规则及与其匹配相关业务数据叫做Activation，在调用fireAllRules 方法后，这些Activation 会依次执行，这些位于Agenda 表中的Activation 的执行顺序在没有设置相关用来控制顺序的属性时（比如salience 属性），它的执行顺序是随机的，不确定的。 Agenda Group 是用来在Agenda 的基础之上，对现在的规则进行再次分组，具体的分组方法可以采用为规则添加agenda-group 属性来实现。agenda-group 属性的值也是一个字符串，通过这个字符串，可以将规则分为若干个Agenda Group，默认情况下，引擎在调用这些设置了agenda-group 属性的规则的时候需要显示的指定某个Agenda Group 得到Focus（焦点），这样位于该Agenda Group 当中的规则才会触发执行，否则将不执行。

auto-focus 前面我们也提到auto-focus 属性，它的作用是用来在已设置了agenda-group 的规则上设置该规则是否可以自动独取Focus，如果该属性设置为true，那么在引擎执行时，就不需要显示的为某个Agenda Group 设置Focus，否则需要。对于规则的执行的控制，还可以使用Agenda Filter 来实现。在Drools 当中，提供了一个名为org.drools.runtime.rule.AgendaFilter 的Agenda Filter 接口，用户可以实现该接口，通过规则当中的某些属性来控制规则要不要执行。org.drools.runtime.rule.AgendaFilter 接口只有一个方法需要实现，方法体如下： public boolean accept(Activation activation); 在该方法当中提供了一个Activation 参数，通过该参数我们可以得到当前正在执行的规则对象或其它一些属性，该方法要返回一个布尔值，该布尔值就决定了要不要执行当前这个规则，返回true 就执行规则，否则就不执行。

ruleflow-group 在使用规则流的时候要用到ruleflow-group 属性，该属性的值为一个字符串，作用是用来将规则划分为一个个的组，然后在规则流当中通过使用ruleflow-group 属性的值，从而使用对应的规则。

 8.Left Hand Side。
Conditions / LHS —匹配模式（Patterns）
没有字段约束的Pattern
Person()
有文本字段约束的Pattern

  Person( name == “bob” )
字段绑定的Pattern
Person( $name : name == “bob” )
变量名称可以是任何合法的java变量，$是可选的，可由于区分字段和变量
Fact绑定的Pattern
$bob : Person( name == “bob” )字段绑定的Pattern
变量约束的Pattern
Person( name == $name )

# Drools提供了十二中类型比较操作符：

> >= < <= == != contains / not contains / memberOf / not memberOf /matches/ not matches

not contains：与contains相反。

memberOf：判断某个Fact属性值是否在某个集合中，与contains不同的是他被比较的对象是一个集合，而contains被比较的对象是单个值或者对象。

not memberOf：正好相反。

matches：正则表达式匹配，与java不同的是，不用考虑'/'的转义问题

not matches:正好相反。

 9.Right Hand Side。

 insert：往当前workingMemory中插入一个新的Fact对象，会触发规则的再次执行，除非使用no-loop限定；

update：更新

modify：修改，与update语法不同，结果都是更新操作

retract：删除

# 后记：

 看到这，你可能还是不了解，drools怎么用，别担心，持续更新中，请等待……


