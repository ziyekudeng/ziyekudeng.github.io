---
layout: post
title: 15个Spring的核心注释示例
category: spring
tags: [spring]
---



众所周知，Spring DI和Spring IOC是Spring Framework的核心概念。让我们从org.springframework.beans.factory.annotation和org.springframework.context.annotation包中探索一些Spring核心注释。我们经常将这些称为“Spring核心注释”，我们将在本文中对它们进行讲解。

# 这是所有已知的Spring核心注释的列表。

## `@Autowired`

我们可以使用@Autowired注释来标记Spring将要解析和注入的依赖关系。我们可以将这个注释与构造函数，setter或字段注入一起使用

*   构造器注入

1.  `@RestController`

2.  `public class CustomerController {`

3.  `private CustomerService customerService;`

4.  `@Autowired`

5.  `public CustomerController(CustomerService customerService) {`

6.  `   this.customerService = customerService;`

7.  `}`

8.  `}`

*   setter注入

1.  `import org.springframework.beans.factory.annotation.Autowired;`

2.  `import org.springframework.web.bind.annotation.RestController;`

3.  `@RestController`

4.  `public class CustomerController {`

5.  `private CustomerService customerService;`

6.  `@Autowired`

7.  `public void setCustomerService(CustomerService customerService) {`

8.  `   this.customerService = customerService;`

9.  `}`

10.  `}`

*   领域注入

1.  `import org.springframework.beans.factory.annotation.Autowired;`

2.  `import org.springframework.web.bind.annotation.RestController;`

3.  `@RestController`

4.  `public class CustomerController {`

5.  `@Autowired`

6.  `private CustomerService customerService;`

7.  `}`

## `@Bean`

*   `@Bean`是方法级注释，是XML元素的直接模拟。 注释支持一些提供的属性，例如init-method，destroy-method，auto-wiring和name。

*   您可以在 `@Configuration`注解或 `@Component`注解类中使用 `@Bean`批注

*   以下是

    方法声明的简单示例：

    上述配置等效于以下Spring XML：

1.  `<beans>`

2.  `   <bean id="customerService" class="com.companyname.projectname.CustomerService"/>`

3.  `   <bean id="orderService" class="com.companyname.projectname.OrderService"/>`

4.  `</beans>`

5.  `import org.springframework.context.annotation.Bean;`

6.  `import org.springframework.context.annotation.Configuration;`

7.  `import com.companyname.projectname.customer.CustomerService;`

8.  `import com.companyname.projectname.order.OrderService;`

9.  `@Configuration`

10.  `public class Application {`

11.  `@Bean`

12.  `public CustomerService customerService() {`

13.  `   return new CustomerService();`

14.  `}`

15.  `@Bean`

16.  `public OrderService orderService() {`

17.  `   return new OrderService();`

18.  `}`

19.  `}`

20.  `@Bean`

## `@Qualifier`

此注释有助于微调基于注释的自动布线。 可能存在这样的情况：我们创建多个相同类型的bean，并且只想使用属性连接其中一个bean。 这可以使用@ `Qualifier`注释以及 `@Autowired`注释来控制。

示例：考虑使用EmailService和SMSService类来实现单个MessageService接口

*   为多个消息服务实现创建MessageService接口。

1.  `public interface MessageService {`

2.  `public void sendMsg(String message);`

3.  `}`

*   接下来，创建实现：EmailService和SMSService。

1.  `public class SMSService implements MessageService{`

2.  `public void sendMsg(String message) {`

3.  `System.out.println(message);`

4.  `}`

5.  `}`

6.  `public class EmailService implements MessageService{`

7.  `public void sendMsg(String message) {`

8.  `System.out.println(message);`

9.  `}`

10.  `}`

*   这时候该看看 `@Qualifier`注释的用法了

1.  `public interface MessageProcessor {`

2.  `public void processMsg(String message);`

3.  `}`

4.  `public class MessageProcessorImpl implements MessageProcessor {`

5.  `private MessageService messageService;`

6.  `// setter based DI`

7.  `@Autowired`

8.  `@Qualifier("emailService")`

9.  `public void setMessageService(MessageService messageService) {`

10.  `   this.messageService = messageService;`

11.  `}`

12.  `// constructor based DI`

13.  `@Autowired`

14.  `public MessageProcessorImpl(@Qualifier("emailService") MessageService messageService) {`

15.  `   this.messageService = messageService;`

16.  `}`

17.  `public void processMsg(String message) {`

18.  `   messageService.sendMsg(message);`

19.  `}`

20.  `}`

## `@Required`

`@Required` 注释是一个方法级注释，并应用于bean的setter方法。此注释仅指示必须将setter方法配置为在配置时使用值依赖注入。例如，对setter方法的 `@Required`标记了我们想要通过XML填充的依赖项：

1.  `@Required`

2.  `void setColor(String color) {`

3.  `   this.color = color;`

4.  `}`

5.  `<bean class="com.javaguides.spring.Car">`

6.  `   <property name="color" value="green" />`

7.  `</bean>`

否则，将抛出BeanInitializationException。

## `@Value`

Spring `@Value` 注释用于为变量和方法参数指定默认值。我们可以使用@Value 注释来读取Spring环境变量以及系统变量 。Spring `@Value` 注释也支持SpEL。让我们看一下使用@Value 注释的一些示例 。 示例：我们可以使用@Value 注释为类属性指定默认值 。

1.  `   @Value("Default DBConfiguration")`

2.  `   private String defaultName;`

该 @Value 注释参数可以是只有字符串，但春天尝试将其转换为指定的类型。以下代码将正常工作，并将布尔值和整数值分配给变量。

1.  `   @Value("true")`

2.  `   private boolean defaultBoolean;`

4.  `   @Value("10")`

5.  `   private int defaultInt;`

这演示了Spring `@Value` - Spring环境变量

1.  `   @Value("${APP_NAME_NOT_FOUND}")`

2.  `   private String defaultAppName;`

接下来，使用 `@Value` 注释分配系统变量 。

1.  `   @Value("${java.home}")`

2.  `   private String javaHome;`

4.  `   @Value("${HOME}")`

5.  `   private String homeDir;`

Spring @Value – SpEL

1.  `   @Value("#{systemProperties['java.home']}")`

2.  `   private String javaHome;`

## `@DependsOn`

该 `@DependsOn` 注释可以强制的Spring IoC容器中的bean，它是由注释之前初始化一个或多个bean `@DependsOn` 注释。

所述 `@DependsOn` 注释可以在直接或间接地注释与任何类使用 `@Component` 或与所述注解的方法 `@Bean`。

示例：让我们创建 `FirstBean` 和 `SecondBean` 类。在此示例中， `SecondBean` 在 `bean`之前初始化 `FirstBean`。

1.  `public class FirstBean {`

2.  `   @Autowired`

3.  `   private SecondBean secondBean;`

4.  `}`

5.  `public class SecondBean {`

6.  `   public SecondBean() {`

7.  `       System.out.println("SecondBean Initialized via Constuctor");`

8.  `   }`

9.  `}`

基于配置类在Java中声明上述bean。

1.  `@Configuration`

2.  `public class AppConfig {`

3.  `   @Bean("firstBean")`

4.  `   @DependsOn(value = {`

5.  `       "secondBean"`

6.  `   })`

7.  `   public FirstBean firstBean() {`

8.  `       return new FirstBean();`

9.  `   }`

10.  `   @Bean("secondBean")`

11.  `   public SecondBean secondBean() {`

12.  `       return new SecondBean();`

13.  `   }`

14.  `}`

## `@Lazy`

默认情况下，Spring IoC容器在应用程序启动时创建并初始化所有单例bean。我们可以通过使用 `@Lazy` 注释来防止单例bean的这种预初始化 。所述 @Lazy 注释可以在任何类中使用，与直接或间接地注释 `@Component` 或与所述注解的方法 `@Bean`。

示例：考虑我们有两个bean - FirstBean 和 SecondBean。在此示例中，我们将FirstBean 使用 `@Lazy`注释显式加载。

1.  `public class FirstBean {`

2.  `   public void test() {`

3.  `       System.out.println("Method of FirstBean Class");`

4.  `   }`

5.  `}`

6.  `public class SecondBean {`

7.  `   public void test() {`

8.  `       System.out.println("Method of SecondBean Class");`

9.  `   }`

10.  `}`

基于配置类在Java中声明上述bean。

1.  `@Configuration`

2.  `public class AppConfig {`

3.  `   @Lazy(value = true)`

4.  `   @Bean`

5.  `   public FirstBean firstBean() {`

6.  `       return new FirstBean();`

7.  `   }`

8.  `   @Bean`

9.  `   public SecondBean secondBean() {`

10.  `       return new SecondBean();`

11.  `   }`

12.  `}`

我们可以看到，bean secondBean 由Spring容器初始化，而bean firstBean 则被显式初始化。

## `@Lookup`

注释的方法 `@Lookup` 告诉Spring在我们调用它时返回方法返回类型的实例。

## `@Primary`

我们使用 `@Primary` 当存在多个相同类型的bean时，我们使用它 给bean更高的优先级。

1.  `@Component`

2.  `@Primary`

3.  `class Car implements Vehicle {}`

4.  `@Component`

5.  `class Bike implements Vehicle {}`

6.  `@Component`

7.  `class Driver {`

8.  `   @Autowired`

9.  `   Vehicle vehicle;`

10.  `}`

11.  `@Component`

12.  `class Biker {`

13.  `   @Autowired`

14.  `   @Qualifier("bike")`

15.  `   Vehicle vehicle;`

16.  `}`

## `@Scope`

我们使用@ Scope注释来定义 `@Component`类的范围或 `@Bean`定义。 它可以是单例，原型，请求，会话，globalSession或某些自定义范围。 举个例子:

1.  `@Component`

2.  `@Scope(value = ConfigurableBeanFactory.SCOPE_SINGLETON)`

3.  `public class TwitterMessageService implements MessageService {`

4.  `}`

5.  `@Component`

6.  `@Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)`

7.  `public class TwitterMessageService implements MessageService {`

8.  `}`

## `@Profile`

如果我们希望Spring仅在特定配置文件处于活动状态时使用 `@Component`类或 `@Bean`方法，我们可以使用 `@Profile`标记它。 我们可以使用注释的value参数配置配置文件的名称：

1.  `@Component`

2.  `@Profile("sportDay")`

3.  `class Bike implements Vehicle {}`

## `@Import`

该 `@Import` 注释指示一个或多个 `@Configuration`类进口。

例如：在基于Java的配置中，Spring提供了 `@Import`注释，允许从另一个配置类加载 `@Bean`定义。

1.  `@Configuration`

2.  `public class ConfigA {`

3.  `   @Bean`

4.  `   public A a() {`

5.  `       return new A();`

6.  `   }`

7.  `}`

8.  `@Configuration`

9.  `@Import(ConfigA.class)`

10.  `public class ConfigB {`

11.  `   @Bean`

12.  `   public B b() {`

13.  `       return new B();`

14.  `   }`

15.  `}`

现在，在实例化上下文时，不需要同时指定ConfigA类和ConfigB类，只需要显式提供ConfigB。

## `@ImportResource`

Spring提供了一个 `@ImportResource`注释，用于将 `applicationContext.xml`文件中的bean加载到ApplicationContext中。 例如：考虑我们在类路径上有 `applicationContext.xml` Spring bean配置XML文件。

1.  `@Configuration`

2.  `@ImportResource({"classpath*:applicationContext.xml"})`

3.  `public class XmlConfiguration {`

4.  `}`

## `@PropertySource`

该 @PropertySource 注释提供了一种方便的声明性机制，用于添加 PropertySource Spring的Eenvironment以与@Configuration类一起使用 。

例如，我们从文件config.properties文件中读取数据库配置，并使用Environment 将这些属性值设置为 DataSourceConfig类。

1.  `import org.springframework.beans.factory.InitializingBean;`

2.  `import org.springframework.beans.factory.annotation.Autowired;`

3.  `import org.springframework.context.annotation.Configuration;`

4.  `import org.springframework.context.annotation.PropertySource;`

5.  `import org.springframework.core.env.Environment;`

6.  `@Configuration`

7.  `@PropertySource("classpath:config.properties")`

8.  `public class ProperySourceDemo implements InitializingBean {`

9.  `   @Autowired`

10.  `   Environment env;`

11.  `   @Override`

12.  `   public void afterPropertiesSet() throws Exception {`

13.  `       setDatabaseConfig();`

14.  `   }`

15.  `   private void setDatabaseConfig() {`

16.  `       DataSourceConfig config = new DataSourceConfig();`

17.  `       config.setDriver(env.getProperty("jdbc.driver"));`

18.  `       config.setUrl(env.getProperty("jdbc.url"));`

19.  `       config.setUsername(env.getProperty("jdbc.username"));`

20.  `       config.setPassword(env.getProperty("jdbc.password"));`

21.  `       System.out.println(config.toString());`

22.  `   }`

23.  `}`

## `@PropertySources`

我们可以使用此批注指定多个 `@PropertySource`配置：

1.  `@PropertySources({`

2.  ` @PropertySource("classpath:config.properties"),`

3.  ` @PropertySource("classpath:db.properties")`

4.  `})`

5.  `public class AppConfig {`

6.  ` //...`

7.  `}`

希望您喜欢这篇关于项目最佳Spring注释的帖子！快乐的编码！

