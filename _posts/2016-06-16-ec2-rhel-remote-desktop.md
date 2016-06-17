---
layout:     post
title:      "如何远程使用在AWS EC2上RHEL7的图形桌面"
subtitle:   "安装Redhat桌面包并启动VNC连接"
date:       2016-06-16 12:00:00
author:     "biaobean"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - EC2
    - AWS
    - Red Hat
    - Remote
---

安装Redhat的桌面包，再启动一个VNC远程连接就行了。

具体操作主要分三个阶段：安装GUI组件、启动VNC服务端和建立连接。具体操作如下：


## 安装GUI组件
1. 更新系统(可选)

```shell
sudo yum update -y
```
2. 安装gnome GUI组件

```shell
sudo yum groupinstall -y "Desktop" "Desktop Platform" "X Window System" "Fonts"
sudo yum groupinstall -y "Server withGUI"
```
3. 启动时默认启动GUI

```shell
sudo systemctl set-default graphical.target
sudo systemctl default
```
4. 安装其他一些可能依赖的组件

```shell
sudo yum install -y pixman pixman-devel libXfont
```
现在所有的GUI组件已经安装OK。
## 安装并启动VNC服务端
5. 安装tiger VNC服务端

```shell
sudo yum install -y tigervnc-server
```
6. 为需要启动VNC桌面的用户创建密码。(通常EC2上的缺省连接用户为ec2-user，暴力点也可以直接用root)

```shell
sudo passwd ec2-user
```
7. 设置VNC连接密码

```shell
vncpasswd
```
8. 修改sshd_config文件，将passwordauthentication参数设置为“yes”
9. 重启sshd服务

```shell
sudo service sshd restart
```
10. ~~启动VNCServer服务：sudo service vncserver start~~
10. 启动VNCServer，可以使用geometry参数设置屏幕大小，注意没空格。

```shell
vncserver -geometry1024x768
```
你可以运行多次vncserver命令启动多个连接供不同用户使用。每一个有不同的连接号。
界面输出样例如下，注意记录下"desktop is"后面的桌面ID，供客户端连接用。

```shell
StartingVNC server: 1:ec2-user 

xauth:creating new authority file /home/ec2-user/.XauthorityNew 'ip-172-31-16-193:1(ec2-user)'

 desktop is ip-172-31-16-193:1

Creatingdefault startup script /home/ec2-user/.vnc/xstartupStarting applications 

specifiedin  /home/ec2-user/.vnc/xstartupLog 

file is/home/ec2-user/.vnc/ip-172-29-4-27:1.log                               [ OK ]
```
11.	配置端口。VNC前99个连接依次使用5901到5999端口。对于第一个启动的VNC桌面，系统使用连接号1，端口为5901。使用如下命令将对应端口打开：

```shell
iptables -AINPUT -m state --state NEW -m tcp -p tcp --dport 5901 -j ACCEPT
```

## 客户端连接
12. 现在服务器端所有工作已经完成，只需要本地启动VNC客户端。可以通过这个[地址](http://www.realvnc.com/download/viewer/)下载并安装不同操作系统的版本。

13. 填写服务器公有IP以及VNC桌面ID，点击连接。
14. 输入在vncpasswd命令（第7步）中建立的密码，打开GUI桌面。
15. 如果GUI桌面需要输入密码，输入在passwd（第6步）中建立的密码。
