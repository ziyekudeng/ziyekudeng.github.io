---
layout: post
title: Java里的双亲委派机制
category: arch
tags: [arch]
keywords: 架构
---

 

## Java里的双亲委派机制


**什么是双亲委派模型？**

首先，先要知道什么是类加载器。简单说，类加载器就是根据指定全限定名称将class文件加载到JVM内存，转为Class对象。

如果站在JVM的角度来看，只存在两种类加载器:

1. 启动类加载器（Bootstrap ClassLoader）

2. 其他类加载器：由Java语言实现，继承自抽象类ClassLoader。

但是实际上却并不是这个样子的。

我们看一幅图描述的类加载器之间的关系：

![](https://ziyekudeng.github.io/assets/images/2019/0907/parental-delegation-model/1.webp) 

**BootStrapClassLoader**：

启动类加载器，该ClassLoader是jvm在启动时创建的，用于加载 $JAVA_HOME/jre/lib下面的类库（或者通过参数-Xbootclasspath 指定）。

由于引导类加载器涉及到虚拟机本地实现细节，开发者无法直接获取到启动类加载器的引用，所以不能直接通过引用进行操作。

**ExtensionClassLoader**：

扩展类加载器，该ClassLoader是在sun.misc.Launcher里作为一个内部类ExtClassLoader定义的（即 sun.misc.Launcher$ExtClassLoader），

ExtClassLoader会加载 $JAVA_HOME/jre/lib/ext下的类库（或者通过参数-Djava.ext.dirs指定）。

**ApplicationClassLoader**：

应用程序类加载器，该 ClassLoader 同样是在sun.misc.Launcher里作为一个内部类AppClassLoader定义的（即 sun.misc.Launcher$AppClassLoader ），

ApplicationClassLoader会加载java环境变量CLASSPATH所指定的路径下的类库，

而CLASSPATH所指定的路径可以通过System.getProperty("java.class.path")获取；

当然，该变量也可以覆盖，可以使用参数-cp，例如：java -cp 路径 （可以指定要执行的class目录）。

**CustomClassLoader**：

自定义类加载器，该 ClassLoader 是指我们自定义的 ClassLoader ，比如tomcat的 StandardClassLoader 属于这一类；

当然，大部分情况下使用 ApplicationClassLoader 就足够了。

所以说双亲委派模型就是说：如果一个类加载器收到了类加载的请求，它首先不会自己尝试去加载这个类，

而是把这个请求委派给父类加载器，每一个层次的类加载器都是加此，因此所有的加载请求最终到达顶层的启动类加载器，

只有当父类加载器反馈自己无法完成加载请求时（指它的搜索范围没有找到所需的类），子类加载器才会尝试自己去加载。

其实这也是双亲委派模型的一个过程。

**双亲委派模型的系统实现**

    public abstract class ClassLoader {
        
         /**
         java.lang.ClassLoader的loadClass()方法中，
         先检查是否已经被加载过，若没有加载则调用父类加载器的loadClass()方法，
         若父加载器为空则默认使用启动类加载器作为父加载器。如果父加载失败，则抛出ClassNotFoundException异常后，
         再调用自己的findClass()方法进行加载。*/
         protected Class<?> loadClass(String name, boolean resolve)
                throws ClassNotFoundException
            {
                synchronized (getClassLoadingLock(name)) {
                    // 首先，检查类是否已加载
                    Class<?> c = findLoadedClass(name);
                    if (c == null) {
                        long t0 = System.nanoTime();
                        try {
                            if (parent != null) {
                                c = parent.loadClass(name, false);
                            } else {
                                c = findBootstrapClassOrNull(name);
                            }
                        } catch (ClassNotFoundException e) {
                            // 如果找不到类，则引发ClassNotFoundException
                            // 从非空父类加载器
                        }
        
                        if (c == null) {
                            // 如果仍然找不到，则按顺序调用findclass
                            // to find the class.
                            long t1 = System.nanoTime();
                            c = findClass(name);
        
                            // 这是定义类加载器；记录统计信息
                            sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                            sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                            sun.misc.PerfCounter.getFindClasses().increment();
                        }
                    }
                    if (resolve) {
                        resolveClass(c);
                    }
                    return c;
                }
            }
    }

ClassLoader的loadClass()方法中,先检查是否已经被加载过，若没有加载则调用父类加载器的loadClass()方法，

若父加载器为空则默认使用启动类加载器作为父加载器。如果父加载失败，则抛出ClassNotFoundException异常后，

再调用自己的findClass()方法进行加载。

以上的代码就是双亲委派模型的一个代码实现的一个案例，其实原理说起来很简单，但是如果你不深入了解类加载机制，那么就比较难理解了

**双亲委派模型原理**

1-类加载器收到类加载的请求；

2-把这个请求委托给父加载器去完成，一直向上委托，直到启动类加载器；

3-启动器加载器检查能不能加载（使用findClass()方法），能就加载（结束）；否则，抛出异常，通知子加载器进行加载。

4-重复步骤三；

其实在Java的日常应用程序开发中，类的加载几乎是由BootStrap类加载器，Extension类加载器，Application类加载器相互配合执行的，

在必要时，我们还可以自定义类加载器，需要注意的是，Java虚拟机对class文件采用的是按需加载的方式，

也就是说当需要使用该类时才会将它的class文件加载到内存生成class对象，而且加载某个类的class文件时，

Java虚拟机采用的是双亲委派模式即把请求交由父类处理。

**为什么要使用双亲委派模型呢？**

其实这个时候就有一道非常有意思的面试题了大家看一下

**能不能自己写个类叫java.lang.System？**

其实这也是阿里面试的一道很经典的面试题，我们来做个分析。

之前看过一个文章说的就是说这个通常不可以，但可以采取另类方法达到这个需求。所谓的另类方法指自己写个类加载器来加载java.lang.System达到目的。

但是实际上却是是不能的，为了输出，我们做个最简单的测试用Math类来进行测试，效果和原理类似，这个例子是我之前在看一个文章的时候随手记录下拉的，当时运行了一下，确实很有道理，大家可以参考一下。

包结构如下：

![](https://ziyekudeng.github.io/assets/images/2019/0907/parental-delegation-model/2.webp) 

代码

    package com.tq;
    import java.io.InputStream;
    public class MyClassLoader extends ClassLoader {
        public MyClassLoader() {
            super(null);
        }
    
        @Override
        public Class<?> loadClass(String name) throws ClassNotFoundException {
            try{
                String className = null;
                if(name.startsWith("java.lang")){
                    className = "/" + name.replace('.', '/') + ".class";
                }else{
                    className = name.substring(name.lastIndexOf('.') + 1) + ".class";
                }
                System.out.println(className);
                InputStream is = getClass().getResourceAsStream(className);
                System.out.println(is);
                if(is == null)
                    return super.loadClass(name);
    
                byte[] b = new byte[is.available()];
                is.read(b);
                return defineClass(name, b, 0, b.length);
            }catch (Exception e) {
                e.printStackTrace();
                throw new ClassNotFoundException();
            }
        }
    }
    
    
    package com.tq;
    public class ClassLoaderTest {
        public static void main(String[] args) throws ClassNotFoundException, InstantiationException, IllegalAccessException {
            ClassLoader myLoader = new MyClassLoader();
            Object obj = myLoader.loadClass("java.lang.Math").newInstance();
            System.out.println(obj);
        }
    }
    
    
    package java.lang;
    public final class Math {
        public static void main(String[] args) {
            System.out.println("hello world");
        }
    }
    
    
    package java.lang;
    public class MyMath {
        public static void main(String[] args) {
            System.out.println("hello world");
        }
    }

我们在运行Math里面的输出方法的时候就会出现问题，

![](https://ziyekudeng.github.io/assets/images/2019/0907/parental-delegation-model/3.webp) 

提示Math类没有main方法，这就是我们在之前预测根据双亲委托原则，Math类首先由启动类加载器去尝试加载，

它找到rt.jar中的java.lang.Math类并加载进内存（并不会加载我们自定义的Math类），然后执行main方法时，发现不存在该方法，

所以报方法不存在错误。也就是说，默认情况下JVM不会加载我们自定义的Math类。

![](https://ziyekudeng.github.io/assets/images/2019/0907/parental-delegation-model/4.webp) 

java.lang.SecurityException: Prohibited package name: java.lang 他不允许我们使用java.lang的包名

我们继续运行下ClassLoaderTest类

![](https://ziyekudeng.github.io/assets/images/2019/0907/parental-delegation-model/5.webp) 

我们看一下ClassLoader的源码

    
    private ProtectionDomain preDefineClass(String name,ProtectionDomain pd){
            if (!checkName(name))
                throw new NoClassDefFoundError("IllegalName: " + name);
    
            // Note:  Checking logic in java.lang.invoke.MemberName.checkForTypeAlias
            // relies on the fact that spoofing is impossible if a class has a name
            // of the form "java.*"
            if ((name != null) && name.startsWith("java.")) {
                throw new SecurityException
                    ("Prohibited package name: " +
                     name.substring(0, name.lastIndexOf('.')));
            }
            if (pd == null) {
                pd = defaultDomain;
            }
    
            if (name != null) checkCerts(name, pd.getCodeSource());
    
            return pd;
        }


通过代码实例及源码分析可以看到，对于自定义的类加载器，强行用defineClass()方法去加载一个以"java."开头的类也是会抛出异常的。

**总结**

1. 双亲委派模型最大的好处就是让Java类同其类加载器一起具备了一种带优先级的层次关系。这句话可能不好理解，我们举个例子。比如我们要加载java.lang.Object类，无论我们用哪个类加载器去加载Object类，这个加载请求最终都会委托给Bootstrap ClassLoader，这样就保证了所有加载器加载的Object类都是同一个类。如果没有双亲委派模型，那就乱了套了，完全可能搞出多个不同的Object类。

2. 自上而下每个类加载器都会尽力加载.

3. 不能自己写以"java."开头的类，其要么不能加载进内存，要么即使你用自定义的类加载器去强行加载，也会收到一个SecurityException。

