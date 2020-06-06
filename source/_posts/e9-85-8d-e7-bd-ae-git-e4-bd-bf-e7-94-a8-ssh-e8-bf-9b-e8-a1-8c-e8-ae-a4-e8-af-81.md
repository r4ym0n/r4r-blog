---
title: 配置 Git 使用 SSH 进行认证
tags:
  - git
  - misc
  - OPS
url: 797.html
id: 797
categories:
  - OP之路
date: 2019-04-15 22:19:00
---

前
-

可能由于服务器的TLS版本不对，所以试着账号密码直接登陆总是显示认证失败。

想想，使用账号密码认证，每次提交还需要进行登陆认证，的确麻烦。所以这里就直接配置使用 SSH进行认证了。

(动态博客，就想写就写了无所谓篇幅了)

配置过程
----

### 生成密钥对

    ssh-keygen -t rsa -C&quot;mail@mail.com&quot;   # 这里换成自己的邮箱

生成的RSA密钥对在默认的 `/root/.ssh/` 目录下

### Github 添加公钥

直接上传公钥

![file](https://i.loli.net/2019/04/15/5cb48b4d176ee.png)

### 测试

    ssh -Tv git@github.com  # 测试基于证书的Github连接

![file](https://i.loli.net/2019/04/15/5cb48b1a75629.png)

### 修改配置

修改项目内容的url为ssh的形式，后面就可以实现免登陆的ssh认证。

      [remote &#039;origin&#039;]
      url = git@github.com:xxxx/xxxx.git

* * *

至此，配置完成。

后
-

这里打算通过 WebHook 来实现 WP 文章的更新触发 GitHub 上的一个commit。算是记录一下自己的进度吧。