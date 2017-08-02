---
layout:     post
title:      "2017 Strata Datad大会Beijing站参会记"
subtitle:   " \"2017 Strata Data Beijing\""
date:       2017-07-16 00:00:00
author:     "biaobean"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Strata
---

Orelly的Strata全球大会引入中国已经第二年，这是门票要大好几千的高B格的技术会议，也是广大海外IT民工蹭会归国的探亲会:)。去蹭吃蹭喝，那是必须的。

这次Strata的官方规格还是很高的，Cloudera公司来的是Hadoop吉祥物道哥和CTO Amr，这对儿胖头陀和瘦头陀在一个技术大会上同时出现，这些年在全球都是屈指可数的；Intel家来的是会议主席Jason的老板马自雅女士。

大公司的调调B格总是很高尚的，Ziya介绍了Intel在大数据上的愿景：
1. “通过硬件升级和软件优化推动技术民主化”，通俗的讲就是让屌丝也用得起大数据和人工智能。确实，正如罗胖所说，商业的目的不是让英国女王拥有更多的丝袜（不是让高大上的HPC变得更快），而是制造老百姓也穿得起丝袜。不过，这里Intel解决的只是计算速度问题，现在云计算已经解决了计算的资源门槛，但技术能力门槛却才是让很多传统企业望尘莫及的。
	1. Intel新一代至强芯片性能提升了数倍。现在无论是大数据还是深度学习等对于计算的要求不是数倍的增长，而是好几个数量级，显然单纯通过CPU的提速完全不满足大数据界应用的需求了。Intel尽力想把任务留在CPU，而N家往CPU以外拉，现在从股票和技术的发展看，明显N家更火。哎，Intel当年没把N家买下来，估计心情就和俺们买房子一样：当初怎么就嫌它贵没买呢？现在可好，涨成这样买不起了....话说发现Intel居然还有自己的Python发行版？！这不长记性的。。。。。。
	1. HBase：非堆存储的读操作提升了5倍。
	1. Spark：基于Intel实现的数学库能将Spark分析的性能提高4.3倍。
1. “为新兴的需求、新技术提供新的解决方案”，这里其实就是推Intel出的BigDL啦，通过它构建Spark+ML端到端深度学习流水线，其优点包括：
	1. 将深度学习放到已有的大数据加速平台，即复用现在的Hadoop/Spark框架；
	1. 现有机器学习框架不是为了Spark和扩展性所设计，BigDL在利用分布式框架获得性能提升方面更有优势；
	1. 硬件利旧。现在的ML通常需要高配置专有的机器，而传统的大数据服务器配置比较低；
	1. 高性能，缩短训练周期。
1. “推动创新，为客户解决更新更复杂的问题”。这个说的是行业里的案例，牛吹的比较虚，还不如不讲，Intel也不是干这事儿的，略过。

[Ziya的PPT](https://cdn.oreillystatic.com/en/assets/1/event/273/英特尔技术加速实现分析与人工智能的未来 - 英特尔赞助 _Accelerating the future for analytics and AI with Intel technologies—sponsored by Intel_ 讲话 1.pdf)



https://strata.oreilly.com.cn/strata-cn/public/schedule/proceedings

Intel
