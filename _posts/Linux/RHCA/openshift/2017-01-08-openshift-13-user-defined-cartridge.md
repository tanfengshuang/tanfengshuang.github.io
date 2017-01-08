---
layout: post
title:  "创建自定义Cartridge"
categories: Openshift
tags: openshift
---

*    OpenShift Enterprise支持开发和运维人员自定义Cartridge
*    自定义Cartridge中包含必要的目录和文件
*    OpenShift Enterprise自定义Cartridge文档为"[Cartridge Specification Guide]"(https://access.redhat.com/documentation/en-US/OpenShift_Enterprise/2/html/Cartridge_Specification_Guide/index.html)

### 自定义Cartridge所需要的目录及文件说明

*    metadata目录：（manifest.yml文件， managed_file.yml文件）
*    bin目录：（setup文件，control文件）

### 自定义Cartridge文件结构示例

```
下载自定义Cartridge打包文件
[root@server0-b] # wget http://classroom.example.com/pub/materials/mock.tar.gz

解开mock.tar.gz包查看目录及文件
[root@server0-b] # tar vxzf mock.tar.gz
[root@server0-b] # cd mock/metadata
[root@server0-b] # ls
manifest.yml        managed_file.yml
```

### 将自定义Cartridge加入OpenShift系统

```
登录所有Node节点，并将自定义Cartridge文件放置在相关目录中
[root@server0-b] # wget http://classroom.example.com/pub/materials/mock.tar.gz
[root@server0-b] # tar vxzf mock.tar.gz -C /usr/libexec/openshift/cartridges

列出当前node节点支持的Cartridge
[root@server0-b] # oo-admin-cartridge --list

激活并更新node节点支持新Cartridge
[root@server0-b] # oo-admin-cartridge --action install --source /usr/libexec/openshift/cartridges/mock

再次列出当前node节点支持的Cartridge，可以看到新的Cartridge
[root@server0-b] # oo-admin-cartridge --list

重启ruby193-mcollective服务，提交信息到broker节点
[root@server0-b] # service ruby193-mcollective restart

登录broker节点，刷新Cartridge信息
[root@server0-a] # oo-admin-ctl-cartridge -c list
[root@server0-a] # oo-admin-ctl-cartridge -c import-node --active
[root@server0-a] # oo-admin-console-cache --clear
[root@server0-a] # oo-admin-ctl-cartridge -c list

登录客户端使用rhc查看可用Cartridge
[student@server0-a] $ rhc cartridges

```

### 将自定义Cartridge移出OpenShift系统

```
首先需要在broker节点上标记Cartridge为非活跃
[root@server0-a] # oo-admin-ctl-cartridge -c list
[root@server0-a] # oo-admin-ctl-cartridge -c deactivate --name mock-0.1
[root@server0-a] # oo-admin-ctl-cartridge -c deactivate --name mock-0.2
[root@server0-a] # oo-admin-ctl-cartridge -c deactivate --name mock-0.3
[root@server0-a] # oo-admin-ctl-cartridge -c deactivate --name mock-0.4
[root@server0-a] # oo-admin-ctl-cartridge -c list

在所有node节点上清除自定义Cartridge信息和文件
[root@server0-b] # oo-admin-ctl-cartridge -c list
[root@server0-b] # rm -rf /usr/libexec/openshift/cartridges/mock
[root@server0-b] # oo-admin-ctl-cartridge --action erase --name mock --version 0.1 --cartridge_version 0.0.1
[root@server0-b] # oo-admin-ctl-cartridge -c list
[root@server0-b] # service ruby193-mcollective restart

在broker节点上清除已经删除的Cartridge信息
[root@server0-a] # oo-admin-ctl-cartridge -c list
[root@server0-a] # oo-admin-ctl-cartridge -c clean
[root@server0-a] # oo-admin-ctl-cartridge -c list

在broker节点上清除过时缓冲
[root@server0-a] # oo-admin-broker-cache --clear --console
[root@server0-a] # oo-admin-broker-cache --clear
```
