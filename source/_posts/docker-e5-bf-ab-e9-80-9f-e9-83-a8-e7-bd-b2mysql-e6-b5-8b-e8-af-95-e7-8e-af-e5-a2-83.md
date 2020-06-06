---
title: Docker 快速部署mysql测试环境
tags:
  - Docker
  - mysql
url: 1350.html
id: 1350
categories:
  - OP之路
date: 2019-10-31 19:40:44
---

使用 Docker 快速的部署一个本地可用的测试环境，再使用Navicat之类的工具来进行表的创建和设计，极大的降低了使用和操作的成本。

* * *

How to
------

具体的操作很简单，使用 docker 对mysql进行部署即可。这里使用 docker-compose文件可以更好的管理。

*   拉取镜像
    
        docker pull mysql
    
*   写 yaml文件
    
        version: '3'
        services:
        db:
            image: mysql #构建mysql镜像
            container_name: mysql-db # 容器名
            command: mysqld --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci #设置utf8字符集
            restart: always
            environment:
              MYSQL_ROOT_PASSWORD: root #root管理员用户密码
              MYSQL_USER: test   #创建test用户
              MYSQL_PASSWORD: test  #设置test用户的密码
              MYSQL_ROOT_HOST: "%"
            ports:
              - '3306:3306'  #host物理直接映射端口为6606
        # 数据持久化
        #    volumes:
        #        #mysql数据库挂载到host物理机目录/e/docker/mysql/data/db
        #      - "/e/docker/mysql/data/db:/var/lib/mysql"
        #        #容器的配置目录挂载到host物理机目录/e/docker/mysql/data/conf
        #      - "/e/docker/mysql/data/conf:/etc/mysql/conf.d"
    
*   启动容器
    
        # docker-compose up
        ➜  mysql docker-compose up
        mysql-db is up-to-date
        Attaching to mysql-db
        mysql-db | Initializing database
        mysql-db | 2019-10-29T08:28:19.673116Z 0 [Warning] [MY-011070] [Server] 'Disabling symbolic links using --skip-symbolic-links (or equivalent) is the default. Consider not using this option as it' is deprecated and will be removed in a future release.
        mysql-db | 2019-10-29T08:28:19.673422Z 0 [System] [MY-013169] [Server] /usr/sbin/mysqld (mysqld 8.0.16) initializing of server in progress as process 30
        mysql-db | 2019-10-29T08:28:21.304245Z 5 [Warning] [MY-010453] [Server] root@localhost is created with an empty password ! Please consider switching off the --initialize-insecure option.
        mysql-db | 2019-10-29T08:28:22.582852Z 0 [System] [MY-013170] [Server] /usr/sbin/mysqld (mysqld 8.0.16) initializing of server has completed
        mysql-db | Database initialized
    

之后 3306 的端口的mysql服务就跑起来了！

EOF