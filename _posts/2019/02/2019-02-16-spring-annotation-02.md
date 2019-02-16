---
layout: post
title: spring事务不回滚throw的Exception异常的解决方法
category: spring
tags: [spring]
---



 spring事务中，throw的Exception异常事务不会回滚，原因是：
用 spring 事务管理器,由spring来负责数据库的打开,提交,回滚.默认遇到运行期例外(throw new RuntimeException("注释");)会回滚，即遇到不受检查（unchecked）的例外时回滚；而遇到需要捕获的例外(throw new Exception("注释");)不会回滚,即遇到受检查的例外（就是非运行时抛出的异常，编译器会检查到的异常叫受检查例外或说受检查异常）时，需我们指定方式来让事务回滚 ：要想所有异常都回滚,要加上 @Transactional( rollbackFor={Exception.class,其它异常}) .如果让unchecked例外不回滚： @Transactional(notRollbackFor=RunTimeException.class)
如下:
@Transactional(rollbackFor=Exception.class) //指定回滚,遇到异常Exception时回滚
public void methodName() {
throw new Exception("注释");
}

@Transactional(noRollbackFor=Exception.class)//指定不回滚,遇到运行期例外(throw new RuntimeException("注释");)会回滚
public ItimDaoImpl getItemDaoImpl() {
throw new RuntimeException("注释");
}

方法1：@Transactional注解指定rollbackFor=Exception.class

方法2：让throw的自定义Exception继承RuntimeException

方法3：使用自定义注解，处理回滚Exception的问题：
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Transactional(rollbackFor=Exception.class)
public @interface DolTransactional {

}


