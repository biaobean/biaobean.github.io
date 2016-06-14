---
layout:     post
title:      "Cloudera Manager API开发"
subtitle:   "cm api"
date:       2016-06-14 12:00:00
author:     "biaobean"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Hadoop
    - API
    - Cloudera
    - Java
---
# 如何查看API版本
1. 通过页面http://[cm-server-host]:[cm-server-port]/api/version查看；
2. 通过Cloudera Manager的API帮助页面http://[cm-server-host]:[cm-server-port]/cmf/static/apidocs可查看；
3. 通过在线文档

# API帮助

1. 通过https://cloudera.github.io/cm_api/
2. 在线资源：http://www.cloudera.com/documentation/enterprise/5-5-x/topics/cm_intro_api.html
3. 在线资源：http://www.cloudera.com/documentation/enterprise/latest/topics/cm_intro_api.html

# Example Code
https://github.com/biaobean/cm_api_example

# 碰到的那些坑

## Hadoop参数必须和范围或者角色对应

不同范围的参数是不能乱射的，比如，服务范围的，不能在角色组或者角色上配置。强制的保证整个服务范围内的配置是一致的，比如dfs.replication这个原本是本地的配置，不能砸在DataNode上设置：
![img](/img/content/clouder-manager-api/1.png)

只能在服务范围：
![img](/img/content/clouder-manager-api/2.png)
## 用户自定义参数
用户定义的参数不能直接做为config push到系统里，这能当作xxxx-safety-valve的内容保存。
![img](/img/content/clouder-manager-api/3.png)
## Hadoop配置中同样的参数，在CM的API上可能在不同的角色对应不同的参数
比如，内存，xxx-env.sh中的值等。

## 缺省有角色组，命名规则为“服务-角色-BASE”。

## 通过API添加的角色，系统不会做自动的缺省配置，如data.dir等。

