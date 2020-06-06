---
title: Traefik 的真实IP转发
url: 865.html
id: 865
categories:
  - 未分类
tags:
---

问题,wp 的 server得到的IP不是nginx传来的真实的IP,是traefik 的假IP 参照很多方法无果.. [https://github.com/containous/traefik/issues/3210](https://github.com/containous/traefik/issues/3210) [https://github.com/containous/traefik/pull/3070](https://github.com/containous/traefik/pull/3070) [https://github.com/containous/traefik/issues/2942](https://github.com/containous/traefik/issues/2942) [https://github.com/containous/traefik/issues/2730](https://github.com/containous/traefik/issues/2730) [https://docs.traefik.io/configuration/entrypoints/#forwarded-header](https://docs.traefik.io/configuration/entrypoints/#forwarded-header) [https://my.oschina.net/guol/blog/2209678](https://my.oschina.net/guol/blog/2209678)