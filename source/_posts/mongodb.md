---
title: 记一次MongoDB线上问题处理
date: 2017-09-20 16:08:09
tags:
    - java
    - MongoDB
---
* [问题发生](#m1)
* [问题表现](#m2)
* [问题分析](#m3)
* [问题解决](#m4)
* [问题总结](#m5)

<span id="m1"><strong>问题发生</strong></span>
> 2017年8月25日，上午10点前后

---
<span id="m2"><strong>问题表现</strong></span>
> 三台Mongodb集群机器的CPU飙高，一直稳定在<font color="red" size="4">95%</font>以上，<font color="red" size="4">基本已经压死了服务</font>

!["CPU飙升"](/images/mongo_1.jpg)
<!--more-->
!["负载飙升"](/images/mongo_2.jpg)

---
<span id="m3"><strong>问题分析</strong></span>

DBA和运维以及开发，通过上述的现象进行分析，可能是
> 1. MongoDB在使用的时候，连接数没有释放，由于长连接导致的CPU过高。后经研发同事发现，并没有出现这种情况，连接数一直保持在合理的状态。
> 2. MongoDB读的collection已经存储了近10亿的数据，可能有慢查询导致负载过高，但是开启mongo的慢查询监控，并没有发现有慢查询。
> 3. MongoDB写入数据慢，经过DBA的排查，监控MongoDB操作日志，发现写入一条数据贼慢，基本在10s以上

至此，我们发现问题的原因是<font color="red" size="4">MongoDb的写入速度很慢，导致了负载过高。</font>

---
<span id="m4"><strong>问题解决</strong></span>

既然我们已经找出了问题的症结，就要着手去解决它。
> 1. 首先，我们尝试不往原有的旧collection中写入数据，因为旧有数据已经达到10亿级别，这样写入肯定会很慢，所以我们<font color="red" size='3'>*先停止向旧collection中写入，往一张新的collection中写入*，</font>看看能不能使得CPU和负载降下来。做了这一次优化调整，发现依旧没有解决问题，CPU和负载在短时间内又飙升的很高。
> 2. 既然这个法子行不通，我们就要考虑机器本身的问题，运维在服务器上监控，发现数据的IO很慢，由于这个机器所使用的磁盘是普通磁盘，所以磁盘的IO速度很慢，不足以满足大数据量的IO，因此，服务器的扩容和升级刻不容缓。在得到有关领导的授权以后，<table><tr><td bgcolor=yellow>运维进行了阿里云的SSD磁盘采购，SSD的磁盘读写速度相比较普通磁盘优势很明显。</td></tr></table>
> 3. 在采购和升级以及极其耗时的数据迁移过后，重新启动的MongoDB的服务。在短时间内，CPU和负载都降下来很多，就在我们以为服务OK，问题已经得到解决时,<font color="red" size='3'>**突然发现机器的CPU和负载瞬间飙升。**</font>

以下两张图片反应了当时的情况：
!["CPU飙升"](/images/mongo_3.jpg)
!["负载飙升"](/images/mongo_4.jpg)

发现问题并没有得到根本的解决，我们继续分析问题，DBA继续在MongoDB的监控中，查看日志，发现查询的数据很慢，我们`怀疑是不是新建的collection没有加上索引`，排查一看，

**发现新建的两张collection中索引建的有问题，导致查询的速度很慢，占用连接数，进而导致负载和CPU飙高。最后，重新建索引，跟踪一段时间发现问题得到的很好的解决**。

下图是最近几天的负载和CPU情况
!["CPU"](/images/mongo_5.jpg)
!["负载"](/images/mongo_6.jpg)

---
<span id="m5"><strong>问题总结</strong></span>
> 分析问题的时候，要多方面的去考虑，还要和别人一起配合探讨，集思广益，可能不会一步解决问题，但是要敢于尝试。




