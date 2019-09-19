---
layout: post
title: Jboss rules规则引擎 Drools 6.4.0 Final 教程(4)
category: drools
tags: [drools]
---



11.1.Drools 6.0的重要变化

随着Drools 6.0发布，Drools与Srping的集成经历了完整的改造。下面是一些非常重要的变化：

Drools在Spring中推荐的前缀从'drools:'变为'kie:'

6.0新的顶级标签：

kie:kmodule

kie:import（6.2新增）

kie:releaseId（6.2新增）

下面的标签不再是有效的顶级标签：

kie:kbase- kie:module的子标签

kie:ksession- kie:kbase的子标签

移除下面的5.X标签

drools:resources

drools:resource

drools:grid

drools:grid-node

11.2.与Drools Expert（Drools核心引擎）集成

下面的章节我们将解释kie命名空间

11.2.1\. KieModule

<kie:kmodule>定义KieBase集合和关联的KieSession。这个kmodule标签有一个强制参数"id"。

表 11.1.示例

| 

属性

 | 

描述

 | 

是否必需

 |
| 

id

 | 

Bean的 id 是其他Bean引用的名称。适用标准的Spring ID语义。

 | 

Yes

 |

一个kmodule标签只能包含以下子节点

*   kie:kbase

需要详细了解kmodule的话，请参阅Drools Expert的kmodule.xml的文档。

11.2.2\. KieBase

11.2.2.1<kie:base>的属性参数

表 11.2\. 示例

| 

属性

 | 

描述

 | 

是否必需

 |
| 

name

 | 

KieBase的名称

 | 

Yes

 |
| 

packages

 | 

资源包名，以"."号分隔

 | 

No

 |
| 

includes

 | 

被包含的kbase名称。一致的kbase资源都会被包括进来。

 | 

No

 |
| 

default

 | 

默认的kbase

 | 

No

 |
| 

default

 | 

布尔值 (TRUE/FALSE). 默认的kbase，如果未提供，它被假定为false。

 | 

No

 |
| 

scope

 | 

prototype | singleton。如果没有提供，默认为singleton

 | 

No

 |
| 

eventProcessingMode

 | 

事件处理模型. 有效的选项是STREAM, CLOUD

 | 

No

 |
| 

equalsBehavior

 | 

有效的选项是IDENTITY, EQUALITY

 | 

No

 |
| 

declarativeAgenda

 | 

有效的选项有enabled, disabled, true, false

 | 

No

 |

11.2.2.2kbase标签仅能包含下面子标签

*   kie:ksession

11.2.2.3.<kie:kbase>例子

一个kmodule标签可以包含多个kbase元素(1..n)

例子 11.1\. kbase定义例子

<kie:kmodule id="sample_module">  <kie:kbase name="kbase1" packages="org.drools.spring.sample"> ... </kie:kbase>  </kie:kmodule>

<bean id="kiePostProcessor"  class="org.kie.spring.KModuleBeanFactoryPostProcessor"/>

11.2.2.4Spring Bean作用范围（用于KieBase和KiesSession）

当定义一个KieBase或KieSession时，你需要声明Bean的作用域。举个例子，为了强制Spring每次请求时都产生一个新的实例，你需要声明Bean的scope属性为'prototype'。同样的，如果你想每次请求都返回相同的实例，你需要声明Bean的scope属性为'singleton'。

11.2.3.重要提示

为了能正确的实例化kmodule对象（kbase和ksession），需要强制定义一个类型为org.kie.spring.KModuleBeanFactoryPostProcessor或org.kie.spring.annotations.KModuleAnnotationPostProcessor的bean。

例子11.2\. 标准的kie-spring post processor bean定义

<bean id="kiePostProcessor"  class="org.kie.spring.KModuleBeanFactoryPostProcessor"/>

例子11.3\. kie-spring post processor bean 使用注解时的定义

<bean id="kiePostProcessor"  class="org.kie.spring.annotations.KModuleAnnotationPostProcessor"/>

注意：没有定义org.kie.spring.KModuleBeanFactoryPostProcessor或org.kie.spring.annotations.KModuleAnnotationPostProcessor时，kie-spring集成将不起作用。

11.2.4\. KieSessions

<kie:session>元素用于定义KieSessions。这个标签同样可以用于定义Stateful（org.kie.api.runtime.KieSession）和Stateless（org.kie.api.runtime.StatelessKieSession）sessions。

11.2.4.1<kie:ksession>的属性参数

表 11.3\. 示例

| 

属性

 | 

描述

 | 

是否必需

 |
| 

name

 | 

ksession的名称

 | 

Yes

 |
| 

type

 | 

是有状态（stateful）还无状态（stateless）会话？如果这个属性没有定义，默认是有状态的（Stateful）

 | 

No

 |
| 

default

 | 

这个是否是默认的ksession？

 | 

no

 |
| 

scope

 | 

prototype | singleton，如果没有定义默认是singleton（单例）

 | 

no

 |
| 

clockType

 | 

REALTIME or PSEUDO

 | 

no

 |
| 

listeners-ref

 | 

指向事件监听器组(参见下面的章节 <a style="color: #336699; text-decoration: none;">'Defining a Group of listeners'</a> 11.2.8.2).

 | 

no

 |

例子11.4\. 定义ksession

<kie:kmodule id="sample-kmodule">  <kie:kbase name="drl_kiesample3" packages="drl_kiesample3">  <kie:ksession name="ksession1" type="stateless"/>  <kie:ksession name="ksession2"/>  </kie:kbase>  </kie:kmodule>

<bean id="kiePostProcessor"  class="org.kie.spring.KModuleBeanFactoryPostProcessor"/>

11.2.4.2.Spring Bean作用范围（用于KieBase和KiesSession）（同11.2.2.4）

当定义一个KieBase或KieSession时，你需要声明Bean的作用域。举个例子，为了强制Spring每次请求时都产生一个新的实例，你需要声明Bean的scope属性为'prototype'。同样的，如果你想每次请求都返回相同的实例，你需要声明Bean的scope属性为'singleton'。

11.2.5.Kie:ReleaseId

11.2.5.1\. <kie:releaseId>属性参数

需要了解Maven及GAV，参见：

[http://blog.csdn.net/smallfish25/article/details/12420913](http://blog.csdn.net/smallfish25/article/details/12420913)

[http://blog.csdn.net/wanghantong/article/details/36427165](http://blog.csdn.net/wanghantong/article/details/36427165)

表11.4\. 示例

| 

属性

 | 

描述

 | 

是否必需

 |
| 

id

 | 

Bean的 id ，其他bean通过id引用此Bean。适用标准的Spring ID语义。

 | 

Yes

 |
| 

groupId

 | 

Maven GAV坐标的groupId

 | 

Yes

 |
| 

artifactId

 | 

Maven GAV坐标的artifactId

 | 

Yes

 |
| 

version

 | 

Maven GAV坐标的version

 | 

Yes

 |

例子11.5\. releaseId定义样例

<kie:releaseId id="beanId" groupId="org.kie.spring"  artifactId="named-artifactId" version="1.0.0-SNAPSHOT"/>

11.2.6.Kie:import

从6.2版本开始，kie-spring允许从classpath的kjars中引入kie对象。目前支持两种模式引入kie对象。

表11.5\.

| 

属性

 | 

描述

 | 

是否必需

 |
| 

releaseId

 | 

指向一个Bean ID. 适用标准的Spring ID语义。

 | 

No

 |
| 

enableScanner

 | 

是否允许Scanner. 当仅仅当'releaseId'显示定义的时候使用此属性

 | 

No

 |
| 

scannerInterval

 | 

间隔多少毫秒扫描一次。当且仅当'releaseId'显示定义的时候使用此属性

 | 

No

 |

11.2.6.1\. 全局引入

import标签将会强制自动扫描所有在classpath下的jar文件，初始化Kie对象（Kbases/KSessions），并引入到spring的上下文中。

表11.1\. 全局引入

<kie:import />

11.2.6.2\. 具体的引入 - ReleaseId

在import标签中使用 releaseId-ref 属性可以初始化我写的Kie对象 (Kbase/KSessions)并引入到srping上下文中。

表11.2\. Import Kie Objects using a releaseId

<kie:import releaseId-ref="namedKieSession"/>  <kie:releaseId id="namedKieSession" groupId="org.drools"  artifactId="named-kiesession" version="6.3.0-SNAPSHOT"/>  

当使用具体的引入时，Kie的自动扫描功能可以被启用。这个功能暂时不能在全局引入中使用。

表11.3\. Import Kie Objects using a releaseId -Enable Scanner

<kie:import releaseId-ref="namedKieSession"  enableScanner="true" scannerInterval="1000"/>

<kie:releaseId id="namedKieSession" groupId="org.drools"  artifactId="named-kiesession" version="6.3.0-SNAPSHOT"/>  

如果Scanner被定义并启用，一个隐式的KieScanner对象将被创建并插入到spring的上下文中。它可以从spring的上下文中获取。

表11.4\. Retriving the KieScanner from SpringContext

  _// theimplicit name would be releaseId#scanner_ KieScanner releaseIdScanner = context.getBean("namedKieSession#scanner",KieScanner.class);
releaseIdScanner.scanNow();

注意：kie-ci依赖包必须包括在classpath中，才能使用按releaseId引入的功能。

11.2.7.注解（与10.2部分内容相同或类似）

@KContainer, @KBase and @KSession都支持的可选的'name'属性. 当他们注入时，spring通常执行'get'。名称相同的一组注解，所有注入都会得到相同的实例。'name'标签会强制给每个名称一个唯一实例，尽管这个名称的所有的实例会在使用equals方法得到的结果是true。

11.2.7.1\. @KReleaseId

确定KieModule的某个版的实例被绑定。如果kie-ci在classpath中，依赖关系会被自动解决，并从远程仓库中下载。

11.2.7.2\. @KContainer

表11.5\. Injects Classpath KieContainer

@KContainer private KieContainer kContainer;

表11.6\. Injects KieContainer for Dynamic KieModule

@KContainer
@KReleaseId(groupId = "jar1", artifactId = "art1", version = "1.1") private KieContainer kContainer;

表11.7\. Injects named KieContainer for DynamicKieModule

@KContainer(name = "kc1")
@KReleaseId(groupId = "jar1", artifactId = "art1", version = "1.1") private KieContainer kContainer;

11.2.7.3\. @KBase

Thedefault argument, if given, maps to the value attribute and specifies the nameof the KieBase from the spring xml file.

如果给的话，默认参数是在spring xml中定义的KieBase的值属性和名称。

表11.8\. Injects the Default KieBase from theClasspath KieContainer

@KBase private KieBase kbase;

表11.9\. Injects the Default KieBase from a DynamicKieModule

@KBase
@KReleaseId( groupId = "jar1", artifactId = "art1", version = "1.0") private KieBase kbase;

表11.10\. Side by side version loading for'jar1.KBase1' KieBase

@KBase("kbase1")
@KReleaseId( groupId = "jar1", artifactId = "art1", version = "1.0") private KieBase kbase1v10;

@KBase("kbase1")
@KReleaseId( groupId = "jar1", artifactId = "art1", version = "1.1") private KieBase kbase1v11;

表11.11\. Side by side version loading for'jar1.ksession1' KieSession

@KSession("ksession1")
@KReleaseId( groupId = "jar1", artifactId = "art1", version = "1.0") private KieSession ksession11kb2;

@KSession("ksession1")
@KReleaseId( groupId = "jar1", artifactId = "art1", version = "1.1") private KieSession ksession11kb2;

11.2.7.4\. @KSession for KieSession

如果指定的话，默认参数是在spring xml中定义的KieSession的值属性和名称。

表11.12\. Injects the Default KieSession from theClasspath KieContainer

@KSession private KieSession ksession;

表11.13\. Injects the Default KieSession from aDynamic KieModule

@KSession
@KReleaseId( groupId = "jar1", artifactId = "art1", version = "1.0") private KieSession ksession;

表11.14\. Side by side version loading for'jar1.KBase1' KieBase

@KSession("ksession1")
@KReleaseId( groupId = "jar1", artifactId = "art1", version = "1.0") private KieSession ksessionv10;

@KSession("ksession1")
@KReleaseId( groupId = "jar1", artifactId = "art1", version = "1.1") private KieSession ksessionv11;

表11.15\. Use 'name' attribute to force newInstance for 'jar1.KBase1' KieSession

@KSession("ksession1")
@KReleaseId( groupId = "jar1", artifactId = "art1", version = "1.0") private KieSession ksession1ks1

@KSession("ksession1")
@KReleaseId( groupId = "jar1", artifactId = "art1", version = "1.0") private KieSession ksession1ks2

11.2.7.5\. @KSession for StatelessKieSession

如果指定的话，默认参数是在spring.xml中定义的KieSession的值属性和名称。

表11.16\. Injects the Default StatelessKieSessionfrom the Classpath KieContainer

@KSession private StatelessKieSession ksession;

表11.17\. Injects the Default StatelessKieSessionfrom a Dynamic KieModule

@KSession
@KReleaseId( groupId = "jar1", artifactId = "art1", version = "1.0") private StatelessKieSession ksession;

表11.18\. Side by side version loading for'jar1.KBase1' KieBase

@KSession("ksession1")
@KReleaseId( groupId = "jar1", rtifactId = "art1", version = "1.0") private StatelessKieSession ksessionv10;

@KSession("ksession1")
@KReleaseId( groupId = "jar1", rtifactId = "art1", version = "1.1") private StatelessKieSession ksessionv11;

表11.19\.

@KSession(value="ksession1", name="ks1")
@KReleaseId( groupId = "jar1", artifactId = "art1", version = "1.0") private StatelessKieSession ksession1ks1

@KSession(value="ksession1", name="ks2")
@KReleaseId( groupId = "jar1", artifactId = "art1", version = "1.0") private StatelessKieSession ksession1ks2

**11.2.7.6\.** 重要提示 **NOTE**

当使用注解的时候，为了能正确实例化kmodule对象（kbase/ksession），强制做下面二者之一：

1 定义一个类型为 org.kie.spring.annotations.KModuleAnnotationPostProcessor 的bean

2 启动spring 组件扫描。

下面代码段之一是必须的

例子11.6\. kie-spring annotations post processorbean definition

<bean id="kiePostProcessor"  class="org.kie.spring.annotations.KModuleAnnotationPostProcessor"/>

例子11.7\. kie-spring annotations - ComponentScanning

<context:component-scan base-package="org.kie.spring.annotations"/>

Note

The post processor is different when annotations are used.

11.2.8.事件监听器

Drools支持添加3种类型的KieSessions监听器——AgendaListener, WorkingMemoryListener, ProcessEventListener

kie-spring模块可以通过使用特殊的xml标签配置KieSessions监听器。这些标签作为实际的监听接口有独立的名称，例如<kie:agendaEventListener....>,<kie:ruleRuntimeEventListener....> 和 <kie:processEventListener....>

kie-spring可以将监听器独立定义，也可以按组定义。

11.2.8.1\. 定义独立的监听器:

11.2.8.1.1\. 属性:

Table 11.6. Sample

| 

Attribute

 | 

Required

 | 

Description

 |
| 

ref

 | 

No

 | 

另外一个定义好的bean的引用

 |

Example 11.8. Listener configuration example - using a bean:ref.

<bean id="mock-agenda-listener" class="mocks.MockAgendaEventListener"/>  <bean id="mock-rr-listener" class="mocks.MockRuleRuntimeEventListener"/>  <bean id="mock-process-listener" class="mocks.MockProcessEventListener"/>

<kie:kmodule id="listeners_kmodule">  <kie:kbase name="drl_kiesample" packages="drl_kiesample">  <kie:ksession name="ksession2">  <kie:agendaEventListener ref="mock-agenda-listener"/>  <kie:processEventListener ref="mock-process-listener"/>  <kie:ruleRuntimeEventListener ref="mock-rr-listener"/>  </kie:ksession>  </kie:kbase>  </kie:kmodule>

<bean id="kiePostProcessor"  class="org.kie.spring.KModuleBeanFactoryPostProcessor"/>

11.2.8.1.2\. 嵌套元素:

*   bean
    *   class = String
    *   name = String (可选)

Example 11.9. Listener configuration example - using nested bean.

<kie:kmodule id="listeners_module">  <kie:kbase name="drl_kiesample" packages="drl_kiesample">  <kie:ksession name="ksession1">  <kie:agendaEventListener>  <bean class="mocks.MockAgendaEventListener"/>  </kie:agendaEventListener>  </kie:ksession>  </kie:kbase>  </kie:kmodule>

<bean id="kiePostProcessor"  class="org.kie.spring.KModuleBeanFactoryPostProcessor"/>

**11.2.8.1.3\. 空标签 : 没有关联引用，也没有嵌套****bean****的声明**

当一个监听器没有引用一个实现的bean，也没有包含一个嵌套bean时，_<drools:ruleRuntimeEventListener/>_的底层实现会添加调试版本的监听器。此调试版监听器会打印交互事件（Event）的toString消息到System.err。

Example 11.10. Listener configuration example - defaulting to the debug versionsprovided by the Knowledge-API .

<bean id="mock-agenda-listener" class="mocks.MockAgendaEventListener"/>  <bean id="mock-rr-listener" class="mocks.MockRuleRuntimeEventListener"/>  <bean id="mock-process-listener" class="mocks.MockProcessEventListener"/>

<kie:kmodule id="listeners_module">  <kie:kbase name="drl_kiesample" packages="drl_kiesample">  <kie:ksession name="ksession2">  <kie:agendaEventListener />  <kie:processEventListener />  <kie:ruleRuntimeEventListener />  </kie:ksession>  </kie:kbase>  </kie:kmodule>

<bean id="kiePostProcessor"  class="org.kie.spring.KModuleBeanFactoryPostProcessor"/>

11.2.8.1.4\. 不同声明风格的混合与配合

Drools-spring模块允许在相同的KieSession中使用不同声明风格的混合与配合。下面是示例。

Example 11.11. Listener configuration example - mix and match of'ref'/nested-bean/empty styles.

<bean id="mock-agenda-listener" class="mocks.MockAgendaEventListener"/>  <bean id="mock-rr-listener" class="mocks.MockRuleRuntimeEventListener"/>  <bean id="mock-process-listener" class="mocks.MockProcessEventListener"/>

<kie:kmodule id="listeners_module">  <kie:kbase name="drl_kiesample" packages="drl_kiesample">  <kie:ksession name="ksession1">  <kie:agendaEventListener>  <bean class="org.kie.spring.mocks.MockAgendaEventListener"/>  </kie:agendaEventListener>  </kie:ksession>  <kie:ksession name="ksession2">  <kie:agendaEventListener ref="mock-agenda-listener"/>  <kie:processEventListener ref="mock-process-listener"/>  <kie:ruleRuntimeEventListener ref="mock-rr-listener"/>  </kie:ksession>  </kie:kbase>  </kie:kmodule>

<bean id="kiePostProcessor"  class="org.kie.spring.KModuleBeanFactoryPostProcessor"/>

11.2.8.1.5\. 同一个类型定义多个监听器

为一个KieSession定义多个类型相同的监听器也是有效的。

Example 11.12. Listener configuration example - multiple listeners of the sametype.

<bean id="mock-agenda-listener" class="mocks.MockAgendaEventListener"/>

<kie:kmodule id="listeners_module">  <kie:kbase name="drl_kiesample" packages="drl_kiesample">  <kie:ksession name="ksession1">  <kie:agendaEventListener ref="mock-agenda-listener"/>  <kie:agendaEventListener>  <bean class="org.kie.spring.mocks.MockAgendaEventListener"/>  </kie:agendaEventListener>  </kie:ksession>  </kie:kbase>  </kie:kmodule>

<bean id="kiePostProcessor"  class="org.kie.spring.KModuleBeanFactoryPostProcessor"/>

11.2.8.2\. 定义一组监听器

drools-spring允许为监听器分组。当你希望定义多个监听器且希望他们用于多个会话时，这个特别有用。分组功能也非常有用，当我们定义一组测试监听器并想在生产上使用它们时。

11.2.8.2.1\. 属性:

Table 11.7. Sample

| 

Attribute

 | 

Required

 | 

Description

 |
| 

ID

 | 

yes

 | 

Unique identifier

 |

11.2.8.2.2\. 包含的元素:

*   kie:agendaEventListener...
*   kie:ruleRuntimeEventListener...
*   kie:processEventListener...

注意：

上面提到的子元素可以以任意顺序进行声明。在同一个组中，每个类型最多只能有一个声明。

11.2.8.2.3\. Example:

Example 11.13\. Group of listeners - example

<bean id="mock-agenda-listener" class="mocks.MockAgendaEventListener"/>  <bean id="mock-rr-listener" class="mocks.MockRuleRuntimeEventListener"/>  <bean id="mock-process-listener" class="mocks.MockProcessEventListener"/>

<kie:kmodule id="listeners_module">  <kie:kbase name="drl_kiesample" packages="drl_kiesample">  <kie:ksession name="statelessWithGroupedListeners" type="stateless"  listeners-ref="debugListeners"/>  </kie:kbase>  </kie:kmodule>

<kie:eventListeners id="debugListeners">  <kie:agendaEventListener ref="mock-agenda-listener"/>  <kie:processEventListener ref="mock-process-listener"/>  <kie:ruleRuntimeEventListener ref="mock-rr-listener"/>  </kie:eventListeners>

<bean id="kiePostProcessor"  class="org.kie.spring.KModuleBeanFactoryPostProcessor"/>

未完待续

11.2.9.Loggers

11.2.10.内置的批处理命令

11.2.11.持久化

11.2.12.利用Spring的其他特点

11.3.与jBPM人工任务集成

11.3.1.如何配置Spring与jBPM人工任务
