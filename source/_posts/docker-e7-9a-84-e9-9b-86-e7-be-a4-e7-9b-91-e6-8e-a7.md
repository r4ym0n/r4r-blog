---
title: Docker的集群监控
url: 448.html
id: 448
categories:
  - 未分类
date: 2019-03-17 00:00:00
tags:
---

前面的话
----

再一次被 Docker 所带来的革命性的变量所震撼。**Build，ship and run**。

想着给之前的搭建的集群加一个监控系统，主机性能监控，和服务日志监控(ES + Kibana + filebeat)。当然很有可能只是幻想了。因为单机性能实在是太过羸弱。

正巧看到了网上的方案使用的是 ：

> cAdvisor(数据收集)+InfluxDB(数据存储)+Grafana(数据可视化)

这样的方案。这里的主要参考文章链接：

> *   [Docker Swarm Monitoring](https://github.com/botleg/swarm-monitoring)
> *   [docker swarm集群监控方案cAdvisor+InfluxDB+Grafana实战](https://blog.csdn.net/dkfajsldfsdfsd/article/details/79977693)

服务部署
----

有了前面的折腾的经验，这里就到了一路绿灯的状态了。毕竟较之前有了更多的理解。这里直接使用 compose 进行结构化的部署：

    version: '3.2'
    
    services:
      influx:
        image: influxdb
        volumes:
          - influx:/var/lib/influxdb
        deploy:
          replicas: 1
          placement:
            constraints:
            # 这里指定部署的角色，数据库放在manager上面
              - node.role == manager
    
      grafana:
        image: grafana/grafana
        ports:
        # 端口映射
          - 0.0.0.0:3000:3000
        volumes:
          - grafana:/var/lib/grafana
        depends_on:
          - influx
        deploy:
          replicas: 1
          placement:
            constraints:
            # 这里也是指定部署角色
              - node.role == manager
    
      cadvisor:
      # 这里需要使用 cadvisor 的ARM 的版本
        image: budry/cadvisor-arm
        # 这里指定hostname 为节点名
        hostname: '{ {.Node.Hostname}}'
        # 配置参数（待会前面的frp可以学习一下）
        command: -logtostderr -docker_only -storage_driver=influxdb -storage_driver_db=cadvisor -storage_driver_host=influx:8086
        volumes:
        # 这里需要对一些目录进行挂载，特别是 sys 以便获得系统信息
          - /:/rootfs:ro
          - /var/run:/var/run:rw
          - /sys:/sys:ro
          - /var/lib/docker/:/var/lib/docker:ro
        depends_on:
          - influx
        deploy:
          mode: global
    
    volumes:
      influx:
        driver: local
      grafana:
        driver: local

上传 `yml` 文件，直接在 manager 节点进行部署：

    docker pull braingamer/cadvisor-arm
    docker pull grafana/grafana
    docker stack deploy -c monitor.yml monitor

其实 这里会自动的拉镜像。(Docker 在大陆连通性不好)所以这里可能对部分镜像需要梯子。如果不出意外，理应是一路绿灯。

* * *

上面的部署过程完成之后，后面需要在 InfluxDB 里面建立一个 cAdvisor 的库

    docker exec `docker ps | grep -i influx | awk '{print $1}'` influx -execute 'CREATE DATABASE cadvisor'

容器内执行的命令。cAdviosr 的配置已经在部署的yml指定：

    command: -logtostderr -docker_only -storage_driver=influxdb -storage_driver_db=cadvisor -storage_driver_host=influx:8086

后面就是在 Grafana 里面对视图进行配置了，（默认账号密码：admin），在项目里面有示例的 dashboard的配置，这里直接 Copy-paste 就好

> *   [swarm-monitoring](https://github.com/botleg/swarm-monitoring)/**[dashboard.json](https://github.com/botleg/swarm-monitoring/blob/master/dashboard.json)**

* * *

这里有个坑，需要修改这个配置文件里面的数据源。由 `influx` 改为 `InfluxDB`。

这里的数据源直接可以修改为 `http://influx:8086` ，数据库填写 `cadvisor` 便可以直接进行连接。

至此配置完成，直接访问 主机的3000 端口即可，看见高大上的 Grafana的界面了。

    root@vm0:~# docker service ls
    ID                  NAME                MODE                REPLICAS            IMAGE                        PORTS
    0m4tl8b6huw9        monitor_cadvisor    global              5/5                 budry/cadvisor-arm:latest    
    ixftiydc1v8k        monitor_grafana     replicated          1/1                 grafana/grafana:latest       *:3000->3000/tcp
    78f3ryphvjc6        monitor_influx      replicated          1/1                 influxdb:latest 

补充
--

由于监控数据的时效性，所以这里为了空间的节省，配置自动删除三天以前的数据：

    docker exec `docker ps | grep -i influx | awk '{print $1}'` influx -execute 'SHOW RETENTION POLICIES ON "cadvisor" '
    
    docker exec `docker ps | grep -i influx | awk '{print $1}'` influx -execute 'CREATE RETENTION POLICY "rp_0" ON "cadvisor" DURATION 3d REPLICATION 1 DEFAULT'

后面的话
----

Docker 的项目和工程至此就结束了，后面开始 Golang 的学习了，加油呢。