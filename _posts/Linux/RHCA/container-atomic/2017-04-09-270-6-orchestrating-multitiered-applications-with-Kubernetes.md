---
layout: post
title:  "使用 Kubernetes 编排多层应用(270-6)"
categories: Linux
tags: RHCA 270
---

### Kubernetes 概念


### 在 RHEL Atomic 主机中配置 Kubernetes 


### 创建 Kubernetes Pod 和服务


### Summary

1. 创建 Kubernetes 群集时要修改的文件包括：

*    /etc/kubernetes/config
*    /etc/kubernetes/controller-manager
*    /etc/kubernetes/apiserver

2. 创建 Kubernetes 节点时要修改/创建的文件包括：

*    /etc/kubernetes/config
*    /etc/kubernetes/proxy
*    /etc/kubernetes/kubelet
*    /var/lib/kubelet/auth

3. kubectl 命令用于查询和应用配置更改到 Kubernetes apiserver 守护进程中。
4. 标签是可分配给任何 Kubernetes 对象的键/值对。标签选择器是可将 Kubernetes 对象组合在一起的标签。


