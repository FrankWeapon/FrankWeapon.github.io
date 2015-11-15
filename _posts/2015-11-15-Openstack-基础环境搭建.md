---
layout: post
title: OpenStack-基础环境搭建 
tags: OpenStack
---
<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1. 网络环境</a>
<ul>
<li><a href="#sec-1-1">1.1. 网络概述</a></li>
<li><a href="#sec-1-2">1.2. 使用配置文件修改网络设置</a></li>
<li><a href="#sec-1-3">1.3. 路由器设置</a></li>
<li><a href="#sec-1-4">1.4. 修改hosts</a></li>
<li><a href="#sec-1-5">1.5. controller</a></li>
<li><a href="#sec-1-6">1.6. network</a></li>
<li><a href="#sec-1-7">1.7. compute1</a></li>
<li><a href="#sec-1-8">1.8. 验证</a></li>
</ul>
</li>
<li><a href="#sec-2">2. Network Time Protocal (NTP)</a>
<ul>
<li><a href="#sec-2-1">2.1. 问题解决</a></li>
<li><a href="#sec-2-2">2.2. 注意事项</a></li>
</ul>
</li>
<li><a href="#sec-3">3. OpenStack packages</a></li>
<li><a href="#sec-4">4. SQL database</a>
<ul>
<li><a href="#sec-4-1">4.1. 注意事项</a></li>
</ul>
</li>
<li><a href="#sec-5">5. Message queue</a>
<ul>
<li><a href="#sec-5-1">5.1. 注意事项</a></li>
</ul>
</li>
</ul>
</div>
</div>

# 网络环境<a id="sec-1" name="sec-1"></a>

openstack网络尤其重要，理解各节点组件作用，在正确稳定的网络结构下，可以避免很多不必要的问题。整体结构如图所示
![网络拓扑](/assets/upload/OpenStack-网络拓扑.jpg)
该文章中采centOS7的网卡命名，集成网卡名为eno1、插入的两张PCI网卡分别为enp2s10和enp2s12。

一定要采用命令行下编辑配置文件的方式来进行网络配置，使用GUI可能会导致配置文件的混乱，要是已经弄乱了的话，后面会有解决方案。

## 网络概述<a id="sec-1-1" name="sec-1-1"></a>

-   10.0.0.0/24
    该网段称作management，用作3台物理机间的数据通信。 需要一个网关（这里使用路由器）接入互联网。
-   10.0.1.0/24
    该网段称作tunnel，用作运行在compute上的实例与network间的数据通信，与controller无关。
-   外网
    只在network上有，不指定IP地址且BOOTPROTO为none，在安装好网络组件之前这块网卡并没有用。 创建网络环境时会将一段IP地址分配到这里，以池的形式提供NAT以及虚拟机的IP地址绑定。

## 使用配置文件修改网络设置<a id="sec-1-2" name="sec-1-2"></a>

centOS7 网络配置文件位于

> *etc/sysconfig/network-scripts*

目录下对应3块网卡的配置文件分别为ifcfg-eno1, ifcfg-enp2s10, ifcfg-enp2s12。

环境中所有IP均为静态指定，所以BOOTPROTO=static。network的第三块网卡除外，要设置为 BOOTPROTO=none。

编辑完成后重启网络

    systemctl restart network

## 路由器设置<a id="sec-1-3" name="sec-1-3"></a>

CentOS下无法直接进入路由器设置界面，需要一台windows接入LAN口输入192.168.1.1进入路由器设置。 路由器网段改为10.0.0.0,子网掩码255.255.255.0，路由器地址10.0.0.1，路由器重启后进入指定3台机器eno1的MAC地址和IP地址
1.  controller
    10.0.0.11
2.  network
    10.0.0.21
3.  compute1
    10.0.0.31
4.  compute2(可选）
    10.0.0.32（以此类推）

注意，如果你之前在其他连入LAN口的电脑上配置了地址10.0.0.1，会导致无法进入路由器设置页面，记得修改它以避免冲突。

## 修改hosts<a id="sec-1-4" name="sec-1-4"></a>

每个节点都需要修改hosts文件，指定各节点的通信地址。配置文件在
    
    > /etc/host

\#controller
10.0.0.11 controller

\#network
10.0.0.21 network

\#compute1
10.0.0.31 compute1

\#compute2
10.0.0.32 compute2

## controller<a id="sec-1-5" name="sec-1-5"></a>

-   ifcfg-eno1
    
    > BOOTPROTO=static
    > IPADDR=10.0.0.11
    > NETMASK=255.255.255.0
    > GATEWAY=10.0.0.1
    > 
    > DNS = 8.8.8.8
    >
    > DNS1 = 8.8.4.4

## network<a id="sec-1-6" name="sec-1-6"></a>

-   ifcfg-eno1
    
    > BOOTPROTO=static
    > IPADDR=10.0.0.21
    > NETMASK=255.255.255.0
    > GATEWAY=10.0.0.1
    > 
    > DNS = 8.8.8.8
    >
    > DNS1 = 8.8.4.4

-   ifcfg-enp2s10
    
    > BOOTPROTO=static
    > IPADDR=10.0.1.21
    > NETMASK=255.255.255.0

-   ifcfg-enp2s12
    
    > BOOTPROTO=none

## compute1<a id="sec-1-7" name="sec-1-7"></a>

-   ifcfg-eno1
    
    > BOOTPROTO=static
    > IPADDR=10.0.0.31
    > NETMASK=255.255.255.0
    > GATEWAY=10.0.0.1
    > 
    > DNS = 8.8.8.8
    >
    > DNS1 = 8.8.4.4
-   ifcfg-enp2s10
    
    > BOOTPROTO=static
    > IPADDR=10.0.1.31
    > NETMASK=255.255.255.0

## 验证<a id="sec-1-8" name="sec-1-8"></a>

-   所有主机都能ping通外网
-   所有主机之间使用hosts文件中的名字都能相互ping通
-   compute和network可以ping通对方tunnel地址

# Network Time Protocal (NTP)<a id="sec-2" name="sec-2"></a>

## 问题解决<a id="sec-2-1" name="sec-2-1"></a>

-   控制节点上要填一个地址，用于统一时间，许多都不能用，在验证的时候显示.INIT.
    
    这里给出国家授时中心的域名
    
    > cn.pool.ntp.org
    > 1.cn.pool.ntp.org
-   防火墙可能会导致NTP的通信异常，可以关闭并禁用防火墙
    
        systemctl stop firewalld
        systemctl disable firewalld

## 注意事项<a id="sec-2-2" name="sec-2-2"></a>

-   其他节点都以controller作为时间标准。以后安装过程中遇到问题可以先验证NTP是否运行正常，有时网络问题会导致组件功能不正常。

# OpenStack packages<a id="sec-3" name="sec-3"></a>

这里的按照官方文档安装没有问题

# SQL database<a id="sec-4" name="sec-4"></a>

## 注意事项<a id="sec-4-1" name="sec-4-1"></a>

-   记得要执行Mysql的安装过程
    
        mysql_secure_installation
    
    少了这一步可能会导致在之后调用程序方法的时候报服务器500错误
-   数据库出现问题是可以删掉重新安装的，数据会存放在另一个地方，将程序重装后数据还在。

# Message queue<a id="sec-5" name="sec-5"></a>

## 注意事项<a id="sec-5-1" name="sec-5-1"></a>

-   消息队列是保证物理机之间消息通信的基础，组件出现问题的时候可以查看消息队列的日志观察是否有问题。 有的时候无法正确解析地址会导致消息队列不通使程序出现问题。 消息队列的问题不会报错，查看日志文档(存放在/var/log/rabbit<sub>mq下</sub>)会发现ERROR，根据提示解决问题即可。