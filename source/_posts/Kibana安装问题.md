---
title: Kibana安装问题
date: 2023-02-04 22:32:31
categories: 错误排查
tags:
    - Elasticsearch
---

## Kibana安装后产生CREATE_NEW_TARGET -> CREATE_NEW_TARGET错误问题

解决方案

找到es中的elasticsearch.yml文件 检查以下配置是否正确
```yml

# Enable security features

xpack.security.enabled: false

xpack.security.enrollment.enabled: true

# Enable encryption for HTTP API client connections, such as Kibana, Logstash, and Agents
xpack.security.http.ssl:

enabled: false

keystore.path: certs/http.p12

# Enable encryption and mutual authentication between cluster nodes
xpack.security.transport.ssl:

enabled: false

verification_mode: certificate

keystore.path: certs/transport.p12

truststore.path: certs/transport.p12

# Create a new cluster with the current node only
# Additional nodes can still join the cluster later

cluster.initial_master_nodes: ["Node1"]

```