---
layout: post
title:  "RHEL Atomic 主机故障排除(270-8)"
categories: Linux
tags: RHCA 270
---

### 红帽工具容器的故障排除



### 在 RHEL Atomic 主机中生成 sosreport



### 使用 kdump 创建 RHEL Atomic 主机 vmcore



### Summary

*    红帽提供名为 rhel-tools 的支持工具容器，它提供了许多未在 RHEL Atomic 主机上安装的诊断实用工具。
*    该 atomic run 命令将启动工具容器以获取 RHEL Atomic 主机和其他容器的访问特权。
*    诊断工具在容器化环境中的行为与其在典型的 Red Hat Enterprise Linux 7 主机上的行为不同。
*    必须从支持工具容器中运行 sosreport 命令，并将 /var/tmp/sosreport-*.tar.gz 文件中的报告存储在 RHEL Atomic 主机上。
*    kdump 将在内存超过 1024 MiB 的 RHEL Atomic 主机系统上工作，且它们将 vmcore 映像默认保存在 /sysroot/crash 目录中。 



