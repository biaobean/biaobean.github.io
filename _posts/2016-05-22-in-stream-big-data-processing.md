【[原文](http://highlyscalable.wordpress.com/2013/08/20/in-stream-big-data-processing/)】



http://dirtysalt.info/in-stream-big-data-processing.html
https://highlyscalable.wordpress.com/2013/08/20/in-stream-big-data-processing/

缺点和面向批处理的数据处理的弊端由大数据被社会广泛认可的相当长一段时间以前。很显然，实时查询处理和流处理在许多实际应用中的燃眉之急。近年来，这个想法得到了很多牵引和一大堆类似Twitter的风暴解决方案，雅虎的S4，Cloudera的黑斑羚，Apache的星火和Apache TEZ出现，并加入了大数据和NoSQL系统的军队。这篇文章是探讨在流数据处理系统的开发者使用的技术，跟踪这些技术的大规模批量处理和OLTP / OLAP数据库的连接，并讨论如何一个统一的查询引擎可以插播支持的努力，批次，在同一时间和OLAP处理。

在电网动态，我们最近面临一个必要构建旨在紧缩每天约8十亿事件提供容错性和严格transactioanlity即没有这些事件可能会丢失或重复的串流数据处理系统。该系统已被​​设计为补充和成功的现有基于Hadoop的系统，该系统具有数据处理的过高延迟和过高的维护成本。的要求和系统本身是如此一般性和典型我们下面描述它作为一个典范模型，就像一个抽象的问题陈述。

我们用下图所示的工作环境的一种高层次的概述：

覆盖-2

人们可以看到，这样的环境中是一个典型的大数据安装：有一组产生多个数据中心中的原始数据的应用程序，该数据是通过数据采集子系统的装置运到位于中央设施HDFS，那么原始数据被聚合，并使用标准的Hadoop栈（MapReduce的，猪，蜂房）和聚合结果存储在HDFS和NoSQL，导入到OLAP数据库和由自定义用户应用程序访问进行分析。我们的目标是为所有的设施，一个新的串流发动机（在图的底部示出），其处理至中央设施最密集的数据流和一般的预聚合的数据，从而减少原始数据和重量批处理作业Hadoop中。中插播处理引擎本身是由下列要求驱动的设计：
•SQL一样的功能。该发动机具有持续评估类似SQL的查询，包括联接随着时间的推移窗口和实现相当复杂的自定义业务逻辑不同的聚合功能。发动机还可以涉及从汇总数据门店加载相对静态的数据（外加剂）。复杂的多通数据挖掘算法是超越眼前的目标。
•模块化和灵活性。这是不是说，人们可以简单地发出类似SQL的查询和相应的管道将被创建并自动部署，但它应该是比较容易通过连接一个街区到另一个组装相当复杂的数据处理链。
•容错。严格的容错是发动机的主要要求。因为它在图中的底部勾勒，发动机的一个可能的设计是使用实施操作喜欢联接和聚合或这些操作的链分布式数据处理管线，并通过容错持久缓冲器的装置连接这些管道。这些缓冲器也通过使发布/订阅通信方式和易于添加/移除管道提高了系统的模块化。该管道可以是有状态和发动机的中间件应该提供一个持久存储，以使国家检查点。所有这些议题将在本文的后面的章节中讨论。
•互操作性用Hadoop。发动机应该能够从Hadoop的即作为自定义查询引擎顶上HDFS的摄取都流数据和数据。
•高性能和移动性。该系统应甚至在最小尺寸的集群，实现每秒数万消息的性能。发动机应紧凑，高效，因此可以在小群多数据中心部署。

要找出这样的系统如何实现，我们将讨论在本文的其余部分以下主题：
•首先，我们将探讨在流数据处理系统，大规模的批量处理系统，以及关系查询引擎之间的关系，以了解如何在流处理可以利用被设计用于其他类的系统的技术，一个巨大的数字。
•其次，我们描述了一些模式和技巧，频繁的在流处理架构和制度建设中。此外，我们调查了当前和新兴的技术，并提供一些实现技巧。

这篇文章是基于在网格动力学实验室开发的研究项目。大部分的功劳归于阿列克谢Kharlamov和Rafael Bagmanov谁领导这个项目和其他贡献者：德米特里·苏斯洛夫，Konstantine Golikov，埃维莉娜诺娃，阿纳托利维诺格拉多夫，罗马别洛乌斯和瓦尔瓦拉Strizhkova。

分布式查询处理的基础知识

很显然，分布在流数据的处理有东西在分布式关系数据库是与查询处理。许多标准的查询处理技术可以通过在流处理引擎被使用，所以它是理解分布式查询处理的经典算法，看看它是如何涉及流处理和其他流行的范式像MapReduce的非常有用的。

分布式查询处理是知识，这是正在开发数十年的一个非常大的领域，所以我们用的主要技术的简要概述开始只是提供进一步讨论的背景。

分区和洗牌

分布式和并行查询处理在很大程度上依赖于数据分割分解的大数据集分割成多个部分，可以由独立的处理器来处理。查询处理可以由多个步骤，每个步骤可能需要它自己的分区策略，因此数据移动是频繁分布式数据库执行的操作。

虽然选择和投影操作中获得最佳的分区可能会非常棘手（例如，对于范围查询），我们可以假设，在流数据过滤实际上是不够分配使用基于散列的分区的处理器之间的数据。

的分布式联接处理不是那么容易的，需要更彻底的检查。在分布式环境中，连接处理的并行性是通过数据分区来实现，即数据处理器之间分配和每个处理器采用串行连接算法（如嵌套循环连接或排序合并或基于散列的加入），以处理其部分的数据。最终的结果是从来自不同处理器所获得的结果合并。

有可以通过分布式处理联接可以采用两种主要的数据分区技术：
•交集数据分区
•鸿沟和广播加盟

不相交的数据划分技术洗牌的数据转换成以这样的方式连接键在不同的分区不重叠几个分区。每个处理器上执行的每个这些分区和最终结果的联接操作来自不同处理器所获得的结果的一个简单的级联获得。考虑在关系R接合用关系S于数字键k和一个简单的基于模散列函数用于产生分区（这是假定该数据最初是基于一些其他策略的处理器之间分布）的例子：

不相交分区

除法和广播连接算法在如下图所示。该方法将第一数据（图中的R1，R2和R3）设置成多个不相交的分区和复制第二数据集到所有处理器。在分布式数据库中，除法通常不在查询处理本身的一部分，因为数据集的多个节点之间最初分布。

广播加盟

这种策略适用于用小关系或两个小的关系大关系的加盟。在流数据处理系统可以使用这种技术流富集即加入一个静态数据（混合）到一个数据流。

GROUPBY查询处理也依靠洗牌，从根本上类似其纯粹的形式MapReduce的范例。考虑其中数据是由数值计算每个组中的一个字符串键和总和分组的例子：

基的由查询

在这个例子中，计算由两个步骤组成：局部聚集和全局汇聚。这些步骤基本上对应于地图和降低运营。本地聚合是可选的，原始的记录可以发射，打乱，并聚集在全球聚集阶段。

本节的整点是，所有的算法以上可以自然使用传递的建筑风格，即查询执行引擎可以被认为是由消息队列连接的节点的分布式网络的消息来实现。这是概念上类似于在流处理管道。

流水线

在上一节中，我们注意到，许多分布式查询处理算法类似于消息传递网络。然而，这是不够的，组织有效的在流处理：查询中所有运营商应该以这样的方式，数据通过IE既不操作整个管道畅通无阻应该等待一大块输入数据块处理被链接没有产生任何输出或写在磁盘上的中间结果。此类物品分拣一些操作与此概念（显然，直到整个输入摄取一个排序块不能产生任何输出）固有不相容，但在许多情况下，流水线算法都适用。流水线的一个典型例子如下所示：

加入管线

在这个例子中，散列连接算法来加入四个关系：R1，S1，S2和S3用3个处理器。该想法是建立为S1，S2和S3在平行哈希表，然后流R1元组逐个尽管这通过在哈希表查找匹配用S1，S2和S3中加入他们的管道。在流处理自然采用这种技术来加入与静态数据（外加剂）的数据流。

在关系数据库中，连接操作可以通过使用对称散列连接算法或者一些先进的变种[1,2]利用流水线。对称散列连接是散列的推广加盟。而正常散列连接要求其输入的至少一个是完全可用的，以产生第一结果（需要的输入建立一个哈希表），对称散列连接能够立即产生第一结果。与此相反的正常哈希联接，它保持对两个输入哈希表和元组到达填充这些表：

对称加盟

作为一个元组到来时，木匠首先查找它在其他流的哈希表。如果发现匹配，输出元组产生。然后将元组插入到其自己的哈希表。

但是，它不会使一个很大的意义进行无限流的一个完整的加盟。在许多情况下，联接的缓冲器例如有限时间窗口或其他类型上执行LFU缓存包含在信息流中最频繁的元组。对称散列连接可以采用如果缓冲区大比较流率或缓冲液是根据一些应用逻辑频繁刷新或缓冲驱逐策略是不可预测的。在其他情况下，简单的散列连接通常就足够了，因为缓冲区满不断而不会阻止处理：

流加入

值得注意的是，在流处理常与其中的记录根据得分指标相匹配，而不是在野外条件下的平等复杂的码流相关的算法交易。缓冲区的一个更复杂的系统可以要求在这种情况下两个流。

在流处理模式

在上一节，我们讨论了一些可在大规模并行流处理可使用标准的查询处理的技术。因此，在概念上，在分布式数据库中的高效的查询引擎可以作为流处理系统，反之亦然，流处理系统可以作为一个分布式数据库查询引擎起作用。洗牌和流水线是分布式查询处理和消息传递网络，可以自然实现它们的关键技术。然而，事情并非如此简单。在对比数据库查询引擎，可靠性并不重要，因为总是可以重新启动只读查询，流媒体系统要多多注意可靠的事件处理。在这一节中，我们讨论了一些由流系统，以提供消息传送的保证和一些其他的模式，不典型标准查询处理所使用的技术。

流回放

能力倒带数据流回来的时间和重放数据是非常重要的，因为以下原因在流处理系统：
•这是保证正确的数据处理的唯一方式。即使数据处理管线是容错，这是非常有问题的，以保证已部署处理逻辑是无缺陷的。人们总是可以面对必要性以固定和重新部署系统和重放管线上的一个新版本的数据。
•问题的调查可能需要即席查询。如果出现问题，我们可以需要重新运行系统上的问题的数据具有更好的记录或代码交替。
•虽然不总是这样，所述串流处理系统可以以这样一种方式，它旨在重新读取从源各个消息中加工误差和局部故障的情况下，即使在一般的系统容错宽容。

作为一个结果，输入数据通常变为从经由持久缓冲器，允许客户端移至其阅读指针来回数据源中插播管道。

重播缓冲

卡夫卡消息队列是众所周知的实现，还支持可扩展分布式部署，容错，并且提供高的性能这样的缓冲液中。

作为一个底线，流回放技术强加了系统设计的以下要求：
•系统能够存储的原始输入数据为预先配置的周期时间。
•系统能够撤销产生的结果的一部分，重放对应的输入数据和产生的结果的一个新的版本。
•系统应工作速度不够快，数据倒带回来的时候，重播它们，然后赶上不断到达的数据流。

沿袭跟踪

在流传输系统中，直到结果到达最终目的地（例如一个外部数据库）的事件流经处理器链。每个输入事件产生，通过最后的结果结束后裔事件（谱系）的有向图。为了保证可靠的数据处理中，有必要确保在整个图形被成功处理，并且在故障的情况下，重新启动处理过程。

高效的血统跟踪是不是一个简单的问题。让我们首先考虑Twitter的风暴如何跟踪消息，以保证在-至少一次传递语义（见下图）：
•由源（在数据处理图表第一节点）发射的所有事件由随机ID标记。对于每个源，该框架保留了一组对[事件ID - >签名]为每个初始事件。签名最初由事件ID初始化。
•下游节点可以基于所接收到的初始事件零或多个事件。每个事件带有自己的随机ID和初始事件的ID。
•如果成功地接收，并通过在图中的下一个节点处理的情况下，该节点通过异或与（a）该输入事件的标识和（b）所有的事件的ID出品基于签名更新相应初始事件的签名对传入事件。在下面图的第2部分，事件01111产生事件01100，10010和00010，所以对于事件01111签名变为11100（= 01111（初始值）异或01111异或01100异或10010异或00010）。
•事件可以基于一个以上的传入事件产生。在这种情况下，它是连接几个初始事件和下游携带一个以上的初始的ID（下图中的第3部分的黄黑色事件）。
•尽快签字变成零，即最终的节点成功处理考虑该事件承认，在图中的最后一个事件被成功处理任何事件都发出下游。该框架将提交信息源节点（见下图中的第3部分）。
•该框架横过的初始事件周期性地寻找老未提交的事件（事件非零签名）的表。这些事件被认为是失败，该框架要求的源节点重放。
•要注意，签名更新的顺序并不重要，由于异或操作的可交换性是很重要的。在下图中，在第2部分中描述的确认可以到达部分3所示确认这使得完全异步处理后。
•一个可以注意到，上述算法并不严格可靠 - 签名可能变成零意外，由于ID的不幸组合。然而，64位ID是足以保证错误的概率非常低，大约2 ^（ - 64），这是在几乎所有的实际应用可以接受的。作为结果，签名表可以有一个小的内存footprint.lineage跟踪风暴

所描述的方法是优雅的，由于其性质decentrilized：节点独立行事发送确认消息，有一个明确的跟踪所有的血统没有百磅实体。然而，可能难以在这种方式来管理事务处理该保持滑动窗口或其它缓冲液流动。例如，一个滑动窗口上的处理可涉及成百上千事件在每个时刻，所以变得难以管理确认因为许多事件保持中立或计算状态应经常保持。

另一种方法是在Apache中使用星火[3]。我们的想法是要考虑的最终结果作为输入数据的函数。为了简化谱系追踪，框架处理批量的事件，所以结果是分批的一个序列，其中每个批次是输入批次的功能。所得批次可以并行计算，如果一些计算失败，则框架简单地重新运行它。考虑一个例子：

流加入-microbatching-TX

在这个例子中，框架联接的滑动窗口上两个流，然后将结果通过多一个处理阶段。该框架认为还不如流，但由于设置批次输入流。每批都有一个ID和框架，可以通过ID在任何时刻把它拿来。所以，流处理可被表示为一串事务，其中每个事务需要一组输入批次，使用的处理功能将它们转换，并持续一个结果。在上图中，此类交易的一个以红色突出显示。如果交易失败，该框架简单地重新运行它。重要的是，事务可以并行执行。

这个简单但功能强大的模式实现了集中式的事务管理和本质提供仅一次的消息处理语义。值得注意的是，这种技术既可以用于批量处理和流处理中使用，因为它把该输入数据为一组分批不管其静态特性流。

国家检查点

在上一节，我们认为，使用签名（校验和），以提供在-至少酮消息传递语义谱系追踪算法。这种技术提高了系统的可靠性，但它留下至少有两个主要的开放性问题：
•在很多情况下，一次准确加工的语义是必需的。例如，计数事件会产生不正确的结果，如果一些消息将被传递两倍的管道。
•在管道的节点可以具有被更新为已处理的消息的计算状态。此状态可能会丢失在节点故障的情况下，因此，有必要持续或复制它。

Twitter的风暴解决了这些问题，通过使用以下协议：
•事件被分组为批次与每批与事务ID相关联。事务ID是单调增长的数值（例如，第一批次的ID是1，第二个编号2，等等）。如果管道没有处理一批，此批被重新发射具有相同的事务ID。
•首先，框架宣布到一个新的事务尝试开始在管道中的节点。第二，该框架以发送通过管道的批次。最后，该框架宣布交易的尝试，如果完成所有节点都可以例如提交他们的状态在外部数据库进行更新。
•该框架保证承诺相全局排序的所有交易，即交易2永远不能交易1.本保证能够处理节点使用下面的持久状态更新逻辑之前承诺：◦The最新交易ID与坚持一起州。
◦If框架请求提交与从ID值不同的标识当前事务数据库中的持续存在，该状态可以例如更新在数据库中的计数器可以被递增。假设交易的一个强大的排序，这样的更新将发生只有一个每个批次。
◦If当前事务ID等于的值在存储持久，节点跳过提交，因为这是一个批次重放。节点必须处理完一批较早，相应地更新状态，但交易失败，原因是一个错误的管道别的地方。
◦Strong提交的顺序是实现一次准确处理语义重要。但是，以前严格顺序处理是不可行的，因为在管道第一节点往往会空闲等待直到完成了下行节点上的处理。这个问题可以通过使交易的并行处理，但序列的仅提交步骤，如在下面的图中所示它被减轻：


流水线-提交-2

这种技术允许一个实现一次准确假设的数据源处理语义是容错，可以重播。然而，持续的状态更新会导致即使使用大批量严重的性能下降。由这个原因，中间计算状态应尽量减少或尽量避免。

作为一个注脚，值得一提的是，国家的写作可以用不同的方式来实现。最直接的方法是将内存状态转储到持久性存储作为交易的一部分提交过程。这不适用于大国（一种等滑动窗口）工作。另一种方法是写一种事务日志，即是改造旧的状态进入新的操作序列的（对于一个滑动窗口也可以是一组添加和驱逐的事件）。这种方法崩溃恢复复杂化，因为该状态必须被从日志重建，但是可以在多种情况下提供性能优势。

加国和草图

的中间和最终计算结果相加是大大简化了设计，实施，维护，并在流数据处理系统的恢复的一个重要属性。加意味着较大的时间范围或更大的数据分区的计算结果可以作为结果更小的时间范围或小的分区的组合来计算。例如，每日数的页面访问可被计算为的页面访问每小时数的总和。添加剂状态允许一个流的处理分割为可以被计算并独立地重新计算和，如我们在前面部分中讨论的批次的处理，这有助于简化谱系追踪和减少的状态维护的复杂性。

它并不总是琐碎实现加：
•在许多情况下，加的确是微不足道的。例如，简单的计数器添加剂。
·在某些情况下，有可能通过存储的附加信息少量实现加性。例如，考虑到在互联网上开店每个小时计算平均购买价值的系统。日均不能从24小时平均值获得。然而，该系统可以很容易地存储多个事务与每个每小时平均沿着和它足以计算每日平均值。
•在许多情况下，这是非常困难或不可能实现加性。例如，考虑才是最重要的一些互联网网站唯一访问者的系统。如果100独立用户访问该网站昨天和100独特的用户现在访问该网站，唯一的用户的两天总人数可以从100到200取决于有多少用户昨天和今天都访问该网站。一要维护用户ID列表通过ID列表的交集/联合来实现加。大小和处理的复杂性，这些列表可以媲美的原始数据的大小和处理的复杂性。

草图是改造非相加值到添加剂的非常有效的方法。在前面的例子中，编号的列表可以由紧凑添加剂统计计数器来代替。这些计数器提供近似值代替精确的结果，但它是对于许多实际应用可以接受的。草图像互联网广告某些地区很受欢迎，可以看作是在流处理的独立模式。的素描技术的全面概述可以在[5]中找到。

合乎逻辑的时间跟踪

这是很常见的插播计算依赖于时间：聚合和联接往往滑动的时间窗口进行;处理逻辑往往取决于事件等等之间的时间间隔。显然，在流处理系统应该具备的，而不是CPU挂钟时间的应用程序的视图的概念。然而，适当的时间跟踪是不平凡的，因为数据流和特定的事件可以在发生故障的情况下进行重播。它经常是有，可以如下来实现全局逻辑时间概念是一个好主意：
•所有事件应标明由原始应用程序生成的时间戳。
•管道中的每个处理器跟踪的最大时间戳它在流已经看到，如果全局时钟是由背后此时间戳更新一个全球性的持续性时钟。所有其它处理器同步其与全局时钟时间。
•全球时钟可以在数据回放的情况下，进行复位。

聚合在持久性存储

我们已经讨论了持久性存储，可用于检查点的状态。然而，这不是唯一的方式聘请外部存储在流处理。让我们考虑了采用卡桑德拉在时间窗口加入多个数据流的例子。相反，保持在内存中事件缓冲区，可以简单地从所有数据流都传入事件保存到卡桑德拉使用连接键作为行键，如下图所示吧：

卡桑德拉联接

在另一侧，所述第二过程周期性地遍历记录，组装和发射加入事件，并逐出该跌出时间窗口的事件。卡桑德拉甚至可以排序的事件根据它们的时间戳推动这一活动。

写单个事件的数据存储可以引入严重的性能瓶颈，即使对于快速商店像卡桑德拉或Redis的 - 要明白，这种技术可以打败流数据处理的全部目的，如果不正确地实现这一点很重要。在另一方面，这种方法提供了计算状态和不同的性能优化的完美持久性 - 比如，批量写入 - 可以帮助实现许多用例可接受的性能。

聚集在滑动窗口

在流数据处理中经常像查询涉及“什么是流中的值在最后10分钟？总和”，即连续查询滑动时间窗。一个简单的方法来这样的查询的处理是计算像总和聚合函数的时间窗口的每个实例独立。很显然，这种方法是因为时间窗口的两个顺序实例之间的高相似性不是最佳的。如果在T中包含的样本的时间{S（0），S（1），S（2），...，S（T-1）中，s（T）}，则在时间T窗口的窗口+ 1含有样品{S（1）中，s（2）中，s（3），...，S（T），S（T + 1）}。这一观察表明，可能使用增量处理​​。

在滑动窗口的增量计算是一组技术，被广泛应用于数字信号处理，在软件和硬件。一个典型的例子是总和函数的计算。如果在当前的时间窗口中的总和是已知的，则在接下来的时间窗的总和，可以通过增加一个新的样本，并减去在窗口长子样本计算：

inremental聚集

类似的技术不仅存在于简单聚合一样款项或产品，同时也为更复杂的转换。例如，该SDFT（滑动的Discreet傅立叶变换）算法[4]是每个窗口计算FFT（快速傅立叶变换）算法的一个计算上有效的替代方案。

查询处理管道：风暴，卡桑德拉，卡夫卡

现在让我们回到到在本文开头所述的实际问题。我们已经设计并实现我们的流数据处理系统上风暴，卡夫卡的顶部，采用卡桑德拉本文前面所述的技术。在这里，我们提供的解决方案只是一个非常简短的概述 - 所有执行陷阱和技巧的详细描述过大，可能需要一个单独的文章。

风暴卡夫卡卡桑德拉系统

该系统采用自然卡夫卡0.8作为分区容错事件缓冲区，使流回放，并轻松添加新的事件生产者和消费者提高了系统的可扩展性。卡夫卡倒带读指针能力也使随机访问与输入批次，因此，火花式谱系追踪。另外，也可以向系统的输入指向HDFS处理历史数据。

Cassandra是用于检查点的状态和店内聚集，如前所述。在许多使用情况下，它也存储最后的结果。

Twitter的风暴是系统的骨干。在风暴的拓扑结构与卡夫卡和卡桑德拉互动进行的所有活动查询处理。一些数据流是简单明了：数据到达卡夫卡;风暴读取并处理它，坚持的结果，卡桑德拉或其他目的地。


