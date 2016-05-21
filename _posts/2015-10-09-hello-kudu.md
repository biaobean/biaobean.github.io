---
layout:     post
title:      "初见Kudu"
subtitle:   "Hello Kudu"
date:       2015-10-09 12:00:00
author:     "biaobean"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 大数据
    - Hadoop
    - Kudu
---

Cloudera刚公布了一个新的工具叫Kudu([http://getkudu.io]())，基于Apache协议开源，按照官网blog中所说，其主要动机在于：

- 同时提供高性能的顺序扫描和随机查询，避免使用HBase+HDFS混合架构的复杂性：
	- 开发：必须编写复杂的代码来管理两个系统之间的数据传输及同步
	- 运维：必须管理跨多个不同系统的一致性备份、安全策略以及监控
	- 业务：新数据从达到HBase到HDFS中有时延，不能马上供分析
	- 在实际运行中，系统通常会遇到数据延时达到，因此需要对过去的数据进行修正等。如果使用不可更改的存储（如HDFS文件），将会非常不便。
- 充分利用现代CPU能力从而提高ROI*［注：比如不使用HBase的read merge路径］*
- 充分利用最新存储技术提高IO性能*［注：这个和Spark的思想相同］*
- 提供就地数据更新，避免额外的处理以及数据移动
- 提供双活集群互备部署，集群可以部署在跨地区的多个数据中心

Cloudera的产品总裁Charles Zedlewski在推上（[‏@zedlewski](https://twitter.com/zedlewski)）说：

1. Kudu时一个运行在YARN以下，支持Hadoop生态圈内大部分的引擎：MapReduce、Impala和Spark。
1. Kudu是可修改的存储。今天发布的Impala版本提供了对ANSI SQL操作的支持，如更新和删除。
1. 如果用户曾经为了删除和更新数据做了很多hacking的事情，希望Kudu能帮忙简化之。
1. Kudu很“坚固”。因为它维护和3份（甚至更多）副本，不会因为节点切换而丢失数据。
1. Kudu很快。无论是在内存还是磁盘上数据都是按列式存储的。单条记录的查询和操作时间约5ms。
1. Kudu是一致性的。据我所知这是基于Spanner设计的第一个开源产品［注：这让cockroachDB等情何以堪？］。你可以配置副本和一致性规则。
1. Kudu是为了支持当前硬件中更大容量的内存而设计的。使用本地化的编码［注：指使用C语言而非Java］实现了自己的内存管理，避免了GC带来的运行中断。
1. Kudu很高效。它同时使用列编码和字典编码，因此会比传统的noSQL存储更多的字节。
1. Kudu不是为所有人设计的，他实际上是一个关系型数据存储（有表，有类型），因此它并不是原始字节（HDFS）或者包含一百万列的表（HBase）。

而从Hadoop生态来看，Kudu并非为替代HDFS或者HBase，而是填补两者之间的灰色地带：HDFS的列式存储非常适合分析，但实时更新性能很差，单纯的离线分析还是HDFS的强项；HBase随机数据插入性能很好，但分析性能很差，对日志查询等场景HBase仍然是不二之选。因此，Kudu是针对既要随机数据插入又要顺序访问数据分析的场景，比如：

- 时间序列（Time Series），比如：stream market data; fraud detection & prevention; rick monitor
- 机器数据分析（Machine Data Analyse），比如：Network threat detection
- 在线报表（Online Reporting），比如：ODS

看Kudu的论文，从架构和实现上和Spanner非常相像。Hadoop是2003年的论文，2006年开源社区出的产品，Spanner的论文时2012年的，今年也应该有开源复制品了。但存储结构完全不同，使用列式存储，因此**Kudu不仅现在不支持OLTP，以后即使支持，性能也肯定是硬伤**。Spanner的目的是提供“跨数据中心”的“事务”，而这两点都没有在现有Kudu的实现中有特别体现，相反，Kudu现在的定位是平衡分析和随机读写。为什么抄作文把主题思想抄变了？百思不得其姐啊。

Kudu和HBase的比较：

* 相同：

	1. 内存部分差别不大，使用Memstore，MVCC等；
	1. 基本思想同样基于LSM架构，不过Kudu的实现更复杂；
	1. 一条记录只在一个Rowset中；
	1. Kudu的事务性和HBase一样，只保证行级事务；

* 不同：

	1. Kudu预先定义数据schema，主要面对结构化数据，HBase主要面对半结构化数据，尤其是可能有几万个列的稀疏矩阵。因此Kudu能更好的支持SQL。据Cloudera说Kudu变更schema，如改变column的效率还不错。
	1. Kudu除了partition还支持bucket，因此可以均衡查询并行（parallelism）和并发（concurrency）。
	1. Kudu使用了列式存储，分析性能好于HBase；
	1. 存储和处理更高效：
		1. 由于有schema，因此不需要存储列名
		1. update等使用offset表示位置，而非rowkey
	1. 性能：Kudu的flush和compact更复杂，因此write的性能（尤其是update）比HBase差；同时由于采用列式存储，读取单条记录的性能（尤其是有很多更新时）也比Hbase差。但换取了更快的scan性能；
	1. compaction：Kudu没有minor和major的区别，没长时间stop-all的compaction，后台使用低IO优先级的线程一直不停地compact。
	1. 副本一致性：每个数据块副本使用leader-follower方式服务，使用Raft（类似Zookeeper、PAXOS）consensus协议，能定义一致性， 秒级MTTR，每个follower都有自己的WAL，能支持(n-1)/2台机器宕机。不同于HBase单Region服务模式，HDFS能支持n-1台机器宕机，服务可靠性更高；
	1. Kudu中的Master更像是一个旁观者，提出建议，而HBase中Master直接管理哪块数据在哪里被服务；
	1. 底层存储架构HBase使用HDFS，而Kudu使用自己的存储架构，直接存储本地磁盘，备份基于日志而非数据，更适合跨数据中心的部署。

因此，小弟有一些话不知道当讲不当讲：

1. 传统HBase+HDFS的混搭架构应用确实可以让Kudu一试，避免了数据生命周期管理啊、大小文件合并啊等等脏活儿。但OLTP并不是Kudu的目的，因此以前HBase/Impala不能替代传统数据库的部分，现在同样不能；
1. Kudu虽然说是面向多数据中心部署，但无论在实验数据还是真实案例都甚为缺乏，更不用说整套解决方案（比如如何跨数据中心时钟同步、权限控制等）、最佳实践等了，因此要跨数据中心功能的产品化以及落地，还需时日；
1. Kudu使用C++开发，因此对于使用C++开发的应用更友好；
1. Kudu是独立于Hadoop原有底层存储的新系统，使用全新的存储架构（非HDFS），向上兼容Hadoop接口支持其他组件，如MapReduce、Impala和Spark等。因此相当于需要重新多部署一套集群，且需要数据迁移，在实际项目中可能遇到工程方面的困难。

从小米给出的数据来看：

* bulk load：Parquet比Kudu快了差不多9倍，所以批量数据导入还是直接HDFS好。

# 参考资源

[http://www.dbms2.com/2015/09/28/introduction-to-cloudera-kudu/]()

这只是看了几篇文章得出的初步结论，日后有了更深的认识再来看看。