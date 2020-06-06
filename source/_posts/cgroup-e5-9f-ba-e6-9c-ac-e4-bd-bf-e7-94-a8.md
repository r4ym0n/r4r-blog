---
title: Cgroup 基本使用
tags:
  - Docker
  - Linux
url: 1316.html
id: 1316
categories:
  - OP之路
date: 2019-10-27 00:47:44
---

Cgroup是Linux 内核中的重要功能，和Namespace 构成了当前的热门技术---容器。

> cgroups的一个设计目标是为不同的应用情况提供统一的接口，从控制单一进程（像nice）到操作系统层虚拟化（像OpenVZ，Linux-VServer，LXC）。cgroups提供： 资源限制：组可以被设置不超过设定的内存限制；这也包括虚拟内存。\[3\] \[4\] 优先级：一些组可能会得到大量的CPU\[5\] 或磁盘IO吞吐量。\[6\] 结算：用来衡量系统确实把多少资源用到适合的目的上。\[7\] 控制：冻结组或检查点和重启动。\[7\] ----wikipedia

简单的说，可以对应用进行资源层面的组控制。这里将使用 Cgroup实现基本的资源控制的应用。

使用准备
----

根据一般的标准流程，是需要对cgroup的目录在挂载点进行挂载，但这里就图简单，直接在控制项目里创建控制组，先可以看到Cgroup中的极大资源类型：

    ➜ cd /sys/fs/cgroup
    ➜  cgroup ll
    total 0
    drwxr-xr-x 5 root root  0 Aug  1 22:06 blkio
    lrwxrwxrwx 1 root root 11 Aug  1 22:06 cpu -> cpu,cpuacct
    lrwxrwxrwx 1 root root 11 Aug  1 22:06 cpuacct -> cpu,cpuacct
    drwxr-xr-x 5 root root  0 Aug  1 22:06 cpu,cpuacct
    drwxr-xr-x 4 root root  0 Aug  1 22:06 cpuset
    drwxr-xr-x 5 root root  0 Aug  1 22:06 devices
    drwxr-xr-x 4 root root  0 Aug  1 22:06 freezer
    drwxr-xr-x 4 root root  0 Aug  1 22:06 hugetlb
    drwxr-xr-x 5 root root  0 Aug  1 22:06 memory
    lrwxrwxrwx 1 root root 16 Aug  1 22:06 net_cls -> net_cls,net_prio
    drwxr-xr-x 4 root root  0 Aug  1 22:06 net_cls,net_prio
    lrwxrwxrwx 1 root root 16 Aug  1 22:06 net_prio -> net_cls,net_prio
    drwxr-xr-x 4 root root  0 Aug  1 22:06 perf_event
    drwxr-xr-x 5 root root  0 Aug  1 22:06 pids
    drwxr-xr-x 5 root root  0 Aug  1 22:06 systemd

* * *

在本次的使用中，选取基本的 `cpu，memory，blkio`这几类，分别是 CPU计算资源，内存，块设备IO。 默认的Cgroup是系统全局，所以我们需要创建自己的一个子系统（subsystem），

    ➜  cgroup cd cpu
    ➜  cpu mkdir foo
    ➜  foo ll
    total 0
    -rw-r--r-- 1 root root 0 Oct 27 17:51 cgroup.clone_children
    --w--w--w- 1 root root 0 Oct 27 17:51 cgroup.event_control
    -rw-r--r-- 1 root root 0 Oct 27 17:51 cgroup.procs
    -r--r--r-- 1 root root 0 Oct 27 17:51 cpuacct.stat
    -rw-r--r-- 1 root root 0 Oct 27 17:51 cpuacct.usage
    -r--r--r-- 1 root root 0 Oct 27 17:51 cpuacct.usage_percpu
    -rw-r--r-- 1 root root 0 Oct 27 17:51 cpu.cfs_period_us
    -rw-r--r-- 1 root root 0 Oct 27 17:51 cpu.cfs_quota_us
    -rw-r--r-- 1 root root 0 Oct 27 17:51 cpu.rt_period_us
    -rw-r--r-- 1 root root 0 Oct 27 17:51 cpu.rt_runtime_us
    -rw-r--r-- 1 root root 0 Oct 27 17:51 cpu.shares
    -r--r--r-- 1 root root 0 Oct 27 17:51 cpu.stat
    -rw-r--r-- 1 root root 0 Oct 27 17:51 notify_on_release
    -rw-r--r-- 1 root root 0 Oct 27 17:51 tasks

CPU资源限制
-------

在Cgroup的 CPU 下可以看到有很多分类

    total 0
    -rw-r--r-- 1 root root 0 Oct 27 17:51 cgroup.clone_children
    --w--w--w- 1 root root 0 Oct 27 17:51 cgroup.event_control
    -rw-r--r-- 1 root root 0 Oct 27 17:51 cgroup.procs
    -r--r--r-- 1 root root 0 Oct 27 17:51 cpuacct.stat
    -rw-r--r-- 1 root root 0 Oct 27 17:51 cpuacct.usage
    -r--r--r-- 1 root root 0 Oct 27 17:51 cpuacct.usage_percpu
    -rw-r--r-- 1 root root 0 Oct 27 17:51 cpu.cfs_period_us
    -rw-r--r-- 1 root root 0 Oct 27 17:51 cpu.cfs_quota_us
    -rw-r--r-- 1 root root 0 Oct 27 17:51 cpu.rt_period_us
    -rw-r--r-- 1 root root 0 Oct 27 17:51 cpu.rt_runtime_us
    -rw-r--r-- 1 root root 0 Oct 27 17:51 cpu.shares
    -r--r--r-- 1 root root 0 Oct 27 17:51 cpu.stat
    -rw-r--r-- 1 root root 0 Oct 27 17:51 notify_on_release
    -rw-r--r-- 1 root root 0 Oct 27 17:51 tasks

* * *

这里用来进程限制的是 `cpu.cfs_quota_us` 这个。quota翻译过来是配额的意思。 这里的 CFS， 全称是 `Completely Fair Scheduler` 完全公平调度器。简单来讲，CPU的时间片，对每个进程是完全公平的。这里设置的 quota，就是对进程所分的时间片的限制。

    ➜  cpuacct cat cpu.cfs_period_us
    100000
    ➜  cpuacct cat cpu.cfs_quota_us
    -1

我们查看当前的配置情况，可以看到配额限制是 `-1` 也就是进程可以占用100%的CPU。

### 占满CPU

使用连续的hash计算来吃满CPU

    ➜  ~ cat /dev/zero| md5sum
    ➜  ~ top
     1461 root      20   0    7448    376    284 R  85.2  0.0   0:12.48 md5sum

使用TOP命令，可以看到其CPU占用是很高的。

### 资源限制

得到来进程的PID`1461` 之后，就开始对进程进行CPU限制，在我们的自建的subsystem中`/sys/fs/cgroup/cpu/foo` ，

    # 把进程加入子系统控制
    echo 1461 > cgroup.procs
    # 设置 CPU配额
    echo 8333 > cpu.cfs_quota_us
    # 查看总时间片
    cat cpu.cfs_period_us
    100000

那么可以推断出，我们现在可以使用的CPU是`8333/10000=8.33%`，我们使用top查看资源使用情况验证：

     1461 root      20   0    7448    376    284 R   8.3  0.0   2:33.05 md5sum

可见，资源现在的CPU使用刚好是稳定在 8.3% 。

块设备IO限制
-------

使用Cgroup，来对块设备的IO进行占用

### IO占用

这里是用dd来进行IO资源的占用；

    dd if=/dev/zero of=./tmp bs=1k count=1024000000 oflag=sync

向当前目录写10G的文件。使用iotop查看io情况：

    10171 be/4 root        0.00 B/s  169.88 M/s  0.00 %  0.00 % dd if=/dev/ze~unt=1024000000

当前的写速度是 169MB/s，注意这里需要设置 `oflag`,否则会默认写入缓存，不受我们的控制组控制。

### 资源限制

这里对最大的IO速度进行限制

    mkdir -p /sys/fs/cgroup/blkio/foo
    # 1048576字节每秒，也就是1M/s throttle 节流阀
    echo '252:16    1024000' >  blkio.throttle.write_bps_device
    echo '252:16    1024000' >  blkio.throttle.read_bps_device
    #252:16对应主设备号和副设备号，可以通过ls -l /dev/sda查看
    ls -l /dev/vdb1
    brw-rw---- 1 root disk 252, 16 Aug 26 16:09 /dev/vdb
    echo ${PID} > cgroup.procs

在对控制组进行写入之后。再使用iotop看到IO用量已经被限制

    13842 be/4 root        0.00 B/s  559.14 K/s  0.00 % 26.37 % dd if=/dev/ze~000 oflag=sync

内存限制
----

### 内存占用

为了精准的空占内存，这里找来一段C，使用malloc来进行固定内存空间的分配；

    #include<stdio.h>
    #include<sys/mman.h>
    #include<stdlib.h>
    #include<unistd.h>
    //要占用100M内存，以字节为单位
    const int alloc_size = 100*1024*1024;
    int main(){
            char *mem = malloc(alloc_size);
            size_t i;
            // 等待20S之后开始
            sleep(20);
            //获得每一个内存页的大小，一般为4K
            size_t page_size = getpagesize();
            for(i=0;i<alloc_size;i+=page_size){
                mem[i] = 0;
            }
            printf("i = %zd\n",i);
            while(1){
                sleep(5);
            }
            return 1;
    }

编译后运行，`free -m -s 1`持续输出内存占用情况，可见；

                        total        used        free      shared  buff/cache   available
    Mem:          15787         475       13040         281        2271       14931
    Swap:             0           0           0
    
                        total        used        free      shared  buff/cache   available
    Mem:          15787         575       12940         281        2271       14831
    Swap:             0           0           0

可见，内存已被分配占用 100M

### 资源限制

    mkdir -p /sys/fs/cgroup/memory/foo
    #分配50MB的内存给这个控制组 50000000
    echo $MEM_LIMIT >  memory.limit_in_bytes   
    echo ${PID} > tasks 

进程运行，等待 20秒之后；

    [root@VM_54_221_centos /data]# ./a.out
    Killed

可见进程因为使用内存超过限制，最终OOM被KILL。

后
-

Cgroup是LINUX内核中一个很好玩好用的属性，控制组的功能，实现了docker的对容器进行资源限制的功能。 其功能还有很多，后面在进行继续的探索。