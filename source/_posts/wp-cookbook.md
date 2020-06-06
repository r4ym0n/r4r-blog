---
title: WP-CookBook
tags:
  - wordpress
url: 1523.html
id: 1523
categories:
  - OP之路
date: 2019-12-11 10:49:36
---

Tricks
------

### 改用户名

想改登录名怎么办？不能直接改，那就改库：

    UPDATE wp_users SET user_login = '新用户名' WHERE user_login = 'admin';

### 忘记密码

这里直接改数据库了，结合这里的docker的部署方式。先attach上容器

    docker exec -it forum_guet_db.1.poow5nr4a081b55yftqmxwj2v  /bin/bash

得到shell之后，直接 `mysql -uroot -p` 连上数据库。`use wordpress` 选库。之后直接对`wp_users` 表内的内容更新即可。语句如下：

    update wp_users set user_pass=md5('123') where user_login='admin';