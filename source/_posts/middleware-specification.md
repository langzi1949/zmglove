---
title: 中间件使用规范
date: 2017-10-26 09:10:43
tags:
    - 中间件规范
---
现在企业级系统中，各个系统之间的交互，以及系统自身的性能提升，不可避免的需要使用中间件，那么系统之间所需要的中间件需要哪些规范呢？结合我的工作，简单的罗列一下：

------
* <strong>Zookeeper 使用规范</strong>

> 1. `Zookeeper` 做为分布式配置服务、同步服务和命名注册。不建议将 Zookeeper 做为数据存储工具，如有分布式快速存储需求可用Redis代替；
> 
> 2. 不建议在相同节点下面放置过多子节点或数据；
> 
> 3. 将`Zookeeper` 做为分布式锁用时不建议自己去写客户端实现，用 `Curator` 现成的分布式锁 `InterProcessMutex` ( [http://www.jianshu.com/p/5d12a01018e1](http://www.jianshu.com/p/5d12a01018e1) );

<!--more-->

* <strong>Redis 使用规范</strong>

> 1. `Redis` 使用时务必设置超时时间;
> 
> 2. 不建议存储大量超大Bean(不必要缓存的Bean字段可精简);
> 
> 3. 不建议在一个集合下面放置过多的 Value；

* <strong>MQ 使用规范</strong>

> 1. 队列或消息在使用时建议设置`Priority`(优先级)，高优先级的队列与消息会优先消费 ([http://www.cnblogs.com/huangxincheng/p/6029214.html](http://www.cnblogs.com/huangxincheng/p/6029214.html) ) ;
> 
> 2. `consumer` 在配置时建议根据业务场景配置并发线程，不宜过高，也不建议配置单线程；

* <strong>中间件运维规范</strong>

> 1. 中间件在使用时务必做到故障隔离，将不同场景或相同场景不同用途的中间件做到故障隔离；
> 
> 2. 建议能给出中间件的部署方案或集群方案图，以便在调优与出故障时做隔离；


