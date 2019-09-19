---
layout: post
title: java自定义注解
category: spring
tags: [spring]
---


Java注解是附加在代码中的一些元信息，用于一些工具在编译、运行时进行解析和使用，起到说明、配置的功能。注解不会也不能影响代码的实际逻辑，仅仅起到辅助性的作用。包含在 java.lang.annotation 包中注解的定义和接口差不多，只是在interface前面多了一个“@”。形象地说注解只是做了如下声明：大家好，代码这里用了这个注解，请大家多多关照。 然后呢，下面那里用到注解就关照下该怎么处理。

      public @interface MyAnnotation{}  
上面的代码是一个最简单的注解。这个注解没有属性。也可以理解为是一个标记注解。就象Serializable接口一样是一个标记接口，里面未定义任何方法。
当然，也可以定义而有属性的注解。

      public @interface MyAnnotation
    {    String value();}  可以按如下格式使用MyAnnotation
      @MyAnnotation(“abc”)
    public void myMethod()
    {}  
看了上面的代码，大家可能有一个疑问。怎么没有使用value，而直接就写”abc”了。那么”abc”到底传给谁了。其实这里有一个约定。如果没有写属性名的值，而这个注解又有value属性，就将这个值赋给value属性，如果没有，就出现编译错误。
除了可以省略属性名，还可以省略属性值。这就是默认值。

      public @interface MyAnnotation
    {    public String myMethod() default "xyz";}  可以直接使用MyAnnotation
      @MyAnnotation // 使用默认值xyz
    public void myMethod()
    {}  也可以这样使用
      @MyAnnotation(myMethod=”abc”)
    public void myMethod()
    {}  如果要使用多个属性的话。可以参考如下代码。
      public @interface MyAnnotation
    {
        public enum MyEnum{A, B, C}
        public MyEnum value1();
        public String value2();
    
    }    @MyAnnotation(value1=MyAnnotation.MyEnum.A, value2 = “xyz”)
    public void myMethod()
    {}  
如果仅仅这样配自定义的注解还差一点，为这些注解再声明一下，这就要讲到元注解

元注解是指注解的注解。包括 @Retention @Target @Document @Inherited四种。

@Target：定义注解的作用目标
其定义的源码为：
    
       @Documented  
        @Retention(RetentionPolicy.RUNTIME)  
        @Target(ElementType.ANNOTATION_TYPE)  
        public @interface Target {  
            ElementType[] value();  
        }   @Target(ElementType.TYPE) //接口、类、枚举、注解
    @Target(ElementType.FIELD) //字段、枚举的常量
    @Target(ElementType.METHOD) //方法
    @Target(ElementType.PARAMETER) //方法参数
    @Target(ElementType.CONSTRUCTOR) //构造函数
    @Target(ElementType.LOCAL_VARIABLE)//局部变量
    @Target(ElementType.ANNOTATION_TYPE)//注解
    @Target(ElementType.PACKAGE) ///包
由以上的源码可以知道，他的elementType 可以有多个，一个注解可以为类的，方法的，字段的等等

    @Retention: 定义注解的保留策略
    @Retention(RetentionPolicy.SOURCE) //注解仅存在于源码中，在class字节码文件中不包含
    @Retention(RetentionPolicy.CLASS) // 默认的保留策略，注解会在class字节码文件中存在，但运行时无法获得
    @Retention(RetentionPolicy.RUNTIME) // 注解会在class字节码文件中存在，在运行时可以通过反射获取到

Documented
这个注解和它的名子一样和文档有关。在默认的情况下在使用javadoc自动生成文档时，注解将被忽略掉。如果想在文档中也包含注解，必须使用Documented为文档注解。
    
      @interface MyAnnotation{ }
    @MyAnnotation
    class Class1{public void myMethod() { }}  使用javadoc为这段代码生成文档时并不将@MyAnnotation包含进去。生成的文档对Class1的描述如下：
    class Class1 extends java.lang.Object
    而如果这样定义MyAnnotation将会出现另一个结果。
      @Documented
    @interface MyAnnotation {}  生成的文档：
    @MyAnnotation // 这行是在加上@Documented后被加上的
    class Class1 extends java.lang.Object

Inherited
继承是java主要的特性之一。在类中的protected和public成员都将会被子类继承，但是父类的注解会不会被子类继承呢？很遗憾的告诉大家，在默认的情况下，父类的注解并不会被子类继承。如果要继承，就必须加上Inherited注解。
      
      @Inherited
    @interface MyAnnotation { }
    @MyAnnotation
    public class ParentClass {}
    public class ChildClass extends ParentClass { }  在以上代码中ChildClass和ParentClass一样都已被MyAnnotation注解了。

下面来分析一段网上自定义的注解，来了解下怎么用的，因为总不能把项目上的代码搞过来让大家分析。这段代码我跑了一下没问题。
    
    HelloWord.class
    
      @Documented
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    @Inherited
    public @interface HelloWorld {
       public String name()default "我";
    }  YTS.class  @Documented
    @Target({ElementType.TYPE,ElementType.METHOD})
    @Retention(RetentionPolicy.RUNTIME)
    public @interface YTS {
        public enum YtsType{util,entity,service,model};
       public YtsType classType() default YtsType.util;
    }  处理定义这两个注解的类ParseAnnotation.class
      public class ParseAnnotation {

    public void parseMethod(Class clazz) throws IllegalArgumentException, IllegalAccessException, InvocationTargetException, SecurityException, NoSuchMethodException, InstantiationException{

       Object obj = clazz.getConstructor(new Class[]{}).newInstance(new Object[]{});
    for(Method method : clazz.getDeclaredMethods()){
        HelloWorld say = method.getAnnotation(HelloWorld.class);
        String name = "";
        if(say != null){
           name = say.name();
           method.invoke(obj, name);
        }
       YTS yts = (YTS)method.getAnnotation(YTS.class);
       if(yts != null){
          if(YtsType.util.equals(yts.classType())){
          System.out.println("this is a util method");
        }else{
            System.out.println("this is a other method");
           }
       }
     }
       }
       @SuppressWarnings("unchecked")
       public void parseType(Class clazz) throws IllegalArgumentException, IllegalAccessException, InvocationTargetException{
        YTS yts = (YTS) clazz.getAnnotation(YTS.class);
           if(yts != null){
               if(YtsType.util.equals(yts.classType())){
                   System.out.println("this is a util class");
               }else{
                   System.out.println("this is a other class");
               }
           }
       }
    
    }  测试类SayHell.class  @YTS(classType =YtsType.util)
    public class SayHell {
    
        @HelloWorld(name = " 小明 ")
        @YTS
        public void sayHello(String name){
            if(name == null || name.equals("")){
                System.out.println("hello world!");
            }else{
                System.out.println(name + "say hello world!");
            }
        }
    
        public static void main(String[] args) throws IllegalArgumentException, IllegalAccessException, InvocationTargetException, SecurityException, NoSuchMethodException, InstantiationException {
            ParseAnnotation parse = new ParseAnnotation();
            parse.parseMethod(SayHell.class);
            parse.parseType(SayHell.class);
    
        }
    }  打印结果：  小明 say hello world!
    this is a util method
    this is a util class  

下面分析这段代码：可以看到@yts这注解可以定义到类也可以定义到方法上。首先掉parse.parseMethod(SayHell.class)这个方法，此方法里先

  clazz.getDeclaredMethods()  通过反射来获取list<Method>,即这个类的全部方法，然后遍历这些方法，看哪个上面配置了@yts和@HelloWord的注解，如果配置helloWord的注解，将会把helloword的name属性作为参数来执行这个方法，在这里因为sayhello方法上配了@HelloWord，所以会执行这个方法，输出 小明 sayhello word。当然，如果这里不传ｎａｍｅ＝小明那将会按默认的‘我’来作为参数，输出　我　sayhello word．后面的同理就不分析了．

通过上面的大家基本上能了解自定义注解是怎么用的。下面说下鄙人对自定义注解的用处的一些心得。（实际项目中用到的地方）

１，假如现在要批量保存１００条数据，怎样保证里面有不重复的数据呢？当然我们可以用ｈｉｂｅｒｎａｔｅ的＠**UniqueConstraint，参照语法如下：**

      @Table(name = "contact", uniqueConstraints = {
      @UniqueConstraint(columnNames = { "name", "email" }
          )})  
      这里只能保证ｇｅｔ出来的ｎａｍｅ和ｅｍａｉｌ来确定唯一数据，但是ｓａｖｅ时呢？我们知道，要保证两个对象不相同就要重写ｅｑｕａｌｓ和ｈａｓｈｃｏｄｅ方法，所以一般来说我们会重写这两个对象的这两个方法来保证对象的唯一性，这样在ｓａｖｅ的时候就可以解决这问题。但是问题又来了，你总不会每个对象都重写一次吧，多麻烦啊，所以我们就可以考虑自定义注解来解决这个问题。思路如下，代码不提供。首先我们可以先自定义个注解假如叫＠ＡＡＡ，然后写个抽象类来获取配＠ＡＡＡ注解的地方，通过equalsBuilder 这个工具把某个类所有配ＡＡＡ的属性都ｇｅｔ到，然后append到equalsBuilder里，通过它的isequal（） 和toHashCode（） 来就可以轻松判断。这样，以后你所配的ｖｏ的ｆｉｅｌｄ上面，只要是要保证唯一的就可以加这个注解。不用再一个对象一个对象的重写equals和HashCode，是不是很方便呢？

２，配缓存的时候。我们知道，hibernate 提供了ehcache来支持二级缓存，ehcache二级缓存我会在以后仔细讲。因为这个缓存hibernate只提供了@cache这个注解来定义某个ＶＯ，一般情况下这满足了我们的需求，但是我们项目有个特殊的地方，比如要掉另外一个系统的东西，封装写参数传过去，再把返回的东西封装下，这个过程可能需要花很多时间，而这个如果要用ehcache做缓存就比较麻烦一点，如下：

      // 使用默认配置文件创建CacheManager
    CacheManager manager =CacheManager.create();
    // 通过manager可以生成指定名称的Cache对象
    Cache cache = cache =manager.getCache("demoCache");

//往cache中添加元素

    Element element = new Element("key", "value");
    cache.put(element);   看到了吧，每一个ｒｅｓｐｏｎｓｅ就必须通过这种方式一个一个ｐｕｔ进某个ｃａｃｈｅ里，是不是很不人性化？

所以解决思路也就来了，我们就自定义某个注解，比如还是＠ａａａ，通过spring 的aop面向切面　@around(“@annotation（com.aaa)”）　来获取到配置这个注解的类（这种写法是不是很另类呢，），然后request作为key response作为value放入Element，当然　放入之前以这个ｋｅｙ　看缓存里有没有在ｐｕｔ进去，要不就失去缓存的意义了。

综上，大家可以看到，自定义注解还是有很多很多地方能用到的，不仅仅是这点，根据实际情况实际需求来自定义注解，一定会让你的代码更简洁，好用。

