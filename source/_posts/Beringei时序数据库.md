---
title: Beringei时序数据库
date: 2017-03-13 08:18:16
comments: true
categories: 数据
tags: [数据库, 时序, Beringei, facebook]
---

# Beringei时序数据库
Facebook于2017年2月3日宣布开源他们的高性能时序数据存储引擎Beringer。Berigei内存数据库是用来解决Facebook内部监控数据存储和查询需求的数据库，具有快速读写、高压缩比等特性。本文将从应用场景、设计思路和特点详细介绍Beringei。
## Beringer诞生背景
Facebook运维大规模的分布式服务，需要监控系统的运行状况和性能指标，以便第一时间发现、诊断、处理出现的问题。Facebook使用时序数据库（TSDB）跟踪和存储系统度量指标，如：cpu、内存、网络，以及服务的统计信息（缓存命中率、mysql查询速率）等，基于这些数据运维人员就可以看到基础设施上的实时负载情况，并作出相应的决策。

Facebook早期采用采用Hbase存储ODS（Operational Data Store）数据。随着系统的不断发展，2013年初，存储引擎监控团队发现Hbase无法灵活扩展，将导致未来无法处理高并发的读取负载。如：同时分析几千个时序数据，将需要几十秒的时间才能返货，对于稀疏数据查询执行时间更长，甚至会出现查询超时。存储引擎监控团队评估和否决了几款基于磁盘和现有内存缓存的解决方案（OpenTSDB压缩率不足，还可能导致数据精度下降；Whisper不支持数据时序抖动，且不是内存数据库，性能不足；InfluxDB由于支持Meta存储，压缩率不足导致内存利用率下降）。

为了满足以上场景和解决以上问题，Beringer必须满足高并发写（TPS：1000w/s）、快速响应查询（如：小窗口聚集查询——最近1~2小时的聚合操作）、存储最近26小时的数据共读写（Facebook 85%的查询是最近26小时的数据）、采用ClusterFS持久化数据、容错能力（容灾、宕机恢复）、水平扩展等场景。 Facebook监控团队在VLD2015大会上发表一篇名为《Gorilla：A Fast, Scalable, In-Memory Time Series Database》的文章，Beringer正是基于这项工作成果的进一步发展。

## Beringer设计思路
设计Beringer的初衷是为了满足更高的写入速率和更低的读取延迟，同时尽可能高效的利用内存来存储时间序列的数据。Facebook团队通过分析监控数据发现，大多数时间序列中的值和相邻数据点的值并没有显著变化且许多数据源值存储整数（系统支持浮点数），所以Facebook采用delta-of-delta编码压缩时间戳，采用XOR压缩64位的浮点数。接下来让我们介绍一下

### 时间戳压缩算法
时间戳的压缩算法采用delta-of-delta编码，具体的算法步骤如下：
1. 头部存储序列的起始时间戳$t_1$，他与两小时的窗口对其，第一个时间戳$t_0$采用14bit存储$t_1-t_2$的deta值;
2. 对于接下来的时间戳$t_n$:
    1. 计算delta of delta：$D=(t_{n-t}-t_{n-1})-(t_{n-1}-t_{n-2})$;
    2. 如果D=0，则存储bit '0'；
    3. 如果$D\in[-63, 64]$，存储'10'，然后在接下来的7bit中存储delta值；
    4. 如果$D\in[-255, 256]$，存储'110'，然后在接下来的9bit中存储delta值；
    5. 如果$D\in[-2047, 2048]$，存储'1110'，然后在接下来的12bit中存储delta值；
    6. 否则，存储'1111'，然后在接下来的32bit中存储delta值；

如** 图1. ** 采用时间压缩算法得：Header存储时间T 2015.03.24 02:00:00；紧接着存储第一个时间点$T_1$ 2015.03.24 02:00:00与Hearder T时间2015.03.24 02:01:02的$t$差值62；第二个时间点$T_2$ 2015.03.24 02:02:02与$t_1$的差值$deltaT_1$ 60，$T_2$ 60与$T_1$ 62的差值-2(既delta of delta),则存储标记位'10' 然后存储值-2；$T_3$ 2015.03.24 02:03:02与$T_2$ 2015.03.24 02:02:02的差值$delta_3$为0，直接存储标记位'0'。

### 值压缩算法
值的压缩算法采用XOR编码，具体的算法步骤如下：

1. 第一个值不压缩，直接存储值；
2. 如果XOR的值为0（与之前的值一样），则存储bit '0'
3. 如果XOR的值不为0，计算XOR后前置0和尾部0的值，存储bit '1'，然后存储a）或b）：
&emsp; a)控制位'0'，如果meaning值的bit块落在之前meaning值的bit块内，（当前块前置0和后置0的个数与之前值的个数一样），存储meaning值的bit块；
&emsp; b)控制位'1'，否则，采用5bit存储前置0的位置值，然后采用6bit存储XOR后的meaning值；

如** 图1. ** 采用值压缩算法得：$T_1$时刻的值$Value_1$ 12 直接存储；$T_2$时刻的值$Value_2$ 12与$T_1$时刻的值$Value_1$ 12的差值$deltaV_2$ 0，所以直接存储标记位'0'；因为$T_2$时刻没有有meaning值，所以   $T_3$时刻的值$Value_3$ 24 与$T_2$ 时刻的值做XOR运算，存储控制为'11',然后采用5bit存储XOR值的第一个bit位为1的值11，接着采用6 bit存储XOR后meaning值的bit的长度1，最后存储meaning值的bit块。（如果XOR之后的Meaning块落在之前XOR后的Meaning块内，则存储控制位'10'，然后存储Meaning值的bit块）

{% img full-image '/images/Beringei时序数据库/graph1.png' 图1. 压缩算法 %}

采用以上两个算法压缩Facebook的监控数据，可知压缩比极高。
时间戳压缩算法，得到各标志位的分布比例如图2。96%的时间戳可以用1bit存储，压缩比极高。

{% img full-image '/images/Beringei时序数据库/graph2.png' 图2. 时间戳压缩算法各bit位分布比 %}

分体值压缩算法，得到各标志位的分布比例如图3。其中标志位'0'占1bit占比51%，标志位'10'的值占26.6bit占比30%，标志位'11'占36.9bit占比19%。
{% img full-image '/images/Beringei时序数据库/graph3.png' 图3. 值压缩算法各bit位分布比 %}

### 存储结构
每两小时产生一个Block块。ShardMap为Vector存储Block块的key和TSmap的地址映射关系。TSmap存储所shards Block块地址和key的映射关系，这样就可以快速复制内存中的数据。Facebook内存存储结构，当无头的数据块还没有刷新时宕机，数据将会丢失。当然，内存数据同样需要持久化硬盘上，当块形成后，Facebook采用Hbase持久化内存数据块。

{% img full-image '/images/Beringei时序数据库/graph4.png' 图4. 值压缩算法各bit位分布比 %}

## Beringei总结
虽然Beringei算法简单，压缩比和性能都有较大的提高，但是采用Beringei存储监控数据需要case by case的分析业务场景，只有业务场景和Beringei设计原则吻合才能达到较好的压缩比。其次，Beringei部署需要依赖较多的其他框架，部署环境不友好，且Beringei需要持久化数据库支撑。

### 参考链接
[Gorilla论文地址](http://www.vldb.org/pvldb/vol8/p1816-teller.pdf)
[Beringei代码库](https://github.com/facebookincubator/beringei)
[深度解读Facebook刚开源的beringei时序数据库](https://yq.aliyun.com/articles/69354)
[Facebook开源内存数据库Beringei，追求极致压缩率](http://weixin.niurenqushi.com/article/2017-02-12/4766476.html)
[浮点数标示方法：fast lossless compression of scientific floating-point data](http://www.cnblogs.com/mlog/archive/2010/12/16/2456368.html)
