---
title: CentOS 下的LVM卷扩容
url: 1530.html
id: 1530
categories:
  - 未分类
date: 2019-12-14 19:44:00
tags:
---

上周在k8s 的集群里进行pod部署的时候，发现出了`evicted` 的情况，意思说是北驱逐，也就是目标的节点无法满足 部署需要。所以导致被驱逐到其他的节点。 检查日志发现是由于 磁盘不满足需求导致pod被驱逐。所以需要北目标节点来进行扩容。

前
-

主机扩容一直是一个麻烦事情，经常出现使用 `fdisk`修改完分区表之后开不了机的情况，所以改磁盘分区的时候还是比较紧张，但是这次在用`df -h` 看了之后，发现

    Filesystem                           Size  Used Avail Use% Mounted on
    /dev/mapper/centos-root               29G   21G  8.2G  72% /

不是磁盘分区直接进行的挂载，而是使用的 `LVM` 来进行的挂载。这样使得磁盘扩容的过程就小了非常多，而且可以进行**不停机**的磁盘扩容。

关于 `LVM` 技术 可以看其百科的说明。

> 因此完美的解决方法应该是在零停机前提下可以自如对文件系统的大小进行调整，可以方便实现文件系统跨越不同磁盘和分区。幸运的是Linux提供的逻辑盘卷管理（LVM，LogicalVolumeManager）机制就是一个完美的解决方案。

这一篇实战贴就算是记录一下整个磁盘扩容的过程，以便后面再次遇上使用。

过程
--

在过程上分为两个：

1.  建立磁盘分区
2.  拓展LVM卷
3.  文件系统扩容
    
    ### 建立磁盘分区
    
    由于节点主机是使用 esxi 来进行虚拟化部署的，所以可在控制台上很容易的进行磁盘的扩容操作。扩容完成之后，登录主机使用 `fdisk` 查看扇区的范围：
    
        Disk /dev/sda: 34.4 GB, 34359738368 bytes, 67108864 sectors
    
    实际上sda1和sda2没有完全使用磁盘的空间，所以这里需要新建一个分区，过程如下:
    

    fdisk /dev/sda
    n # 创建新分区
        p # primary 分区
        3 #分区号
    t # 修改分区类型
        3 # 选择上面的分区号
        8e # LVM 的类型
    w # 写入磁盘分区表
    
    # 让系统刷新分区情况
    portprobe
    
    # 选择刚刚创建的分区进行格式化
    mkfs.ext3 /dev/sda3

完成上上面的步骤之后就有了一个可用的ext3的分区了。完成创建分区的步骤

### 添加分区至LVM组

这里需要把新建的分区添加到已经挂载在`/`的卷。这里需要使用 `LVM` 来进行管理了，流程如下：

    #进入lvm管理
    lvm
    
    # 这是初始化刚才的分区3
    lvm>pvcreate /dev/sda3
    # 将初始化过的分区加入到虚拟卷组centos (卷和卷组的命令可以通过 vgdisplay )
    lvm>vgextend centos /dev/sda3    
    # vgdisplay查看free PE /Site
    lvm>vgdisplay -v
    #扩展已有卷的容量（6143 是通过vgdisplay查看free PE /Site的大小）
    lvm>lvextend -l+6143 /dev/mapper/centos-root
    
    #查看卷容量，这时你会看到一个很大的卷了
    lvm>pvdisplay
    #退出
    lvm>quit

### 文件系统扩容

在上面LVM扩容完之后，就需要进行文件系统的扩容，在Centos7 中有对应的工具，由于安装的系统是直接挂载在根目录的。所以这里直接

    xfs_growfs /dev/mapper/centos-root

在执行完之后，可以用df命令看到，已经玩完成了扩容的过程，整个过程完全无痛，不用停机的调整，nice～

参考
--

> [VMware虚拟机中CentOS7的硬盘空间扩容](https://www.cnblogs.com/Sungeek/p/9084510.html)