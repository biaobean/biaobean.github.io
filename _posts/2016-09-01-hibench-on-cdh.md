---
layout:     post
title:      "如何在CDH上跑HiBench"
subtitle:   "How to run HiBench on CDH"
date:       2016-09-01 12:00:00
author:     "biaobean"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Hadoop
    - CDH
    - HiBench
---

## 配置Hadoop路径

修改conf/99-user_defined_properties.conf文件，找到包含下面左边参数的相应行，并将值修改为右边：

```
hibench.hadoop.home             /opt/cloudera/parcels/CDH/lib/hadoop

hibench.spark.home              /opt/cloudera/parcels/CDH/lib/spark

hibench.hdfs.master             hdfs://172.31.14.227 ##这个需要按实际的部署来配

hibench.spark.master            yarn-client

hibench.hadoop.executable       /usr/bin/hadoop ## 这行被注释了，一定要配，否则monitor不起作用

hibench.hadoop.version          hadoop2

hibench.spark.version          spark1.6 ##这个需要按实际的Spark版本来配
```

## 其它bug的临时解决方案

### exmaple.jar

现在对于CDH的hadoop-mapreduce-examples.jar的查找是这样写死的：

```
HibenchConf["hibench.hadoop.examples.jar"] = OneAndOnlyOneFile(HibenchConf['hibench.hadoop.mapreduce.home'] + "/share/hadoop/mapreduce2/hadoop-mapreduce-examples-*.jar")
```

CDH的parcel方式安装中并没有这个目录。由于code没有加入判断逻辑，让参数没有设置的时候才进行查找，因此必须手动地查找相应代码，并修改为：
```
HibenchConf["hibench.hadoop.examples.jar"] = OneAndOnlyOneFile(HibenchConf['hibench.hadoop.mapreduce.home'] + "/../../jars/hadoop-mapreduce-examples-*.jar")
```

参见https://github.com/intel-hadoop/HiBench/issues/301。

### 其它 

如果在map-site.xml里设置了mapreduce.map.java.opts和mapreduce.reduce.java.opts，需要对应地设置在脚本内部人肉修改MAP_JAVA_OPTS和RED_JAVA_OPTS。

原因是现在MAP_JAVA_OPTS的获得是通过下面的shell命令：

```
cat /opt/cloudera/parcels/CDH/lib/hadoop/etc/hadoop/mapred-site.xml | grep "mapreduce.map.java.opts" | awk -F\< '{print $5}' | awk -F\> '{print $NF}''
```

而如果mapreduce.map.java.opts参数的名称和值不在同一行，MAP_JAVA_OPTS就会被设成空。RED_JAVA_OPTS也类似。参见https://github.com/intel-hadoop/HiBench/issues/302。
