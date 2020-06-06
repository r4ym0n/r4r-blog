---
title: 读本好书 《Docker 进阶与实践》
url: 484.html
id: 484
categories:
  - 未分类
date: 2018-12-31 00:00:00
tags:
---

前
-

读书

简介
--

*   书名：Docker 进阶与实践
*   作者：华为Docker实践小组
*   ISBN：9787111523390

* * *

这本书作为一个 对 Docker 的系统的学习和了解，由理论到实践，讲解了Docker 技术，以及其应用。

粗浅阅读，作为对Docker 了解，准备一个简单的project。（在线编译器？）

Docker 简介
---------

其核心的思想是 ： **Build Ship and Run**。

在磁盘占用，性能以及效率都在传统的虚拟化技术上有了明显的提高。

*   Docker 之于传统的虚拟化技术是没有 Hypervisor 层。
*   Docker 使用了层级镜像的应用，可以实现存储空间的复用。

* * *

### 基础概念

*   Docker 客户端
    
    使用 command 发起请求，或者使用 RESTful API 进行请求
    
*   Docker deamon
    
    整个Docker 的核芯引擎，可以理解为 Docker Server。
    
*   Docker 容器
    
    容器是一个核心内容，很好的诠释了集装箱的概念，可以实现一个标准隔离执行环境。
    
    迁移和部署的时候，不用关心容器里面是装了什么，也不需要了解是怎么配置。整个就是一个一切完整的集装箱。
    
*   Docker 镜像
    
    可以理解为 容器是镜像的实例， 镜像是容器的模板。
    
*   Registry
    
    可以理解为镜像站点，可以直接使用 pull 从其拉去镜像。
    

### 安装

在内核编译的阶段，需要开启 `Cgroup` 以及 `Namespace`的编译选项。

### Docker 的基本使用

直接键入docker 会得到其提示页面， 使用 `docker COMMAND --help`可以直接显示相关的帮助页面

容器技术
----

在社区的合作下形成了Cgroup 以及 namespace 。

*   Mount Namespace
*   UTS Namespace
*   IPC Namespace
*   PID Namespace
*   Net Namespace
*   User Namespace

* * *

Cgroup :

*   cpuset
*   CPU
*   memory
*   device
*   freezer
*   blkio

* * *

### 容器组成

容器技术通过 Cgroup 和 Namespace 为核心技术。在其基础上由了根文件系统，以及容器引擎。书中有以下公式：

> `容器 = cgroup + namespace + rootfs + 容器引擎(用户态工具)`

*   Cgroup: 资源控制
*   Namespace: 访问隔离
*   rootfs： 文件系统隔离
*   容器引擎: 生命周期控制

### 容器的创建

书中的这里使用了三段伪代码，来说明一个容器的构成，这里说下自己的理解。

1.  通过系统的clone调用，来传入各个的Namespace的Clone flag，对其命名空间进行拷贝 ，之后得到一个新的进程，这个进程就会拥有一个属于自己的 clone 的Namespace。
2.  `echo $pid > /sys/fs/cgroup/cpu/tesks` 建立的进程的PID 写入了各个 Cgroup的子系统之中，受到相应的Cgroup的子系统控制。
3.  通过 系统调用，使得进程可以进入一个 新的 rootfs ，之后使用 `exec("/bin/bash")` 来启动一个 shell。

这样就完成了一个容器的创建

* * *

### Cgroup

**Cgroup 的目的就是实现了对与系统资源的`QoS`**

在CG 之前，通过 sched_setaffinity 设定一个进程的CPU 亲和性（nginx中的配置）。

> `$$` 表示当前进程的PID

用户可以甚至可以新建自己的 Cgroup的规则，系统默认的规则在 `/sys/fs/cgroup` 下面进行配置

对进程配置生效，只需要将pid 写入以下文件 ： `/sys/fs/cgroup/pids` 这样是整个的 Cgroup 规则对当前进程生效。

或者，可以直接配置独立项目里面的 xxx.procs 。

blkio子系统 可以对块设备的I/O带宽进行限制。

devices子系统 可以对设备的权限进行控制 `a *.* rmw` 代表所有设备可被访问 `c 1:3 r` 控制 主设备号:子设备号设备 只读

### Namespace

将内核的全局资源做封装，每一个NS 都有其独立的资源拷贝。

    ll /proc/$$/ns 

查看当前的进程的namespace。

可以很容易的使用 系统调用建立一个独立的namespace的进程：

    static char stack[STACK_SIZE];
    static char* const child_args[] = {"/bin/bash", NULL};
    
    static int child(void* arg)
    {
      execv("/bin/bash", child_args);
      return 0;
    }
    
    int main(int argc, char **argv)
    {
      pid_t pid;
      pid = clone(child, stack+STACK_SIZE, SIGCHLD|CLONE_NEWUTS, NULL); // 这里配置克隆命名空间的操作
      waitpid(pid, NULL, 0);
    }

直接 GCC 编译， 在返回的SHELL 里，对hostname 进行修改 ，logout 之后，原shell中的hostname并没有被改变。

如前面提到的，namespace 有几个种类，

type

desc

UTS

struct utsname ，uname 系统调用里面的结构体

IPC

进程间通信的隔离，如消息队列？ ipcmk

PID

PID 之间的隔离，（ps从procfs里读取），实际上的操作是隔离的，如kill

Mount

挂载点隔离

Network

网络接口隔离

User

用户以及用户权限隔离

Docker 的镜像
----------

image是启动容器的只读模板，是容器启动需要的rootfs。下面是一些 词汇：

word

meaning

doker hub

可以理解为一个镜像站

Namespace

类似 Github 中的命名空间，代表一个种类，用户或组织

Repository

一个 Git 仓库，可以有多个镜像

Tag

类似Git 的tag，区别于不同的版本

Layer

类似于Git的 **commit** ，一个长的 Hash 串

Image ID

镜像的唯一标识，可以等同 repo:tag

后面的部分简述了，**build，ship and run** 的操作。

* * *

Docker 的image的组织结构，颗粒理解为积木堆叠的形式。书中对其源文件做了探索：

    docker pull busybox # 拉取镜像
    docker history busybox  # 查看镜像的历史版本
    docker inspect busybox:latest

* * *

### Docker image 的技术亮点

*   联合挂载
    
    可以把多个目录同时挂载到同一个目录。
    
*   写时复制
    
    在和系统调用的 `fork`类似，在创建子进程的时候，并不进行新的一个内存区的复制，而是在修改了共享的内的时候触发了一次缺页的中断，这时候进行一次页分配。这时候才会有真正的写。
    

仓库进阶
----

git 的思想， 可以很容易的进行 `pull/push`

容器间的网络
------

这里是在实践过程中比较重要的一部分。过程较为复杂，这里记录一些 kword `Weave`，`Flannel`，这些是现有的Docker 的网络解决方案

*   Weave
*   Flannel
*   SocketPlane

容器卷管理
-----

在执行 `run/create` 使用 `-v` 来添加数据卷 volume，当然也可以将主机上的卷挂载到 容器中来：

    docker run -d -v /var/vol --name mytest busybox     # 添加新的卷
    docker run -d -v /var/vol:/home/test --name mytest busybox  # 挂载主机卷

可以创建一个共有的存储容器，在其他的实例上使用 `--vloume-from` 来共享卷

* * *

问题：数据卷的悬挂问题

在删除容器的时候，需要显式的指明所挂载的卷，分配卷的空间才会被删除，如果不指定，就会出现数据卷的悬挂（dangling），浪费大量空间

    docker rm mytest
    docker rm mytest -v /var/vol

Docker API
----------

Docker 的API 设计 满足 **RESTful(Representational State Transfer)** 表达性状态转移。其API 设计：清晰，简单，低耦合，无状态，面向资源。

Docker 提供了API接口，使得其拓展性得到极大的提高。有提供三类API：

*   Docker Remote API 比如 docker run 这种服务控制的命令
*   Docker Registry API 可以用来控制镜像存储
*   Docker Hub API 与Docker 的关系不大

* * *

示例请求，Docker 在启动的时候，会默认开启API，并且监听本地的unix 套接字，默认值为 `unix:///var/run/docker.sock`，可以直接向套接字写内容从而实现了API的调用

这里直接向套接字写了HTTP的请求，server 会直接返回了当前的所有的镜像的状态。

* * *

当然，Docker 也会监听 `localhost:5678` 作为API 的请求端口，（之前有用过一个 GUI 的Docker的管理工具，其核心就是通过API与服务进行交互）

    curl -XGET localhost:5678/image/debian/history  | python -m json.tool   # 等同：
    docker history debian

* * *

Docker API 的监听在其启动的时候进行配置：

    docker -d -H unix:///var/run/docker.sock -H tcp://localhost:32768

* * *

### API的应用场景

书中在这里以神奇的背景展开了一个使用案例：

*   一个python 写的httpserver
*   打包成一个 Docker 镜像
    *   Dockerfile 和其依赖的文件打压缩包
    *   使用远程的Docker 主机对镜像进行构建】
*   发布镜像到Registry pash
*   其他主机 Pull 到本地
*   本地运行容器实例并验证
*   至此完成一个docker 的helloworld。

这里对这个场景进行简单的描述，后面将会有操作的实例。

Docker 的安全
----------

由于 Docker 和物理主机是公用同一个内核，因此受攻击的面将会特别的广，**而且一旦容器内的程序导致了内核的panic，物理机的内核也将 Painc**

共用内核使用Cgroup和Namespace 实现了容器隔离和资源限制的目的。Namespace 目前不是十分完善。**所以导致了虚拟容器中的逃逸问题**，（比如 PS 使用的 procfs ，就可以看到物理机的所有的 ps）这个是特性导致，所以很难完美的处理。现在应用的安全策略：

1.  Cgroup 进行资源限制
2.  ulimit 资源限制
3.  容器组网 安全
4.  容器+全虚拟化 公有云，安全需求很高的解决方案
5.  监控 `docker ps -a` 监控容器状态
6.  文件级防护 对文件的权限进行严格的控制
7.  capability
8.  SELinux 严格的DAC ，资助访问控制
9.  AppArmor
10.  Seccomp
11.  grsecurity 内核的 patch 大大提高安全性

### 安全示例

#### 主机逃逸

**shocker攻击** 通过 `open_by_handle_at` 和 `name_to_handle_at` 这两个系统调用实现。先在容器内打开一个文件描述符，

    fd = open("/.dockerinit", O_RDONLY);

之后使用 `open_by_handle_at` 获取文件的句柄。

    fd = open_by_handle_at(bfd, (struct file_handle *)ih, O_RDONLY);
    dir = fdopendir(fd)

并且在这里得到其打开的所有目录？

    de = readdir(dir);

得到其目录的结构体，之后进行对比与 `/etc/shadow`

    strncmp(de->d_name, path, strlen(de->d_name));

找到了包含 shadow 的de目录之后，开始穷举句柄：

* * *

后面略过部分的内容。讲解了Libcontainer 的一些技术原理。

Docker 实践篇
----------

### Dockerfile 的hello world

Dockerfile 用来制作镜像，像是一个蓝图一样，

*   以 # 为注释
*   每一行一个命令
*   四部分组成
    *   基础镜像信息
    *   维护者信息
    *   镜像操作指令
    *   容器启动指令

书中给了一个 Nginx 的基础镜像的配置，这里进行cpoy：

    FROM ubuntu
    MAINTAINER admin xxx
    RUN echo "[软件源]" >> /etc/apt/source.list        # 添加软件源
    RUN apt-get update && apt-get install -y nginx 
    RUN echo "\ndeamon off" >> /etc/nginx/nginx.conf    # 写配置前台运行
    CMD /usr/sbin/nginx     # 启动服务

* * *

KeyWord

DESC

FROM 

继承自哪个镜像

MAINTAINER

指定维护者信息

RUN 

执行SHELL指令，类似 `sh -c`

EXPOSE

暴露容器端口

CMD

启动容器时执行的命令，**只能有一条**

VOLUME

创建挂载点

ENV

定义环境变量

ADD

复制指定src(URL，tar，相对路径)的指定文件到目的卷

COPY \ \

复制本地主机的文件到容器的目

* * *

### Docker 镜像 的制作与启动

书中给出了一个DockerFile 的示例：

    FROM busybox
    RUN date;sleep 100;date
    RUN echo "abc" > /mytest
    RUN date;sleep 100; date
    CMD /bin/bash

RUN 命令都是在容器内部执行的，最后的CMD为最后返回的内容。

在书中是以一个 Tomcat 的例子来演示的。这里主要设计到了镜像的拉去，以及 卷的 静态挂载，SSL 证书的生成。之后贼了容器内部，把 证书保存在 指定的目录，并且修改一下 server 的配置。

完成所有的配置之后，直接进行一次 commit。之后 跑容器实例即可。

**源码导入** 使用静态导入以及动态导入两种方式。

    FROM tomecat:https
    MAINTAINER xxx
    COPY ./websrc /usr/local/tomecat/webapps/myproj

**动态挂载** 把本地的数据卷挂载在容器中，现在船舰一个挂载点，把文件动态挂载到容器中。

    FROM tomcat:http
    MAINTAINER xxx 
    RUN mkdir -p /usr/local/tomcat/webapps/myproj
    VOLUME /usr/local/tomcat/webapps/myproj

在启动容器的时候指定参数：

    docker run -ti -v $(pwd)/websrc:/usr/local/tomcat/webapps/myproj

* * *

### Docker 的架构

微服务之核心就在于此,把服务的多个部分以容器的形式，分散在各处。比如实现一个 web服务架构，除了上面的web服务器之外，还有后端的数据库，或者是 一些其他的模块。

**Docker-compose** 提供了一个Docker 的工程的管理，可以实现对多个容器的构成一个项目的整合。

Docker 的集群管理
------------

在生产环境的使用的时候，当然是需要进行统一管理的。这里在书中就引入了几个工具：

### Compose

> 一个应用往往有多个组件构成，Docker 的最佳实践是一个容器运行一个进程

所以为了实现容器之间的同意管理，以及协作工作，这里就有了 Compose 。

### Machine

简化Docker 安装的工具

### Swarm

集群管理工具，实现把多个Docker主机组成的系统，整合为统一的虚拟Docker 主机。

> swap,plug and play

* * *

**K8S**

FAQ（_Frequently asked questions_ ）
----------------------------------

书后面的 FAQ，这里挑选部分摘录：

*   Docker 容器中管理多个进程
    *   使用 `supervisord,runit`等进行进程的统一管理，保持管理进程工作即可
*   ATTACH 到容器之后怎么退出
    *   操作和 `screen` 有些类似使用 `ctrl+P，ctrl+Q` 进行退出，容器继续进行，如果使用 `ctrl+c` 可能导致进程结束从而容器退出

后面的话
----

通过这本书，对Docker 的认识，和使用更上了一个台阶，后面准备实现一个 基于Docker 的在线编译执行的东西。

基础构想是 使用 Nginx 跑 Python的CGI，post 源码，之后编译执行，再返回前端，把这个打包成镜像

下一步就是把服务拆开，单容器单进程，把 nginx 和 uwsgi 拆分开来，使用多容器合作。

后面看看 Docker 的源码吧。

书后推荐书籍：

*   Docker 技术入门与实践 9787111488521
*   Docker 源码分析 9787111510727
*   Linux 内核设计与实现
*   Linux 内核精髓