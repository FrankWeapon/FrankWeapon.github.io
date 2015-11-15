---
layout: post
title: Openstack-功能介绍
tags: OpenStack
---
<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1. 写在前面</a></li>
<li><a href="#sec-2">2. 节点功能</a>
<ul>
<li><a href="#sec-2-1">2.1. controller</a></li>
<li><a href="#sec-2-2">2.2. network</a></li>
<li><a href="#sec-2-3">2.3. compute</a></li>
<li><a href="#sec-2-4">2.4. block storage (可选）</a></li>
<li><a href="#sec-2-5">2.5. object storage (可选）</a></li>
</ul>
</li>
<li><a href="#sec-3">3. 组件功能</a>
<ul>
<li><a href="#sec-3-1">3.1. Identity Service 身份验证服务 （Keystone)</a></li>
<li><a href="#sec-3-2">3.2. Image Service 镜像服务 （Glance)</a></li>
<li><a href="#sec-3-3">3.3. Compute Service 计算服务 (Nova)</a></li>
<li><a href="#sec-3-4">3.4. Networking Service 网络服务 (Neutron)</a></li>
<li><a href="#sec-3-5">3.5. DashBoard 图形化管理界面 (Horizon)</a></li>
</ul>
</li>
</ul>
</div>
</div>

# 写在前面<a id="sec-1" name="sec-1"></a>

-   openstack作为一个开源项目，每隔一段时间都会发布一个版本而且有两种部署方式，在不同平台上有不同的安装方式。 这里介绍的为kilo版本采用neutron布局，在centOS7下，3台物理机上部署安装。 在官方网站提供了详尽的部署文档（英文版），应该以该文档为主，这里记录一些问题的补充说明作为辅助。
-   根据个人经验，总体上对openstack有个概念性的了解，明白组件功能所在对部署时问题的解决有很大帮助，否则面对问题会有些无从下手。 请花些时间耐心阅读组件功能介绍。
-   有些问题源于配置文件的错误，可能是在复制过程中无意发生差错，这种问题在查找过程中不太容易发现。 若出现解决不掉的问题可以考虑将问题组件的配置文件备份后删除，然后重新安装该组件，再次配置问题可能就不复存在了。
-   注意验证工作，不要轻易跳过验证不通过的部分，除非你对该部分的原理相当了解，能弄明白验证不通过的原因所在。 否则请认真进行每一步验证工作，这样在问题出现时才有相当于检查点的存在。 不至于找不到问题所在而彻底重新安装。

# 节点功能<a id="sec-2" name="sec-2"></a>

节点指一台计算机，若采用neutron布局，则最少需要3台计算机。 包括一台计算节点（controller)需要1张网卡，一台网络节点（network)需要3张网卡，一台计算节点（compute1)需要两张网卡。 若需要加入存储节点请参照官网配置。

## controller<a id="sec-2-1" name="sec-2-1"></a>

该节点相当于项目经理，大部分的组件都需要安装在该节点上，任务的分配也都在这里进行，但该节点并不承担虚拟机的运行负载，也不承担网络负载。数据库只运行在该节点上，Dashboard也部署在这台机器的Apache上，所以如果想要外网访问Dashboard就需要给它额外一张网卡并提供公网地址。

## network<a id="sec-2-2" name="sec-2-2"></a>

该节点承担所有实例（运行在openstack上的虚拟机）的网络访问职责，注意这里只负责实例的网络访问，和物理机无关。每个物理机通过自己的网线上网。除基本环境外只需要安装网络组件。

## compute<a id="sec-2-3" name="sec-2-3"></a>

所有的实例都运行在该节点上，虚拟机所占用的资源均为该节点的硬件资源，可在环境中添加多个compute节点。

## block storage (可选）<a id="sec-2-4" name="sec-2-4"></a>

简单来说相当于硬盘，以块存储序列化后的内容，添加该节点后需要额外网卡进行存储通信。

## object storage (可选）<a id="sec-2-5" name="sec-2-5"></a>

简单来说相当于内存，以对象的形式存储数据，添加该节点后需要额外网卡进行存储通信。

# 组件功能<a id="sec-3" name="sec-3"></a>

组件指安装在各节点上的软件，在不同节点上需要的组件有所不同，安装时注意选择。

## Identity Service 身份验证服务 （Keystone)<a id="sec-3-1" name="sec-3-1"></a>

-   功能：
    1.  记录用户以及其权限
    2.  提供了每个用户的可用服务列表

## Image Service 镜像服务 （Glance)<a id="sec-3-2" name="sec-3-2"></a>

-   功能：
    提供对包括制作在内的镜像管理。

## Compute Service 计算服务 (Nova)<a id="sec-3-3" name="sec-3-3"></a>

-   功能：
    作为IaaS最主要的组成部分，计算服务负责云操作系统的运行。

## Networking Service 网络服务 (Neutron)<a id="sec-3-4" name="sec-3-4"></a>

-   功能：
    提供所有虚拟机的网络功能，包括基础网络设施的搭建以及网络访问控制。

## DashBoard 图形化管理界面 (Horizon)<a id="sec-3-5" name="sec-3-5"></a>

-   功能：
    通过浏览器打开网页操作Openstack，否则只能记住大量API通过控制台控制。