---
title: 基于K8S的logstash无状态部署
tags:
  - Docker
  - K8S
  - OPS
url: 1093.html
id: 1093
categories:
  - DEV
date: 2019-08-18 00:33:50
---

前段时间就瞅中了Kubernetes，现在有机会用上那么一两下。这篇文章主要就是记载K8S的一个标标准准的使用过程，

标准化
---

其实现在发现自己是越来越喜欢标准化的东西来，使用默认的端口，使用默认的路径。 因为自己的无意修改，提升了熵，提升了复杂度。那么就需要付出额外的努力来使得熵降低（也就是维护自己所修改的那么一两个地方）

所以，这里使用k8s进行服务部署的思想，主要就是标准。标准也是自动化的前提。

* * *

所以这里使用了logstash的官方镜像使用差异化配置来进行功能的区分。对于这些无状态的组件是十分好用和方便的。 直接使用一个yaml文件，就可以进行快速的部署。在控制台也可以进行统一的管理。

配置文件
----

示例的配置文件在：[项目地址](https://github.com/zhoufwind/k8s-elastic)，这个项目的内容便是使用yaml来进行logstash的节点的快速搭建。

配置文件分为两个 部分。Deployment和configmap。Deployment是之前的swarm里的service的一部分。用于说明容器属性，环境变量等等。 service在k8s中 被独立了出来，用于进行一个deployment的端口的统一的管理。示例的yaml文件如下

    ---
    kind: ConfigMap
    apiVersion: v1
    metadata:
      name: logstash-xxx
      namespace: test
    data:
      logstash-config-named-k8s: |
        input {
          kafka {
            codec => "json"
            bootstrap_servers => "192.168.240.61:9092"
            topics => "named"
            group_id => "logstash-named-qa-k8s-topic"
            consumer_threads => 3
          }
        }
    
        filter {
          grok {
            match => ["message", "(?<timestamp>%{MONTHDAY}-%{MONTH}-%{YEAR} %{TIME}) queries: client %{IPV4:c_ip}#%{NUMBER:c_port}: query: %{NOTSPACE:queryrec} %{NOTSPACE:dnsclass} %{NOTSPACE:dnstype} \+ \(%{IPV4:dnsbind}\)"]
          }
        }
        output {
          #stdout { codec => rubydebug }
          elasticsearch {
            hosts => ["192.168.36.145:29200"]
            index => "logstash-kafka-named-%{+YYYY.MM.dd}"
          }
        }
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:           
      name: logstash-xxx
      namespace: test
      labels:
        app: logstash-test
    spec:
      serviceName: logstash-xxx
      replicas: 4
      selector:
        matchLabels:
          app: logstash-test
      template:
        metadata:
          labels:
            app: logstash-test
        spec:
          containers:
          - name: logstash-xxx
            image: docker.elastic.co/logstash/logstash:6.8.1
            env:
              - name: "NODE_NAME"
                valueFrom:
                  fieldRef:
                    fieldPath: metadata.name
              - name: "LS_JAVA_OPTS"
                value: "-Xmx5g -Xms5g -XX:ParallelGCThreads=8"
              - name: "XPACK_MONITORING_ENABLED"
                value: "true"
              - name: "XPACK_MONITORING_ELASTICSEARCH_HOSTS"
                value: "xxx"
              - name: "PIPELINE_WORKERS"
                value: "24"
              - name: "PIPELINE_OUTPUT_WORKERS"
                value: "24"
              - name: "PIPELINE_BATCH_SIZE"
                value: "1024"
              - name: "PIPELINE_BATCH_DELAY"
                value: "5"
              - name: "CONFIG_RELOAD_AUTOMATIC"
                value: "true"
              - name: "CONFIG_RELOAD_INTERVAL"
                value: "60"
              - name: "HTTP_HOST"
                valueFrom:
                  fieldRef:
                    fieldPath: status.podIP
            volumeMounts:
            - name: vm-config
              mountPath: /usr/share/logstash/pipeline
    
          volumes:
            - name: vm-config
              configMap:
                name: logstash-xxx
                items:
                - key: logstash-config-named-k8s
                  path: indexer-kafka-named-k8s.conf

文件主要是分为两个部分。Deployment和configmap。这里重点讲configmap。这个是k8s的特性，在原来的swarm的体系中，实现配置文件的统一管理的体验是十分糟糕的，只能通过吧目录向外部进挂载来实现配置管理。而如果存在多个主机和节点的话，还需要使用NFS来进行多主机的文件共享。显得十分的冗余。 而在k8s中引入了configmap的功能，把一个配置文件以键值的进行保存，且这个保存是多个节点共享的，所以这样的话就很自然的实现了配置文件的统一管理，所以这个配置很是好用。

后
-

到了k8s的环境下实际上还是进行写yaml的过程，这个过程中，把应用从主机的层面抽象出来，可以方便的实现快速扩容，等等。 除了对语法和概念的了解需要补充，其他的用的挺舒服