---
layout: post
title: JUnit4注解基本介绍 
category: java
tags: [java]
---



# JUnit4注解基本介绍

 2013年08月01日 09:28:44 [sean-zou](https://me.csdn.net/a19881029) 阅读数：21454

 版权声明：本文为博主原创文章，未经博主允许不得转载。 https://blog.csdn.net/a19881029/article/details/9668969

**@After**

If you allocate external resources in a Before method you need to release them after the test runs.Annotating a public void method with @After causes that method to be run after the Test method. All @After methods are guaranteed to run even if a Before or Test method throws an exception. The @After methods declared in superclasses will be run after those of the current class.

如果在@Before注解方法中分配了额外的资源，那么在测试执行完后，需要释放分配的资源。

使用@After注解一个public void方法会使该方法在@Test注解方法执行后被执行

即使在@Before注解方法、@Test注解方法中抛出了异常，所有的@After注解方法依然会被执行，见示例

父类中的@After注解方法会在子类@After注解方法执行后被执行

<code class="language-java">public class MathTest {
	@Before
	public void setUp() throws Exception {
		throw new Exception();
	}

	@Test
	public void testAdd() {
		Math m = new Math();
		assertTrue(m.add(1, 1) == 2);
	}

	@After
	public void tearDown() throws Exception {
		System.out.println("after");
	}
}</code><code class="language-plain">after</code>![](https://img-blog.csdn.net/20130731180209203?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE5ODgxMDI5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

**@AfterClass**

If you allocate expensive external resources in a Before Class method you need to release them after all the tests in the class have run. Annotating a public static void method with @AfterClass causes that method to be run after all the tests in the class have been run. All @AfterClass methods are guaranteed to run even if a Before Class method throws an exception.The @AfterClass methods declared in superclasses will be run after those of thecurrent class.

如果在@BeforeClass注解方法中分配了代价高昂的额外的资源，那么在测试类中的所有测试方法执行完后，需要释放分配的资源。

使用@AfterClass注解一个public static void方法会使该方法在测试类中的所有测试方法执行完后被执行

即使在@BeforeClass注解方法中抛出了异常，所有的@AfterClass注解方法依然会被执行

父类中的@AfterClass注解方法会在子类@AfterClass注解方法执行后被执行

**@Before**

When writing tests, it is common to find that several tests need similar objects created before they can run. Annotating a public void method with @Before causes that method to be run before the Test method. The @Before methods of superclasses will be run before those of the current class. No other ordering is defined.

当编写测试方法时，经常会发现一些方法在执行前需要创建相同的对象

使用@Before注解一个public void 方法会使该方法在@Test注解方法被执行前执行（那么就可以在该方法中创建相同的对象）

父类的@Before注解方法会在子类的@Before注解方法执行前执行

**@BeforeClass**

Sometimes several tests need to share computationally expensive setup (like logging into a database). While this can compromise the independence of tests, sometimes it is a necessary optimization.Annotating a public static void no-arg method with @BeforeClass causes it to be run once before any of the test methods in the class. The @BeforeClass methods of superclasses will be run before those the current class.

有些时候，一些测试需要共享代价高昂的步骤（如数据库登录），这会破坏测试独立性，通常是需要优化的

使用@BeforeClass注解一个public static void 方法，并且该方法不带任何参数，会使该方法在所有测试方法被执行前执行一次，并且只执行一次

父类的@BeforeClass注解方法会在子类的@BeforeClass注解方法执行前执行

**@Ignore**

Sometimes you want to temporarily disable a test or a group of tests. Methods annotated with Test that are also annotated with @Ignore will not be executed as tests. Also, you can annotate a class containing test methods with @Ignore and none of the containing tests will be executed. Native JUnit 4 test runners should report the number of ignored tests along with the number of tests that ran and the number of tests that failed.

对包含测试类的类或@Test注解方法使用@Ignore注解将使被注解的类或方法不会被当做测试执行

JUnit执行结果中会报告被忽略的测试数

<code class="language-java">public class MathTest {
	@Ignore("do not test")
	@Test
	public void testAdd() {
		Math m = new Math();
		assertTrue(m.add(1, 1) == 2);
	}
}</code><code class="language-java">@Ignore
public class MathTest {
	@Test
	public void testAdd() {
		Math m = new Math();
		assertTrue(m.add(1, 1) == 2);
	}
}</code>

执行结果相同：

![](https://img-blog.csdn.net/20130731172321953?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE5ODgxMDI5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

**@Test**

The Test annotation tells JUnit that the public void method to which it is attached can be run as a test case. To run the method, JUnit first constructs a fresh instance of the class then invokes the annotated method. Any exceptions thrown by the test will be reported by JUnit as a failure. If no exceptions are thrown, the test is assumed to have succeeded.

The Test annotation supports two optional parameters.

The first, expected,declares that a test method should throw an exception. If it doesn't throw an exception or if it throws a different exception than the one declared, the test fails.

The second optional parameter, timeout, causes a test to fail if it takes longer than a specified amount of clock time (measured in milliseconds).

@Test注解的public void方法将会被当做测试用例

JUnit每次都会创建一个新的测试实例，然后调用@Test注解方法

任何异常的抛出都会认为测试失败

@Test注解提供2个参数：

1，“expected”，定义测试方法应该抛出的异常，如果测试方法没有抛出异常或者抛出了一个不同的异常，测试失败

2，“timeout”，如果测试运行时间长于该定义时间，测试失败（单位为毫秒）

<code class="language-java">public class MathTest {
	@Test(expected=Exception.class)
	public void testAdd() throws Exception{
		throw new Exception();
	}
}</code>![](https://img-blog.csdn.net/20130801092840062?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE5ODgxMDI5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)<code class="language-java">public class MathTest {
	@Test(timeout=5000)
	public void testAdd() {
		for(;;){

		}
	}
}</code>![](https://img-blog.csdn.net/20130801092410531?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYTE5ODgxMDI5/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

