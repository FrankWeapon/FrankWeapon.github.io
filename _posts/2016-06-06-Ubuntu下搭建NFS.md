---
layout: post
title: Ubuntu 下搭建 NFS
comments: true
---

# 基本介绍

NFS (Network File System)是一种分布式文件系统，可以在客户端将服务器的某一目录挂载，像访问本地文件一样访问服务器文件。

-   服务器：192.168.0.1
-   客户端：192.168.0.101

# 软件安装

## 服务器:

在服务器上安装 nfs-kernel-server 执行以下命令：

```shell
    sudo apt-get update
    sudo apt-get install nfs-kernel-server
```
## 客户端

在客户端上安装 nfs-common

```shell
    sudo apt-get update
    sudo apt-get install nfs-common
```

# 在服务器设置共享目录<a id="orgheadline5"></a>

1.  在服务器上创建共享目录
```shell
        sudo mkdir /var/nfs
```
2.  修改该目录拥有者和用户组
```shell
        sudo chown nobody:nogroup /var/nfs
```
3.  修改配置文件
    配置文件在 `/etc/exports` 内有示例。
    比如：我们挂载/home以及/var/nfs 供IP地址为192.168.1.101的客户端访问
    >    /var/nfs    192.168.1.101(rw,sync,no_subtree_check)

    其中参数含义如下：
    -   rw: 允许客户端对该目录读和写。
    -   sync: 强制NFS 在返回消息之前向磁盘中写入文件。 通常情况下，该参数给环境带来更高稳定性。
    -   nosubtreecheck: 在每次请求之前都查看当前用户是否有权限对其子目录进行操作，带来更高安全性。
    -   norootsquash: 关闭客户端对该目录的root权限。
4.  创建NFS表项 状态并启动服务
```shell
        sudo exportfs -a
        sudo service nfs-kernel-server start
```
# 在客户端设置挂载点<a id="orgheadline6"></a>

1.  在mnt目录下创建挂载点
```shell
       sudo mkdir -p /mnt/nfs/var/nfs
```
2.  挂载服务器目录
```shell
       sudo mount 192.168.1.1:/var/nfs /mnt/nfs/var/nfs
```
3.  查看挂载目录
```shell
        df -h
        mount -t nfs
```
    我们可以在最后看到刚挂载的目录。
# 设置自动挂载目录<a id="orgheadline7"></a>
向 `/etc/fstab` 中写入配置项
>   192.168.1.1:/var/nfs    /mnt/nfs/var/nfs   nfs auto,noatime,nolock,bg,nfsvers=4,sec=krb5p,intr,tcp,actimeo=1800 0 0
