---
title: 去除只读操作中的事务控制
date: 2017-09-09 11:23:09
tags:
    - java
---
java中事务的管理有两类: 声明式和编程式. 编程式实用比较繁琐耦合度较高不常用,这里主要讲下声明式事务.

---

* **声明式事务常用的使用方法有两种:**

> 1. 基于AspectJ的方式
    通过spring的面向切面编程AOP来进行配置，按照方法的名字进行管理，无需在类中添加任何东西.
> 2. 基于注解的方式
    配置简单，在业务层类上添加注解`@Transactional`.

<!--more-->
配置事例:
```java
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">

   <!--dataSource是数据库连接的数据源-->

<property name="dataSource" ref="dataSource" />

</bean>

    <tx:advice id="txAdvice" transaction-manager="transactionManager">

       <tx:attributes>

           <tx:method name="add*" propagation="REQUIRES_NEW" rollback-for="Exception"/>

           <tx:method name="del*" propagation="REQUIRES_NEW" rollback-for="Exception"/>

           <tx:method name="save*" propagation="REQUIRES_NEW" rollback-for="Exception"/>

           <tx:method name="update*" propagation="REQUIRES_NEW" rollback-for="Exception"/>

  <tx:method name="*" propagation="NOT_SUPPORTED" rollback-for="Exception" />  

       </tx:attributes>

   </tx:advice>

   <aop:config proxy-target-class="true">

      <aop:pointcut expression="execution(* com.zmglove.user.service.*.*(..))" id="pointcut" />

      <aop:advisor advice-ref="txAdvice" pointcut-ref="pointcut"/>

   </aop:config>
```
解释:
1. 传播特性:
> REQUIRED（默认值）：在有transaction状态下执行；如当前没有transaction，则创建新的transaction；
> SUPPORTS：如当前有transaction，则在transaction状态下执行；如果当前没有transaction，在无transaction状态下执行；
> MANDATORY：必须在有transaction状态下执行，如果当前没有transaction，则抛出异常IllegalTransactionStateException；
> REQUIRES_NEW：创建新的transaction并执行；如果当前已有transaction，则将当前transaction挂起；
> NOT_SUPPORTED：在无transaction状态下执行；如果当前已有transaction，则将当前transaction挂起
> NEVER：在无transaction状态下执行；如果当前已有transaction，则抛出异常IllegalTransactionStateException。 

2. 切点pointcut:
> 如上配置对service业务逻辑层进行切面添加事务, 对于service层中以add,del,save,update开头的接口是需要事务控制的,默认其他是不需要的,如需要事务控制可以在对应实现接口方法上面添加支持事务的注解:
```java
@Transactional(propagation = Propagation.REQUIRES_NEW, rollbackFor = Exception.class)
```
---
所以为了提高系统性能,在只读的接口上去掉事务的控制. 配置可参考如上非add,del,save,update开头的接口和没有`@Transactional`注解的接口都是不支持事务的.
