---
layout:     post
title:      "2017 Strata Datad大会Beijing站参会记（第一篇）：周五Keynotes"
subtitle:   "2017 Strata Data Beijing(1): Keynotes 1"
date:       2017-07-16 00:00:00
author:     "biaobean"
header-img: "img/content/2017-strata-data-beijing/doug.jpg"
catalog: true
tags:
    - Strata
---

Orelly的Strata全球大会引入中国已经第二年，这是门票要大好几千的高B格的技术会议，也是广大海外IT民工蹭会归国的探亲会:)。有幸能去蹭吃蹭喝，那是必须去现眼的。

这次Strata的官方规格还是很高的，Cloudera公司来的是Hadoop吉祥物道哥和CTO Amr，这对儿胖头陀和瘦头陀在一个技术大会上同时出现，这些年在全球都屈指可数。

![img](/img/content/2017-strata-data-beijing/doug.jpg)

Strata Hadoopy已经改名成了Strata Data，不过这一届其实被“掰弯”成了Strata AI，全场Hadoop的Session仅在个位数。“大数据”本身已经不再是技术界，甚至不再是业界的宠儿，现在花魁的名字叫“人工智能”。

总体看，百度、京东等互联网的AI技术已经在开始走出实验室，并且碾压传统技术。而传统IT行业或厂商更多还在干着OLAP，OLTP的活儿。就分享的Session看，也多来自互联网圈，有钱有数据节奏快，让现在的现在BATJ等互联网码农几乎垄断国内的大数据技术。传统IT数据工程师和与之的代购，别说分享了，现在估计连听都不怎么听的懂了。各种技术大会几乎只是圈里人的自high的Meetup而已。

# Keynotes

## Intel

樱桃厂来的是马自雅女士，会议主席Jason的老板。以前在厂里老听说，但从来没见过，原来本人是华人，还很年轻。

大公司的调调B格总是很高尚的，Ziya介绍了Intel在大数据上的愿景：

1. “通过硬件升级和软件优化推动技术民主化”，通俗的讲就是让屌丝也用得起大数据和人工智能。确实，正如罗胖所说，商业的目的不是让英国女王拥有更多的丝袜（不是让高大上的HPC变得更快），而是制造老百姓也穿得起丝袜。不过，这里Intel解决的只是计算速度问题，现在云计算已经解决了计算的资源门槛，但技术能力门槛却才是让很多传统企业望尘莫及的。
	1. Intel新一代至强芯片性能提升了数倍。现在无论是大数据还是深度学习等对于计算的要求不是数倍的增长，而是好几个数量级，显然单纯通过CPU的提速完全不满足大数据界应用的需求了。Intel尽力想把任务留在CPU，而N家往CPU以外拉，现在从股票和技术的发展看，明显N家更火。哎，Intel当年没把N家买下来，估计心情就和俺们买房子一样：当初怎么就嫌它贵没买呢？现在可好，涨成这样买不起了....话说发现Intel居然还有自己的Python发行版？！这不长记性的。。。。。。
	![img](/img/content/2017-strata-data-beijing/intel_ai.png)
	1. HBase：非堆存储的读操作提升了5倍。（在这么高逼格的Keynotes上讲这个就有点low了吧，这玩意儿的应用场景和对整体提升有限，HBase的平均处理速度已经足够满足应用需要了，它的痛点不在于快慢，而在于间歇性的抽风。）
	1. Spark：基于Intel实现的数学库能将Spark分析的性能提高4.3倍。（用Native甚至汇编来优化现有实现，利用Intel CPU的特性提升速度，尤其是数学向量计算，是樱桃软件事业部的看家本领，以前的C语言库、XML处理库、Android优化啦....）
1. “为新兴的需求、新技术提供新的解决方案”，这里其实就是推Intel出的BigDL啦，通过它构建Spark+ML端到端深度学习流水线，其优点包括：
	1. 将深度学习放到已有的大数据加速平台，即复用现在的Hadoop/Spark框架；
	1. 现有机器学习框架不是为了Spark和扩展性所设计，BigDL在利用分布式框架获得性能提升方面更有优势；
	1. 硬件利旧。现在的ML通常需要高配置专有的机器，而传统的大数据服务器配置比较低；
	1. 高性能，缩短训练周期。
	![img](/img/content/2017-strata-data-beijing/intel_bigdl.png)
1. “推动创新，为客户解决更新更复杂的问题”。这个说的是行业里的案例，牛吹的比较虚，还不如不讲，Intel也不是干这事儿的，略过。

[PPT](https://cdn.oreillystatic.com/en/assets/1/event/273/英特尔技术加速实现分析与人工智能的未来 - 英特尔赞助 _Accelerating the future for analytics and AI with Intel technologies—sponsored by Intel_ 讲话 1.pdf)


## Linkedin

这次来的是张喆，前Cloudera的工程师，去年跳槽去了Linked做核心大数据平台研发经理。哎呀呀，忍忍呀，今年Cloudera就上市了呀，人帅就是人性...不好意思我三俗了。

这次张帅哥却没什么太多干货，更多的是“高屋建瓴”地介绍了一下Linkedin的Hadoop现状，毕竟，升官了嘛：）。

现在Linkedin已经部署了超过10个Hadoop集群，总节点数过万，他们遇到的问题也许是以后大家的问题：（在Keynotes里的分享比较少，后面有个session张喆更详细地讲了，一并写在这里。）

**问题一：新的计算需求：深度学习**

Linkedin在做了联系人推荐(People You May Know)这个大数据应用以后，现在也在做一些AI的尝试，甚至在使用深度学习来满足新的计算需求，如用户资料自动翻译。场景包括：

* 文本标准化：比如简历中Java程序员、前端开发、Spark Committer等不同的说法都表示软件工程师。
* 潜在关联/相似：比如北京大学毕业的腾讯员工和清华大学毕业的阿里云员工。

再比如回答：

* 我大概预期应该几年会升职？
* 哪些因素影响升职的进度？
* 现在我应该学习哪种编程语言？
* 兼职MBA对我有用吗？

新的需求对于资源的利用习性差别非常大，带来了新的计算平台的挑战：规模和异构化。比如：

* 深度学习框架TensorFlow：任务运行时间甚至长达数天，因此计算资源需求稳定；通常混合使用CPU+GPU；需要定期存档中间计算结果
* 服务交互式SQL查询Presto：任务执行需要秒级甚至毫秒级响应时间；计算资源需求起伏非常大；容错性要求低
* 流处理引擎Samza：连续数据输入；计算资源需求稳定；长时间运行

张喆认为Hadoop 3.0以后YARN对于长任务（一直运行的服务，而不像MR那种资源“用完即走”的计算任务）的支持会更好，因此可能会很有帮助，他们也正在做测试。比如最新的TF支持动态包括：

1. Yahoo已经开发出了TF on spark，但性能尚有问题；
2. HD 3.0后是否能更好支持TF；
3. Intel在做YARN的Native支持TF，原理就是为TF定制一个Application Master。

我个人不是很看好，资源管理这块Hadoop走得太慢，我在去年上一次Strata里分享的使用Mesos对Hadoop资源进行管理的很多功能（(传送门]()）到现在Hadoop自身都还没有做到。现在一些Native云化厂商已经对Hadoop进行本地化支持，甚至声称由于更有效地利用了CPU和内存，性能比原生部署Hadoop还好10%以上。长此以往，YARN够呛啦。

另外，到底应该专属架构和共享资源原本是各有所好。专属部署能最大提高应用性能，有更好的隔离性和安全性，而共享资源能提高资源利用率，通常如果性能下降能被资源更好的利用所抵消，当然共享资源池更好了，至少能减少管理成本。因此，比如像Facebook一样将不同业务（离线和在线）分别部署应该是常态，而相同资源要求属性的任务可以共享资源。

**问题二：新的存储需求**

这里说的是HDFS的规模化问题。为什么在机器性能越来越好的现在，NN单机仍然有性能问题呢？

1. NN表空间和文件的全局读写锁。虽然可以分配给NN很多CPU核，但只能有1-2个Thread进行同时读。Linkedin实现了unfairi的读写锁。（不过据说可能会有写饿死的情况，如何重现和保证不产生饿死的情况尚未知，如果生产系统使用可能需要应用层有相关的控制。）
	1. 目标：10万QPS
	2. 目前：3-4万
	3. 已完成：精确测量每个操作用锁时间
	4. 正在做：茄花岛unfairi读写锁
	5. 计划：集合同类操作，分周期用锁
2. JVM的GC。大规模的文件操作，比如 ls -r /，会产生大量的Java对象和垃圾收集。Linkedin的最佳实践是尽量将不需要的文件移到其他集群（呃，这废话好有道理！）。比如Linkedin发现其集群里面很多的是YARN的日志，平均大小很小（100-200K），将其迁移到其他集群进行远程访问并不会带来太大的访问性能损耗，却将NN响应时间缩小了一半。（为什么不直接删？我想是因为Linkedin每天都运行至少15万以上的任务，即使一段时间后就删除，NN压力也不小吧。）
	1. 目标：5亿文件
	2. 目前：~3亿
	3. 已完成：完善文件夹list的throttle
	4. 正在做：测试G1GC
	5. 计划：可重用的文件和block对象池

另外为了应对大规模测试的挑战，Linkedin开发了大规模HDFS集群的虚拟器Dynamometer：

* 装载生成环境的fsimage
* 1键部署需要测试的Hadoop版本
* 重放生产环境的audit logs
* 只需要生产环境集群的1/10硬件

![img](/img/content/2017-strata-data-beijing/dynamometer.jpg)

Linkedin还实现了**基于Router的多集群方案**，可以访问不同的集群的**异构**文件系统（不一定是HDFS哦），可以看做viewFS的改进，以前是静态的，只能在客户端，现在放在服务器端，管理员可以自己改动。具体可以看
[HDFS-10467](https://issues.apache.org/jira/browse/HDFS-10467)。

![img](/img/content/2017-strata-data-beijing/hdfs10467.png)

**问题三：新的大数据工作者，工程生产力需求**

现在Linkedin有超过1000个数据科学家和工程师在使用Hadoop集群，所以开发了Dr. elephant工具保障集群任务质量：

1. 监控和分析所有的任务
1. 作为应用上线前的必须通过此工具的检测

这个东西是开源的，粗略看了一下，基于经验规则，无法给出性能影响估计，而且只能用于MR任务。作为一个不要钱的建议是可以的，何况界面还是很方便易用的。

[PPT](https://cdn.oreillystatic.com/en/assets/1/event/273/%E6%88%90%E9%95%BF%E7%9A%84%E7%83%A6%E6%81%BC--%E9%A2%86%E8%8B%B1%E5%A4%A7%E6%95%B0%E6%8D%AE%E5%B9%B3%E5%8F%B0500%E5%80%8D%E6%89%A9%E5%B1%95%E4%B8%AD%E5%BA%94%E5%AF%B9%E7%9A%84%E6%8C%91%E6%88%98%20_Growing%20pains_%20When%20your%20big%20data%20platform%20grows%20really%20big_%20%E8%AE%B2%E8%AF%9D.pdf)

# Orielly

先是两个Jeff的名人名言，旨在说ML/DL的普适性。

"Machine learning and AI is a horizontal enabling layer. ... basically there’s no 
institution in the world that cannot be improved with machine learning." Jeff Bezos, CEO of Amazon

"Deep Learning is essential for every computer scientist to know." Jeff Dean, Google

Ben主要介绍了一下深度学习在影像声音领域以外的一些成果，都还在研究阶段。比如：

* Google出了一篇论文，将DL应用在推荐系统中
* 新加坡国立的论文，又玩了协同过滤，Neural Collaborative Filtering
* Stanford在自然语言处理中尝试DL
* 还有将DL用于时序分析
* Uber将神经网络用于出行预测

Ben总结了DL的三个特点：

* Big Model：有成千上万的参数
* Big Compute：需要很多的机器计算资源
* Big Data：需要很多数据做监督训练。不过，准确的说，要的不是Big Data，而是good clean training data

另外，Ben说了现在机器学习从传统离线学习到在线持续学习的趋势，并介绍了UC Berkeley大学新成立了一个RiseLab，旨在研究如何在live data上做实时决策。以前UC Berkeley成立过一个AMPLab，旨在做批量数据上的高级分析，孵化出了Mesos、Spark和Alluxio等业界爆品，真可谓是全球大数据界的半壁江山。这次的RiseLab值得关注，可以在github.com/ucbrise找到他们。

[PPT](https://cdn.oreillystatic.com/en/assets/1/event/273/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E6%97%B6%E4%BB%A3%EF%BC%88The%20Age%20of%20Machine%20Learning%EF%BC%89%20%E8%AE%B2%E8%AF%9D.pdf)

## 腾讯

腾讯现在主要业务分三块：

* 游戏：2016年游戏收入草果100亿美元。
* 社交：微信月活用户9.38亿（啥？没错，已经超过中国移动用户）
* 内容：腾讯的内容服务包括视频、音乐、新闻（“快报”日活用户2500万）和文学作品。

腾讯深圳AI实验室有50多位科学家和专家，200多研究工程师，今年5月在美国西雅图还建立了分舵。强化学习中心总监刘晗介绍，利用游戏+社交+内容带来的大数据，使用深度和强化学习，在计算机视觉（Computer Vision）、语音识别（Speech Recognition）、自然语言处理（NLP）、机器人（Robotics）和博弈（Gaming）方面进行尝试，并反馈到应用。

**游戏**

这次主要分享的是在游戏领域的AI。通常游戏分为：

* 棋牌类（Chess & Board），比如腾讯扑克，Go。AI相对简单点，利用大数据（Big Data，用Imitation Learning）开发出一个牛X的模拟器（Simulator，用Reinforement Learning）就行了。腾讯开发的Jueyi还赢得了2017年一个什么冠军。
* 多人在线（MOBA），比如农药，1.6亿MAU。这类游戏与棋牌类相比，一方面很简单，小学生都能上手，但对机器要求高，很难将棋类如alphago移植，具体难点在于：
	1. 多用户：决策需要有战略思想
	2. 实时：Multi-agent system

如果能解决这个问题，在真实社会中也能有很大的应用，如智能交通。

**社交**

介绍了对话机器人（Conversational AI），分两类：

* 聊天(Open-domain)：闲聊，尽量聊天时间长，要求像个博学者（Generalist）。这个就类似于图灵测试（Tuning测试其实就定义了什么是人工智能），现在的解决方案是基于搜索，需要海量的文本数据。
* 客服(Task-oriented)：时间尽可能短，要求像个专家（Specialist），如订票机器人。在这里可能做出比较好的用户模拟器(user simulator)。聊天机器人(Chatbot）通过Data进行Imitation Learning和Reinforcement Learning强化学习能学习型的场景，然后通过Learning methods进化到能解决以前没有遇到的问题(Generalizable to new unknown cases)，即zero shot。(别怪我说的这么绕，图揍是酱紫画的。没看懂不要紧，其实就是Imitation Learning和Reinforcement Learning）

![img](/img/content/2017-strata-data-beijing/tencent_ai_1.jpg)

**内容**

这里主要介绍的天天快报的精准推荐。基于新闻特征(News characteristics)、环境特征(Evironmental characteristics)、用户特征(User characteristics)和上下文特征(Context characteristics)为文章打分。

现在准备建立的AI平台，将NLP
、Speech、VIsion和Games等方面的能力对行业合作伙伴开放。估计就是能力搬上腾讯云。

## 京东: 电子商务的未来：AI和大数据

没介绍什么实质的东西，聊聊几页PPT，唯一的信息量是：

京东的核心竞争力：物流(Logistics)、无人商店(Smart Shop)和物联网(Smart IoT)。（我没听错？是Smart shop还是无人商店？无人商店新闻才出来多久，京东做了啥就成了“核心竞争力”了？）

京东AI的应用场景：智慧医疗(Medtech)、智慧城市(Smart City)、智能投顾(Fintech)和办公自动化(Auto-office)。（简而言之，京东的AI要出墙了。）

[PPT](https://cdn.oreillystatic.com/en/assets/1/event/273/%E7%94%B5%E5%AD%90%E5%95%86%E5%8A%A1%E7%9A%84%E6%9C%AA%E6%9D%A5%EF%BC%9AAI%E5%92%8C%E5%A4%A7%E6%95%B0%E6%8D%AE%EF%BC%88An%20ecommerce%20future_%20AI%20and%20big%20data%EF%BC%89%20%E8%AE%B2%E8%AF%9D.pdf)

## 滴滴：大数据在滴滴出行的应用

![img](/img/content/2017-strata-data-beijing/didi_1.jpg)

DiDi现在每天70TB新增数据，处理4.5PB数据，超过200亿次路径规划请求，140亿次定位。核心的项目包括：

* 路径规划(Route Planning)：这是派单的核心，主要目的是：
	* 最小化成本
	* 最大化司机效率
	* 最优化交通效率
* Estimated Time of Arrival(ETA)：估计路上时间和等待时间之和。以上两个地图服务是DiDi最核心也是用的最多的机器学习，架构如：
![img](/img/content/2017-strata-data-beijing/didi_2.jpg)
* Intelligent Order-Dispatching
![img](/img/content/2017-strata-data-beijing/didi_3.jpg)
* Tranportation Capacity Management
* Supply & Demand Forcasting
![img](/img/content/2017-strata-data-beijing/didi_4.jpg)
* Ride-pooling
* Jiu Xiao Visualization
* Safety Assessment
* Arbitration & Dispute Resolution

其中Supply & Demand Forcasting/Tranportation Capacity Management,Guess Your Destination和Sugggest Pickup Spots等还发表在了KDD 2017。

#资源

https://strata.oreilly.com.cn/strata-cn/public/schedule/proceedings
