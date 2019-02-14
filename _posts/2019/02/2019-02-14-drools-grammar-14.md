---
layout: post
title: Jboss rules规则引擎 Drools 6.4.0 Final 教程(4)
category: drools
tags: [drools]
---

4.1.概观

4.1.1.项目剖析

Drools和jBPM集成知识解决方案的研究过程中曾被简单称为“droolsjbpm”。这个名称渗透到GitHub的帐户和Maven的POM中。随着范围扩大和新项目发展，KIE作为一个新的名称，全称为Knowledgeis Everything。KIE名称也用于该系统的享方面；如统一构建，部署和利用。

KIE目前包括以下子项目：

Figure 4.1\. KIE 全貌

![KIE Anatomy](http://blog.csdn.net/buxulianxiang/article/details/45132689)

OptaPlanner，本地搜索和优化工具，已经从Drools Planner剥离出来，现在是Drools和jBPM的一个顶级项目。OptaPlanner是自然演变出来的，一直以来都相对Drools独立，又能与Drools非常强的整合在一起。

DashboardBuilder提供强大的报表能力，是从Polymita收购，同时收购的还有其他东西。Dashboard Builder目前是一个临时的名称，在6.0发布以后会重新选择名字。Dashboard Builder将完全独立与Drools和jBPM，并将用于JBoss的很多项目，也有可能用于JBoss之外的项目。

经过底层的重写，UberFire成为一个新的基础项目。UberFire提供一个类似Eclipse的工作台，由插件和视图组件。此项目同样独立于Drools和jBPM，任何人都可以使用它，用来构建一个灵活而强大的工作台。UberFire将会被用于控制台和工作台的开发，并贯穿于JBoss。

从Guvnor品牌计划的角色中泄漏了太多；以管理功能为例，就像决策表，正在考虑使用Guvnor组件，而不是Drools组件。5.x版本中的Guvnor对整体项目结构并没有帮助。在6.x时代，Guvnor的重点已经缩小到封装UberFire插件集，提供WEBIDE的基础。又如集成Maven用于构建和部署，通过邮箱管理Maven仓库和活动通知。Drools和jBPM基于UberFire构建工作台套件，并包括一系列插件，如Guvnor，连同他们插件包括决策表、向导编辑器、MPMN2设计器、人工任务等。Drools的工作台称为Drools-WB。KIE-WB是结合了Guvnor,Drools和jBPM插件的Uber工作台。jPBM-WB是虚构的，因为它实际上并不存在，被KIE-WB覆盖了。

4.1.2.生命周期

无论是Drools还是jBPM，KIE的工作内容或流程，通常可以细分为以下内容：

Author（管理，创建）

使用UI界面创建知识，比如DRL，BPMN2，决策表，类模型

Build（构建，编译）

将创建的知识编译成可部署单元。

对于KIE，这个单元是一个JAR。

Test（测试）

在部署到应用之前，测试KIE知识。

Deploy（部署）

将单元部署到一个地方，让应用程序可以使用（消耗）他们

Utilize（使用，利用）

加载一个JAR，提供一个KIE session（KieSession），这样的话，程序就可以与之交互

在运行时通过KIE [Container](http://lib.csdn.net/base/4 "Docker知识库")（KieContainer）解压JAR包。

用于运行时交互的KieSession，是从KieContainer中创建的。

Run（运行）

系统通过API与KieSession交互

Work（工作）

用户使用UI或命令行与KieSession交互

Manage（管理）

管理任何的KieSession或KieContainer

4.2.构建，部署，使用和运行

4.2.1.介绍

6.0引入新的配置和约定方法，用于构建知识基础，替代5.X中的使用的代码构建方式。构建者依然可以回退，因为它是用于工具集成。

现在使用Maven构建，用法与Maven实践一致。一个KIE项目或模块，同时也是一个Maven [Java](http://lib.csdn.net/base/17 "Java EE知识库")项目或模块；增加一个额外的元数据文件，名称为META-INF/kmodule.xml。这个kmodule.xml文件用于指定知识基础（knowledge base）对应的资源文件，并配置知识对应的base和session。这个配置也可以选择[spring](http://lib.csdn.net/base/17 "Java EE知识库")和OSCi BluePrints的XML。

标准的Maven可以构建并打包KIE资源，但并不在构建时校验。有一个Maven插件可以提供建构时的校验，建议使用。该插件还可以生成很多类，使得运行时加载快更快。

下面的截图，展示项目结构和Maven POM结构

Figure 4.2\. Example project layout and MavenPOM

![Example project layout and Maven POM](http://blog.csdn.net/buxulianxiang/article/details/45132689)

为减少配置，KIE使用很多默认配置。使用一个空的kmodule.xml文件是最简单的配置。一定要有一个kmodule.xml文件，即便是空的，因为它用于查找JAR和它的内容。

可以使用'mvninstall'命令部署KieModule到本地机器，这样本地的其他程序可以使用它。也可以使用'mvn deploy'命令将KieModule推送到远程的Maven仓库。构建程序会拉取KieModule并应用到本地Maven库。

Figure 4.3\. Example project layout and MavenPOM

![Example project layout and Maven POM](http://blog.csdn.net/buxulianxiang/article/details/45132689)

JAR文件有两种部署方式。其一是加入到classpath中，就像Maven的依赖列表的中其他JAR文件一样；其二是在运行时动态加载。KIE可以扫描所有包含kmodule.xml的JAR包。每个找到的JAR都用KieModule接口表示。术语Classpath KieModule和动态KieModule指代两种不同的加载方式。动态模块支持并行版本，classpath则不支持。而且一旦有一个模块在classpath中，就无法动态加载其他版本了。

下面是API的详细参考资料，着急的话可以直接跳到例子处，看了例子就明白是怎么回事了。

4.2.2.建筑

Figure 4.4\. org.kie.api.core.builder

![org.kie.api.core.builder](http://blog.csdn.net/buxulianxiang/article/details/45132689)

**4.2.2.1\. 创建并构建一个****KIE****项目**

一个KIE项目有着与普通Maven类似的结构，唯一的不同的是kmodule.xml文件，用于声明KieBase和KieSession。此文件被旋转在Maven项目的resources/META-INF目录，而其他的KIE文件，如DRL EXCEL文件，都必须保存在resources目录下，或它的任意子目录的中。

由于有意义的默认值涵盖所有的配置，所以最简单的kmodule.xml文件可以是一个空的kmodule标签，如下：

Example 4.1. An empty kmodule.xml file

**<?xml version="1.0"encoding="UTF-8"?>**  <kmodule xmlns="http://jboss.org/kie/6.0.0/kmodule"/>

这种方式下，kmodule将包含一个默认的KieBase。在resources目录及子目录下的所有元素都会被编译，并添加进来。创建一个KieContainer，就足以触发这些组件的构建。

Figure 4.5. KieContainer

![KieContainer](http://blog.csdn.net/buxulianxiang/article/details/45132689)

这种简单的情况下，下面的代码可以创建一个KieContainer，用来从classpath读取文件并构建：**Example** **4.2.** **Creating a KieContainer from the classpath**

KieServices kieServices =KieServices.Factory.get();
KieContainer kContainer = kieServices.getKieClasspathContainer();

KieServices是一个接口，可以用来访问几乎所有的关于KIE构建和运行时的工具。

Figure 4.6. KieServices

![KieServices](http://blog.csdn.net/buxulianxiang/article/details/45132689)

这种方式下，所有 的Java源代码和KIE资源都被编译并部署到KieContainer中。

4.2.2.2.kmodule.xml文件介绍

如前如所述，kmodule.xml文件用来声明式配置KieBase(s)和KieSession(s)的，这些KieBase(s)和KieSession(s)又可以从项目中创建。

特别要指出的是，KieBase是应用程序中所有的Knowledge定义的仓库。包括规则、流程、方法、类型模型。KieBase本身并不包含数据；想反的，从KieBase中创建的会话，既可以插入数据，也可以发起流程。创建KieBase很重，而创建会话很轻，所以建议缓存KieBase对象，这样可以重复创建KieSession。不过，最终用户通常不必担心此问题，因为KieContainer已经具备这个缓存机制。

Figure 4.7. KieBase

![KieBase](http://blog.csdn.net/buxulianxiang/article/details/45132689)

想反的，KieSession在运行时内存中保存并执行。它可以从KieBase中创建。还有一种更简单的方式是从KieContainer中创建，前提是它已在kmodule.xml文件中定义。

Figure 4.8. KieSession

![KieSession](http://blog.csdn.net/buxulianxiang/article/details/45132689)

kmodule.xml文件允许定义并配置一个或多个KieBase，不同的KieSession都可以从每个KieBase中创建出来。下面是例子：

Example 4.3. A sample kmodule.xml file

<kmodule xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  xmlns="http://jboss.org/kie/6.0.0/kmodule">  <configuration>  <property key="drools.evaluator.supersetOf" value="org.mycompany.SupersetOfEvaluatorDefinition"/>  </configuration>  <kbase name="KBase1" default="true" eventProcessingMode="cloud" equalsBehavior="equality"declarativeAgenda="enabled" packages="org.domain.pkg1">  <ksession name="KSession2_1" type="stateful" default="true"/>  <ksession name="KSession2_2" type="stateless" default="false" beliefSystem="jtms"/>  </kbase>  <kbase name="KBase2" default="false" eventProcessingMode="stream" equalsBehavior="equality"declarativeAgenda="enabled" packages="org.domain.pkg2, org.domain.pkg3" includes="KBase1">  <ksession name="KSession3_1" type="stateful" default="false" clockType="realtime">  <fileLogger file="drools.log" threaded="true" interval="10"/>  <workItemHandlers>  <workItemHandler name="name" type="org.domain.WorkItemHandler"/>  </workItemHandlers>  <listeners>  <ruleRuntimeEventListener type="org.domain.RuleRuntimeListener"/>  <agendaEventListener type="org.domain.FirstAgendaListener"/>  <agendaEventListener type="org.domain.SecondAgendaListener"/>  <processEventListener type="org.domain.ProcessListener"/>  </listeners>  </ksession>  </kbase>  </kmodule>

这里'<configuration>'是一些由'键-值对'组成的配置项，用来配置在KieBase创建过程中用到的配置项。这些配置项都是可选的。比如上面的例子中配置了一个superSetOf并通过org.mycompany.SupersetOfEvaluatorDefinition类来实现自定义操作。

上面定义了两个KieBase对象，前一个KieBase有两个 类型不同的KieSession，后面一个KieBase只有一个KieSession。通过标签可以定义一系列属性和对应的值，下表进行详细说明：

Table 4.1. kbase Attributes

| 

属性名称

 | 

默认值

 | 

有效值

 | 

含义

 |
| 

name

 | 

none

 | 

any

 | 

这是唯一必需的属性，用于从KieContainer获得KieBase

 |
| 

includes

 | 

none

 | 

任何','（逗号）分隔的列表

 | 

在当前kmodule.xml文件中定义的其他的KieBase的名称，若有多个，可以用';'分隔。当前KieBase会包含定义的这些其他KieBase。

 |
| 

packages

 | 

all

 | 

任何','（逗号）分隔的列表

 | 

默认情况下，在resources目录下的所有Drools文件，包括子目录，都会纳入KieBase。此属性可以指定哪些将要被KieBase编译Drools文件。每个package都是一个目录，多个package以','分隔。

 |
| 

default

 | 

false

 | 

true, false

 | 

如果KieBase被指定为缺省(true)，则可以直接从KieContainer中创建，而不需要指定name(名称)。每个模块中最多只有一个缺省的KieBase。

 |
| 

equalsBehavior

 | 

identity

 | 

identity, equality

 | 

定义当一个新的Fact（事实）插入到工作区时Drools的行为。使用identity时，总是创建一个新的FactHandle除非相同的对象已经不存在于工作区中；使用equality时，只有当新插入的对象与已经存在的对象不相等时（使用equals方法判断），才创建新的FactHandle。

简单的话，identity是通过对象地址来判断是否相等，而equality是通过判断对象值是否相等；只有不相等的时候才创建新的FactHandle。

 |
| 

eventProcessingMode

 | 

cloud

 | 

cloud, stream

 | 

当使用cloud模式编译时，KieBase把事件当做一个普通的Fact，使用stream模式时，可以基于事件做时间推理。

 |
| 

declarativeAgenda

 | 

disabled

 | 

disabled, enabled

 | 

定义是否声明的Agenda是否启用。Agenda译为议程，实际使用中起到过滤器的作用。

 |

同样的，ksession标签的所有属性都有有意义的默认值（当然name除外），详见下表：

Table 4.2. ksession Attributes

| 

属性名称

 | 

默认值

 | 

有效值

 | 

含义

 |
| 

name

 | 

none

 | 

any

 | 

这是唯一必需的属性，是KieSession的唯一名称，用于从KieContainer中获取。

 |
| 

type

 | 

stateful

 | 

stateful, stateless

 | 

一个stateful（有状态）会议可以多次与工作区交互；stateless（无状态）则基于工作区提供的数据一次性执行。

 |
| 

default

 | 

false

 | 

true, false

 | 

如果的默认的，则可以直接从KieContainer中创建，而不需要知道name。在一个模块中(kmodule.xml)，每种类型最多只能有一个默认的KieSession，分别对应有状态和无状态。

 |
| 

clockType

 | 

realtime

 | 

realtime, pseudo

 | 

定义事件的时间戳是由系统时间，还是由程序伪造的时间。这个配置项在时间规则的单元测试中非常有用。

 |
| 

beliefSystem

 | 

simple

 | 

simple, jtms, defeasible

 | 

定义KieSession的beliefSystem（信念系统？完全不知道是干什么的，用到的时候再补充）

 |

如之前的kmodule.xml例子，可以为每一个KieSession声明一个文件日志（或控制台日志），一个或多个WorkItemHandlers，还可以定义3种类型的监听器，分别是：ruleRuntimeEventListener,agendaEventListener and processEventListener。

根据之前例子定义好kmodule.xml文件后，就可以根据name从KieContainer中获取需要的KieBase和KieSession。

Example 4.4. Retriving KieBases and KieSessions from the KieContainer

KieServices kieServices =KieServices.Factory.get();
KieContainer kContainer = kieServices.getKieClasspathContainer();

KieBase kBase1 = kContainer.getKieBase("KBase1");
KieSession kieSession1 = kContainer.newKieSession("KSession2_1");
StatelessKieSession kieSession2 = kContainer.newStatelessKieSession("KSession2_2");

必须指出的是，KSession2_1和KSession2_2是两个不同类型的KieSession(一个有状态，一个无状态）。根据配置的不同，需要调用2种不同的方法来获得他们。如果请求的KieSession类型与kmodule.xml中配置的不相符，就发抛出一个RuntimeException。

此外，如果一个KieBase或KieSession声明为默认的（default为true），就可以不使用名称而获取他们。

Example 4.5. Retriving default KieBases and KieSessions from the KieContainer

KieContainer kContainer = ...

KieBase kBase1 = kContainer.getKieBase(); _// returns KBase1_ KieSession kieSession1 = kContainer.newKieSession(); _// returnsKSession2_1_

Kie项目同时也是Maven项目，在pom.xml中声明的groupId, artifactId和version（GAV）也可以生成一个唯一的ReleaseId，用于确定一个项目。这也允许传递一个ReleaseId参数给KieServices来创建一个新的KieContainer。

Example 4.6. Creating a KieContainer of an existing project by ReleaseId

KieServices kieServices =KieServices.Factory.get();
ReleaseId releaseId = kieServices.newReleaseId( "org.acme", "myartifact", "1.0" );
KieContainer kieContainer = kieServices.newKieContainer( releaseId );

**4.2.2.3\. 使用****Maven****构建**

Kie的Maven插件用来确保组件资源是被验证和编译，所以建议一直使用它。只需要将它添加到pom.xml的build模块即可使用此插件。

Example 4.7. Adding the KIE plugin to a Maven pom.xml

  <build>  <plugins>  <plugin>  <groupId>org.kie</groupId>  <artifactId>kie-maven-plugin</artifactId>  <version>${project.version}</version>  <extensions>true</extensions>  </plugin>  </plugins>  </build> 不使用Maven插件编译，就是复制所有的资源到一个打包好的JAR中。在运行时加载的时候，系统会尝试构建它们。如果编译有问题，会返回一个空的KieContainer（null）。这也会推高运行时编译开销。一般不建议这样做，应该和Maven插件一起使用。

**4.2.2.4\. 以编程方式定义一个****KieModule**

也可以使用编程方式定义KieModule及附属的KieBase和KieSession，而不是在kmodule.xml中定义它们。相同的编程API还可以显式添加任何包含Kie组件的文件，而不是自动从项目资源文件夹读取他们。首先建立一个KieFileSystem，一种虚拟的文件系统，并向其中添加资源。

Figure 4.9. KieFileSystem

![KieFileSystem](http://blog.csdn.net/buxulianxiang/article/details/45132689)

像Kie的其他组件一样，你也可以从KieServices类中获取KieFileSystem的实例。首先必须将kmodule.xml配置文件添加到这个虚拟的文件系统中。Kie提供一个方便流畅的API来创建kmodule.xml文件，由KieModuleModel类提供实现。

Figure 4.10. KieModuleModel

![KieModuleModel](http://blog.csdn.net/buxulianxiang/article/details/45132689)

实际上，你需要先用KieServices创建 KieModuleModel，再根据需要配置相应的KieBase和KieSession，并将它们转换成XML再添加到虚拟文件系统中。下面是例子：

Example 4.8. Creating a kmodule.xml programmatically and adding it to aKieFileSystem

KieServices kieServices =KieServices.Factory.get();
KieModuleModel kieModuleModel = kieServices.newKieModuleModel();

KieBaseModel kieBaseModel1 =kieModuleModel.newKieBaseModel( "KBase1 ")
.setDefault( true )
.setEqualsBehavior(EqualityBehaviorOption.EQUALITY )
.setEventProcessingMode(EventProcessingOption.STREAM );

KieSessionModel ksessionModel1 =kieBaseModel1.newKieSessionModel( "KSession1" )
.setDefault( true )
.setType(KieSessionModel.KieSessionType.STATEFUL )
.setClockType(ClockTypeOption.get("realtime") );

KieFileSystem kfs = kieServices.newKieFileSystem();

之后，需要通过API将其他的Kie组件添加到虚拟文件系统中。这个组件必须加到与对应的Maven项目相同的位置上。

Example 4.9. Adding Kie artifacts to a KieFileSystem

KieFileSystem kfs = ...
kfs.write( "src/main/resources/KBase1/ruleSet1.drl", stringContainingAValidDRL)
.write( "src/main/resources/dtable.xls",
kieServices.getResources().newInputStreamResource( dtableFileStream ) );

上面的例子说明，有两种方式添加组件，一种是使用简单的String对象，一种是使用Resource。后一种情况下，资源可以通过KieResource工作创建，API由KieServices提供。KieResource提供很多非常方便的工厂方法，可以接收InputStream, URL, 文件, 或表示文件路径的字符串，并将其转化为可以被KieFileSystem接收的Resource。

Figure 4.11. KieResources

![KieResources](http://blog.csdn.net/buxulianxiang/article/details/45132689)

通常情况下，在添加文件到KieFileSystem中时，可以从文件的后缀推断Resource的类型。然而，有时候后缀可能与文件类型无关。这时，就需要显式指定ResourceType，示例如下：

Example 4.10. Creating and adding a Resource with an explicit type

KieFileSystem kfs = ...
kfs.write( "src/main/resources/myDrl.txt",
kieServices.getResources().newInputStreamResource( drlStream)
.setResourceType(ResourceType.DRL) );

将所有的资源添加到KieFileSystem后，就可能使用KieBuilder编译它了。

Figure 4.12. KieBuilder

![KieBuilder](http://blog.csdn.net/buxulianxiang/article/details/45132689)

当KieFileSystem的内容被成功地构建后，产生的KieModule被自动添加到Kie库。该库是充当所有可用KieModules仓库里的单例。

Figure 4.13. KieRepository

![KieRepository](http://blog.csdn.net/buxulianxiang/article/details/45132689)

之后，可以通过使用KieServices为之前添加的KieMoudle创建一个新的KieContainer，需要使用它的ReleaseId。然而，这种情况下，KieFileSystem不包含的pom.xml文件（可以使用KieFileSystem.writePomXML文件创建一个），Kie并不知道它的ReleaseId，所以会赋给它一个默认的ReleaseId。这个默认的ReleaseId可以从KieModule中获取，并用来在KieRepository内部唯一标识这个KieModule。下面的例子说明整个过程：

Example 4.11. Building the contents of a KieFileSystem and creating a KieContainer

KieServices kieServices =KieServices.Factory.get();
KieFileSystem kfs = ...
kieServices.newKieBuilder( kfs ).buildAll();
KieContainer kieContainer =kieServices.newKieContainer(kieServices.getRepository().getDefaultReleaseId());

现在，你可以直接从KieContainer中获取KieBase和并创建新的KieSession，就像从classpath中获取KieContainer一样。

最佳实践提示要检查编译结果。KieBuilder会反馈3种级别的编译结果：ERROR, WARNING, INFO。ERROR表示项目编译失败，并且不会产生KieModule，也没有什么被添加到KieRepository。WARNING和INFO可以被忽略，但可供参考。

Example 4.12. Checking that a compilation didn't produce any error

KieBuilder kieBuilder = kieServices.newKieBuilder(kfs ).buildAll();
assertEquals( 0, kieBuilder.getResults().getMessages(Message.Level.ERROR ).size() );

4.2.2.5\. 更改编译结果的默认级别（Build ResultSeverity这个翻译的不好）

在某些情况下，有可能需要修改编译结果的默认级别。例如，当与现有规则同名的新规则被添加到一个包时，默认行为是使用新规则替代旧规则，并返回一个INFO级别消息。这种做法可能适合大多情况，但在某些部署用户那可能想要阻止规则更新，并返回一个ERROR。

改变编译结果的默认级别，就像配置Drools的其他选项一样，可以通过调用API，系统属性或配置文件来完成。从这个版本（6.2）起，Drools支持修改规则更新和方法更新的编译结果级别。要实现这一点，可以使用下面的属性：

Example 4.13\. Setting the severity usingproperties

// sets the severity of ruleupdates
drools.kbuilder.severity.duplicateRule = <INFO|WARNING|ERROR>
// sets the severity of functionupdates
drools.kbuilder.severity.duplicateFunction = <INFO|WARNING|ERROR>
