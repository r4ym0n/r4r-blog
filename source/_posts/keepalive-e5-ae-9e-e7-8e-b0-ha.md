---
title: Keepalive 实现 HA
tags:
  - HA
  - Linux
  - Nginx
  - OPS
url: 462.html
id: 462
categories:
  - OP之路
date: 2019-03-01 00:00:00
---

前
-

`HA(High Availability)` 高可用，是衡量一个系统的重要指标。所以双机热备，成为一个好的和通用的选择。前面的的方案直接使用了nginx 的Upstream，来实现了 **双机** 的过程。 虽然是可以实现在单节点down掉之后系统依旧可用，不过显得不是那么专业。这里就使用 KeepAlive 来实现系统的 **双机过程**。

* * *

> 1.  nginx进程基于Master+Slave(worker)多进程模型，自身具有非常稳定的子进程管理功能。在Master进程分配模式下，Master进程永远不进行业务处理，只是进行任务分发，从而达到Master进程的存活高可靠性，Slave(worker)进程所有的业务信号都 由主进程发出，Slave(worker)进程所有的超时任务都会被Master中止，属于非阻塞式任务模型。
> 2.  Keepalived是Linux下面实现VRRP备份路由的高可靠性运行件。基于Keepalived设计的服务模式能够真正做到主服务器和备份服务器故障时IP瞬间无缝交接。二者结合，可以构架出比较稳定的软件LB方案。
> 
> 引用自 [Nginx+keepalived 双机热备（主从模式）](https://www.cnblogs.com/kevingrace/p/6138185.html)

* * *

Keepalive 基本概念
--------------

KeepAlived 使用了 `VRRP` 来避免IP的单点故障。VRRP全称 Virtual Router Redundancy Protocol，即 虚拟路由冗余协议。简单的说，在一个子网IP，后面其实对应了 N 台主机。这些主机再构成一个路由器组。这个单独的虚拟ip，复杂对数据包的转发。这里也就有了传说中的 **VIP （virtual IP）**。

简单的一句话来讲：多个IP被放在一个路由组，虚拟成一个 IP。

这里有主机和从机。当主机不可用时候，回自主更新路由对应的MAC（设备）。

Keepailve 简易配置
--------------

这里直接在两台主机上安装 `keepalive` ，在 `/etc/keepalive/keepalive.conf` 里面进行配置。 这里也直接贴配置了，因为是直接 Copy的。。。

> 示例配置在项目官网可见：[http://www.keepalived.org/manpage.html](http://www.keepalived.org/manpage.html)

主机配置如下，定义了节点名称，网络接口，认证方式以及 **VIP**：

    global_defs {  
        router_id NodeA # 不同节点的不同命名  
        router_id NodeB #################
    }  
    vrrp_instance VI_1 {  
        state MASTER        #设置为主服务器  
        state BACKUP        #设置为备服务器 ###############
    
        interface eth0      #监测网络接口  
        virtual_router_id 51    #主、备必须一样  
    
        priority 99         #(主、备机取不同的优先级，主机值较大，备份>机值较小,值越大优先级越高)
    
        advert_int 1        #VRRP Multicast广播周期秒数  
        authentication {  
            auth_type PASS  #VRRP认证方式，主备必须一致  
            auth_pass ***** #(密码)  
        }
        virtual_ipaddress {  
            192.168.1.200/24  #VRRP HA虚拟地址  
        }
    }

* * *

上面直接给出了器可用的配置文件，主从之间的差距就是那么几行。之后使用 `service` 重启服务即可。

### 效果验证

在主机或者路由里可以看到 `arp -a` 里面，192.168.1.200 的这个IP的路由已经被注册了，而且后面的MAC正是我们的 **Master** 的MAC地址。

    192.168.1.1           20-76-93-46-68-e3     动态
    ...
    192.168.1.200         2c-4d-54-42-9b-0a     动态
    ######################################################################
    ➜  keepalived ifconfig 
    eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
            inet 192.168.1.129  netmask 255.255.255.0  broadcast 192.168.1.255
            inet6 fe80::cffe:4eee:f129:de22  prefixlen 64  scopeid 0x20<link>
            ether 2c:4d:54:42:9b:0a  txqueuelen 1000  (Ethernet)
            RX packets 464842  bytes 188082136 (179.3 MiB)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 506574  bytes 162269596 (154.7 MiB)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
            device interrupt 43 

当**拔掉 MASTER 主机的网线**之后：可以看到地址绑定的MAC地址，飞速的就切换了。

    192.168.1.1           20-76-93-46-68-e3     动态
    ...
    192.168.1.200         b8-27-eb-eb-17-72     动态
    #######################################################################

至此可见，Keepalive 的热备效果实现了在一个 VIP 下的双机热备。在单主机下线情况下快速切换。

* * *

### 问题

当前的配置实际上并不合理，只有在主机掉线后才会进行主从切换。很多实话是某个服务（比如 Nginx）挂掉了导致的服务不可用。所以后面回就需要更高阶的使用，加上对服务进行检测的功能。

Keepavlived 监控配置
----------------

入上面的提到的问题，在这种情况下，只适用于 **MASTER** 节点down掉的情况下，才会重新选取 Backup节点。然很多时候实际上是服务出了问题而不是主机（比如 Nginx 服务挂掉），那么在这种情况下就不能及时的切换到备机。

所以这里的配置就需要更进一步，来实现监控的过程。对于 Nignx 的web服务来讲，可以有以下的方式：

*   监控 Nginx 进程
*   监控 Nginx 端口
*   监控 Nginx 的请求返回

这三种方法，的可靠性也是依次提升的。

> 这里参考文章：[Nginx+keepalived 双机热备（主从模式）](https://www.cnblogs.com/kevingrace/p/6138185.html)
> 
> 里面对细节讲的相当的详细

* * *

在配置文件里，可以使用 `vrrp_script` 域来自定义检测脚本。示例如下：

    vrrp_script chk_http_port {         
        script "/opt/chk_nginx.sh"      # 指定检测脚本
        interval 2                      # 检测间隔
        weight -5                       # 故障时权重值 -5
        fall 2                          # 故障判断次数，连续两次算故障
        rise 1                          # 检测成功，一次算成功
    }

> 注意：这里要提示一下keepalived.conf中vrrp_script配置区的script一般有2种写法： 1）通过脚本执行的返回结果，**改变优先级**，keepalived继续发送通告消息，backup比较优先级再决定。这是直接监控Nginx进程的方式。 2）脚本里面检测到异常，**直接关闭keepalived进程**，backup机器接收不到advertisement会抢占IP。这是检查NginX端口的方式。

* * *

用于检测的 **Nginx** 进程的脚本如下，用Shell脚本对进程进行检测，统计进程数量，如果进程挂掉，就尝试重启进程，如果进程无法拉起，那么就直接停掉本机的 `keepalive` ，让 VIP 漂移到从机上。这里使用的就是第二种方法。

    counter=$(ps -C nginx --no-heading|wc -l)
    if [ "${counter}" = "0" ]; then
        /usr/local/nginx/sbin/nginx
        sleep 2
        counter=$(ps -C nginx --no-heading|wc -l)
        if [ "${counter}" = "0" ]; then
            /etc/init.d/keepalived stop
        fi
    fi

* * *

至此，基于进程监控的 Keepalive 已经实现。

后面的话
----

这里用 Keepalive 实现了一个 VIP 下的双机热备，和实际业务需求更加接近了一点。当然真实架构不是这样的，这样导致了，从机的完全的空闲，其压测效果还不如两个机器分别在 upstream里面。后面将会使用 Docker集群进行统一管理。