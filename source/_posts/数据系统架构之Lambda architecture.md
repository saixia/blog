---
title: 数据系统架构之Lambda architecture
date: 2016-06-28 19:53:05
comments: true
categories: 数据
tags: [Lambda architecture, 数据架构]
---

# 传统系统的问题

“我们正在从IT时代走向DT时代(数据时代)。IT和DT之间，不仅仅是技术的变革，更是思想意识的变革，IT主要是为自我服务，用来更好地自我控制和管理，DT则是激活生产力，让别人活得比你好”——阿里巴巴董事局主席马云。

数据量从M的级别到G的级别到现在T的级、P的级别。数据量的变化，数据管理系统（DBMS）和数仓系统（DW）也在悄然的变化着。
传统应用的数据系统架构设计时，应用直接访问数据库系统。当用户访问量增加时，数据库无法支撑日益增长的用户请求的负载时，从而导致数据库服务器无法及时响应用户请求，出现超时的错误。出现这种情况以后，在系统架构上就采用图（A）的架构，在数据库和应用中间过一层缓冲隔离，缓解数据库的读写压力。
{% img full-image '/images/数据系统架构之Lambda architecture/graph1.png' 图（A） %} 
然而，当用户访问量持续增加时，就需要考虑读写分离技术（Master－Slave）架构如图（B），分库分表技术。现在，架构变得越来越复杂了，增加队列、分区、复制等处理逻辑。应用程序需要了解数据库的schema，才能访问到正确的数据。  
{% img full-image '/images/数据系统架构之Lambda architecture/graph2.png' 图（B） %} 

# Lambda架构的背景

大数据处理技术需要解决这种可伸缩性与复杂性。  
首先要认识到这种分布式的本质，要很好地处理分区与复制，不会导致错误分区引起查询失败，而是要将这些逻辑内化到数据库中。当需要扩展系统时，可以非常方便地增加节点，系统也能够针对新节点进行rebalance。  
其次是要让数据成为不可变的。原始数据永远都不能被修改，这样即使犯了错误，写了错误数据，原来好的数据并不会受到破坏。  
Storm的作者Nathan Marz提出的一个实时大数据处理框架（Lambda架构）就满足以上两点。Marz在Twitter工作期间开发了著名的实时大数据处理框架Storm，Lambda架构是其根据多年进行分布式大数据系统的经验总结提炼而成。  
Lambda架构的目标是设计出一个能满足实时大数据系统关键特性的架构，包括有：高容错、低延时和可扩展等。Lambda架构整合离线计算和实时计算，融合不可变性（Immunability），读写分离和复杂性隔离等一系列架构原则，可集成Hadoop，Kafka，Storm，Spark，Hbase等各类大数据组件。

# 大数据系统的关键特性
Marz介绍Big Data System许具备的属性：
a、Robust and fault-tolerant（容错性和鲁棒性）：对大规模分布式系统来说，机器是不可靠的，可能会当机，但是系统需要是健壮、行为正确的，即使是遇到机器错误。除了机器错误，人更可能会犯错误。在软件开发中难免会有一些Bug，系统必须对有Bug的程序写入的错误数据有足够的适应能力，所以比机器容错性更加重要的容错性是人为操作容错性。对于大规模的分布式系统来说，人和机器的错误每天都可能会发生，如何应对人和机器的错误，让系统能够从错误中快速恢复尤其重要。
b、Low latency reads and updates（低延时）：很多应用对于读和写操作的延时要求非常高，要求对更新和查询的响应是低延时的。
c、Scalable（横向扩容）：当数据量/负载增大时，可扩展性的系统通过增加更多的机器资源来维持性能。也就是常说的系统需要线性可扩展，通常采用scale out（通过增加机器的个数）而不是scale up（通过增强机器的性能）。
d、General（通用性）：系统需要能够适应广泛的应用，包括金融领域、社交网络、电子商务数据分析等。
e、Extensible（可扩展）：需要增加新功能、新特性时，可扩展的系统能以最小的开发代价来增加新功能。
f、Allows ad hoc queries（方便查询）：数据中蕴含有价值，需要能够方便、快速的查询出所需要的数据。
d、Minimal maintenance（易于维护）：系统要想做到易于维护，其关键是控制其复杂性，越是复杂的系统越容易出错、越难维护。
h、Debuggable（易调试）：当出问题时，系统需要有足够的信息来调试错误，找到问题的根源。其关键是能够追根溯源到每个数据生成点。

# 数据系统的本质

Marz认为：数据系统通过查询过去的（部分、全部）数据去回答问题。如：他是一个什么样的人？他有多少朋友？这个账号是否收支平衡？。因此，Data System的通用定义为：
Query ＝ Function（all data）。
对通用的表达式进行分解得到：
数据系统 ＝ 数据 ＋ 查询
从而可以从数据和查询两个方面认识大数据系统的本质。

## 数据本本质：When&What

数据是一个不可分割的单元，数据有两个关键的特性：When 和 What。
When是只数据是与时间相关的，也就是数据是在某个时间产生的。这个非常重要，在具有事务特性的数据库中，操作的先后顺序对结果至关重要。例如数据库的Binlog日志。因此，数据的时间性质决定了数据的全局发生先后，也就决定了数据的结果。
What是只数据的本身。由于数据跟某个时间点相关，所以数据的本身是不可变的(immutable)，过往的数据已经成为事实（Fact），你不可能回到过去的某个时间点去改变数据事实。这也就意味着对数据的操作其实只有两种：读取已存在的数据和添加更多的新数据。采用数据库的记法，CRUD就变成了CR，Update和Delete本质上其实是新产生的数据信息，用C来记录。

## 数据的存储：Store Everything Rawly and Immutably

根据上述对数据特性的分析，lambda架构中对数据的存储采用的方式是：数据不可变，存储所有数据。
采用这两种方式存储的好处：
a、简单。采用不可变的数据模型，存储数据时只需要简单的往主数据集后追加数据即可。相比于采用可变的数据模型，为了Update操作，数据通常需要被索引，从而能快速找到要更新的数据去做更新操作。
b、应对人为和机器的错误。人和机器每天都可能会出错，如何应对人和机器的错误，让数据系统快速恢复极其重要。不可变和可重复计算是应对认为和机器错误的常用方法。采用可变数据模型，引发错误的数据有可能被覆盖而丢失。相比于采用不可变的数据模型，因为所有的数据都在，引发错误的数据也在。修复的方法就可以简单的是遍历数据集上存储的所有的数据，丢弃错误的数据，重新计算得到Views。重新计算的关键点在于利用数据的时间特性决定的全局次序，依次顺序重新执行，必然能得到正确的结果。
当前业界有很多采用不可变数据模型来存储所有数据的例子。比如分布式数据库Datomic，基于不可变数据模型来存储数据，从而简化了设计。分布式消息中间件Kafka，基于Log日志，以追加append-only的方式来存储消息。

# Lambda架构

Lambda架构的主要思想是将大数据系统架构为多层个层次，分别为批处理层（batch layer）、实时处理层（speed layer）、服务层（serving layer）如图（C）。
理想状态下，任何数据访问都可以从表达式Query = function(all data)开始，但是，若数据达到相当大的一个级别（例如PB），且还需要支持实时查询时，就需要耗费非常庞大的资源。一个解决方式是预运算查询函数（precomputed query funciton）。书中将这种预运算查询函数称之为Batch View（A），这样当需要执行查询时，可以从Batch View中读取结果。这样一个预先运算好的View是可以建立索引的，因而可以支持随机读取（B）。于是系统就变成：
（A）batch view = function(all data)；
（B）query = function(batch view)。

{% img full-image '/images/数据系统架构之Lambda architecture/graph3.png' 图（B） %} 

## Batch Layer

在Lambda架构中，实现（A）batch view = function(all data)的部分称之为Batch Layer。他承担两个职责：
a、存储Master Dataset，这是一个不变的持续增长的数据集
b、针对这个Master Dataset进行预运算
在全体数据集上在线运行查询函数得到结果的代价太大，同时处理查询时间过长，导致用户体验不好。如果我们预先在数据集上计算并保存预计算的结果，查询的时候直接返回预计算的结果，而无需重新进行复制耗时的计算。显然，batch view 是一个批处理过程，如采用Hadoop或spark支持的map－reduce方式。采用这种方式计算得到的每个view都支持再次计算，且每次计算的结果都相同。

{% img full-image '/images/数据系统架构之Lambda architecture/graph4.png' 图（D） %} 

对View的理解： 
View是一个和业务关联性比较大的概念，View的创建需要从业务自身的需求出发。一个通用的数据库查询系统，查询对应的函数千变万化，不可能穷举。但是如果从业务自身的需求出发，可以发现业务所需要的查询常常是有限的。Batch Layer需要做的一件重要的工作就是根据业务的需求，考察可能需要的各种查询，根据查询定义其在数据集上对应的Views。
Batch Layer的Immutable data模型和Views
如图（E）坐席（agentid＝50023）的人，在10:00:06分的时候，状态是calling，在10:00:10的时候状态为waiting。在传统的数据库设计中，直接后面的纪录覆盖前面的纪录，而在Immutable 数据模型中，不会对原有数据进行更改，而是采用插入修改纪录的形式更改历史纪录。

{% img full-image '/images/数据系统架构之Lambda architecture/graph5.png' 图（E） %}

上文所提及的View是图（E）中预先计算得到的相关视图，例如：2016-06-21当天所有上线的agent数，每条热线、公司下上线的Agent数。根据业务需要，预先计算出结果。此过程相当于传统数仓建模的应用层，应用层也是根据业务场景，预先加工出的view。
Speed Layer

Batch Layer能够很好的处理离线数据，但是在很多场景数据不断产生，并且业务场景需要实时查询。Speed Layer就是设计用来处理增量实时数据。
Speed Layer和Batch Layer比较类似，对数据进行计算并生成Realtime View，其主要的区别在于：
a、Speed Layer处理的数据是最近的增量数据流，Batch Layer处理的是全体数据集
b、Speed Layer为了效率，接收到新数据及时更新Realtime View，而Batch Layer根据全体离线数据直接得到Batch View。Speed Layer是一种增量计算，而非重新计算（recomputation）。
c、Speed Layer因为采用增量计算，所以延迟小，而Batch Layer是全数据集的计算，耗时比较长。
综上所诉，Speed Layer是Batch Layer在实时性上的一个补充。如图（F）

{% img full-image '/images/数据系统架构之Lambda architecture/graph6.png' 图（F） %}

Speed Layer可总结为以（C）Realtime View ＝ function（Realtime View， new data）；
Lambda Architecture将数据处理分解为Batch Layer 和Speed Layer有如下优点：
a、容错性：Speed Layer中处理的数据不断写入Batch Layer，当Batch Layer中重新计算数据集包含Speed Layer处理的数据集后，当前的Realtime View就可以丢弃，这就意味着Speed Layer处理中引入的错误，在Batch Layer重新计算时都可以得到修证。这点也可以看成时CAP理论中的最终一致性（Eventual Consistency）的体现。
b、复杂性隔离。Batch Layer处理的是离线数据，可以很好的掌控。Speed Layer采用增量算法处理实时数据，复杂性比Batch Layer要高很多。通过分开Batch Layer和Speed Layer，把复杂性隔离到Speed Layer，可以很好的提高整个系统的鲁棒性和可靠性。
Serving Layer

Batch Layer通过对Master Dataset执行查询获得Batch View，Speed Layer通过增量计算提供Realtime View。Lambda架构的Serving Layer用于响应用户的查询请求，合并Batch View和Realtime View中的结果数据集到最终的数据集，如图（G）。因此，Serving Layer的职责包含：
a、对batch View和RealTime View的随机访问
b、更新Batch Veiw和RealTime View，并负责结合两者的数据，对用户提供统一的接口

{% img full-image '/images/数据系统架构之Lambda architecture/graph7.png' 图（G） %} 

综上所诉，Serving Layer采用如下等式（D）表示：Query ＝ function（Batch Views， Realtime View）。
Lambda 架构组件选型

下图给出了Lambda架构中各组件在大数据生态系统中和阿里集团的常用组件。数据流存储选用不可变日志的分布式系统Kafa、TT、Metaq；Batch Layer数据集的存储选用Hadoop的HDFS或者阿里云的ODPS；Batch View的加工采用MapReduce；Batch View数据的存储采用Mysql（查询少量的最近结果数据）、Hbase（查询大量的历史结果数据）。Speed Layer采用增量数据处理Storm、Flink；Realtime View增量结果数据集采用内存数据库Redis。

{% img full-image '/images/数据系统架构之Lambda architecture/graph8.png' 图（H） %} 

Lambda是一个通用框架，各模块选型不要局限于上面给出的组件，特别是view的选型。因为View是和各业务关联非常大的概念，View选择组件时要根据业务的需求，选择最合适的组件。
Lambda架构的评估

优点：
a、数据的不可变性。里面给出的数据传输模型是在初始化阶段对数据进行实例化，这样的做法是能获益良多的。能够使得大量的MapReduce工作变得有迹可循，从而便于在不同阶段进行独立调试。
b、强调了数据的重新计算问题。在流处理中重新计算是个主要挑战，但是经常被忽视。比方说，某工作流的数据输出是由输入决定的，那么一旦代码发生改动，我们将不得不重新计算来检视变更的效度。什么情况下代码会改动呢？例如需求发生变更，计算字段需要调整或者程序发出错误，需要进行调试。
缺点：
a、Jay Kreps认为Lambda包含固有的开发和运维的复杂性。Lambda需要将所有的算法实现两次，一次是为批处理系统，另一次是为实时系统，还要求查询得到的是两个系统结果的合并。

由于存在以上缺点，Linkedin的Jay kreps提出了Kappa架构如图（I）：

{% img full-image '/images/数据系统架构之Lambda architecture/graph9.png' 图（I） %} 

1、使用Kafka或其它系统来对需要重新计算的数据进行日志记录，以及提供给多个订阅者使用。例如需要重新计算30天内的数据，我们可以在Kafka中设置30天的数据保留值。
2、当需要进行重新计算时，启动流处理作业的第二个实例对之前获得的数据进行处理，之后直接把结果数据放入新的数据输出表中。
3、当作业完成时，让应用程序直接读取新的数据记录表。
4、停止历史作业，删除旧的数据输出表。
 
Kappa架构暂时未做深入了解，在此不做评价。我个人觉得，不同的数据架构有各自的优缺点，我们使用的时候只能根据应用场景，选择更合适的架构，才能扬长避短。
 
参考资料：

Big Data: Principles and best practices of scalable real-time data systems——Nathan Marz
http://blog.csdn.net/brucesea/article/details/45937875
https://zhuanlan.zhihu.com/p/20510974
http://www.infoq.com/cn/news/2014/09/lambda-architecture-questions


