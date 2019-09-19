---
layout: post
title: Java运行时动态生成class的方法
category: arch
tags: [arch]
keywords: 架构
---
 


Java是一门静态语言，通常，我们需要的class在编译的时候就已经生成了，为什么有时候我们还想在运行时动态生成class呢？

因为在有些时候，我们还真得在运行时为一个类动态创建子类。比如，编写一个ORM框架，如何得知一个简单的JavaBean是否被用户修改过呢？

以`User`为例：

    public class User {
        private String id;
        private String name;
    
        public String getId() {
            return id;
        }
    
        public void setId(String id) {
            this.id = id;
        }
    
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    }

其实`UserProxy`实现起来很简单，就是创建一个`User`的子类，覆写所有`setXxx()`方法，做个标记就可以了：
    
    public class UserProxy extends User {
        private boolean dirty;
    
        public boolean isDirty() {
            return this.dirty;
        }
    
        public void setDirty(boolean dirty) {
            this.dirty = dirty;
        }
    
        @Override
        public void setId(String id) {
            super.setId(id);
            setDirty(true);
        }
    
        @Override
        public void setName(String name) {
            super.setName(name);
            setDirty(true);
        }
    }

但是这个`UserProxy`就必须在运行时动态创建出来了，因为编译时ORM框架根本不知道`User`类。

现在问题来了，动态生成字节码，难度有多大？

如果我们要自己直接输出二进制格式的字节码，在完成这个任务前，必须先认真阅读[JVM规范第4章](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html)，详细了解class文件结构。估计读完规范后，两个月过去了。

所以，第一种方法，自己动手，从零开始创建字节码，理论上可行，实际上很难。

第二种方法，使用已有的一些能操作字节码的库，帮助我们创建class。

目前，能够操作字节码的开源库主要有[CGLib](https://github.com/cglib/cglib)和[Javassist](https://jboss-javassist.github.io/javassist/)两种，它们都提供了比较高级的API来操作字节码，最后输出为class文件。

比如CGLib，典型的用法如下：

    Enhancer e = new Enhancer();
    e.setSuperclass(...);
    e.setStrategy(new DefaultGeneratorStrategy() {
        protected ClassGenerator transform(ClassGenerator cg) {
            return new TransformingGenerator(cg,
                new AddPropertyTransformer(new String[]{ "foo" },
                        new Class[] { Integer.TYPE }));
        }});
    Object obj = e.create();

比自己生成class要简单，但是，要学会它的API还是得花大量的时间，并且，上面的代码很难看懂对不对？

> 有木有更简单的方法？

# 有！

换一个思路，如果我们能创建`UserProxy.java`这个源文件，再调用Java编译器，直接把源码编译成class，再加载进虚拟机，任务完成！

毕竟，创建一个字符串格式的源码是很简单的事情，就是拼字符串嘛，高级点的做法可以用一个模版引擎。

如何编译？

Java的编译器是`javac`，但是，在很早很早的时候，Java的编译器就已经用纯Java重写了，自己能编译自己，行业黑话叫“自举”。从Java 1.6开始，编译器接口正式放到JDK的公开API中，于是，我们不需要创建新的进程来调用`javac`，而是直接使用编译器API来编译源码。

使用起来也很简单：

    JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();
    int compilationResult = compiler.run(null, null, null, '/path/to/Test.java');

这么写编译是没啥问题，问题是我们在内存中创建了Java代码后，必须先写到文件，再编译，最后还要手动读取class文件内容并用一个ClassLoader加载。

> 有木有更简单的方法？

有！

其实Java编译器根本不关心源码的内容是从哪来的，你给它一个`String`当作源码，它就可以输出`byte[]`作为class的内容。

所以，我们需要参考Java Compiler API的文档，让Compiler直接在内存中完成编译，输出的class内容就是`byte[]`。

代码改造如下：

     results;
    JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();
    StandardJavaFileManager stdManager = compiler.getStandardFileManager(null, null, null);
    try (MemoryJavaFileManager manager = new MemoryJavaFileManager(stdManager)) {
        JavaFileObject javaFileObject = manager.makeStringSource(fileName, source);
        CompilationTask task = compiler.getTask(null, manager, null, null, null, Arrays.asList(javaFileObject));
        if (task.call()) {
            results = manager.getClassBytes();
        }
    }

上述代码的几个关键在于：

1.  用`MemoryJavaFileManager`替换JDK默认的`StandardJavaFileManager`，以便在编译器请求源码内容时，不是从文件读取，而是直接返回`String`；
2.  用`MemoryOutputJavaFileObject`替换JDK默认的`SimpleJavaFileObject`，以便在接收到编译器生成的`byte[]`内容时，不写入class文件，而是直接保存在内存中。

最后，编译的结果放在`Map<String, byte[]>`中，Key是类名，对应的`byte[]`是class的二进制内容。

> 为什么编译后不是一个`byte[]`呢？

因为一个`.java`的源文件编译后可能有多个`.class`文件！只要包含了静态类、匿名类等，编译出的class肯定多于一个。

> 如何加载编译后的class呢？

加载class相对而言就容易多了，我们只需要创建一个`ClassLoader`，覆写`findClass()`方法：
    
    class MemoryClassLoader extends URLClassLoader {
    
        Map<String, byte[]> classBytes = new HashMap<String, byte[]>();
    
        public MemoryClassLoader(Map<String, byte[]> classBytes) {
            super(new URL[0], MemoryClassLoader.class.getClassLoader());
            this.classBytes.putAll(classBytes);
        }
    
        @Override
        protected Class<?> findClass(String name) throws ClassNotFoundException {
            byte[] buf = classBytes.get(name);
            if (buf == null) {
                return super.findClass(name);
            }
            classBytes.remove(name);
            return defineClass(name, buf, 0, buf.length);
        }
    }

> 除了写ORM用之外，还能干什么？

可以用它来做一个Java脚本引擎。实际上本文的代码主要就是参考了[Scripting](https://java.net/projects/scripting)项目的源码。

> 完整的源码呢？

也就200行代码吧！动态创建class不是梦！

廖雪峰：[https://github.com/michaelliao/compiler](https://github.com/michaelliao/compiler)

子夜枯灯：[https://github.com/ziyekudeng/compiler](https://github.com/ziyekudeng/compiler)




**作者：廖雪峰**  
来源：[廖雪峰官方网站](https://www.liaoxuefeng.com/)
**版权归作者所有，转载请注明出处** 
