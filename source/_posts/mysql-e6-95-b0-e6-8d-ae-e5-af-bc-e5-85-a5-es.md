---
title: Mysql 数据导入 ES
tags:
  - ELK
  - logstash
url: 1109.html
id: 1109
categories:
  - OP之路
date: 2019-08-25 17:33:48
---

这篇算是一个自己的实践贴，虽然都可以看到有现成的方案。但是其中有一些坑，还是自己得记下来。

缘起
--

自己后面有想做一个和检索相关的小站点，所以就现在开始倒腾数据。 看起来是一个 `select * from` 就能解决的问题，但是考虑到实际情况这就有问题了。 所以这里用上了elasticsearch，作为一个全文搜索的引擎。

这次的目的，就是把mysql 的数据导入到e 送中去。

过程
--

导入过程需要logstash来进行，使用jdbc的插件，对数据库进行查询后直接output到es集群。 先给ls来安装插件：

    ./logstash-plugin install logstash-input-jdbc

这部分很快可以搞定，直接会拉取在线安装

* * *

在java连接数据库需要使用到connect的驱动。所以这一步需要准备mysql的驱动。这里有个坑，在文章后面会写。这里就直接使用 `mysql-connector-java-8.0.14.jar` 就好了，驱动可以直接在官方上下到。 由于自己使用的是mariaDB10 ，所以需要使用8的版本，如果是mysql5可以使用5版本。而且需要注意的是：`jdbc_driver_class` 这个配置项，在mysql8下已经变成了：`com.mysql.cj.jdbc.Driver` 如果指定之前的`com.mysql.jdbc.Driver`会出现：**cannot be loaded**的错误。

* * *

上面的都准备好之后，就开始写导入过程的pipeline，这里直接给出来：

    input {
        jdbc {
          jdbc_connection_string => "jdbc:mysql://mysql:3307/mydb"
          jdbc_user => "root"
          jdbc_password => "password"
          jdbc_driver_library => "/usr/local/logstash/config/mysql-connector-java-8.0.14.jar"
          jdbc_driver_class => "com.mysql.cj.jdbc.Driver"
          jdbc_paging_enabled => "true"
          jdbc_page_size => "50000"
    
          jdbc_default_timezone => "Asia/Shanghai"
          statement_filepath => "select * from test_table" # 查询的语句
          schedule => "* * * * *"   # 每分钟一次
          type => "jdbc"
        }
    }
    
    output {
        elasticsearch {
            hosts => "es1:9200"
            index => "mysql_test"
        }
    }
    

有了 pipeline之后，直接运行logstash：

    ./bin/logstash -f ./pipeline/mysql.conf

然后就开始导入，导入完成之后就可以体验es带来的全文搜索的快感了。

遇到的问题
-----

这里的驱动只能使用openjdk1.8 的版本来运行。由于之前使用的是最新的java12，发现无论怎么修改配置，都会报错类无法被加载的问题。