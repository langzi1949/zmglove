---
title: ActiveMQ重连问题
date: 2017-10-13 19:00:08
tags:
    - java
    - ActiveMQ
---
* [问题描述](#active_1)
* [解决方案](#active_2)

<span id="active_1"><strong>问题描述</strong></span>

----
生产发现，当消息中间件activeMQ挂掉后，我们的应用并不能重新连接到activeMQ,而必须通过重启应用才能连接。
本地测试应用持续抛异常：

```java
2017-02-27 20:05:37.297 ERROR [jmsContainer-1] o.s.j.l.DefaultMessageListenerContainer - Could not refresh JMS Connection for destination 'queue://gateway.qly.queue' - retrying using FixedBackOff{interval=5000, currentAttempts=27, maxAttempts=unlimited}. Cause: this connection
org.apache.activemq.AlreadyClosedException: this connection
 at org.apache.activemq.pool.PooledConnection.assertNotClosed(PooledConnection.java:208) ~[activemq-pool-5.7.0.jar:5.7.0]
 at org.apache.activemq.pool.PooledConnection.start(PooledConnection.java:97) ~[activemq-pool-5.7.0.jar:5.7.0]
 at org.springframework.jms.connection.SingleConnectionFactory$SharedConnectionInvocationHandler.localStart(SingleConnectionFactory.java:632) ~[spring-jms-4.2.1.RELEASE.jar:4.2.1.RELEASE]
 at org.springframework.jms.connection.SingleConnectionFactory$SharedConnectionInvocationHandler.invoke(SingleConnectionFactory.java:569) ~[spring-jms-4.2.1.RELEASE.jar:4.2.1.RELEASE]
 at com.sun.proxy.$Proxy58.start(Unknown Source) ~[na:na]
 at org.springframework.jms.listener.AbstractJmsListeningContainer.refreshSharedConnection(AbstractJmsListeningContainer.java:400) ~[spring-jms-4.2.1.RELEASE.jar:4.2.1.RELEASE]
 at org.springframework.jms.listener.DefaultMessageListenerContainer.refreshConnectionUntilSuccessful(DefaultMessageListenerContainer.java:915) [spring-jms-4.2.1.RELEASE.jar:4.2.1.RELEASE]
 at org.springframework.jms.listener.DefaultMessageListenerContainer.recoverAfterListenerSetupFailure(DefaultMessageListenerContainer.java:890) [spring-jms-4.2.1.RELEASE.jar:4.2.1.RELEASE]
 at org.springframework.jms.listener.DefaultMessageListenerContainer$AsyncMessageListenerInvoker.run(DefaultMessageListenerContainer.java:1061) [spring-jms-4.2.1.RELEASE.jar:4.2.1.RELEASE]
 at java.lang.Thread.run(Thread.java:745) [na:1.7.0_79]
```
<!--more-->
<span id="active_2"><strong>解决方案</strong></span>

-----
activeMQ采用failover方式，在断开连接后由client起一个新线程不断从配置的url获取一个url重试连接

```java
failOver:(tcp://xxx.xxx.xxx.xxx:61616)?startupMaxReconnectAttempts=2
```

`startupMaxReconnectAttempts`为启动阶段的最大重连次数，默认-1，无限重连

当使用spring jms连接activeMQ时，经测试brokerURL不配置failOver时应用也会尝试重连，但持续报上述错误。
原因是没有设置`SingleConnectionFactory`对象的`reconnectOnException`属性。
它的默认值是false，意味着即使JMS异常抛出，SingleConnectionFactory自带的connection没有重置。
在调用getConnection()方法时,由于连接没有重置，this.connection != null，仍然返回旧的connection.
从而DefaultMessageListenerContainer引用的连接仍然失败连接，不断抛出异常。
正确配置如下：
```java
 <bean id="connectionFactory"
  class="org.springframework.jms.connection.SingleConnectionFactory">
  <property name="targetConnectionFactory" ref="pooledConnectionFactory" />
  <property name="reconnectOnException" value="true"/>
 </bean>
```
另外，一下配置可以更改断线重连时间，recoveryInterval默认时间5000ms。由于每一个DefaultMessageListenerContainer对象均会尝试重连，
合理设置该属性，避免过量打印错误日志
```java
<bean id="xxxContainer" 
        class="org.springframework.jms.listener.DefaultMessageListenerContainer"> 
        ..................
  <property name="recoveryInterval" value="30000" />
</bean>
```

