layout: post
title: Spring Boot 项目中的三种多数据源方案
category: arch
tags: [arch]
keywords: 架构
---
## **写在前面**

以下文章来源于[芋道源码](http://www.iocoder.cn/Spring-Boot/dynamic-datasource/)
> 
> 本文在提供完整代码示例，可见 https://github.com/YunaiV/SpringBoot-Labs 的 lab-19 目录。
> 
> 原创不易，给点个 Star 嘿，一起冲鸭！

## **正文**
 

> “
> 
> 摘要: 原创出处 http://www.iocoder.cn/Spring-Boot/dynamic-datasource/ 「芋道源码」欢迎转载，保留摘要，谢谢！

*   1\. 概述
*   2\. 实现方式

*   2.1 方案一
*   2.2 方案二
*   2.3 方案三

*   3\. baomidou 多数据源
*   4\. baomidou 读写分离
*   5\. MyBatis 多数据源
*   6\. Spring Data JPA 多数据源
*   7\. JdbcTemplate 多数据源
*   8\. Sharding-JDBC 多数据源
*   9\. Sharding-JDBC 读写分离
*   666\. 彩蛋

* * *

> “
> 
> 本文在提供完整代码示例，可见 https://github.com/YunaiV/SpringBoot-Labs 的 lab-19 目录。
> 
> 原创不易，给点个 Star 嘿，一起冲鸭！

# 1\. 概述

在项目中，我们可能会碰到需要多数据源的场景。例如说：

*   读写分离：数据库主节点压力比较大，需要增加从节点提供读操作，以减少压力。
*   多数据源：一个复杂的单体项目，因为没有拆分成不同的服务，需要连接**多个业务**的数据源。

本质上，读写分离，仅仅是多数据源的一个场景，从节点是只提供读操作的数据源。所以只要实现了多数据源的功能，也就能够提供读写分离。

# 2\. 实现方式

目前，实现多数据源有三种方案。我们逐个小节来看。

## 2.1 方案一

**基于 Spring AbstractRoutingDataSource 做拓展**。

简单来说，通过继承 AbstractRoutingDataSource 抽象类，实现一个管理项目中多个 DataSource 的**动态** DynamicRoutingDataSource 实现类。这样，Spring 在获取数据源时，可以通过 DynamicRoutingDataSource 返回**实际**的 DataSource 。

然后，我们可以自定义一个 `@DS` 注解，可以添加在 Service 方法、Dao 方法上，表示其**实际**对应的 DataSource 。

如此，整个过程就变成，执行数据操作时，通过“配置”的 `@DS` 注解，使用 DynamicRoutingDataSource 获得**对应的实际的** DataSource 。之后，在通过该 DataSource 获得 Connection 连接，最后发起数据库操作。

可能这么说，没有实现过多数据源的胖友会比较懵逼，比较大概率。所以推荐胖胖看看艿艿的基友写的 《剖析 Spring 多数据源》 文章。

不过呢，这种方式在结合 Spring 事务的时候，会存在无法切换数据源的问题。具体我们在 「3\. baomidou 多数据源」 中，结合示例一起来看。

艿艿目前找了一圈开源的项目，发现比较好的是 baomidou 提供的 `dynamic-datasource-spring-boot-starter` 。所以我们在 「3\. baomidou 多数据源」 和 「4\. baomidou 读写分离」 中，会使用到它。

## 2.2 方案二

**不同操作类，固定数据源**。

关于这个方案，解释起来略有点晦涩。以 MyBatis 举例子，假设有 `orders` 和 `users` 两个数据源。那么我们可以创建两个 SqlSessionTemplate `ordersSqlSessionTemplate` 和 `usersSqlSessionTemplate` ，分别使用这两个数据源。

然后，配置不同的 Mapper 使用不同的 SqlSessionTemplate 。

如此，整个过程就变成，执行数据操作时，通过 Mapper 可以对应到其  SqlSessionTemplate ，使用 SqlSessionTemplate 获得**对应的实际的** DataSource 。之后，在通过该 DataSource 获得 Connection 连接，最后发起数据库操作。

咳咳咳，是不是又处于懵逼状态了？！没事，咱在 「5\. MyBatis 多数据源」、「6\. Spring Data JPA 多数据源」、「7\. JdbcTemplate 多数据源」 中，结合案例一起看。「Talk is cheap. Show me the code」

不过呢，这种方式在结合 Spring 事务的时候，也会存在无法切换数据源的问题。淡定淡定。多数据源的情况下，这个基本是逃不掉的问题。

## 2.3 方案三

**分库分表中间件**。

对于分库分表的中间件，会解析我们编写的 SQL ，路由操作到对应的数据源。那么，它们天然就支持多数据源。如此，我们仅需配置好每个表对应的数据源，中间件就可以**透明**的实现多数据源或者读写分离。

目前，Java 最好用的分库分表中间件，就是 Apache ShardingSphere ，没有之一。

那么，这种方式在结合 Spring 事务的时候，会不会存在无法切换数据源的问题呢？答案是**不会**。在上述的方案一和方案二中，在 Spring 事务中，会获得对应的 DataSource ，再获得 Connection 进行数据库操作。而获得的 Connection 以及其上的事务，会通过 ThreadLocal 的方式和当前线程进行绑定。这样，就导致我们无法切换数据源。

难道分库分表中间件不也是需要 Connection 进行这些事情么？答案是的，但是不同的是分库分表中间件返回的 Connection 返回的实际是**动态**的 DynamicRoutingConnection ，它管理了整个请求（逻辑）过程中，使用的所有的 Connection ，而最终执行 SQL 的时候，DynamicRoutingConnection 会解析 SQL ，获得表对应的真正的 Connection 执行 SQL 操作。

难道方案一和方案二不可以这么做吗？答案是，当然可以。前提是，他们要实现解析 SQL 的能力。

那么，分库分表中间件就是多数据源的完美方案落？从一定程度上来说，是的。但是，它需要解决多个 Connection 可能产生的多个事务的一致性问题，也就是我们常说的，分布式事务。关于这块，艿艿最近有段时间没跟进 Sharding-JDBC 的版本，所以无法给出肯定的答案。不过我相信，Sharding-JDBC 最终会解决分布式事务的难题，提供**透明**的多数据源的功能。

在 「8\. Sharding-JDBC 多数据源」、「9\. Sharding-JDBC 读写分离」 中，我们会演示这种方案。

# 3\. baomidou 多数据源

> “
> 
> 示例代码对应仓库：lab-17-dynamic-datasource-baomidou-01 。

本小节，我们使用实现开源项目 `dynamic-datasource-spring-boot-starter` ，来实现多数据源的功能。我们会使用 `test_orders` 和 `test_users` 两个数据源作为两个数据源，然后实现在其上的 SQL 操作。并且，会结合在 Spring 事务的不同场景下，会发生的结果以及原因。

另外，关于 `dynamic-datasource-spring-boot-starter` 的介绍，胖友自己看 官方文档 。😈 它和 MyBatis-Plus 都是开发者 baomidou 提供的。

## 3.1 引入依赖

在 `pom.xml` 文件中，引入相关依赖。
    
    <code>
    <project xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <parent>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-parent</artifactId>
            <version>2.1.3.RELEASE</version>
            <relativePath/> <!-- lookup parent from repository -->
        </parent>
        <modelVersion>4.0.0</modelVersion>
    
        <artifactId>lab-17-dynamic-datasource-baomidou-01</artifactId>
    
        <dependencies>
            <!-- 实现对数据库连接池的自动化配置 -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-jdbc</artifactId>
            </dependency>
            <dependency> <!-- 本示例，我们使用 MySQL -->
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>5.1.48</version>
            </dependency>
    
            <!-- 实现对 MyBatis 的自动化配置 -->
            <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
                <version>2.1.1</version>
            </dependency>
    
            <!-- 实现对 dynamic-datasource 的自动化配置 -->
            <dependency>
                <groupId>com.baomidou</groupId>
                <artifactId>dynamic-datasource-spring-boot-starter</artifactId>
                <version>2.5.7</version>
            </dependency>
            <!-- 不造为啥 dynamic-datasource-spring-boot-starter 会依赖这个 -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-actuator</artifactId>
            </dependency>
    
            <!-- 方便等会写单元测试 -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
                <scope>test</scope>
            </dependency>
    
        </dependencies>
    
    </project></code> 

具体每个依赖的作用，胖友自己认真看下艿艿添加的所有注释噢。

## 3.2 Application

创建 `Application.java` 类，代码如下：

// Application.java

    @SpringBootApplication
    @MapperScan(basePackages = "cn.iocoder.springboot.lab17.dynamicdatasource.mapper")
    @EnableAspectJAutoProxy(exposeProxy = true) // http://www.voidcn.com/article/p-zddcuyii-bpt.html
    public class Application {
    }

*   添加 `@MapperScan` 注解，`cn.iocoder.springboot.lab17.dynamicdatasource.mapper` 包路径下，就是我们 Mapper 接口所在的包路径。
*   添加 `@EnableAspectJAutoProxy` 注解，重点是配置 `exposeProxy = true` ，因为我们希望 Spring AOP 能将当前代理对象设置到 AopContext 中。具体用途，我们会在下文看到。想要提前看的胖友，可以看看 《Spring AOP 通过获取代理对象实现事务切换》 文章。

## 3.3 应用配置文件

在 `resources` 目录下，创建 `application.yaml` 配置文件。配置如下：

    spring:
      datasource:
        # dynamic-datasource-spring-boot-starter 动态数据源的配置内容
        dynamic:
          primary: users # 设置默认的数据源或者数据源组，默认值即为 master
          datasource:
            # 订单 orders 数据源配置
            orders:
              url: jdbc:mysql://127.0.0.1:3306/test_orders?useSSL=false&useUnicode=true&characterEncoding=UTF-8
              driver-class-name: com.mysql.jdbc.Driver
              username: root
              password:
            # 用户 users 数据源配置
            users:
              url: jdbc:mysql://127.0.0.1:3306/test_users?useSSL=false&useUnicode=true&characterEncoding=UTF-8
              driver-class-name: com.mysql.jdbc.Driver
              username: root
              password:

# mybatis 配置内容
    
    mybatis:
      config-location: classpath:mybatis-config.xml # 配置 MyBatis 配置文件路径
      mapper-locations: classpath:mapper/*.xml # 配置 Mapper XML 地址
      type-aliases-package: cn.iocoder.springboot.lab17.dynamicdatasource.dataobject # 配置数据库实体包路径</code> 

*   `spring.datasource.dynamic` 配置项，设置 dynamic-`datasource-spring-boot-starter` 动态数据源的配置内容。

*   `primary` 配置项，设置默认的数据源或者数据源组，默认值即为 master 。
*   `datasource` 配置项，配置**每个**动态数据源。这里，我们配置了 `orders`、`users` 两个动态数据源。

*   `mybatis` 配置项，设置 `mybatis-spring-boot-starter` MyBatis 的配置内容。

## 3.4 MyBatis 配置文件

在 `resources` 目录下，创建 `mybatis-config.xml` 配置文件。配置如下：
    
    <code>
    <!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <configuration>
    
        <settings>
            <!-- 使用驼峰命名法转换字段。 -->
            <setting name="mapUnderscoreToCamelCase" value="true"/>
        </settings>
    
        <typeAliases>
            <typeAlias alias="Integer" type="java.lang.Integer"/>
            <typeAlias alias="Long" type="java.lang.Long"/>
            <typeAlias alias="HashMap" type="java.util.HashMap"/>
            <typeAlias alias="LinkedHashMap" type="java.util.LinkedHashMap"/>
            <typeAlias alias="ArrayList" type="java.util.ArrayList"/>
            <typeAlias alias="LinkedList" type="java.util.LinkedList"/>
        </typeAliases>
    
    </configuration></code> 

因为在数据库中的表的字段，我们是使用下划线风格，而数据库实体的字段使用驼峰风格，所以通过 `mapUnderscoreToCamelCase = true` 来自动转换。

## 3.5 实体类

在 `cn.iocoder.springboot.lab17.dynamicdatasource.dataobject` 包路径下，创建 `UserDO.java` 和 `OrderDO.java` 类。代码如下：
    
    // OrderDO.java
    /**
     * 订单 DO
     */
    public class OrderDO {
    
        /**
         * 订单编号
         */
        private Integer id;
        /**
         * 用户编号
         */
        private Integer userId;
    
        // 省略 setting/getting 方法
    
    }
    
    // UserDO.java
    /**
     * 用户 DO
     */
    public class UserDO {
    
        /**
         * 用户编号
         */
        private Integer id;
        /**
         * 账号
         */
        private String username;
    
        // 省略 setting/getting 方法
    } 

对应的创建表的 SQL 如下：

    -- 在 `test_orders` 库中。
    CREATE TABLE `orders` (
      `id` int(11) DEFAULT NULL COMMENT '订单编号',
      `user_id` int(16) DEFAULT NULL COMMENT '用户编号'
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin COMMENT='订单表';
    
    -- 在 `test_users` 库中。
    CREATE TABLE `users` (
      `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '用户编号',
      `username` varchar(64) COLLATE utf8mb4_bin DEFAULT NULL COMMENT '账号',
      `password` varchar(32) COLLATE utf8mb4_bin DEFAULT NULL COMMENT '密码',
      `create_time` datetime DEFAULT NULL COMMENT '创建时间',
      PRIMARY KEY (`id`),
      UNIQUE KEY `idx_username` (`username`)
    ) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin;
## 3.6 Mapper

在 `cn.iocoder.springboot.lab17.dynamicdatasource.mapper` 包路径下，创建 `UserDO.java` 和 `UserMapper.java` 接口。代码如下：

// OrderMapper.java
    
    @Repository
    @DS(DBConstants.DATASOURCE_ORDERS)
    public interface OrderMapper {
    
        OrderDO selectById(@Param("id") Integer id);
    
    }
    
    // UserMapper.java
    @Repository
    @DS(DBConstants.DATASOURCE_USERS)
    public interface UserMapper {
    
        UserDO selectById(@Param("id") Integer id);
    
    }

*   `DBConstants.java` 类，枚举了 `DATASOURCE_ORDERS` 和 `DATASOURCE_USERS` 两个数据源。
*   `@DS` 注解，是 `dynamic-datasource-spring-boot-starter` 提供，可添加在 Service 或 Mapper 的类/接口上，或者方法上。在其 `value` 属性种，填写数据源的名字。

*   OrderMapper 接口上，我们添加了 `@DS(DBConstants.DATASOURCE_ORDERS)` 注解，访问 `orders` 数据源。
*   UserMapper 接口上，我们添加了 `@DS(DBConstants.DATASOURCE_USERS)` 注解，访问 `users` 数据源。

*   为了让整个测试用例精简，我们在 OrderMapper 和 UserMapper 中，只添加了根据编号查询单条记录的方法。

在 `resources/mapper` 路径下，创建 `OrderMapper.xml` 和 `UserMapper.xml` 配置文件。代码如下：
    
    <code>
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="cn.iocoder.springboot.lab17.dynamicdatasource.mapper.OrderMapper">
    
        <sql id="FIELDS">
            id, user_id
        </sql>
    
        <select id="selectById" parameterType="Integer" resultType="OrderDO">
            SELECT
                <include refid="FIELDS" />
            FROM orders
            WHERE id = #{id}
        </select>
    
    </mapper></code>
    
    <!-- UserMapper.xml -->
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="cn.iocoder.springboot.lab17.dynamicdatasource.mapper.UserMapper">
    
        <sql id="FIELDS">
            id, username
        </sql>
    
        <select id="selectById" parameterType="Integer" resultType="UserDO">
            SELECT
                <include refid="FIELDS" />
            FROM users
            WHERE id = #{id}
        </select>
    
    </mapper></code> 
## 3.7 简单测试

创建 UserMapperTest 和 OrderMapperTest 测试类，我们来测试一下简单的 UserMapper 和 OrderMapper 的每个操作。代码如下：

// OrderMapperTest.java
    
    @RunWith(SpringRunner.class)
    @SpringBootTest(classes = Application.class)
    public class OrderMapperTest {
    
        @Autowired
        private OrderMapper orderMapper;
    
        @Test
        public void testSelectById() {
            OrderDO order = orderMapper.selectById(1);
            System.out.println(order);
        }
    
    }

// UserMapperTest.java
    
    @RunWith(SpringRunner.class)
    @SpringBootTest(classes = Application.class)
    public class UserMapperTest {
    
        @Autowired
        private UserMapper userMapper;
    
        @Test
        public void testSelectById() {
            UserDO user = userMapper.selectById(1);
            System.out.println(user);
        }
    
    }

胖友自己跑下测试用例。如果跑通，说明配置就算成功了。

## 3.8 详细测试

在本小节，我们会编写 5 个测试用例，尝试阐述 `dynamic-datasource-spring-boot-starter` 在和 Spring 事务结合碰到的情况，以便胖友更好的使用。当然，这个不仅仅是 `dynamic-datasource-spring-boot-starter` 独有的，而是方案一【基于 Spring AbstractRoutingDataSource 做拓展】都存在的情况。

在 `cn.iocoder.springboot.lab17.dynamicdatasource.service` 包路径下，创建 `OrderService.java` 类。代码如下：

// OrderService.java
    
    @Service
    public class OrderService {
    
        @Autowired
        private OrderMapper orderMapper;
        @Autowired
        private UserMapper userMapper;
    
        private OrderService self() {
            return (OrderService) AopContext.currentProxy();
        }
    
        public void method01() {
            // ... 省略代码
        }
    
        @Transactional
        public void method02() {
            // ... 省略代码
        }
    
        public void method03() {
            // ... 省略代码
        }
    
        public void method04() {
            // ... 省略代码
        }
    
        @Transactional
        @DS(DBConstants.DATASOURCE_ORDERS)
        public void method05() {
            // ... 省略代码
        }
    
    }

*   `#self()` 方法，通过 AopContext 获得自己这个代理对象。举个例子，在 `#method01()` 方法中，如果直接使用 `this.method02()` 方法进行调用，因为 `this` 代表的是 OrderService Bean 自身，而不是其 AOP 代理对象。这样会导致，无法触发 AOP 的逻辑，在此处，就是 Spring 事务的逻辑。因此，我们通过 AopContext 获得自己这个代理对象。
*   每一个 `#methodXX()` 方法，都代表一个测试用例，胖友可以使用 OrderServiceTest 进行测试。

下面，我们来一个一个看。

**场景一：`#method01()`**

// OrderService.java
    
    public void method01() {
        // 查询订单
        OrderDO order = orderMapper.selectById(1);
        System.out.println(order);
        // 查询用户
        UserDO user = userMapper.selectById(1);
        System.out.println(user);
    }

*   方法未使用 `@Transactional` 注解，不会开启事务。
*   对于 OrderMapper 和 UserMapper 的查询操作，分别使用其接口上的 `@DS` 注解，找到对应的数据源，执行操作。
*   这样一看，在未开启事务的情况下，我们已经能够自由的使用多数据源落。

**场景二：`#method02()`**

// OrderService.java
    
    @Transactional
    public void method02() {
        // 查询订单
        OrderDO order = orderMapper.selectById(1);
        System.out.println(order);
        // 查询用户
        UserDO user = userMapper.selectById(1);
        System.out.println(user);
    } 

*   和 `#method01()` 方法，差异在于，方法上增加了 `@Transactional` 注解，声明要使用 Spring 事务。

*   执行方法，抛出如下异常：


    NoUniqueBeanDefinitionException: No qualifying bean of type 'org.springframework.transaction.PlatformTransactionManager' available: expected single matching bean but found 2: ordersTransactionManager,usersTransactionManager

*   在执行 OrderMapper 查询订单操作时，抛出在 `test_users` 库中，不存在 `orders` 表。

*   这是为什么呢？咱不是在 OrderMapper 上，声明使用 `orders` 数据源了么？结果为什么会使用 `users` 数据库，路由到 `test_users` 库上呢。

*   这里，就和 Spring 事务的实现机制有关系。因为方法添加了 `@Transactional` 注解，Spring 事务就会生效。此时，Spring TransactionInterceptor 会通过 AOP 拦截该方法，创建事务。而创建事务，势必就会获得数据源。那么，TransactionInterceptor 会使用 Spring DataSourceTransactionManager 创建事务，并将事务信息通过 ThreadLocal 绑定在当前线程。
*   而事务信息，就包括事务对应的 Connection 连接。那也就意味着，还没走到 OrderMapper 的查询操作，Connection 就已经被创建出来了。并且，因为事务信息会和当前线程绑定在一起，在 OrderMapper 在查询操作需要获得 Connection 时，就直接拿到当前线程绑定的 Connection ，而不是 OrderMapper 添加 `@DS` 注解所对应的 DataSource 所对应的 Connection 。
*   OK ，那么我们现在可以把问题聚焦到 DataSourceTransactionManager 是怎么获取 DataSource 从而获得 Connection 的了。对于每个 DataSourceTransactionManager 数据库事务管理器，创建时都会传入其需要管理的 DataSource 数据源。在使用 `dynamic-datasource-spring-boot-starter` 时，它创建了一个 DynamicRoutingDataSource ，传入到 DataSourceTransactionManager 中。
*   而 DynamicRoutingDataSource 负责管理我们配置的多个数据源。例如说，本示例中就管理了 `orders`、`users` 两个数据源，并且默认使用 `users` 数据源。那么在当前场景下，DynamicRoutingDataSource 需要基于 `@DS` 获得数据源名，从而获得对应的 DataSource ，结果因为我们在 Service 方法上，并没有添加 `@DS` 注解，所以它只好返回默认数据源，也就是 `users` 。故此，就发生了 `Table 'test_users.orders' doesn't exist` 的异常。
*   咳咳咳，这里涉及 Spring 事务的实现机制，如果胖友不是很了解源码会比较懵逼，推荐可以尝试将 TransactionInterceptor 作为入口，进行调试。当然，也欢迎胖友给艿艿留言。

**场景三：`#method03()`**

// OrderService.java
    
    public void method03() {
        // 查询订单
        self().method031();
        // 查询用户
        self().method032();
    }
    
    @Transactional // 报错，因为此时获取的是 primary 对应的 DataSource ，即 users 。
    public void method031() {
        OrderDO order = orderMapper.selectById(1);
        System.out.println(order);
    }
    
    @Transactional
    public void method032() {
        UserDO user = userMapper.selectById(1);
        System.out.println(user);
    } 

*   执行方法，抛出如下异常：

    
        Caused by: com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: Table 'test_users.orders' doesn't exist
*   按照艿艿在场景二的解释，胖友可以思考下原因。

    > “
    > 
    > 😈 其实，场景三和场景二是等价的。

*   如果此时，我们将 `#self()` 代码替换成 `this` 之后，诶，结果就正常执行。这又是为什么呢？胖友在思考一波。

    > “
    > 
    > 😈 其实，这样调整后，因为 `this` 不是代理对象，所以 `#method031()` 和 `#method032()` 方法上的 `@Transactional` 直接没有作用，Spring 事务根本没有生效。所以，最终结果和场景一是等价的。

**场景四：`#method04()`**

// OrderService.java
    
    public void method04() {
        // 查询订单
        self().method041();
        // 查询用户
        self().method042();
    }
    
    @Transactional
    @DS(DBConstants.DATASOURCE_ORDERS)
    public void method041() {
        OrderDO order = orderMapper.selectById(1);
        System.out.println(order);
    }
    
    @Transactional
    @DS(DBConstants.DATASOURCE_USERS)
    public void method042() {
        UserDO user = userMapper.selectById(1);
        System.out.println(user);
    } 

*   和 `@method03()` 方法，差异在于，`#method041()` 和 `#method042()` 方法上，添加 `@DS` 注解，声明对应使用的 DataSource 。
*   执行方法，正常结束，未抛出异常。是不是觉得有点奇怪？
*   在执行 `#method041()` 方法前，因为有 `@Transactional` 注解，所以 Spring 事务机制触发。DynamicRoutingDataSource 根据 `@DS` 注解，获得对应的 `orders` 的 DataSource ，从而获得 Connection 。所以后续 OrderMapper 执行查询操作时，即使使用的是线程绑定的 Connection ，也可能不会报错。😈 嘿嘿，实际上，此时 OrderMapper 上的 `@DS` 注解，也没有作用。
*   对于 `#method042()` ，也是同理。但是，我们上面不是提了 Connection 会绑定在当前线程么？那么，在 `#method042()` 方法中，应该使用的是 `#method041()` 的 `orders` 对应的 Connection 呀。在 Spring 事务机制中，在一个事务执行完成后，会将事务信息和当前线程解绑。所以，在执行 `#method042()` 方法前，又可以执行一轮事务的逻辑。
*   【重要】总的来说，对于声明了 `@Transactional` 的 Service 方法上，也同时通过 `@DS` 声明对应的数据源。

**场景五：`#method05()`**

// OrderService.java
    
    @Transactional
    @DS(DBConstants.DATASOURCE_ORDERS)
    public void method05() {
        // 查询订单
        OrderDO order = orderMapper.selectById(1);
        System.out.println(order);
        // 查询用户
        self().method052();
    }
    
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    @DS(DBConstants.DATASOURCE_USERS)
    public void method052() {
        UserDO user = userMapper.selectById(1);
        System.out.println(user);
    } 

*   和 `@method04()` 方法，差异在于，我们直接在 `#method05()` 方法中，**此时处于一个事务中**，直接调用了 `#method052()` 方法。
*   执行方法，正常结束，未抛出异常。是不是觉得有点奇怪？
*   我们仔细看看 `#method052()` 方法，我们添加的 `@Transactionl` 注解，使用的事务传播级别是 `Propagation.REQUIRES_NEW` 。此时，在执行 `#method052()` 方法之前，TransactionInterceptor 会将原事务**挂起**，暂时性的将原事务信息和当前线程解绑。

*   所以，在执行 `#method052()` 方法前，又可以执行一轮事务的逻辑。
*   之后，在执行 `#method052()` 方法完成后，会将原事务**恢复**，重新将原事务信息和当前线程绑定。

*   编写这个场景的目的，是想告诉胖友，如果在使用方案一【基于 Spring AbstractRoutingDataSource 做拓展】，在事务中时，如何切换数据源。当然，一旦切换数据源，可能产生多个事务，就会碰到多个事务一致性的问题，也就是分布式事务。😈

😝 五个场景，胖友在好好理解。可以尝试调试下源码，更好的帮助理解。

咳咳咳，如果有解释不到位的地方，欢迎胖友给艿艿留言。

# 4\. baomidou 读写分离

> “
> 
> 示例代码对应仓库：lab-17-dynamic-datasource-baomidou-02 。

在绝大多数情况下，我们使用多数据源的目的，是为了实现读写分离。所以，在本小节中，我们来使用 `dynamic-datasource-spring-boot-starter` ，实现一个读写分离的示例。

## 4.1 引入依赖

和 「3.1 引入依赖」 一致。

## 4.2 Application

和 「3.2 Application」 一致。

## 4.3 应用配置文件

在 `resources` 目录下，创建 `application.yaml` 配置文件。配置如下：
    
    <code>spring:
      datasource:
        # dynamic-datasource-spring-boot-starter 动态数据源的配置内容
        dynamic:
          primary: master # 设置默认的数据源或者数据源组，默认值即为 master
          datasource:
            # 订单 orders 主库的数据源配置
            master:
              url: jdbc:mysql://127.0.0.1:3306/test_orders?useSSL=false&useUnicode=true&characterEncoding=UTF-8
              driver-class-name: com.mysql.jdbc.Driver
              username: root
              password:
            # 订单 orders 从库数据源配置
            slave_1:
              url: jdbc:mysql://127.0.0.1:3306/test_orders_01?useSSL=false&useUnicode=true&characterEncoding=UTF-8
              driver-class-name: com.mysql.jdbc.Driver
              username: root
              password:
            # 订单 orders 从库数据源配置
            slave_2:
              url: jdbc:mysql://127.0.0.1:3306/test_orders_02?useSSL=false&useUnicode=true&characterEncoding=UTF-8
              driver-class-name: com.mysql.jdbc.Driver
              username: root
              password:

# mybatis 配置内容
    
    mybatis:
      config-location: classpath:mybatis-config.xml # 配置 MyBatis 配置文件路径
      mapper-locations: classpath:mapper/*.xml # 配置 Mapper XML 地址
      type-aliases-package: cn.iocoder.springboot.lab17.dynamicdatasource.dataobject # 配置数据库实体包路径</code> 

*   相比 「3.3 应用配置」 来说，我们配置了订单库的多个数据源：

*   `master` ：订单库的主库。
*   `slave_1` 和 `slave_2` ：订单库的两个从库。

*   在 `dynamic-datasource-spring-boot-starter` 中，多个相同角色的数据源可以形成一个数据源组。判断标准是，数据源名以下划线 `_` 分隔后的**首部**即为组名。例如说，`slave_1` 和  `slave_2` 形成了 `slave` 组。

*   我们可以使用 `@DS("slave_1")` 或 `@DS("slave_2")` 注解，明确访问数据源组的指定数据源。
*   也可以使用 `@DS("slave")` 注解，此时会负载均衡，选择分组中的某个数据源进行访问。目前，负载均衡默认采用轮询的方式。

*   因为艿艿本地并未搭建 MySQL 一主多从的环境，所以是通过创建了 `test_orders_01`、`test_orders_02` 库，手动模拟作为 `test_orders` 的从库。

## 4.4 MyBatis 配置文件

和 「3.4 MyBatis 配置文件」 一致。

## 4.5 OrderDO

只使用 「3.5 实体类」 的 `OrderDO.java` 类。

## 4.6 OrderMapper

在 `cn.iocoder.springboot.lab17.dynamicdatasource.mapper` 包路径下，创建 `OrderMapper.java` 接口。代码如下：

// OrderMapper.java
    
    @Repository
    public interface OrderMapper {
    
        @DS(DBConstants.DATASOURCE_SLAVE)
        OrderDO selectById(@Param("id") Integer id);
    
        @DS(DBConstants.DATASOURCE_MASTER)
        int insert(OrderDO entity);
    
    } 

*   `DBConstants.java` 类，枚举了 `DATASOURCE_MASTER` 和 `DATASOURCE_SLAVE` 两个数据源。
*   对 `#selectById(Integer id)` 读操作，我们配置了 `@DS(DBConstants.DATASOURCE_SLAVE)` ，访问从库。
*   对 `#insert(OrderDO entity)` 写操作，我们配置了 `@DS(DBConstants.DATASOURCE_MASTER)` ，访问主库。

在 `resources/mapper` 路径下，创建 `OrderMapper.xml` 配置文件。代码如下：
    
    <code>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="cn.iocoder.springboot.lab17.dynamicdatasource.mapper.OrderMapper">
    
        <sql id="FIELDS">
            id, user_id
        </sql>
    
        <select id="selectById" parameterType="Integer" resultType="OrderDO">
            SELECT
                <include refid="FIELDS" />
            FROM orders
            WHERE id = #{id}
        </select>
    
        <insert id="insert" parameterType="OrderDO" useGeneratedKeys="true" keyProperty="id">
            INSERT INTO orders (
              user_id
            ) VALUES (
              #{userId}
            )
        </insert>
    
    </mapper></code> 
## 3.7 简单测试

创建 UserMapperTest 测试类，我们来测试一下简单的 UserMapper 的读写操作。代码如下：

// UserMapperTest.java
    
    @RunWith(SpringRunner.class)
    @SpringBootTest(classes = Application.class)
    public class OrderMapperTest {
    
        @Autowired
        private OrderMapper orderMapper;
    
        @Test
        public void testSelectById() {
            for (int i = 0; i < 10; i++) {
                OrderDO order = orderMapper.selectById(1);
                System.out.println(order);
            }
        }
    
        @Test
        public void testInsert() {
            OrderDO order = new OrderDO();
            order.setUserId(10);
            orderMapper.insert(order);
        }
    
    } 

胖友自己跑下测试用例。如果跑通，说明配置就算成功了。

另外，在 `#testSelectById()` 测试方法中，艿艿会了看看 `slave` 分组是不是真的在负载均衡。所以在数据库中，分别插入数据如下。
    
    主库：[id = 1, user_id = 1]
    从库 01：[id = 1, user_id = 2]
    从库 02：[id = 1, user_id = 3]

*   这样，通过手动设置相同 `id = 1` 的记录，对应不同的 `user_id` ，那么我们就可以观察 `#testSelectById()` 测试方法的输出结果。如果是，`user_id = 2` 和 `user_i = 3` 循环输出，说明就正常了。

## 3.8 详细测试

在 `cn.iocoder.springboot.lab17.dynamicdatasource.service` 包路径下，创建 `OrderService.java` 类。代码如下：

// OrderService.java
    
    @Service
    public class OrderService {
    
        @Autowired
        private OrderMapper orderMapper;
    
        @Transactional
        @DS(DBConstants.DATASOURCE_MASTER)
        public void add(OrderDO order) {
            // 这里先假模假样的读取一下
            orderMapper.selectById(order.getId());
    
            // 插入订单
            orderMapper.insert(order);
        }
    
        public OrderDO findById(Integer id) {
            return orderMapper.selectById(id);
        }
    
    } 

*   对于 `#add(OrderDO order)` 方法，我们希望在 `@Transactional` 声明的事务中，读操作也访问主库，所以声明了 `@DS(DBConstants.DATASOURCE_MASTER)` 。因此，后续的所有 OrderMapper 的操作，都访问的是订单库的 `MASTER` 数据源。
*   对于 `#findById(Integer id)` 方法，读取指定订单信息，使用 OrderMapper 的 `#selectById(Integer id)` 配置的 `SLAVE` 数据源即可。

创建 OrderServiceTest 测试类，测试 OrderService 的读写逻辑。代码如下：

// OrderServiceTest.java
    
    @RunWith(SpringRunner.class)
    @SpringBootTest(classes = Application.class)
    public class OrderServiceTest {
    
        @Autowired
        private OrderService orderService;
    
        @Test
        public void testAdd() {
            OrderDO order = new OrderDO();
            order.setUserId(20);
            orderService.add(order);
        }
    
        @Test
        public void testFindById() {
            OrderDO order = orderService.findById(1);
            System.out.println(order);
        }
    
    } 

*   胖友自己跑下测试用例。如果跑通，说明配置就算成功了。

另外，如果胖友的业务场景，是纯的读写分离，可以看看 《纯读写分离(mybatis 环境)》 文档。

# 5\. MyBatis 多数据源

> “
> 
> 示例代码对应仓库：lab-17-dynamic-datasource-mybatis 。

本小节，我们会基于方案二【不同操作类，固定数据源】的方式，实现 MyBatis 多数据源。

整个配置过程会相对繁琐，胖友请保持耐心。

如果胖友对 Spring Data JPA 不了解的话，可以看看 《芋道 Spring Boot MyBatis 入门》》 文章。

## 5.1 引入依赖

在 `pom.xml` 文件中，引入相关依赖。
    
    <code>
    <project xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <parent>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-parent</artifactId>
            <version>2.1.3.RELEASE</version>
            <relativePath/> <!-- lookup parent from repository -->
        </parent>
        <modelVersion>4.0.0</modelVersion>
    
        <artifactId>lab-17-dynamic-datasource-mybatis</artifactId>
    
        <dependencies>
            <!-- 实现对数据库连接池的自动化配置 -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-jdbc</artifactId>
            </dependency>
            <dependency> <!-- 本示例，我们使用 MySQL -->
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>5.1.48</version>
            </dependency>
    
            <!-- MyBatis 相关依赖 -->
            <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
                <version>2.1.1</version>
            </dependency>
    
            <!-- 保证 Spring AOP 相关的依赖包 -->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-aspects</artifactId>
            </dependency>
    
            <!-- 方便等会写单元测试 -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
                <scope>test</scope>
            </dependency>
    
        </dependencies>
    
    </project></code> 

*   具体每个依赖的作用，胖友自己认真看下艿艿添加的所有注释噢。
*   对于 `mybatis-spring-boot-starter` 依赖，这里并不使用它实现对 MyBatis 的自动化配置。这么引入，只是单纯方便，实际只要引入 `mybatis` 和 `mybatis-spring` 即可。

## 5.2 Application

创建 `Application.java` 类，代码如下：

    @SpringBootApplication
    @EnableAspectJAutoProxy(exposeProxy = true) // http://www.voidcn.com/article/p-zddcuyii-bpt.html
    public class Application {
    } 

*   我们并没有添加 `@MapperScan` 注解，为什么呢?答案我们在 「5.5 配置类」 上看。

## 5.3 应用配置文件

在 `resources` 目录下，创建 `application.yaml` 配置文件。配置如下：

    spring:
      # datasource 数据源配置内容
      datasource:
        # 订单数据源配置
        orders:
          jdbc-url: jdbc:mysql://127.0.0.1:3306/test_orders?useSSL=false&useUnicode=true&characterEncoding=UTF-8
          driver-class-name: com.mysql.jdbc.Driver
          username: root
          password:
        # 用户数据源配置
        users:
          jdbc-url: jdbc:mysql://127.0.0.1:3306/test_users?useSSL=false&useUnicode=true&characterEncoding=UTF-8
          driver-class-name: com.mysql.jdbc.Driver
          username: root
          password:

# mybatis 配置内容
    mybatis:
      config-location: classpath:mybatis-config.xml # 配置 MyBatis 配置文件路径
      type-aliases-package: cn.iocoder.springboot.lab17.dynamicdatasource.dataobject # 配置数据库实体包路径</code> 

*   在 `spring.datasource` 配置项中，我们设置了 `orders` 和 `users` 两个数据源。
*   注释掉 `mybatis` 配置项，因为我们不使用 `mybatis-spring-boot-starter` 自动化配置 MyBatis ，而是自己写配置类，自定义配置 MyBatis 。

## 5.4 MyBatis 配置文件

和 「3.4 MyBatis 配置文件」 一致。

## 5.5 MyBatis 配置类

在 `cn.iocoder.springboot.lab17.dynamicdatasource.config` 包路径下，我们会分别创建：

*   MyBatisOrdersConfig 配置类，配置使用 `orders` 数据源的 MyBatis 配置。
*   MyBatisUsersConfig 配置类，配置使用 `users` 数据源的 MyBatis 配置。

两个 MyBatis 配置类代码是一致的，只是部分配置项的值不同。所以我们仅仅来看下 MyBatisOrdersConfig 配置类，而 MyBatisUsersConfig 配置类胖友自己看看即可。代码如下：

// MyBatisOrdersConfig.java
    
    @Configuration
    @MapperScan(basePackages = "cn.iocoder.springboot.lab17.dynamicdatasource.mapper.orders", sqlSessionTemplateRef = "ordersSqlSessionTemplate")
    public class MyBatisOrdersConfig {
    
        /**
         * 创建 orders 数据源
         */
        @Bean(name = "ordersDataSource")
        @ConfigurationProperties(prefix = "spring.datasource.orders")
        public DataSource dataSource() {
            return DataSourceBuilder.create().build();
        }
    
        /**
         * 创建 MyBatis SqlSessionFactory
         */
        @Bean(name = "ordersSqlSessionFactory")
        public SqlSessionFactory sqlSessionFactory() throws Exception {
            SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
            // <2.1> 设置 orders 数据源
            bean.setDataSource(this.dataSource());
            // <2.2> 设置 entity 所在包
            bean.setTypeAliasesPackage("cn.iocoder.springboot.lab17.dynamicdatasource.dataobject");
            // <2.3> 设置 config 路径
            bean.setConfigLocation(new PathMatchingResourcePatternResolver().getResource("classpath:mybatis-config.xml"));
            // <2.4> 设置 mapper 路径
            bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mapper/orders/*.xml"));
            return bean.getObject();
        }
    
        /**
         * 创建 MyBatis SqlSessionTemplate
         */
        @Bean(name = "ordersSqlSessionTemplate")
        public SqlSessionTemplate sqlSessionTemplate() throws Exception {
            return new SqlSessionTemplate(this.sqlSessionFactory());
        }
    
        /**
         * 创建 orders 数据源的 TransactionManager 事务管理器
         */
        @Bean(name = DBConstants.TX_MANAGER_ORDERS)
        public PlatformTransactionManager transactionManager() {
            return new DataSourceTransactionManager(this.dataSource());
        }
    
    } 

*   `#dataSource()` 方法，创建 `orders` 数据源。
*   `#sqlSessionFactory()` 方法，创建 MyBatis SqlSessionFactory Bean 。

*   `<2.1>` 处，设置 `orders` 数据源。
*   `<2.2>` 处，设置 `entity` 所在包，作为类型别名。
*   `<2.3>` 处，设置 `config` 路径，这里我们使用 `classpath:mybatis-config.xml"` 配置文件。
*   `<2.4>` 处，设置 Mapper 路径，这里我们使用 `classpath:mapper/orders/*.xml` 。我们将 `resource/mapper` 路径下，拆分为 `orders` 路径下的 Mapper XML 用于 `orders` 数据源，`users` 路径下的 Mapper XML 用于 `users` 数据源。
*   通过上述设置，我们就创建出使用 `orders` 数据源的 SqlSessionFactory Bean 对象。

*   `#sqlSessionTemplate()` 方法，创建 MyBatis SqlSessionTemplate Bean 。其内部的 `sqlSessionFactory` 使用的就是对应 `orders` 数据源的 SqlSessionFactory 对象。
*   在类上，有 `@MapperScan` 注解：

*   配置 `basePackages` 属性，它会扫描 `cn.iocoder.springboot.lab17.dynamicdatasource.mapper` 包下的 `orders` 包下的 Mapper 接口。和 `resource/mapper` 路径一样，我们也将 `mapper` 包路径，拆分为 `orders` 包下的 Mapper 接口用于 `orders` 数据源，`users` 包下的 Mapper 接口用于 `users` 数据源。
*   配置 `sqlSessionTemplateRef` 属性，它会使用 `#sqlSessionTemplate()` 方法创建的 SqlSessionTemplate Bean 对象。
*   这样，我们就能保证 `cn.iocoder.springboot.lab17.dynamicdatasource.mapper.orders` 下的 Mapper 使用的是操作 `orders` 数据源的 SqlSessionFactory ，从而操作 `orders` 数据源。

*   `#transactionManager()` 方法，创建 `orders` 数据源的 Spring 事务管理器。因为，我们项目中，一般使用 Spring 管理事务。另外，我们在 `DBConstants.java` 枚举了 `TX_MANAGER_ORDERS` 和 `TX_MANAGER_USERS` 两个事务管理器的名字。

> “
> 
> 艿艿：相比来说，这种方式会相对繁琐。但是如果项目中大量采用，可以封装自己的 Spring Boot Starter ，以实现自动化配置。

## 5.6 实体类

和 「3.5 实体类」 一致。

## 5.7 Mapper

和 「3.6 Mapper」 基本一致，差别在于分出了 `orders` 和 `users` 两个。具体看如下两个传送门：

*   Mapper 接口
*   Mapper XML

## 5.8 简单测试

创建 UserMapperTest 和 OrderMapperTest 测试类，我们来测试一下简单的 UserMapper 和 OrderMapper 的每个操作。代码如下：
    
    // OrderMapperTest.java
    @RunWith(SpringRunner.class)
    @SpringBootTest(classes = Application.class)
    public class OrderMapperTest {
    
        @Autowired
        private OrderMapper orderMapper;
    
        @Test
        public void testSelectById() {
            OrderDO order = orderMapper.selectById(1);
            System.out.println(order);
        }
    
    }
    
    // UserMapperTest.java
    @RunWith(SpringRunner.class)
    @SpringBootTest(classes = Application.class)
    public class UserMapperTest {
    
        @Autowired
        private UserMapper userMapper;
    
        @Test
        public void testSelectById() {
            UserDO user = userMapper.selectById(1);
            System.out.println(user);
        }
    
    } 

胖友自己跑下测试用例。如果跑通，说明配置就算成功了。

## 5.9 详细测试

在本小节，我们会编写 4 个测试用例，尝试方案二【不同操作类，固定数据源】存在的情况。

在 `cn.iocoder.springboot.lab17.dynamicdatasource.service` 包路径下，创建 `OrderService.java` 类。代码如下：

// OrderService.java
    
    @Service
    public class OrderService {
    
        @Autowired
        private OrderMapper orderMapper;
        @Autowired
        private UserMapper userMapper;
    
        private OrderService self() {
            return (OrderService) AopContext.currentProxy();
        }
    
        public void method01() {
            // ... 省略代码
        }
    
        @Transactional // 报错，找不到事务管理器
        public void method02() {
            // ... 省略代码
        }
    
        public void method03() {
            // ... 省略代码
        }
    
        @Transactional(transactionManager = DBConstants.TX_MANAGER_ORDERS)
        public void method05() {
            // 查询订单
            OrderDO order = orderMapper.selectById(1);
            System.out.println(order);
            // 查询用户
            self().method052();
        }
    
    } 

*   每个测试场景，和 「3.8 详细测试」 的测试场景是相对应的，按照编号。
*   每一个 `#methodXX()` 方法，都代表一个测试用例，胖友可以使用 OrderServiceTest 进行测试。

下面，我们来一个一个看。

**场景一：`#method01()`**

// OrderService.java
    
    public void method01() {
        // 查询订单
        OrderDO order = orderMapper.selectById(1);
        System.out.println(order);
        // 查询用户
        UserDO user = userMapper.selectById(1);
        System.out.println(user);
    } 
    
    *   方法未使用 `@Transactional` 注解，不会开启事务。
    *   对于 OrderMapper 和 UserMapper 的查询操作，分别使用其接口对应的 SqlSessionTemplate ，找到对应的数据源，执行操作。
    *   这样一看，在未开启事务的情况下，我们已经能够自由的使用多数据源落。

**场景二：`#method02()`**

// OrderService.java
    
    @Transactional // 报错，找不到事务管理器
    public void method02() {
        // 查询订单
        OrderDO order = orderMapper.selectById(1);
        System.out.println(order);
        // 查询用户
        UserDO user = userMapper.selectById(1);
        System.out.println(user);
    } 

*   和 `#method02()` 方法，差异在于，方法上增加了 `@Transactional` 注解，声明要使用 Spring 事务。

*   执行方法，抛出如下异常：



*   在 `@Transactional` 注解上，如果未设置使用的事务管理器，它会去选择一个事务管理器。但是，我们这里创建了 `ordersTransactionManager` 和 `usersTransactionManager` 两个事务管理器，它就不知道怎么选了。此时，它只好抛出 NoUniqueBeanDefinitionException 异常。

**场景三：`#method03()`**
    
    // OrderService.java
    
    public void method03() {
        // 查询订单
        self().method031();
        // 查询用户
        self().method032();
    }
    
    @Transactional(transactionManager = DBConstants.TX_MANAGER_ORDERS)
    public void method031() {
        OrderDO order = orderMapper.selectById(1);
        System.out.println(order);
    }
    
    @Transactional(transactionManager = DBConstants.TX_MANAGER_USERS)
    public void method032() {
        UserDO user = userMapper.selectById(1);
        System.out.println(user);
    } 

*   执行方法，正常结束，未抛出异常。
*   `#method031()` 和 `#method032()` 方法上，声明的事务管理器，和后续 Mapper 操作是同一个 DataSource 数据源，从而保证不报错。

**场景四：`#method05()`**

// OrderService.java
    
    @Transactional(transactionManager = DBConstants.TX_MANAGER_ORDERS)
    public void method05() {
        // 查询订单
        OrderDO order = orderMapper.selectById(1);
        System.out.println(order);
        // 查询用户
        self().method052();
    }
    
    @Transactional(transactionManager = DBConstants.TX_MANAGER_USERS,
            propagation = Propagation.REQUIRES_NEW)
    public void method052() {
        UserDO user = userMapper.selectById(1);
        System.out.println(user);
    } 

*   执行方法，正常结束，未抛出异常。
*   我们仔细看看 `#method052()` 方法，我们添加的 `@Transactionl` 注解，使用的事务传播级别是 `Propagation.REQUIRES_NEW` 。此时，在执行 `#method052()` 方法之前，TransactionInterceptor 会将原事务**挂起**，暂时性的将原事务信息和当前线程解绑。

*   所以，在执行 `#method052()` 方法前，又可以执行一轮事务的逻辑。
*   之后，在执行 `#method052()` 方法完成后，会将原事务**恢复**，重新将原事务信息和当前线程绑定。

*   编写这个场景的目的，是想告诉胖友，如果在使用方案二【不同操作类，固定数据源】，在事务中时，如何切换数据源。当然，一旦切换数据源，可能产生多个事务，就会碰到多个事务一致性的问题，也就是分布式事务。😈

😝 四个场景，胖友在好好理解。可以尝试调试下源码，更好的帮助理解。

咳咳咳，如果有解释不到位的地方，欢迎胖友给艿艿留言。

## 5.10 读写分离

按照这个思路，如果想要实现 MyBatis 读写分离。还是类似的思路。只是将**从库**作为一个“**特殊**”的数据源，需要做的是：

*   应用配置文件增加**从库**的数据源。
*   增加一套**从库**的 MyBatis 配置类。
*   增加一套**从库**相关的 MyBatis Mapper 接口、Mapper XML 文件。

相比方案一【基于 Spring AbstractRoutingDataSource 做拓展】来说，更加麻烦。并且，万一有多从呢？嘿嘿。

所以呢，实际项目在选型时，方案一会**优**于方案二，被更普遍的采用。

# 6\. Spring Data JPA 多数据源

> “
> 
> 示例代码对应仓库：lab-17-dynamic-datasource-springdatajpa 。

本小节，我们会基于方案二【不同操作类，固定数据源】的方式，实现 Spring Data JPA 多数据源。

整个配置过程会相对繁琐，胖友请保持耐心。

> “
> 
> 艿艿：整个过程，和 「5\. MyBatis 多数据源」 是类似的，所以讲解会想对精简一些。
> 
> 内心 OS ：就是想偷懒，嘿嘿。

如果胖友对 Spring Data JPA 不了解的话，可以看看 《芋道 Spring Boot JPA 入门》》 文章。

## 6.1 引入依赖

在 `pom.xml` 文件中，引入相关依赖。
    
    <code>
    <project xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <parent>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-parent</artifactId>
            <version>2.1.3.RELEASE</version>
            <relativePath/> <!-- lookup parent from repository -->
        </parent>
        <modelVersion>4.0.0</modelVersion>
    
        <artifactId>lab-17-dynamic-datasource-springdatajpa</artifactId>
    
        <dependencies>
            <!-- 实现对数据库连接池的自动化配置 -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-jdbc</artifactId>
            </dependency>
            <dependency> <!-- 本示例，我们使用 MySQL -->
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>5.1.48</version>
            </dependency>
    
            <!-- JPA 相关依赖 -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-data-jpa</artifactId>
            </dependency>
    
            <!-- 方便等会写单元测试 -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
                <scope>test</scope>
            </dependency>
    
        </dependencies>
    
    </project></code> 

*   具体每个依赖的作用，胖友自己认真看下艿艿添加的所有注释噢。
*   对于 `spring-boot-starter-data-jpa` 依赖，这里并不使用它实现对 JPA 的自动化配置。这么引入，只是单纯方便，不然需要引入 `spring-data-jpa` 和 `hibernate-core` 等等依赖。

## 6.2 Application

创建 `Application.java` 类，代码如下：

    @SpringBootApplication
    @EnableAspectJAutoProxy(exposeProxy = true) // http://www.voidcn.com/article/p-zddcuyii-bpt.html
    public class Application {
    } 
## 6.3 应用配置文件

在 `resources` 目录下，创建 `application.yaml` 配置文件。配置如下：

    spring:
      # datasource 数据源配置内容
      datasource:
        # 订单数据源配置
        orders:
          jdbc-url: jdbc:mysql://127.0.0.1:3306/test_orders?useSSL=false&useUnicode=true&characterEncoding=UTF-8
          driver-class-name: com.mysql.jdbc.Driver
          username: root
          password:
        # 用户数据源配置
        users:
          jdbc-url: jdbc:mysql://127.0.0.1:3306/test_users?useSSL=false&useUnicode=true&characterEncoding=UTF-8
          driver-class-name: com.mysql.jdbc.Driver
          username: root
          password:
      jpa:
        show-sql: true # 打印 SQL 。生产环境，建议关闭
        # Hibernate 配置内容，对应 HibernateProperties 类
        hibernate:
          ddl-auto: none 

*   在 `spring.datasource` 配置项中，我们设置了 `orders` 和 `users` 两个数据源。

## 6.4 Spring Data JPA 配置类

在 `cn.iocoder.springboot.lab17.dynamicdatasource.config` 包路径下，创建 `HibernateConfig.java` 配置类。代码如下：

// HibernateConfig.java
    
    @Configuration
    public class HibernateConfig {
    
        @Autowired
        private JpaProperties jpaProperties;
        @Autowired
        private HibernateProperties hibernateProperties;
    
        /**
         * 获取 Hibernate Vendor 相关配置
         */
        @Bean(name = "hibernateVendorProperties")
        public Map<String, Object> hibernateVendorProperties() {
            return hibernateProperties.determineHibernateProperties(
                    jpaProperties.getProperties(), new HibernateSettings());
        }
    
    } 

*   目的是获得 Hibernate Vendor 相关配置。不用纠结它是什么，知道需要获得即可。

在 `cn.iocoder.springboot.lab17.dynamicdatasource.config` 包路径下，我们会分别创建：

*   JpaOrdersConfig 配置类，配置使用 `orders` 数据源的 Spring Data JPA 配置。
*   JpaUsersConfig 配置类，配置使用 `users` 数据源的 Spring Data JPA 配置。

两个 Spring Data JPA 配置类代码是一致的，只是部分配置项的值不同。所以我们仅仅来看下 JpaOrdersConfig 配置类，而 JpaUsersConfig 配置类胖友自己看看即可。代码如下：

// JpaOrdersConfig.java
    
    @Configuration
    @EnableJpaRepositories(
            entityManagerFactoryRef = DBConstants.ENTITY_MANAGER_FACTORY_ORDERS,
            transactionManagerRef = DBConstants.TX_MANAGER_ORDERS,
            basePackages = {"cn.iocoder.springboot.lab17.dynamicdatasource.repository.orders"}) // 设置 Repository 接口所在包
    public class JpaOrdersConfig {
    
        @Resource(name = "hibernateVendorProperties")
        private Map<String, Object> hibernateVendorProperties;
    
        /**
         * 创建 orders 数据源
         */
        @Bean(name = "ordersDataSource")
        @ConfigurationProperties(prefix = "spring.datasource.orders")
        @Primary // 需要特殊添加，否则初始化会有问题
        public DataSource dataSource() {
            return DataSourceBuilder.create().build();
        }
    
        /**
         * 创建 LocalContainerEntityManagerFactoryBean
         */
        @Bean(name = DBConstants.ENTITY_MANAGER_FACTORY_ORDERS)
        public LocalContainerEntityManagerFactoryBean entityManagerFactory(EntityManagerFactoryBuilder builder) {
            return builder
                    .dataSource(this.dataSource()) // 数据源
                    .properties(hibernateVendorProperties) // 获取并注入 Hibernate Vendor 相关配置
                    .packages("cn.iocoder.springboot.lab17.dynamicdatasource.dataobject") // 数据库实体 entity 所在包
                    .persistenceUnit("ordersPersistenceUnit") // 设置持久单元的名字，需要唯一
                    .build();
        }
    
        /**
         * 创建 PlatformTransactionManager
         */
        @Bean(name = DBConstants.TX_MANAGER_ORDERS)
        public PlatformTransactionManager transactionManager(EntityManagerFactoryBuilder builder) {
            return new JpaTransactionManager(entityManagerFactory(builder).getObject());
        }
    
    } 

*   `#dataSource()` 方法，创建 `orders` 数据源。
*   `#entityManagerFactoryPrimary(EntityManagerFactoryBuilder builder)` 方法，创建 LocalContainerEntityManagerFactoryBean Bean ，它是创建 EntityManager 实体管理器的工厂 Bean ，**最终会创建对应的 EntityManager Bean** 。

*   `<2.1>` 处，设置使用的数据源是 `orders` 。
*   `<2.2>` 处，设置 Hibernate Vendor 相关配置。
*   `<2.3>` 处，设置数据库实体 Entity 所在包。
*   `<2.4>` 处，设置持久单元的名字，需要唯一。

*   `#transactionManager(EntityManagerFactoryBuilder builder)` 方法，创建使用上述 EntityManager 的 JpaTransactionManager Bean 对象。这样，该事务管理器使用的也是 `orders` 数据源。
*   最终，通过 `@EnableJpaRepositories` 注解，串联在一起：

*   `entityManagerFactoryRef` 属性，保证了使用 `orders` 数据源的 EntityManager 实体管理器的工厂 Bean 。
*   `transactionManagerRef` 属性，保证了使用 `orders` 数据源的 PlatformTransactionManager 事务管理器 Bean 。
*   `basePackages` 属性，它会扫描 `cn.iocoder.springboot.lab17.dynamicdatasource.repository` 包下的 `orders` 包下的 Repository 接口。我们将 `repository` 包路径，拆分为 `orders` 包下的 Repository 接口用于 `orders` 数据源，`users` 包下的 Repository 接口用于 `users` 数据源。

*   另外，我们在 DBConstants.java 类中，枚举了：

*   `TX_MANAGER_ORDERS` 和 `TX_MANAGER_USERS` 两个事务管理器的名字，方便代码中使用。
*   `ENTITY_MANAGER_FACTORY_ORDERS` 和 `ENTITY_MANAGER_FACTORY_USERS` 两个实体管理器的名字。

> “
> 
> 艿艿：相比来说，这种方式会相对繁琐。但是如果项目中大量采用，可以封装自己的 Spring Boot Starter ，以实现自动化配置。

## 6.5 实体类

和 「3.5 实体类」 基本一致，差别在于增加了 JPA 相关注解。具体看如下两个传送门：

*   `OrderDO.java`
*   `UserDO.java`

## 6.6 Repository

和 「3.6 Mapper」 基本一致，差别在于使用 Spring Data Repository 接口。具体看如下两个传送门：

*   OrderRepository
*   UserRepository

## 6.7 简单测试

和 「5.8 简单测试」 基本一致，具体看如下两个传送门：

*   OrderRepositoryTest
*   UserRepositoryTest

## 6.8 详细测试

和 「5.9 详细测试」 基本一致，具体看如下两个传送门：

*   OrderService
*   OrderServiceTest

## 6.9 读写分离

和 「5.10 读写分离」 **思路**基本一致。

# 7\. JdbcTemplate 多数据源

> “
> 
> 示例代码对应仓库：lab-17-dynamic-datasource-jdbctemplate 。

本小节，我们会基于方案二【不同操作类，固定数据源】的方式，实现 Spring JdbcTemplate 多数据源。

整个配置过程会相对繁琐，胖友请保持耐心。

> “
> 
> 艿艿：整个过程，和 「5\. MyBatis 多数据源」 是类似的，所以讲解会想对精简一些。
> 
> 内心 OS ：我只是想赶紧进入 Sharding-JDBC 的环节，真的不是想偷懒，哈哈哈哈。

如果胖友对 Spring JdbcTemplate 不了解的话，可以看看 《芋道 Spring Boot JdbcTemplate 入门》》 文章。

## 7.1 引入依赖

在 `pom.xml` 文件中，引入相关依赖。
    
    <code>
    <project xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <parent>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-parent</artifactId>
            <version>2.1.3.RELEASE</version>
            <relativePath/> <!-- lookup parent from repository -->
        </parent>
        <modelVersion>4.0.0</modelVersion>
    
        <artifactId>lab-17-dynamic-datasource-jdbctemplate</artifactId>
    
        <dependencies>
            <!-- 实现对数据库连接池的自动化配置 -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-jdbc</artifactId>
            </dependency>
            <dependency> <!-- 本示例，我们使用 MySQL -->
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>5.1.48</version>
            </dependency>
    
            <!-- 保证 Spring AOP 相关的依赖包 -->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-aspects</artifactId>
            </dependency>
    
            <!-- 方便等会写单元测试 -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
                <scope>test</scope>
            </dependency>
        </dependencies>
    
    </project></code> 

具体每个依赖的作用，胖友自己认真看下艿艿添加的所有注释噢。

## 7.2 Application

创建 `Application.java` 类，代码如下：

    @SpringBootApplication
    @EnableAspectJAutoProxy(exposeProxy = true) // http://www.voidcn.com/article/p-zddcuyii-bpt.html
    public class Application {
    } 
## 7.3 应用配置文件

在 `resources` 目录下，创建 `application.yaml` 配置文件。配置如下：

    spring:
      # datasource 数据源配置内容
      datasource:
        # 订单数据源配置
        orders:
          jdbc-url: jdbc:mysql://127.0.0.1:3306/test_orders?useSSL=false&useUnicode=true&characterEncoding=UTF-8
          driver-class-name: com.mysql.jdbc.Driver
          username: root
          password:
        # 用户数据源配置
        users:
          jdbc-url: jdbc:mysql://127.0.0.1:3306/test_users?useSSL=false&useUnicode=true&characterEncoding=UTF-8
          driver-class-name: com.mysql.jdbc.Driver
          username: root
          password:

*   在 `spring.datasource` 配置项中，我们设置了 `orders` 和 `users` 两个数据源。

## 7.4 JdbcTemplate 配置类

在 `cn.iocoder.springboot.lab17.dynamicdatasource.config` 包路径下，我们会分别创建：

*   JdbcTemplateOrdersConfig 配置类，配置使用 `orders` 数据源的 MyBatis 配置。
*   JdbcTemplateUsersConfig 配置类，配置使用 `users` 数据源的 MyBatis 配置。

两个 JdbcTemplate 配置类代码是一致的，只是部分配置项的值不同。所以我们仅仅来看下 JdbcTemplateOrdersConfig 配置类，而 JdbcTemplateUsersConfig 配置类胖友自己看看即可。代码如下：

// JdbcTemplateOrdersConfig.java
    
    @Configuration
    public class JdbcTemplateOrdersConfig {
    
        /**
         * 创建 orders 数据源
         */
        @Bean(name = "ordersDataSource")
        @ConfigurationProperties(prefix = "spring.datasource.orders")
        public DataSource dataSource() {
            return DataSourceBuilder.create().build();
        }
    
        /**
         * 创建 orders JdbcTemplate
         */
        @Bean(name = DBConstants.JDBC_TEMPLATE_ORDERS)
        public JdbcTemplate jdbcTemplate() {
            return new JdbcTemplate(this.dataSource());
        }
    
        /**
         * 创建 orders 数据源的 TransactionManager 事务管理器
         */
        @Bean(name = DBConstants.TX_MANAGER_ORDERS)
        public PlatformTransactionManager transactionManager() {
            return new DataSourceTransactionManager(this.dataSource());
        }
    
    } 

*   `#dataSource()` 方法，创建 `orders` 数据源。
*   `#jdbcTemplate()` 方法，创建使用 `orders` 数据源的 JdbcTemplate Bean 。
*   `#transactionManager()` 方法，创建 `orders` 数据源的 Spring 事务管理器。因为，我们项目中，一般使用 Spring 管理事务。另外，我们在 `DBConstants.java` 枚举了 `TX_MANAGER_ORDERS` 和 `TX_MANAGER_USERS` 两个事务管理器的名字。

> “
> 
> 艿艿：相比来说，这种方式会相对繁琐。但是如果项目中大量采用，可以封装自己的 Spring Boot Starter ，
>以实现自动化配置。

## 7.5 实体类

和 「3.5 实体类」 一致。

## 7.6 Dao

和 「5.8 简单测试」 基本一致，具体看如下两个传送门：

*   OrderDao
*   UserDao

## 7.7 简单测试

和 「5.8 简单测试」 基本一致，具体看如下两个传送门：

*   OrderDaoTest
*   UserDaoTest

## 7.8 详细测试

和 「5.9 详细测试」 基本一致，具体看如下两个传送门：

*   OrderService
*   OrderServiceTest

## 7.9 读写分离

和 「5.10 读写分离」 **思路**基本一致。

# 8\. Sharding-JDBC 多数据源

> “
> 
> 示例代码对应仓库：lab-17-dynamic-datasource-sharding-jdbc-01 。

Sharding-JDBC 是 Apache ShardingSphere 下，基于 JDBC 的分库分表组件。对于 Java 语言来说，
我们推荐选择 Sharding-JDBC 优于 Sharding-Proxy ，主要原因是：

*   减少一层 Proxy 的开销，性能更优。
*   去中心化，无需多考虑一次 Proxy 的高可用。

下面，我们来使用 Sharding-JDBC 来实现多数据源。整个的示例，我们会和 「2\. baomidou 多数据源」 是一样的功能，
方便胖友做类比。

## 8.1 引入依赖

在 `pom.xml` 文件中，引入相关依赖。
    
    <code>
    <project xmlns="http://maven.apache.org/POM/4.0.0"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        <parent>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-parent</artifactId>
            <version>2.1.3.RELEASE</version>
            <relativePath/> <!-- lookup parent from repository -->
        </parent>
        <modelVersion>4.0.0</modelVersion>
    
        <artifactId>lab-17-dynamic-datasource-sharding-jdbc-01</artifactId>
    
        <dependencies>
            <!-- 实现对数据库连接池的自动化配置 -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-jdbc</artifactId>
            </dependency>
            <dependency> <!-- 本示例，我们使用 MySQL -->
                <groupId>mysql</groupId>
                <artifactId>mysql-connector-java</artifactId>
                <version>5.1.48</version>
            </dependency>
    
            <!-- 实现对 MyBatis 的自动化配置 -->
            <dependency>
                <groupId>org.mybatis.spring.boot</groupId>
                <artifactId>mybatis-spring-boot-starter</artifactId>
                <version>2.1.1</version>
            </dependency>
    
            <!-- 实现对 Sharding-JDBC 的自动化配置 -->
            <dependency>
                <groupId>org.apache.shardingsphere</groupId>
                <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
                <version>4.0.0-RC2</version>
            </dependency>
    
            <!-- 保证 Spring AOP 相关的依赖包 -->
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-aspects</artifactId>
            </dependency>
    
            <!-- 方便等会写单元测试 -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-test</artifactId>
                <scope>test</scope>
            </dependency>
    
        </dependencies>
    
    </project></code> 

具体每个依赖的作用，胖友自己认真看下艿艿添加的所有注释噢。

## 8.2 Application

创建 `Application.java` 类，代码如下：

// Application.java
    
    @SpringBootApplication
    @MapperScan(basePackages = "cn.iocoder.springboot.lab17.dynamicdatasource.mapper")
    @EnableAspectJAutoProxy(exposeProxy = true) // http://www.voidcn.com/article/p-zddcuyii-bpt.html
    public class Application {
    } 

*   和 「3.2 Application」 是完全一致的。

## 8.3 应用配置文件

在 `resources` 目录下，创建 `application.yaml` 配置文件。配置如下：

    spring:
      # ShardingSphere 配置项
      shardingsphere:
        datasource:
          # 所有数据源的名字
          names: ds-orders, ds-users
          # 订单 orders 数据源配置
          ds-orders:
            type: com.zaxxer.hikari.HikariDataSource # 使用 Hikari 数据库连接池
            driver-class-name: com.mysql.jdbc.Driver
            jdbc-url: jdbc:mysql://127.0.0.1:3306/test_orders?useSSL=false&useUnicode=true&characterEncoding=UTF-8
            username: root
            password:
          # 订单 users 数据源配置
          ds-users:
            type: com.zaxxer.hikari.HikariDataSource # 使用 Hikari 数据库连接池
            driver-class-name: com.mysql.jdbc.Driver
            jdbc-url: jdbc:mysql://127.0.0.1:3306/test_users?useSSL=false&useUnicode=true&characterEncoding=UTF-8
            username: root
            password:
        # 分片规则
        sharding:
          tables:
            # orders 表配置
            orders:
              actualDataNodes: ds-orders.orders # 映射到 ds-orders 数据源的 orders 表
            # users 表配置
            users:
              actualDataNodes: ds-users.users # 映射到 ds-users 数据源的 users 表

# mybatis 配置内容
    
    mybatis:
      config-location: classpath:mybatis-config.xml # 配置 MyBatis 配置文件路径
      mapper-locations: classpath:mapper/*.xml # 配置 Mapper XML 地址
      type-aliases-package: cn.iocoder.springboot.lab17.dynamicdatasource.dataobject # 配置数据库实体包路径</code> 

*   `spring.shardingsphere.datasource` 配置项下，我们配置了 `ds_orders` 和 `ds_users` 两个数据源。

*   `spring.shardingsphere.sharding` 配置项下，我们配置了分片规则，将 `orders` 逻辑表的操作路由到 `ds-orders` 数据源的 `orders` 真实表 ，将 `users` 逻辑表的操作路由到 `ds-users` 数据源的 `users` 真实表 。

    > “
    > 
    > 艿艿：这里涉及到了一些 ShardingSphere 的概念，后续胖友最好可以看看 官方文档 。

*   `mybatis` 配置项，设置 `mybatis-spring-boot-starter` MyBatis 的配置内容。

## 8.4 MyBatis 配置文件

和 「3.4 MyBatis 配置文件」 一致。

## 8.5 实体类

和 「3.5 实体类」 一致。

## 8.6 Mapper

和 「3.6 Mapper」 一致。

## 8.7 简单测试

和 「3.7 简单测试」 一致。

## 8.8 详细测试

和 「3.8 详细测试」 代码一致，**结果略有差异**。

在 「3.8 详细测试」 的场景二 `#method02()` 的测试，它会抛出异常。而对于本小节使用 Sharding-JDBC 的情况下，正常跑通。这是为什么呢？

原因实际在 「2.3 方案三」 已经解释了：**分库分表中间件返回的 Connection 返回的实际是动态的 DynamicRoutingConnection ，它管理了整个请求（逻辑）过程中，使用的所有的 Connection ，而最终执行 SQL 的时候，DynamicRoutingConnection 会解析 SQL ，获得表对应的真正的 Connection 执行 SQL 操作**。

所以，即使在和 Spring 事务结合的时候，会通过 ThreadLocal 的方式将 Connection 和当前线程进行绑定。此时这个 Connection 也是一个 动态的 DynamicRoutingConnection 连接。

# 9\. Sharding-JDBC 读写分离

> “
> 
> 示例代码对应仓库：lab-17-dynamic-datasource-sharding-jdbc-02 。

Sharding-JDBC 已经提供了读写分离的支持，胖友可以看看如下两个文档：

*   ShardingSphere > 概念 & 功能 > 读写分离
*   ShardingSphere > 用户手册 > Sharding-JDBC > 使用手册 > 读写分离

当然，也可以先不看。

下面，我们来使用 Sharding-JDBC 来实现读写分离。整个的示例，我们会和 「3\. baomidou 读写分离」 是一样的功能，
方便胖友做类比。

## 9.1 引入依赖

和 「8.1 引入依赖」 一致。

## 9.2 Application

和 「8.2 Application」 一致。

## 9.3 应用配置文件

在 `resources` 目录下，创建 `application.yaml` 配置文件。配置如下：

    spring:
      # ShardingSphere 配置项
      shardingsphere:
        # 数据源配置
        datasource:
          # 所有数据源的名字
          names: ds-master, ds-slave-1, ds-slave-2
          # 订单 orders 主库的数据源配置
          ds-master:
            type: com.zaxxer.hikari.HikariDataSource # 使用 Hikari 数据库连接池
            driver-class-name: com.mysql.jdbc.Driver
            jdbc-url: jdbc:mysql://127.0.0.1:3306/test_orders?useSSL=false&useUnicode=true&characterEncoding=UTF-8
            username: root
            password:
          # 订单 orders 从库数据源配置
          ds-slave-1:
            type: com.zaxxer.hikari.HikariDataSource # 使用 Hikari 数据库连接池
            driver-class-name: com.mysql.jdbc.Driver
            jdbc-url: jdbc:mysql://127.0.0.1:3306/test_orders_01?useSSL=false&useUnicode=true&characterEncoding=UTF-8
            username: root
            password:
          # 订单 orders 从库数据源配置
          ds-slave-2:
            type: com.zaxxer.hikari.HikariDataSource # 使用 Hikari 数据库连接池
            driver-class-name: com.mysql.jdbc.Driver
            jdbc-url: jdbc:mysql://127.0.0.1:3306/test_orders_02?useSSL=false&useUnicode=true&characterEncoding=UTF-8
            username: root
            password:
        # 读写分离配置，对应 YamlMasterSlaveRuleConfiguration 配置类
        masterslave:
          name: ms # 名字，任意，需要保证唯一
          master-data-source-name: ds-master # 主库数据源
          slave-data-source-names: ds-slave-1, ds-slave-2 # 从库数据源

# mybatis 配置内容
    
    mybatis:
      config-location: classpath:mybatis-config.xml # 配置 MyBatis 配置文件路径
      mapper-locations: classpath:mapper/*.xml # 配置 Mapper XML 地址
      type-aliases-package: cn.iocoder.springboot.lab17.dynamicdatasource.dataobject # 配置数据库实体包路径</code> 

*   `spring.shardingsphere.datasource` 配置项下，我们配置了 一个主数据源 `ds-master` 、两个从数据源 `ds-slave-1`、`ds-slave-2` 。
*   `spring.shardingsphere.masterslave` 配置项下，配置了读写分离。对于从库来说，Sharding-JDBC 提供了多种负载均衡策略，默认为轮询。
*   `mybatis` 配置项，设置 `mybatis-spring-boot-starter` MyBatis 的配置内容。
*   因为艿艿本地并未搭建 MySQL 一主多从的环境，所以是通过创建了 `test_orders_01`、`test_orders_02` 库，手动模拟作为 `test_orders` 的从库。

## 9.4 MyBatis 配置文件

和 「3.4 MyBatis 配置文件」 一致。

## 9.5 OrderDO

只使用 「3.5 实体类」 的 `OrderDO.java` 类。

## 9.6 OrderMapper

和 「4.6 OrderMapper」 基本一致，差别是无需 `@DS` 注解，具体看如下两个传送门：

*   OrderMapper
*   OrderMapper.xml

## 9.7 简单测试

创建 OrderMapperTest 测试类，我们来测试一下简单的 OrderMapper 的读写操作。代码如下：

// OrderMapper.java
    
    @RunWith(SpringRunner.class)
    @SpringBootTest(classes = Application.class)
    public class OrderMapperTest {
    
        @Autowired
        private OrderMapper orderMapper;
    
        @Test
        public void testSelectById() { // 测试从库的负载均衡
            for (int i = 0; i < 10; i++) {
                OrderDO order = orderMapper.selectById(1);
                System.out.println(order);
            }
        }
    
        @Test
        public void testSelectById02() { // 测试强制访问主库
            try (HintManager hintManager = HintManager.getInstance()) {
                // 设置强制访问主库
                hintManager.setMasterRouteOnly();
                // 执行查询
                OrderDO order = orderMapper.selectById(1);
                System.out.println(order);
            }
        }
    
        @Test
        public void testInsert() { // 插入
            OrderDO order = new OrderDO();
            order.setUserId(10);
            orderMapper.insert(order);
        }
    
    } 

*   `#testSelectById()` 方法，测试从库的负载均衡查询。
*   `#testSelectById02()` 方法，测试强制访问主库。在一些业务场景下，对数据延迟敏感，所以只能强制读取主库。此时，可以使用 HintManager 强制访问主库。

*   不过要注意，在使用完后，需要去清理下 HintManager （HintManager 是基于线程变量，透传给 Sharding-JDBC 的内部实现），避免污染下次请求，一直强制访问主库。
*   Sharding-JDBC 比较贴心，HintManager 实现了 AutoCloseable 接口，可以通过 Try-with-resources 机制，自动关闭。

*   `#testInsert()` 方法，测试主库的插入。

胖友自己跑下测试用例。如果跑通，说明配置就算成功了。

另外，在 `#testSelectById()` 测试方法中，艿艿会了看看 `slave` 分组是不是真的在负载均衡。所以在数据库中，分别插入数据如下。

    主库：[id = 1, user_id = 1]
    从库 01：[id = 1, user_id = 2]
    从库 02：[id = 1, user_id = 3]

*   这样，通过手动设置相同 `id = 1` 的记录，对应不同的 `user_id` ，那么我们就可以观察 `#testSelectById()` 测试方法的输出结果。如果是，`user_id = 2` 和 `user_i = 3` 循环输出，说明就正常了。

## 9.8 详细测试

在 `cn.iocoder.springboot.lab17.dynamicdatasource.service` 包路径下，创建 `OrderService.java` 类。代码如下：

// OrderService.java
    
    @Service
    public class OrderService {
    
        @Autowired
        private OrderMapper orderMapper;
    
        @Transactional
        public void add(OrderDO order) {
            // <1.1> 这里先假模假样的读取一下。读取从库
            OrderDO exists = orderMapper.selectById(1);
            System.out.println(exists);
    
            // <1.2> 插入订单
            orderMapper.insert(order);
    
            // <1.3> 这里先假模假样的读取一下。读取主库
            exists = orderMapper.selectById(1);
            System.out.println(exists);
        }
    
        public OrderDO findById(Integer id) {
            return orderMapper.selectById(id);
        }
    
    } 

*   我们创建了 OrderServiceTest 测试类，可以测试上面编写的两个方法。
*   在 `#add(OrderDO order)` 方法中，开启事务，插入一条订单记录。

*   `<1.1>` 处，往**从库**发起一次订单查询。在 Sharding-JDBC 的读写分离策略里，默认读取从库。
*   `<1.2>` 处，往**主库**发起一次订单写入。写入，肯定是操作主库的。
*   `<1.3>` 处，往**主库**发起一次订单查询。在 Sharding-JDBC 中，读写分离约定：**同一线程且同一数据库连接内，如有写入操作，以后的读操作均从主库读取，用于保证数据一致性。**

*   在 `#findById(Integer id)` 方法，往**从库**发起一次订单查询。

# 666\. 彩蛋

我们看完了三种多数据源的方案，实际场景下怎么选择呢？

首先，我们基本排除了方案二【不同操作类，固定数据源】。配置繁琐，使用不变。艿艿也去问了一圈朋友，
暂时没有这么做的。这种方案，更加适合**不同类型**的数据源，例如说一个项目中，既有 MySQL 数据源，
又有 MongoDB、Elasticsarch 等其它数据源。

然后，对于大多数场景下，方案一【基于 SpringAbstractRoutingDataSource 做拓展】，基本能够满足。
这种方案，目前是比较主流的方案，大多数项目都采用。在实现上，我们可以比较容易的自己封装一套，
当然也可以考虑使用 `dynamic-datasource-spring-boot-starter` 开源项目。不过呢，建议可以把它的源码撸一下
，核心代码估计 1000 行左右，不要慌。

当然，方案一和方案二，会存在和 Spring 事务结合的时候，在事务中无法切换数据源。
这是因为 Spring 事务会将 Connection 和当前线程变量绑定定，后续会通过线程变量重用该 Connection ，
导致无法切换数据源。所以，方案一和方案二，可以理解成 DataSource 级别上实现的数据源方案。

最后，方案三【分库分表中间件】是完美解决方案，基本满足了所有的场景。
艿艿个人强烈推荐使用 Apache ShardingSphere 的 Sharding-JDBC 组件，
无论胖友是有多数据源，还是分库分表，还是读写分离，都能完美的匹配。
并且，Apache ShardingSphere 已经提供多种分布式事务方案，也能解决在文章的开头，
艿艿提到的分布式事务的问题。这种类型的方案，目前很多大厂都是这样去玩的。

> “
> 
> *   京东：采用 client 模式的读写分离和分库分表。
> *   美团：采用 client 模式的读写分离和分库分表。
> *   陌陌：采用 client 模式的读写分离和分库分表。
> 
> ... 继续补充调研 ing 。

