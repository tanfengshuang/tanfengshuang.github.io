---
layout: post
title:  "安装 RHEL Atomic(270-1)"
categories: Linux
tags: RHCA 270
---

### RHEL Atomic 概念

###### 什么是 Docker 包装模型？

名称空间 	功能
*    PID 	用于进程隔离。
*    net 	用于管理网络接口以便每个容器都有自己的 IPv4 和 IPv6 网络协议栈、路由表和防火墙规则。
*    IPC 	用于管理进程间的通信资源以便每个容器将有独立的共享内存区域、信号量集和消息队列。
*    mount 	基于容器映像为容器提供隔离的文件系统数据。
*    UTS 	用于隔离内核和版本号。设置主机名和域名不会影响系统的其他部分。


RHEL Atomic 主机默认配置为使用两个公共的映像注册表：
1. registry.hub.docker.com（也称为 “Docker 枢纽”）
2. registry.access.redhat.com

###### 什么是 RHEL Atomic 主机？

红帽企业 Linux Atomic 主机是红帽企业 Linux 7 的变体，针对在 Docker 容器化环境中运行的 Linux 容器进行了优化。它随 Red Hat Enterprise Linux 7.1 一起发布。RHEL Atomic 主机预装有以下工具：

*    docker 实用工具
*    Docker 守护进程
*    Kubernetes，一种用于协调容器的工具
*    rpm-ostree
*    systemd
*    基于红帽企业 Linux 7 的内核，支持 cgroups、名称空间和 SELinux

RHEL Atomic 是指一个运行时环境。该环境不用于开发容器应用。它的主要用例是用于运行容器应用。典型的开发和部署工具在 RHEL Atomic 主机环境中不可用。RHEL Atomic 在以下几个重要方面与标准的红帽企业 Linux 不同：

*    yum 不用于安装软件或升级系统；RHEL Atomic 包括支持 Atomic 升级和回滚的新工具。
*    只有两个用于本地系统配置的可写目录：/etc 和 /var。
*    / 目录上设置了不可改变位。
*    /usr 以只读方式挂载，但 /usr/local 是 /var/usrlocal 的符号链接。
*    其他目录通过符号链接到可写位置；例如 /home 是 /var/home 的符号链接。
*    RHEL Atomic 主机中未安装 firewalld 软件；仅 Netfilter 和 iptables 可用于网络数据包过滤。
*    未安装客户端 Kerberos 工具（通常由 krb5-workstation 软件包提供）。
*    未安装 iSCSI 客户端或启动器软件。 


### 安装 RHEL Atomic 软件

要安装 RHEL Atomic 主机，第一步是从红帽客户门户下载安装 ISO 或虚拟机映像。RHEL Atomic 主机以几种不同的方式分发：
1. 可配合 OpenStack 或 KVM 使用的 .qcow2 映像。
2. 可配合 RHEV 或 VMware 使用的 .ova 映像。
3. 可配合 Hyper-V 使用的 .vhd 映像。
4. 可用于裸机硬件或虚拟客户机安装的 .iso 映像。
 
 
### Summary

RHEL Atomic 概念 .  在本节，您学到:
1. 有关 Linux 容器的架构和关键术语。
2. RHEL Atomic 主机是红帽企业 Linux 7 的特殊版，为容器提供了一个运行的环境。

安装 RHEL Atomic 软件 .  在本节，您学到:
1. 通过红帽客户门户，可找到不同类型的 RHEL Atomic 主机安装映像。
2. 如何使用 RHEL Atomic 主机安装程序安装一台服务器。 




