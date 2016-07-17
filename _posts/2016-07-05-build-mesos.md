---
layout:     post
title:      "Build Mesos"
subtitle:   "Mesos Step 1"
date:       2016-07-05 12:00:00
author:     "biaobean"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Mesos
---

今天下载了最新的Mesos代码，在CentOS 7上按照官网http://mesos.apache.org/gettingstarted/进行编译，将爬过的坑记录如下：

1. SVN 1.9是不需要的，yum自带的1.7就行了。安装1.9还有一堆不兼容错误，直接跳过安装。
2. 需要一堆的依赖并没在官网页面终列出，如libapr-1，libz等等，Build前先运行下面的语句：

```
sudo yum install -y zlib-devel apr-devel libcurl-devel cyrus-sasl cyrus-sasl-devel cyrus-sasl-md5 subversion-devel

## libelf
sudo yum install -y elfutils-libelf-devel*

```
