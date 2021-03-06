---
layout: post
title: Lombok 注解
category: tools
tags: [tools]
---

 

来源：http://t.cn/EXhxRzV

> *   1）引入相应的maven包
>     
>     
> *   2）添加IDE工具对Lombok的支持
>     
>     
> *   3）Lombok实现原理
>     
>     
> *   4) Lombok注解的使用

* * *

以前的Java项目中，充斥着太多不友好的代码：POJO的getter/setter/toString；异常处理；I/O流的关闭操作等等，这些样板代码既没有技术含量，又影响着代码的美观，Lombok应运而生。

任何技术的出现都是为了解决某一类问题，如果在此基础上再建立奇技淫巧，不如回归Java本身，应该保持合理使用而不滥用。

Lombok的使用非常简单：

# 1）引入相应的maven包

    <dependency>  
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.16.18</version>
        <scope>provided</scope>
    </dependency>

Lombok的scope=provided，说明它只在编译阶段生效，不需要打入包中。事实正是如此，Lombok在编译期将带Lombok注解的Java文件正确编译为完整的Class文件。

# 2）添加IDE工具对Lombok的支持

IDEA中引入Lombok支持如下：

点击File-- Settings设置界面，安装Lombok插件：

![](https://ziyekudeng.github.io/assets/images/2019/0611/idea-lombok/1.webp)

点击File-- Settings设置界面，开启 `AnnocationProcessors`：

![](https://ziyekudeng.github.io/assets/images/2019/0611/idea-lombok/2.webp)

开启该项是为了让Lombok注解在编译阶段起到作用。

Eclipse的Lombok插件安装可以自行百度，也比较简单，值得一提的是，由于Eclipse内置的编译器不是Oracle javac，而是eclipse自己实现的Eclipse Compiler for Java (ECJ).要让ECJ支持Lombok，需要在eclipse.ini配置文件中添加如下两项内容：

    -Xbootclasspath/a:[lombok.jar所在路径]

# 3）Lombok实现原理

自从Java 6起，javac就支持“JSR 269 Pluggable Annotation Processing API”规范，只要程序实现了该API，就能在javac运行的时候得到调用。

Lombok就是一个实现了"JSR 269 API"的程序。在使用javac的过程中，它产生作用的具体流程如下：

1.  javac对源代码进行分析，生成一棵抽象语法树(AST)

2.  javac编译过程中调用实现了JSR 269的Lombok程序

3.  此时Lombok就对第一步骤得到的AST进行处理，找到Lombok注解所在类对应的语法树(AST)，然后修改该语法树(AST)，增加Lombok注解定义的相应树节点

4.  javac使用修改后的抽象语法树(AST)生成字节码文件

# 4) Lombok注解的使用

POJO类常用注解：

@Getter/@Setter: 作用类上，生成所有成员变量的getter/setter方法；作用于成员变量上，生成该成员变量的getter/setter方法。可以设定访问权限及是否懒加载等。

    package com.trace;
    import lombok.AccessLevel;import lombok.Getter;import lombok.Setter;
    /** * Created by Trace on 2018/5/19.<br/> * DESC: 测试类 */@SuppressWarnings("unused")public class TestClass {
        public static void main(String[] args) {
        }
    
        @Getter(value = AccessLevel.PUBLIC)    @Setter(value = AccessLevel.PUBLIC)    public static class Person {        private String name;        private int age;        private boolean friendly;    } nbsp;   public static class Animal {        private String name;        private int age;        @Getter @Setter private boolean funny;    }}

在Structure视图中，可以看到已经生成了getter/setter等方法：

![](https://ziyekudeng.github.io/assets/images/2019/0611/idea-lombok/3.webp)

编译后的代码如下：[这也是传统Java编程需要编写的样板代码]
    
    //// Source code recreated from a .class file by IntelliJ IDEA// (powered by Fernflower decompiler)//
    package com.trace;
    public class TestClass {
        public TestClass() {
        }
    
        public static void main(String[] args) {
        }
    
        public static class Animal {
            private String name;
            private int age;
            private boolean funny;
    
            public Animal() {
            }
    
            public boolean isFunny() {
                return this.funny;
            }
    
            public void setFunny(boolean funny) {
                this.funny = funny;
            }
        }
    
        public static class Person {
            private String name;
            private int age;
            private boolean friendly;
    
            public Person() {
            }
    
            public String getName() {
                return this.name;
            }
    
            public int getAge() {
                return this.age;
            }
    
            public boolean isFriendly() {
                return this.friendly;
            }
    
            public void setName(String name) {
                this.name = name;
            }
    
            public void setAge(int age) {
                this.age = age;
            }
    
            public void setFriendly(boolean friendly) {
                this.friendly = friendly;
            }
        }
    }

@ToString：作用于类，覆盖默认的toString()方法，可以通过of属性限定显示某些字段，通过exclude属性排除某些字段。

![](https://ziyekudeng.github.io/assets/images/2019/0611/idea-lombok/4.webp)

@EqualsAndHashCode：作用于类，覆盖默认的equals和hashCode

@NonNull：主要作用于成员变量和参数中，标识不能为空，否则抛出空指针异常。

![](https://ziyekudeng.github.io/assets/images/2019/0611/idea-lombok/5.webp)

@NoArgsConstructor, @RequiredArgsConstructor, @AllArgsConstructor：作用于类上，用于生成构造函数。有staticName、access等属性。

staticName属性一旦设定，将采用静态方法的方式生成实例，access属性可以限定访问权限。

@NoArgsConstructor：生成无参构造器；

@RequiredArgsConstructor：生成包含final和@NonNull注解的成员变量的构造器；

@AllArgsConstructor：生成全参构造器。

![](https://ziyekudeng.github.io/assets/images/2019/0611/idea-lombok/6.webp)

编译后结果：
    
    public static class Person {
        @NonNull
        private String name;
        private int age;
        private boolean friendly;
    
        private Person() {
        }
    
        @ConstructorProperties({"name"
        })
        Person(@NonNull
        String name) {
            if (name == null) {
                throw new NullPointerException("name");
            } else {
                this.name = name;
            }
        }
    
        @ConstructorProperties({"name",
            "age",
            "friendly"
        })
        public Person(@NonNull
        String name, int age, boolean friendly) {
            if (name == null) {
                throw new NullPointerException("name");
            } else {
                this.name = name;
                this.age = age;
                this.friendly = friendly;
            }
        }
    
        public String toString() {
            return "TestClass.Person(name=" + this.getName() + ", age=" +
            this.getAge() + ")";
        }
    
        @NonNull
        public String getName() {
            return this.name;
        }
    
        public int getAge() {
            return this.age;
        }
    
        public boolean isFriendly() {
            return this.friendly;
        }
    
        public void setName(@NonNull
        String name) {
            if (name == null) {
                throw new NullPointerException("name");
            } else {
                this.name = name;
            }
        }
    
        public void setAge(int age) {
            this.age = age;
        }
    
        public void setFriendly(boolean friendly) {
            this.friendly = friendly;
        }
    
        private static TestClass.Person of() {
            return new TestClass.Person();
        }
    }

@Data：作用于类上，是以下注解的集合：@ToString @EqualsAndHashCode @Getter @Setter @RequiredArgsConstructor

@Builder：作用于类上，将类转变为建造者模式

@Log：作用于类上，生成日志变量。针对不同的日志实现产品，有不同的注解：

![](https://ziyekudeng.github.io/assets/images/2019/0611/idea-lombok/7.webp)

其他重要注解：

@Cleanup：自动关闭资源，针对实现了java.io.Closeable接口的对象有效，如：典型的IO流对象

![](https://ziyekudeng.github.io/assets/images/2019/0611/idea-lombok/8.webp)

编译后结果如下：

![](https://ziyekudeng.github.io/assets/images/2019/0611/idea-lombok/9.webp)

是不是简洁了太多。

@SneakyThrows：可以对受检异常进行捕捉并抛出，可以改写上述的main方法如下：

![](https://ziyekudeng.github.io/assets/images/2019/0611/idea-lombok/10.webp)

@Synchronized：作用于方法级别，可以替换synchronize关键字或lock锁，用处不大。

* * *

* * *

