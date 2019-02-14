---
layout: post
title: Drools规则引擎(十)
category: drools
tags: [drools]
---



# 1、例子




我们假定如下情景：网站伴随业务产生而进行的积分发放操作。比如支付宝信用卡还款奖励积分等。
我们定义一下发放规则：
积分的发放参考因素有：交易笔数、交易金额数目、信用卡还款次数、生日特别优惠等。
定义规则：
过生日，则加10分，并且将当月交易比数翻倍后再计算积分
2011-01-08 - 2011-08-08每月信用卡还款3次以上，每满3笔赠送30分
当月购物总金额100以上，每100元赠送10分
当月购物次数5次以上，每五次赠送50分
特别的，如果全部满足了要求，则额外奖励100分
发生退货，扣减10分
退货金额大于100，扣减100分



代码思路规划，一共6个文件：两个drl规则文件，一个用于添加积分，一个用于扣减积分；一个积分计算对象（PointDomain Fact）；一个RuleBaseFacatory工厂类；一个积分规则接口，一个积分规则接口实现类（包括测试方法）。



#### 1.1、规则文件

addpoint.drl

package com.esom.tech.drools.pointrule.addpoint.drlimport com.esom.tech.drools.pointrule.PointDomain;// 过生日，则加10分，并且将当月交易比数翻倍后再计算积分rule birthdayPointsalience 100when$pointDomain : PointDomain(birthDay == true)then$pointDomain.setPoint($pointDomain.getPoint()+10);$pointDomain.setBuyNums($pointDomain.getBuyNums()*2);$pointDomain.setBuyMoney($pointDomain.getBuyMoney()*2);$pointDomain.setBillThisMonth($pointDomain.getBillThisMonth()*2);$pointDomain.recordPointLog($pointDomain.getUserName(),"birthdayPoint");end// 2013-01-08 - 2013-08-08每月信用卡还款3次以上，每满3笔赠送30分rule billThisMonthPointsalience 99date-effective "2013-01-08 23:59:59"date-expires "2013-08-08 23:59:59"when$pointDomain : PointDomain(billThisMonth >= 3)then$pointDomain.setPoint($pointDomain.getPoint()+$pointDomain.getBillThisMonth()/3*30);$pointDomain.recordPointLog($pointDomain.getUserName(),"billThisMonthPoint");end// 当月购物总金额100以上，每100元赠送10分rule buyMoneyPointsalience 98when$pointDomain : PointDomain(buyMoney >= 100)then$pointDomain.setPoint($pointDomain.getPoint()+ (int)$pointDomain.getBuyMoney()/100 * 10);$pointDomain.recordPointLog($pointDomain.getUserName(),"buyMoneyPoint");end// 当月购物次数5次以上，每五次赠送50分rule buyNumsPointsalience 97when$pointDomain : PointDomain(buyNums >= 5)then$pointDomain.setPoint($pointDomain.getPoint()+$pointDomain.getBuyNums()/5 * 50);$pointDomain.recordPointLog($pointDomain.getUserName(),"buyNumsPoint");end// 特别的，如果全部满足了要求，则额外奖励100分rule allFitPointsalience 96when$pointDomain:PointDomain(buyNums >= 5 && billThisMonth >= 3 && buyMoney >= 100)then$pointDomain.setPoint($pointDomain.getPoint()+ 100);$pointDomain.recordPointLog($pointDomain.getUserName(),"allFitPoint");end

subpoint.drl

package com.esom.tech.drools.pointrule.subpoint.drlimport com.esom.tech.drools.pointrule.PointDomain;// 发生退货，扣减10分rule subBackNumsPointsalience 10when$pointDomain : PointDomain(backNums >= 1)then$pointDomain.setPoint($pointDomain.getPoint()-10);$pointDomain.recordPointLog($pointDomain.getUserName(),"subBackNumsPoint");end// 退货金额大于100，扣减100分rule subBackMondyPointsalience 9when$pointDomain : PointDomain(backMondy >= 100)then$pointDomain.setPoint($pointDomain.getPoint()-10);$pointDomain.recordPointLog($pointDomain.getUserName(),"subBackMondyPoint");end



#### 1.2、积分计算对象

PointDomain.java

package com.esom.tech.drools.pointrule;public class PointDomain {// 用户名private String userName;// 是否当日生日private boolean birthDay;// 增加积分数目private long point;// 当月购物次数private int buyNums;// 当月退货次数private int backNums;// 当月购物总金额private double buyMoney;// 当月退货总金额private double backMondy;// 当月信用卡还款次数private int billThisMonth;/** * 记录积分发送流水，防止重复发放 * @param userName 用户名 * @param type 积分发放类型 */public void recordPointLog(String userName, String type){System.out.println("增加对"+userName+"的类型为"+type+"的积分操作记录.");}//get、set方法...}
#### 1.3、RuleBaseFacatory工厂类

RuleBaseFacatory.java

package com.esom.tech.drools.pointrule;import org.drools.RuleBase;import org.drools.RuleBaseFactory;import com.esom.tech.pattern.Singleton;public class RuleBaseFacatory {private static RuleBase ruleBase;public static RuleBase getRuleBase() {if (ruleBase == null) {synchronized (Singleton.class) {if (ruleBase == null) {ruleBase = RuleBaseFactory.newRuleBase();}}}return ruleBase;}}



#### 1.4、积分规则接口及实现

PointRuleEngine.java

package com.esom.tech.drools.pointrule;public interface PointRuleEngine {//初始化规则引擎public void initEngine();//刷新规则引擎中的规则public void refreshEnginRule();/** * 执行规则引擎 * @param pointDomain 积分Fact */public void executeRuleEngine(final PointDomain pointDomain);}





PointRuleEngineImpl.java

package com.esom.tech.drools.pointrule;import java.io.BufferedReader;import java.io.File;import java.io.FileNotFoundException;import java.io.FileReader;import java.io.InputStream;import java.io.InputStreamReader;import java.io.Reader;import java.util.ArrayList;import java.util.List;import org.drools.RuleBase;import org.drools.StatefulSession;import org.drools.compiler.PackageBuilder;import org.drools.spi.Activation;public class PointRuleEngineImpl implements PointRuleEngine {private RuleBase ruleBase;//初始化规则引擎：规则收集public void initEngine() {// 设置时间格式System.setProperty("drools.dateformat", "yyyy-MM-dd HH:mm:ss");ruleBase = RuleBaseFacatory.getRuleBase();try {PackageBuilder backageBuilder = getPackageBuilderFromDrlFile();ruleBase.addPackages(backageBuilder.getPackages());} catch (Exception e) {e.printStackTrace();}}//刷新规则引擎中的规则public void refreshEnginRule() {ruleBase = RuleBaseFacatory.getRuleBase();org.drools.rule.Package[] packages = ruleBase.getPackages();for (org.drools.rule.Package pg : packages) {ruleBase.removePackage(pg.getName());}initEngine();}//执行规则引擎：规则执行public void executeRuleEngine(final PointDomain pointDomain) {if (null == ruleBase.getPackages()|| 0 == ruleBase.getPackages().length) {return;}StatefulSession statefulSession = ruleBase.newStatefulSession();statefulSession.insert(pointDomain);// firestatefulSession.fireAllRules(//Agenda 负有规划激活规则运行的责任//Agenda Filter，过滤名字含test的规则new org.drools.spi.AgendaFilter() {public boolean accept(Activation activation) {return !activation.getRule().getName().contains("test");}});statefulSession.dispose();}//从Drl规则文件中读取规则：规则编译private PackageBuilder getPackageBuilderFromDrlFile() throws Exception {// 获取测试脚本文件List<String> drlFilePath = getTestDrlFile();// 装载测试脚本文件List<Reader> readers = readRuleFromDrlFile(drlFilePath);PackageBuilder backageBuilder = new PackageBuilder();for (Reader r : readers) {backageBuilder.addPackageFromDrl(r);}// 检查脚本是否有问题if (backageBuilder.hasErrors()) {throw new Exception(backageBuilder.getErrors().toString());}return backageBuilder;}//装载规则文件private List<Reader> readRuleFromDrlFile(List<String> drlFilePath)throws FileNotFoundException {if (null == drlFilePath || 0 == drlFilePath.size()) {return null;}List<Reader> readers = new ArrayList<Reader>();for (String ruleFilePath : drlFilePath) {readers.add(new FileReader(new File(ruleFilePath)));}return readers;}//获取规则文件private List<String> getTestDrlFile() {List<String> drlFilePath = new ArrayList<String>();drlFilePath.add("D:\\workspace\\helloworld\\src\\main\\java\\com\\esom\\tech\\drools\\pointrule\\addpoint.drl");drlFilePath.add("D:\\workspace\\helloworld\\src\\main\\java\\com\\esom\\tech\\drools\\pointrule\\subpoint.drl");return drlFilePath;}public static void main(String[] args) throws Exception {PointRuleEngine pointRuleEngine = new PointRuleEngineImpl();while(true){System.out.println("请输入命令：");InputStream is = System.in;BufferedReader br = new BufferedReader(new InputStreamReader(is));String input = br.readLine();if(null != input && "s".equals(input)){System.out.println("初始化规则引擎...");pointRuleEngine.initEngine();System.out.println("初始化规则引擎结束.");}else if("e".equals(input)){final PointDomain pointDomain = new PointDomain();System.out.println("初始化规则引擎...");pointRuleEngine.initEngine();System.out.println("初始化规则引擎结束.");pointDomain.setUserName("hello kity");pointDomain.setBackMondy(100d);pointDomain.setBuyMoney(500d);pointDomain.setBackNums(1);pointDomain.setBuyNums(5);pointDomain.setBillThisMonth(5);pointDomain.setBirthDay(true);pointDomain.setPoint(0l);pointRuleEngine.executeRuleEngine(pointDomain);System.out.println("执行完毕BillThisMonth："+pointDomain.getBillThisMonth());System.out.println("执行完毕BuyMoney："+pointDomain.getBuyMoney());System.out.println("执行完毕BuyNums："+pointDomain.getBuyNums());System.out.println("执行完毕规则引擎决定发送积分："+pointDomain.getPoint());} else if("r".equals(input)){System.out.println("刷新规则文件...");pointRuleEngine.refreshEnginRule();System.out.println("刷新规则文件结束.");}}}}



执行main方法，输入'e'，输出：

请输入命令：
e
初始化规则引擎...
初始化规则引擎结束.
增加对hello kity的类型为birthdayPoint的积分操作记录.
增加对hello kity的类型为billThisMonthPoint的积分操作记录.
增加对hello kity的类型为buyMoneyPoint的积分操作记录.
增加对hello kity的类型为buyNumsPoint的积分操作记录.
增加对hello kity的类型为allFitPoint的积分操作记录.
增加对hello kity的类型为subBackNumsPoint的积分操作记录.
增加对hello kity的类型为subBackMondyPoint的积分操作记录.
执行完毕BillThisMonth：10
执行完毕BuyMoney：1000.0
执行完毕BuyNums：10
执行完毕规则引擎决定发送积分：380
请输入命令：



#### 2、Droolsv API解释

Drools API可以分为三类：规则编译、规则收集和规则的执行



1）PackageBuilder规则编译：规则文件进行编译， 最终产生一批编译好的规则包(Package)供其它的应用程序使用。

2）RuleBase：提供的用来收集应用当中规则（rule）定义的规则库对象，在一个RuleBase当中可以包含普通的规则（rule）、规则流(rule flow)、函数定义(function)、用户自定义对象（type model）等。

3）StatefulSession：是一种最常用的与规则引擎进行交互的方式，它可以与规则引擎建立一个持续的交互通道，在推理计算的过程当中可能会多次触发同一数据集。在用户的代码当中，最后使用完StatefulSession对象之后，一定要调用其dispose()方法以释放相关内存资源。有状态的。

4）StatelessSession：使用StatelessSession对象时不需要再调用dispose()方法释放内存资源不能进行重复插入fact 的操作、也不能重复的调用fireAllRules()方法来执行所有的规则，对应这些要完成的工作在StatelessSession当中只有execute(…)方法，通过这个方法可以实现插入所有的fact 并且可以同时执行所有的规则或规则流，事实上也就是在执行execute(…)方法的时候就在StatelessSession内部执行了insert()方法、fireAllRules()方法和dispose()方法。

StatefulSession和StatelessSession都继承了WorkingMemory。

5）Fact ：是指在Drools 规则应用当中，将一个普通的JavaBean插入到规则的WorkingMemory当中后的对象规则可以对Fact 对象进行任意的读写操作，当一个JavaBean 插入到WorkingMemory 当中变成Fact 之后，Fact 对象不是对原来的JavaBean 对象进行Clone，而是原来JavaBean 对象的引用。

#### 3、Drools规则

规则文件格式：

package package-name //包名是必须的，并必须放在第一行。除外其他的位置没要求。

imports
globals
functions
queries
rules



## 3.1、规则语言

rule "name"//规则名称
 attributes //属性部分
when
 LHS //left hand sid条件部分
then
 RHS //right hand sid结果部分
end

### 3.1.1、条件部分

条件部分又被称之为Left Hand Side，简称为LHS，条件又称之为pattern（匹配模式）:在一个规则当中when与then 中间的部分就是LHS 部分。在LHS 当中，可以包含0~n 个条件，如果LHS 部分为空的话，那么引擎会自动添加一个eval(true)的条件，由于该条件总是返回true，所以LHS 为空的规则总是返回true，在Drools
当中在pattern 中没有连接符号，那么就用and 来作为默认连接，所以在该规则的LHS 部分中两个pattern 只有都满足了才会返回true。默认情况下，每行可以用“;”来作为结束符（和Java 的结束一样），当然行尾也可以不加“;”结尾。
约束连接：对于对象内部的多个约束的连接，可以采用“&&”（and）、“||”(or)和“,”(and)来实现，表面上看“,”与“&&”具有相同的含义，但是有一点需要注意，“，”与“&&”和“||”不能混合使用，也就是说在有“&&”或“||”出现的LHS 当中，是不可以有“,”连接符出现的，反之亦然。



条件操作符：共计12种：>、>=、<、<=、= =、!=、contains、not contains、memberof、not memberof、matches、not matches



1） contains：用来检查一个Fact 对象的某个字段（该字段要是一个Collection或是一个Array 类型的对象）是否包含一个指定的对象。
contains 只能用于对象的某个Collection/Array 类型的字段与另外一个值进行比较，作为比较的值可以是一个静态的值，也可以是一个变量(绑定变量或者是一个global 对象)

package testrule "rule1"when  $order:Order();  $customer:Customer(age >20, orders contains $order);then  System.out.println($customer.getName());end

2）not contains：与contains作用相反



3）member of：是用来判断某个Fact 对象的某个字段是否在一个集合（Collection/Array）当中，用法与contains 有些类似，但也有不同，member of 前边是某个数据对象且一定要是一个变量(绑定变量或者是一个global 对象)，后边是数据对象集合

package testglobal String[] orderNames;rule "rule1"when  $order:Order(name memberOf orderNames);then  System.out.println($order.getName());end

4）not member of：与member of作用相反



5）matches: 用来对某个Fact 的字段与标准的Java 正则表达式进行相似匹配，被比较的字符串可以是一个标准的Java 正则表达式，但有一点需要注意，那就是正则表达式字符串当中不用考虑“\”的转义问题

package testimport java.util.List;rule "rule1"when  $customer:Customer(name matches "李.*");then  System.out.println($customer.getName());end
### 6）notmatches:与matches相反3.1.2、结果部分

结果部分又被称之为Right Hand Side，简称为RHS，在一个规则当中then 后面部分就是RHS，只有在LHS 的所有条件都满足时RHS 部分才会执行, salience该属性的作用是通过一个数字来确认规则执行的优先级，数字越大，执行越靠前。

在结果部分可以直接使用drools内置的宏函数。



### 3.1.2.1、宏函数介绍

1）insert：作用与我们在Java类当中调用StatefulSession对象的insert 方法的作用相同，都是用来将一个Fact 对象插入到当前的WorkingMemory 当中。一旦调用insert宏函数，那么Drools会重新与所有的规则再重新匹配一次。



2）insertLogical：作用与insert 类似，它的作用也是将一个Fact 对象插入到当前的WorkingMemroy 当中。



3）update：用来实现对当前WorkingMemory 当中的Fact 进行更新。如果希望规则只执行一次，那么可以通过设置规则的no-loop属性为true 来实现。



4）retract：用来将Working Memory当中某个Fact对象从Working Memory当中删除。

package com.esom.tech.drools.drlimport com.esom.tech.drools.Message;import com.esom.tech.drools.Person;query "QUERY MESSAGE BY STATUS"(int $status)    $message : Message( status == $status )endquery "QUERY ALL MESSAGE"    $message : Message()enddeclare Addresscity : StringaddressName : Stringendrule "HelloWorld"    when       $msg : Message(status == Message.HELLO, $info: info)       $person : Person(age > 30)    then       System.out.println( $info + ", old man " + $person.getName() );        $msg.setInfo( "Goodbye World" );       $msg.setStatus( Message.GOODBYE );       update($msg);       Message m = new Message();       m.setInfo(  "Goodbye Again" );       m.setStatus( Message.GOODBYE );       insert( m );endrule "GoodBye"    when       $person : Person()       $m : Message( status == Message.GOODBYE, info != null )    then       System.out.println( $m.getInfo() + ", old man " + $person.getName() ); end



Java Bean：

package com.esom.tech.drools;public class Message {public static final int HELLO = 0;public static final int GOODBYE = 1;private String info;private int status;//get、set方法...}



package com.esom.tech.drools;public class Person {private String name;private int age;//get、set方法...}
### 3.1.3、属性部分

规则的属性共有13 个分别是：salience、no-loop、date-effective、date-expires、enabled、dialect、duration、lock-on-active、activation-group、agenda-group、auto-focus、ruleflow-group、when



1）salience：属性的值是一个数字，数字越大执行优先级越高，同时它的值可以是一个负数。默认情况下，规则的salience默认值为0，所以如果我们不手动设置规则的salience属性，那么它的执行顺序是随机的。



2）no-loop：属性的值是一个布尔型，默认值为false，如果no-loop属性值为true，那么就表示该规则只会被引擎检查一次，如果满足条件就执行规则的RHS部分。



3）date-effective：在规则运行时，引擎会自动拿当前操作系统的时间与date-effective设置的时间值进行比对，只有当系统时间>=date-effective设置的时间值时，规则才会触发执行，否则执行将不执行。默认日期格式：dd-MM-yyyy。可以通过代码设置它的格式：

System.setProperty("drools.dateformat", "yyyy-MM-dd HH:mm:ss");



4）date-expires：该属性的作用与date-effective属性恰恰相反，如果大于系统时间，那么规则就执行，否则就不执行。默认日期格式：dd-MM-yyyy



5）enabled：表示该规则是否可用。true表示可用，false表示不可用，默认值为true。



6）dialect：该属性用来定义规则当中要使用的语言类型：mvel 和java，如果没有手工设置规则的dialect，默认使用的java 语言。



7）duration：对于一个规则来说，如果设置了该属性，那么规则将在该属性指定的值之后在另外一个线程里触发。该属性对应的值为一个长整型，单位是毫秒，比如规则rule1 添加了duration 属性，它的值为3000，表示该规则将在3000 毫秒之后在另外一个线程里触发。



8）lock-on-active：当在规则上使用ruleflow-group 属性或agenda-group 属性的时候，将lock-on-action 属性的值设置为true，可能避免因某些Fact 对象被修改而使已经执行过的规则再次被激活执行。可以看出该属性与no-loop 属性有相似之处，no-loop 属性是为了避免Fact 修改或调用了insert、retract、update 之类而导致规则再次激活执行，这里的lock-on-action 属性也是起这个作用，lock-on-active 是no-loop 的增强版属性，它主要作用在使用ruleflow-group 属性或agenda-group 属性的时候。lock-on-active 属性默认值为false。



9）activation-group：该属性的作用是将若干个规则划分成一个组，用一个字符串来给这个组命名，这样在执行的时候，具有相同activation-group 属性的规则中只要有一个会被执行，其它的规则都将不再执行。也就是说，在一组具有相同activation-group 属性的规则当中，只有一个规则会被执行，其它规则都将不会被执行。当然对于具有相同activation-group 属性的规则当中究竟哪一个会先执行，则可以用类似salience 之类属性来实现。



10）agenda-group：规则的调用与执行是通过StatelessSession 或StatefulSession 来实现的，一般的顺序是创建一个StatelessSession 或StatefulSession，将各种经过编译的规则的package 添加到session当中，接下来将规则当中可能用到的Global 对象和Fact 对象插入到Session 当中，最后调用fireAllRules 方法来触发、执行规则。在没有调用最后一步fireAllRules 方法之前，所有的规则及插入的Fact 对象都存放在一个名叫Agenda 表的对象当中，这个Agenda 表中每一个规则及与其匹配相关业务数据叫做Activation，在调用fireAllRules 方法后，这些Activation 会依次执行，这些位于Agenda 表中的Activation 的执行顺序在没有设置相关用来控制顺序的属性时（比如salience 属性），它的执行顺序是随机的，不确定的。 Agenda Group 是用来在Agenda 的基础之上，对现在的规则进行再次分组，具体的分组方法可以采用为规则添加agenda-group 属性来实现。agenda-group 属性的值也是一个字符串，通过这个字符串，可以将规则分为若干个Agenda Group，默认情况下，引擎在调用这些设置了agenda-group 属性的规则的时候需要显示的指定某个Agenda Group 得到Focus（焦点），这样位于该Agenda Group 当中的规则才会触发执行，否则将不执行。



11）auto-focus：前面我们也提到auto-focus 属性，它的作用是用来在已设置了agenda-group 的规则上设置该规则是否可以自动独取Focus，如果该属性设置为true，那么在引擎执行时，就不需要显示的为某个Agenda Group 设置Focus，否则需要。

对于规则的执行的控制，还可以使用Agenda Filter 来实现。在Drools 当中，提供了一个名为org.drools.spi.AgendaFilter 的Agenda Filter 接口，用户可以实现该接口，通过规则当中的某些属性来控制规则要不要执行。org.drools.runtime.rule.AgendaFilter 接口只有一个方法需要实现。

public boolean accept(Activation activation); 

在该方法当中提供了一个Activation 参数，通过该参数我们可以得到当前正在执行的规则对象或其它一些属性，该方法要返回一个布尔值，该布尔值就决定了要不要执行当前这个规则，返回true 就执行规则，否则就不执行。用法请参照上面例子。



12）ruleflow-group：在使用规则流的时候要用到ruleflow-group属性，该属性的值为一个字符串，作用是用来将规则划分为一个个的组，然后在规则流当中通过使用ruleflow-group 属性的值，从而使用对应的规则。

### 3.2、函数

函数的编写位置可以是规则文件当中package 声明后的任何地方

function void/Object functionName(Type arg...) {    /*函数体的业务代码*/}

函数以function标记开头，可以有或无返回类型，然后定义方法名和参数，语法基本同java一致，不同规则文件的函数相互之间是不可见的。

Drools允许我们导入和使用java类的静态方法
格式：import function 类名.静态方法名
例子：import function com.demo.fun.RuleTools.printInfo

### 3.3、查询

查询是Drools 当中提供的一种根据条件在当前的WorkingMemory当中查找Fact 的方法，在Drools当中查询可分为两种：一种是不需要外部传入参数；一种是需要外部传入参数。

### 3.3.1、无参数查询

在Drools当中查询以query 关键字开始，以end 关键字结束，在package 当中一个查询要有唯一的名称，查询的内容就是查询的条件部分，条件部分内容的写法与规则的LHS 部分写法完全相同。

query "QUERY All MESSAGE"    $message : Message()end

查询的调用是由StatefulSession完成的，通过调用StatefulSession对象的getQueryResults(String queryName)方法实现对查询的调用，该方法的调用会返回一个QueryResults对象，QueryResults是一个类似于Collection接口的集合对象，在它当中存放在若干个QueryResultsRow对象，通过QueryResultsRow可以得到对应的Fact对象，从而实现根据条件对当前WorkingMemory当中Fact对象的查询。

QueryResults results = workingMemory.getQueryResults("QUERY ALL MESSAGE", null);        for ( Iterator it = results.iterator(); it.hasNext(); ) {            QueryResult result = ( QueryResult ) it.next();            Message m = ( Message ) result.get( "$message" );            System.out.println( m.getInfo() + "111\n" );        }
### 3.3.2、带参数查询

query "QUERY MESSAGE BY STATUS"(int $status)    $message : Message( status == $status )end

调用时带入参数

QueryResults results = workingMemory.getQueryResults("QUERY MESSAGE BY STATUS", Integer.valueOf(Message.GOODBYE));
### 3.4、Fact对象

在Drools中，除了向WorkingMemory当中插入Jave Bean作为Fact对象，也可以在规则文件里面定义

declare Addresscity : StringaddressName : Stringend



