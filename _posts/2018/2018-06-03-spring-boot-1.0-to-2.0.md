---
layout: post
title: Spring Boot 1.0 升级到 2.0 版时解决问题记录
no-post-nav: true
category: springboot
tags: [springboot]
---

将项目从 Spring Boot 1.0 升级到 2.0 的时候也遇到了一些问题，在修改的过程中记录下来，今天整理一下分享出来，方便后续升级的朋友少踩一些坑。

1、第一个问题：启动类报错

Spring Boot 部署到 Tomcat 中去启动时需要在启动类添加`SpringBootServletInitializer`，2.0 和 1.0 有区别。

``` java
// 1.0
import org.springframework.boot.web.support.SpringBootServletInitializer;
// 2.0
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;

@SpringBootApplication
public class UserManageApplication extends SpringBootServletInitializer {
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(UserManageApplication.class);
    }

    public static void main(String[] args) throws Exception {
        SpringApplication.run(UserManageApplication.class, args);
    }
}
```

这个问题好解决只需要重新导包就行。

2、日志类报错：Spring Boot 2.0 默认不包含 log4j，建议使用 slf4j 。

```
import org.apache.log4j.Logger;
protected Logger logger = Logger.getLogger(this.getClass());
```

改为：

```
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
protected Logger logger =  LoggerFactory.getLogger(this.getClass());
```

这个也比较好改动，就是需要替换的文件比较多。


3、Spring Boot 2.0 去掉了`findOne()`方法。

以前的`findOne()`方法其实就是根据传入的 Id 来查找对象，所以在 Spring Boot 2.0 的 Repository 中我们可以添加`findById(long id)`来替换使用。

例如：

```
User user=userRepository.findOne(Long id)
```

改为手动在`userRepository`手动添加`findById(long id)`方法，使用时将`findOne()`调用改为`findById(long id)`

```
User user=userRepository.findById(long id)
```

`delete()`方法和`findOne()`类似也被去掉了，可以使用`deleteById(Long id)`来替换，还有一个不同点是`deleteById(Long id)`默认实现返回值为`void`。

```
Long deleteById(Long id);
```

改为

```
//delete 改为 void 类型
void deleteById(Long id);
```

当然我们还有一种方案可以解决上述的两种变化，就是自定义 Sql，但是没有上述方案简单不建议使用。

```
@Query("select t from Tag t where t.tagId = :tagId")
Tag getByTagId(@Param("tagId") long tagId);
```


4、插入数据会报错，错误信息如下：

```
org.springframework.dao.DataIntegrityViolationException: could not execute statement; SQL [n/a]; constraint [PRIMARY]; nested exception is org.hibernate.exception.ConstraintViolationException: could not execute statement
....
Caused by: org.hibernate.exception.ConstraintViolationException: could not execute statement
...
Caused by: com.mysql.jdbc.exceptions.jdbc4.MySQLIntegrityConstraintViolationException: Duplicate entry '299' for key 'PRIMARY'
```

这个问题稍稍花费了一点时间，报错提示的是主键冲突，跟踪数据库的数据发现并没有主键冲突，最后才发现是 Spring Boot 2.0 需要指定主键的自增策略，这个和 Spring Boot 1.0 有所区别，1.0 会使用默认的策略。

```
@Id
@GeneratedValue(strategy= GenerationType.IDENTITY)
private long id;
```

改动也比较简单，需要在所有的主键上面显示的指明自增策略。

5、Thymeleaf 3.0 默认不包含布局模块。

这个问题比较尴尬，当我将 Pom 包升级到 2.0 之后，访问首页的时候一片空白什么都没有，查看后台也没有任何的报错信息，首先尝试着跟踪了 http 请求，对比了一下也没有发现什么异常，在查询 Thymeleaf 3.0 变化时才发现：Spring Boot 2.0 中`spring-boot-starter-thymeleaf` 包默认并不包含布局模块，需要使用的时候单独添加，添加布局模块如下：

```
<dependency>
   <groupId>nz.net.ultraq.thymeleaf</groupId>
   <artifactId>thymeleaf-layout-dialect</artifactId>
</dependency>
```

改完之后再访问首页，一切正常，但是回头查看日志信息发现有一个告警信息：

```
2018-05-10 10:47:00.029  WARN 1536 --- [nio-8080-exec-2] n.n.u.t.decorators.DecoratorProcessor    : The layout:decorator/data-layout-decorator processor has been deprecated and will be removed in the next major version of the layout dialect.  Please use layout:decorate/data-layout-decorate instead to future-proof your code.  See https://github.com/ultraq/thymeleaf-layout-dialect/issues/95 for more information.
2018-05-10 10:47:00.154  WARN 1536 --- [nio-8080-exec-2] n.n.u.t.expressions.ExpressionProcessor  : Fragment expression "layout" is being wrapped as a Thymeleaf 3 fragment expression (~{...}) for backwards compatibility purposes.  This wrapping will be dropped in the next major version of the expression processor, so please rewrite as a Thymeleaf 3 fragment expression to future-proof your code.  See https://github.com/thymeleaf/thymeleaf/issues/451 for more information.
```

跟踪地址看了一下，大概的意思是以前布局的标签已经过期了，推荐使用新的标签来进行页面布局，解决方式也比较简单，修改以前的布局标签 `layout:decorator` 为 `layout:decorate`即可。

6、分页组件`PageRequest`变化。

在 Spring Boot 2.0 中 ，方法`new PageRequest(page, size, sort)` 已经过期不再推荐使用，推荐使用以下方式来构建分页信息：

```
Pageable pageable =PageRequest.of(page, size, Sort.by(Sort.Direction.ASC,"id"));
```

跟踪了一下源码发现`PageRequest.of()`方法，内部还是使用的`new PageRequest(page, size, sort)`，只是最新的写法更简洁一些。

```
public static PageRequest of(int page, int size, Sort sort) {
    return new PageRequest(page, size, sort);
}
```

7、关联查询时候组合返回对象的默认值有变化。

在使用 Spring Boot 1.0 时，使用 Jpa 关联查询时我们会构建一个接口对象来接收结果集，类似如下：

``` java
public interface CollectView{
   Long getId();
   Long getUserId();
   String getProfilePicture();
   String getTitle();
}
```

在使用 Spring Boot 1.0 时，如果没有查询到对应的字段会返回空，在 Spring Boot 2.0 中会直接报空指针异常，对结果集的检查会更加严格一些。




