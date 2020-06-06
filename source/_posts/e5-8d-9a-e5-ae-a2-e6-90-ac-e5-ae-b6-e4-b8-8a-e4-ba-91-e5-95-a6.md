---
title: 博客搬家上云啦
tags:
  - Docker
  - Nginx
  - OPS
  - wordpress
url: 903.html
id: 903
categories:
  - OP之路
  - 日记？
date: 2019-05-03 23:38:45
---

前情提要
----

由于自己的本地主机的情况发生了重大变化，本地的两主机小机器的使命算是完成了。所以，最后没办法不得不上云了。网站整体的内容和域名不作改变。 还好还好，基于 Docker 的部署，所以整体的迁移是相当相当的方便的。本来想着折腾好久要，最后发现，**前前后后没到20分钟**，就差不多搞定了，也顺便解决了之前部署中的一些BUG。这一篇，主要就是记录迁移过程中的各种操作，后面可以很方便的进行复用。

过程记录
----

这里之间参考前面的文章，[Docker集群搭建](https://ray.i124u.cf/2019/03/05/docker%E9%9B%86%E7%BE%A4%E6%90%AD%E5%BB%BA/)，直接部署portainer，有了可视化的管理前端。

    mkdir -p /opt/portainer
    
    docker service create \
    --name portainer \
    --publish 9000:9000 \
    --replicas=1 \
    --constraint 'node.role == manager' \
    --mount type=bind,src=//var/run/docker.sock,dst=/var/run/docker.sock \
    --mount type=bind,src=//opt/portainer,dst=/data \
    portainer/portainer \
    -H unix:///var/run/docker.sock

* * *

服务器上nginx配置好反向代理。

    location ^~ /portainer/ {
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        access_log off;
        proxy_pass http://portainer/;
        rewrite ^/portainer/(.*)$ /$1 break;
    }
    

这里使用 traefik 实现内部的反向代理的功能，理由和之前一样，为了整体更加的和谐，和和系统的更加的兼容。这里就直接使用了。下面直接给出了compose：

    version: '3'
    
    services:
      reverse-proxy:
        image: traefik
        command: --api --docker --docker.swarmMode --docker.watch --web
        ports:
          - "8080:80"
          - "8081:8080"
        volumes:
          - /var/run/docker.sock:/var/run/docker.sock
        networks:
          - traefik_default
        deploy:
          placement:
            constraints:
              - node.role == manager
    networks:
      traefik_default:

使用上面的配置，直接进行部署即可，一般没有什么大的问题。关键期方便的地方，可以进行热配置，直接通过对于服务的label的配置指定转发的规则。

* * *

后面直接，再上我们的 docker-compose，进行wp-blog 的部署：

    version: '3'
    
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
              define('WP_REDIS_HOST', 'redis');
              define('WP_REDIS_PORT', '6379');
              define('WP_REDIS_DATABASE', '0');
    
         networks:
            - tfk_traefik_default
            - wp_network
         deploy:
           labels:
             traefik.frontend.rule: "Host:ray.i124u.cf"
             traefik.port: "80"
             traefik.docker.network: tfk_traefik_default
             traefik.frontend.passHostHeader: "true"
             traefik.frontend.whiteList.useXForwardedFor: "true"
    
       redis:
         image: redis
         restart: always
         networks:
          - wp_network
    networks:
      wp_network:
      tfk_traefik_default:
        external: true
    volumes:
        db_data:
        wp_data:

然后直接部署，问题就应该差不多了。

HTTPS Mix-content 的解决
---------------------

另外，这里胡乱的解决了一个前面很烦人的问题，就是使用这种方式的部署，wp的部分的静态资源是直接去请求 http 的站点的，最后会因为是 **mix-content** 最后被服务器的安全策略直接屏蔽了，导致各种的样式错误。

* * *

上了云了，就不需要进行本地访问了，所以前面的通过在`wp-config`里面对 home和root 进行动态修改的操作就不需要了。直接硬编码即可。

* * *

在配置文件里面有注意到这样的一个部分：

    if (isset($_SERVER['HTTP_X_FORWARDED_PROTO']) &&    $_SERVER['HTTP_X_FORWARDED_PROTO'] === 'https') {
        $_SERVER['HTTPS'] = 'on';
     }

这里面是有一个 https 的宏的，需要一个 `HTTP_X_FORWARDED_PROTO` 的头才能将它置位，所以我们就直接在nginx 里面对其进行设置。

     location / {
         proxy_set_header        Host $host;
         proxy_ignore_headers    Set-Cookie Cache-Control;   #这句代码很关键，尤其要忽略set-cookie
         proxy_set_header            X-real-ip $remote_addr;
         proxy_set_header            X-Forwarded-For $proxy_add_x_forwarded_for;
         proxy_set_header            X-Forwarded-Proto https; # 《--这里
         client_max_body_size        100m;
         proxy_pass http://127.0.0.1:xxxx;
     }

后面就是全站的 https 了，就不会再向http来拉取内容。

配置站点多域名
-------

由于路由和转发是在 traefik 上实现的，所以之于Nginx还是显得比较陌生，所以这里就试着来使用两个域名作为本站点的访问，在issue里面找到了和自己情况类似的内容，所以参考之

*   [Docker - 2 independant rules on same container](https://github.com/containous/traefik/issues/444)

* * *

在我们的容器的label 里面直接配置：

    traefik.port: '80'
    traefik.frontend.rule: 'Host:ray.i124u.cf,blog.diglp.xyz'

就可以实现多个域名对应同一个容器了，使用 `,` 表示是规则的 或关系，使用 `;` 表示是与关系。另外别忘了设置 `proxy_set_header Host $host;` 否则是无法正确路由的