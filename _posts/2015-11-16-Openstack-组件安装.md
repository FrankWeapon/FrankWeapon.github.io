---
layout: post
title: OpenStack-组件安装 
tags: OpenStack
---
<div id="table-of-contents">
<h2>Table of Contents</h2>
<div id="text-table-of-contents">
<ul>
<li><a href="#sec-1">1. controller</a>
<ul>
<li><a href="#sec-1-1">1.1. 注意事项</a></li>
<li><a href="#sec-1-2">1.2. Identity Service</a></li>
<li><a href="#sec-1-3">1.3. Image Service</a></li>
<li><a href="#sec-1-4">1.4. Compute Service</a></li>
<li><a href="#sec-1-5">1.5. Network Service</a></li>
<li><a href="#sec-1-6">1.6. DashBoard</a></li>
</ul>
</li>
<li><a href="#sec-2">2. network</a>
<ul>
<li><a href="#sec-2-1">2.1. 组件介绍</a></li>
<li><a href="#sec-2-2">2.2. Network Service</a></li>
<li><a href="#sec-2-3">2.3. 创建网络</a></li>
</ul>
</li>
<li><a href="#sec-3">3. compute</a>
<ul>
<li><a href="#sec-3-1">3.1. Compute Service</a></li>
<li><a href="#sec-3-2">3.2. Network Service</a></li>
</ul>
</li>
</ul>
</div>
</div>

# controller<a id="sec-1" name="sec-1"></a>

## 注意事项<a id="sec-1-1" name="sec-1-1"></a>

-   安装好组件后使用API执行命令可能会报错Server Internal Error 500。 这多数是由于数据库服务配置不正确导致的。 请检查上一节中提到的数据库安装
-   控制节点的Endpoint可能会由于不正当操作导致异常， 这时其他节点相关服务的日志中会有关于地址解析的日志记录。 可以直接进入数据库相关服务查看endpoints表，若发现其中有localhost.localdomain相关内容可试着改为controller。

## Identity Service<a id="sec-1-2" name="sec-1-2"></a>

-   依照官方提供的安装过程并未出现问题
-   请注意安装过程中的几个概念
    1.  Project：一个Project中有自己的quotas（配额，包括该项目中有多少资源权限，可以创建多少虚拟机等）
    2.  User：可以属于一个或多个Project，包含用户名密码，作为个人登录的凭证。
    3.  Role：定义了一组操作的权限集合，可赋予User。

## Image Service<a id="sec-1-3" name="sec-1-3"></a>

-   执行镜像创建语句
    
        glance image-create --name "cirros-0.3.4-x86_64" --file /tmp/images/ cirros-0.3.4-x86_64-disk.img   --disk-format qcow2 --container-format bare --visibility public --progress
    
    可能出现Bad Argument错误，搜索相关问题可能会找到关于Endpoint API版本的问题，而实际上问题可能不在这里。 我曾遇到这个问题始终找不到原因所在， 而重装后问题消失。 所以这个问题八成是配置文件的错误引起的，请仔细检查或重装该节点。
-   这里的镜像是不带GUI的，测试环境足够了。关于创建带有GUI的镜像在以后的章节中介绍。

## Compute Service<a id="sec-1-4" name="sec-1-4"></a>

-   控制节点上安装完Compute节点是没有验证的，在配置完计算节点后进行验证操作。
    
        nova service-list
    
    结果应至少包含以下几个：
    
    <table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">
    
    
    <colgroup>
    <col  class="left" />
    
    <col  class="left" />
    
    <col  class="left" />
    
    <col  class="left" />
    
    <col  class="left" />
    </colgroup>
    <thead>
    <tr>
    <th scope="col" class="left">Binary</th>
    <th scope="col" class="left">Host</th>
    <th scope="col" class="left">Zone</th>
    <th scope="col" class="left">Status</th>
    <th scope="col" class="left">State</th>
    </tr>
    </thead>
    
    <tbody>
    <tr>
    <td class="left">nova-conductor</td>
    <td class="left">controller</td>
    <td class="left">internal</td>
    <td class="left">enabled</td>
    <td class="left">up</td>
    </tr>
    
    
    <tr>
    <td class="left">nova-cert</td>
    <td class="left">controller</td>
    <td class="left">internal</td>
    <td class="left">enabled</td>
    <td class="left">up</td>
    </tr>
    
    
    <tr>
    <td class="left">nova-scheduler</td>
    <td class="left">controller</td>
    <td class="left">internal</td>
    <td class="left">enabled</td>
    <td class="left">up</td>
    </tr>
    
    
    <tr>
    <td class="left">nova-consoleauth</td>
    <td class="left">controller</td>
    <td class="left">internal</td>
    <td class="left">enabled</td>
    <td class="left">up</td>
    </tr>
    
    
    <tr>
    <td class="left">nova-compute</td>
    <td class="left">compute1</td>
    <td class="left">nova</td>
    <td class="left">enabled</td>
    <td class="left">up</td>
    </tr>
    </tbody>
    </table>
    
    可能包含其他项（应该是不启动状态），但这几项一定要正常。若有问题请检查控制、计算节点的NTP服务和消息队列日志。
-   执行Endpoint验证操作
    
        nova endpoints
    
    结果可能包含Warning，不影响使用。

## Network Service<a id="sec-1-5" name="sec-1-5"></a>

-   控制节点上的网络组件比较简单，保证验证结果包含文档中给出的内容即可。

## DashBoard<a id="sec-1-6" name="sec-1-6"></a>

-   Dashboard需要用到Apache，请确保其正确性。
-   安装后可能出现404，400错误。这种错误一般是由于配置不正确导致的，找不出错误的话重新安装。
-   登陆session超时之后就没法再登陆了（至少在FireFox上如此），没有详细错误信息，只提示请联系管理员。 这时候需要清理保存的session和cookie，重新打开主页即可。
-   安装完Dashboard之后很多操作就可以不用命令去执行了，要熟悉其功能。

# network<a id="sec-2" name="sec-2"></a>

除基础环境外，网络节点只配置了Network Service

## 组件介绍<a id="sec-2-1" name="sec-2-1"></a>

-   Modular Layer 2 (ML2) plug-in
    为实例提供虚拟网络的建立
-   Layer-3 (L3) agent
    为实例提供路由服务
-   DHCP agent
    在虚拟网络使用动态地址分配
-   metadata agent
    为实例提供配置信息，比如身份验证等

## Network Service<a id="sec-2-2" name="sec-2-2"></a>

-   4个组件有四个配置文件，同时还要将ml2的相关内容添加到neutron.conf中。 要确保这5个文件的正确性，分别查看日志排除错误信息。
-   这里需要一个网桥，ex-br与enp2s12（之前没设置IP地址的网卡）绑定，在创建网络之后所有的虚拟机都将由这条通路出去。

## 创建网络<a id="sec-2-3" name="sec-2-3"></a>

-   需要一些基本的网络概念，IP地址、子网掩码、子网划分。
-   内部图解
    ![网络节点内部](/assets/upload/OpenStack-网络节点.jpg)
-   External network
    1.  外部网络设置时的IP浮动范围，子网掩码等取决于enp2s12网卡的网络环境。
    2.  范围内的地址可作为租户网络的网关，绑定给租户路由，为实例提供网络访问。
    3.  范围内的地址可申请并绑定给某实例，这时该实例可以由ssh或windows远程桌面访问到。其可见性与enp2s12网段相同， 即若外部网络浮动范围内的IP地址为公网地址，则所有人都可以访问；若为内网地址则内网同一网段的人可以访问。
-   Tenant network
    1.  租户网络不唯一，可将虚拟机分配到不同的网络下方便管理。
    2.  每一个租户网络要对应一个租户路由，该路由器要绑定一个外部网络的IP地址作为网关，使该租户网络可获得网络连接。
    3.  分配给租户网络的虚拟机将由DHCP agent分配到一个该租户网络下的IP地址。

# compute<a id="sec-3" name="sec-3"></a>

计算节点配置基础环境后只需要安装计算和网络组件

## Compute Service<a id="sec-3-1" name="sec-3-1"></a>

-   在配置完后访问
    
    > <http://controller:6080/vnc_auto.html>
    
    可能会报错could not connect to server (1006)。 这里可以先不解决，在\*网络节点配置完成后，访问实例\*仍然报错的话，在确保网络通畅的情况下清除iptables表项即可。
    
    1.  在控制节点上执行
        
            tcpdump -nni eno1 port 5900
    2.  使用浏览器访问，应该在能看到相关输出，若没有请检查网络。
    3.  在计算节点上执行
        
            iptables -F
    4.  这时应该正常了
    5.  参考内容 <https://ask.openstack.org/en/question/46158/vnc-console-failed-to-connect-to-server-code-1006/>
-   计算节点要保证硬件支持虚拟化，在BIOS中CPU的BIOS CPU virtualization 要启用。

## Network Service<a id="sec-3-2" name="sec-3-2"></a>

-   计算节点只安装了openvswitch服务（ML2），提供了实例的虚拟网络建立。