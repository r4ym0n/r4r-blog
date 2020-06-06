---
title: GIT 配置 SSH 证书访问
url: 1643.html
id: 1643
categories:
  - DEV
date: 2020-06-06 02:51:41
tags:
---

Q：为什么需要使用证书来进行认证呢？ A：自己的电脑上有登录其他的GIT的账号，来回切换太麻烦

配置
--

创建得 puk 和 一个key文件在 `~/.ssh/` 下面，上传 公钥到 github 上来进行保存

    # 生成RSA密钥对
    ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
    
    # 自定义命名
    # > Enter a file in which to save the key (/Users/you/.ssh/id_rsa): [Press enter]
    # 这里直接回车，或者创建自己名字的证书，不建议覆盖

配置 ssh 的策略文件，在请求到 github 域的时候使用这个公钥来进行通信。

    Host github.com
      AddKeysToAgent yes
      UseKeychain yes
      IdentityFile ~/.ssh/github_rsa
    Host *.github.com
      AddKeysToAgent yes
      UseKeychain yes
      IdentityFile ~/.ssh/github_rsa

添加到 ssh 的agent 中

    ssh-add -K ~/.ssh/id_rsa

到这里，证书的添加应该就是 OK的了，进行认证测试

    ssh -T git@github.com

使用
--

这时候，应该就可以直接通过 ssh 来对代码进行 clone 和 push了，但是在之前，有一个细节需要注意，因为这里是多账号环境，如果不在目标目录来设置一下当前的username 等信息，就会以默认的 globle 的git 配置的账号来进行操作。

    git config user.name quartz010
    git config user.email quartz010@outlook.com