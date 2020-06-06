---
title: Compose file for Dos Game
tags:
  - Docker
url: 1085.html
id: 1085
categories:
  - DEV
date: 2019-08-11 20:58:50
---

这一篇只是存档作用啦，因为之前的旧集群又搬走了，不过还好整体是跑的docker，迁移起来还是很容易的。 这个是一个快速搭建一个线上的Dos游戏的yaml文件。挺好玩的就先留下来

compose 文件
----------

    version: '3'
    
    services:
       web_game:
         image: oldiy/dosgame-web-docker:latest
         restart: always
         ports:
           - 262
         networks:
            - tfk_traefik_default
         deploy:
           labels:
             traefik.frontend.rule: "Host:game.diglp.xyz"
             traefik.port: "262"
             traefik.frontend.passHostHeader: "true"
             traefik.docker.network: tfk_traefik_default
    
    networks:
      tfk_traefik_default:
        external: true