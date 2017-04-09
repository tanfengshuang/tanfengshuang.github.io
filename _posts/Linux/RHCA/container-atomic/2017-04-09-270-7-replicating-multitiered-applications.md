---
layout: post
title:  "复制多层应用(270-7)"
categories: Linux
tags: RHCA 270
---

### 配置 Kubernetes 使用 Flannel 网络


### 创建 Kubernetes 复制控制器


### Kubernetes 复制故障排除


### Summary

*    将 Flannel 配置规范上传到 etcd，然后修改 Kubernetes 节点上的 /etc/sysconfig/flanneld 以指向 etcd 服务器。
*    Kubernetes 复制通过 desiredState 配置对象中定义的值控制。replicas 定义在 replicaSelector 对象中应运行多少个通过标签匹配的容器。
*    通过使用私有 pod IP 地址来访问容器服务可识别错误的 Kubernetes 节点。 
