---
title: traefik----更优雅的Docker反代方式
tags:
  - Docker
  - OPS
  - proxy
  - traefik
url: 696.html
id: 696
categories:
  - OP之路
date: 2019-04-05 15:40:58
---

前面的话
----

在这篇POST里面，主要记录，如何把后台的反代工具从 Nginx 切换到 traefik 的过程。实现Docker 的网络中更加优雅反代。

较之Nginx 的最大的特点，容器（服务）的网络，不必再通过ingress（虚拟入口），来把端口映射到host的网络上，而导致，占有了一大堆一大堆的 奇怪的端口 ：`30000，32767...` 所以下面就简单的介绍一下Traefik：

Why Traefik？
------------

traefik 的特点在官网上有很多的介绍了，这里找几点重要的：

*   Continuously updates its configuration (No restarts!)
*   High Availability with cluster mode (beta)
*   Fast

* * *

这里列出了三点点特性，**其中第一点也就是最重要的一点**，这也是之所以选择 Traefik 的原因。

之于Nginx 不同，在每当服务进行调整的时候，Nginx 需要去修改 配置文件，之后去手动的reload服务。而且如上面所说到，这样的直接把容器的端口，通过 Ingress 给映射到host的网络上面，占用一批批的端口，**而且每多一个服务就得自己加一个配置**，显得实在是不优雅。所以，Traefik 很完美的解决了这个问题，**只需要在服务配置里面定义标签**，Traefik 会自动的，去检测配置并且显示自动更新。显得就是十分方便了

* * *

其二，在集群模式可以很方便的实现高可用，像是前面的KeepAlive，在集群模式统统省掉了。直接 `docker servicxe scale mytraefik=2` 就很容易的实现一个高可用的集群来。

* * *

其三，快。当然对于目前的小业务量，没什么影响了。列出这点因为官方给出了数据：

> Traefik is **obviously slower than Nginx**, but **not so much**: Traefik can serve 28392 requests/sec and Nginx 33591 requests/sec which gives a ratio of 85%. Not bad for young project :) !

较之Nginx更加方便了，所以我选择 Traefik。

* * *

下面上图来解释这两个架构：

这里是之前传统的使用 Nginx 进行反代的配置，这里面可见，每个服务都需要在host 的主机网络中映射出一个端口，配置显得十分的冗余 ![file](/wp-content/uploads/2019/04/image-1554438668384.png)

在使用了 Traefik 之后的结构如下，所有的转发配置和规则在 Traefik 进行统一的管理和自动配置。 ![file](/wp-content/uploads/2019/04/image-1554444686951.png) 这一有一点的问题是，可能也被观察出来了，这里依然使用了 Nginx 作为前级（因为目前没有使用 Https 的配置，和缓存，所以Nginx提供 缓存和Https）。

* * *

至此为什么使用 traefik 也就解释完了，下面就开始部署和配置阶段了。

快速部署
----

这里给出的是一键部署的命令啦，这里的环境是在 Swarm 集群下的，所以这里后面指定了一下参数，用于对其进行配置，需要注意的是，这里需要 配置在 manager 的节点上，因为需要和 Docker 的socket 进行通信。映射出了两个端口，其中一个就是反代的统一入口，另一个是提供了一个健康监控的 UI界面。

    docker service create \
         --name traefik \
         --constraint=node.role==manager \
         --publish 8090:80 --publish 8099:8080 \
         --mount type=bind,source=/var/run/docker.sock,target=/var/run/docker.sock\
         --network traefik_default \
         traefik \
         --docker \
         --docker.swarmMode \
         --docker.domain=i124u.cf  \
         --docker.watch \
         --logLevel=DEBUG \
         --web

上面的命令可以很快速的部署一个 Traefik 的服务。

* * *

这里给出一个 Nginx 的服务配置，值得人注意的是，这里的网络需要接入traefik的子网中午，否则是无法连通的。 之后重点来了，也就是 traefik 的亮点，在于，这里的介入，只需要一个简简单单的label来说明转发规则，**就可以被自动的配置为转发对象了。**

    docker service  create \
        --replicas 2 \
        --network traefik_default \     # 这里
        --label traefik.port=80 \
        --label traefik.frontend.rule=Host:test.example.org \
        --name hello nginx

* * *

在配置完成之后，我们直接访问 Traefik 的dashboard可以直接看到我这里的配置的转发规则已经生效了。完全不需要进行手动的修改配置以及reload。

![file](/wp-content/uploads/2019/04/image-1554446423351.png)

进行访问测试，分别测试 123，和example的host。

    ➜  ~ curl -H Host:123.org http://127.0.0.1:8080
    404 page not found
    
    ➜  ~ curl -H Host:test.example.org http://127.0.0.1:8080
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    <style>
        body {
            width: 35em;
            margin: 0 auto;
            font-family: Tahoma, Verdana, Arial, sans-serif;
        }
    </style>
    </head>
    <body>
    <h1>Welcome to nginx!</h1>
    <p>If you see this page, the nginx web server is successfully installed and
    working. Further configuration is required.</p>
    
    <p>For online documentation and support please refer to
    <a href="http://nginx.org/">nginx.org</a>.<br/>
    Commercial support is available at
    <a href="http://nginx.com/">nginx.com</a>.</p>
    
    <p><em>Thank you for using nginx.</em></p>
    </body>
    </html>
    

由此可见，这时的traefik 已经可以进行正常的转发。一个简单的配置到这里就 OK了

项目使用
----

为了便于更新和管理，工程还是需要使用 compose 进行配置的。对于上面的命令这里改成 compose 配置，配置内容如下：

    version: '3'
    
    services:
      reverse-proxy:
        image: traefik
        command: --api --docker --docker.swarmMode --docker.watch --web
        ports:
          - "8080:80"   # 代理的总端口
          - "8081:8080" # 可视化ui的端口映射
        volumes:
          - /var/run/docker.sock:/var/run/docker.sock
        networks:
          - traefik_default
        deploy:
          placement:
            constraints:
              - node.role == manager
    networks:
        # 建立一个专用的网络
      traefik_default:

建立一个 overlay 的网络是十分有必要的。 参考链接：

> <[https://my.oschina.net/guol/blog/1942373>](https://my.oschina.net/guol/blog/1942373>);

* * *

关于在在stack内使用的配置，这里笔者可是折腾了一番。（因为中午的博客环境也知道的，爬虫爬来爬去，搜的的也就是那么几篇）。因为给出的例子都是单个容器或者是服务的使用方法。然而这里遇到了需要使用 Stack 进行部署的方法，网络在这里就有了些问题。

遇到问题，最好的方法是去看官方的手册

> <[https://docs.traefik.io/>](https://docs.traefik.io/>); [https://docs.traefik.io/configuration/backends/docker/#labels-overriding-default-behavior](https://docs.traefik.io/configuration/backends/docker/#labels-overriding-default-behavior) # 去 override 一个默认的网络

     version: "3"
        services:
          whoami:
            deploy:
              labels:
                traefik.docker.network: traefik

> [https://docs.docker-cn.com/compose/compose-file/#external-1](https://docs.docker-cn.com/compose/compose-file/#external-1) If set to true, specifies that this network has been created outside of Compose. docker-compose up will not attempt to create it, and will raise an error if it doesn’t exist. external cannot be used in conjunction with other network configuration keys (driver, driver_opts, ipam, internal). In the example below, **proxy is the gateway to the outside world**. Instead of attempting to create a network called \[projectname\]_outside, Compose will **look for an existing network simply called outside and connect the proxy service’s containers to it.**

所以在这里的网络配置里面需要声明外部网络。并且在标签里面指定 traefik 的默认网络

这里直接贴出`docker-compose.yml` 了，当个保存，大家也可以直接拿去用，重点的配置部分也已经标出

    version: &#039;3&#039;
    
    services:
       db:
         image: mysql:5.7
         volumes:
           - db_data:/var/lib/mysql
         restart: always
         environment:
           MYSQL_ROOT_PASSWORD: ${MYSQL_DATABASE_PASSWORD}
           MYSQL_DATABASE: wordpress
           MYSQL_USER: wordpress
           MYSQL_PASSWORD: wordpress
         networks:
            - wp_network
    
       wordpress:
         image: wordpress:latest
         volumes:
          - wp_data:/var/www/html
         restart: always
         ports:
           - 80
         environment:
           WORDPRESS_DB_HOST: db:3306
           WORDPRESS_DB_USER: wordpress
           WORDPRESS_DB_PASSWORD: wordpress
           WORDPRESS_CONFIG_EXTRA: |
              /* Multisite */
              define(&#039;WP_SITEURL&#039;, &#039;http://&#039; . $$_SERVER[&#039;HTTP_HOST&#039;]);
              define(&#039;WP_HOME&#039;, &#039;http://&#039; . $$_SERVER[&#039;HTTP_HOST&#039;]);
              define(&#039;WP_REDIS_HOST&#039;, &#039;redis&#039;);
              define(&#039;WP_REDIS_PORT&#039;, &#039;6379&#039;);
              define(&#039;WP_REDIS_DATABASE&#039;, &#039;0&#039;);
         networks:
            - tfk_traefik_default
            - wp_network
        ##########################重点#############################
         deploy:
           labels:
             traefik.frontend.rule: &quot;Host:ray.i124u.cf&quot;
             traefik.port: &quot;80&quot;
             traefik.docker.network: tfk_traefik_default
       redis:
         image: redis
         restart: always
         networks:
          - wp_network
        ##########################重点#############################
    networks:
      wp_network:
      tfk_traefik_default:
        external: true
    volumes:
        db_data:
          driver_opts:
            type: &quot;nfs4&quot;
            o: &quot;addr=192.168.1.230,nolock,soft,rw&quot;
            device: &quot;:/srv/nfs/blog_${BLOG_NAME}/wp_db&quot;
        wp_data:
          driver_opts:
            type: &quot;nfs4&quot;
            o: &quot;addr=192.168.1.230,nolock,soft,rw&quot;
            device: &quot;:/srv/nfs/blog_${BLOG_NAME}/wp_wp&quot;
    

配置完成之后，直接更新服务，去dashboard里面可以直接看书工作状态 ![file](/wp-content/uploads/2019/04/image-1554449695399.png)

至此Traefik 的切换基本完成。

后
-

此站点现在就是使用的基于 Traefik 支持的反代服务，整体还是很稳定的，重点是很方便，不需要，一次又一次的修改 Conf 和 reload nginx的服务了。 所以基于 Docker 的微服务，Traefik 有了更好的兼兼容性。值得选择 ![file](/wp-content/uploads/2019/04/image-1554450038231.png)