---
title: Metricbeat 容器化启动
url: 1308.html
id: 1308
categories:
  - 未分类
tags:
---

目前迁移至Docker平台，需要根据业务对监控进行调整。 预计使用metric的容器在同一namespace 内进行访问 官方示例的直接启动命令：

    docker run \
    docker.elastic.co/beats/metricbeat:7.3.2 \
    setup -E setup.kibana.host=kibana:5601 \
    -E output.elasticsearch.hosts=["elasticsearch:9200"]

官方的示例配置：

> curl -L -O [https://raw.githubusercontent.com/elastic/beats/7.3/deploy/kubernetes/metricbeat-kubernetes.yaml](https://raw.githubusercontent.com/elastic/beats/7.3/deploy/kubernetes/metricbeat-kubernetes.yaml)

可以通过-E 指定或者直接使用 yml文件。

filebeat使用配置：

    metricbeat.modules:
    - module: elasticsearch
    metricsets:
        - node
        - node_stats
        - index
        - index_recovery
        - index_summary
        - shard
    metricbeats: ["node","node_stats","index","index_recovery","index_summary","shard"]
    # - ml_job
    period: 10s
    hosts: ["http://elasticsearch68-client-service:9202"]
    #username: "user"
    #password: "secret"