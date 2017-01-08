---
layout: post
title:  "使用域来隔离环境"
categories: Openshift
tags: openshift
---

*    OpenShift默认使用Apache Basic Auth认证用户
*    OpenShift用户信息保存在MongoDB中
*    OpenShift默认使用域（Domain）方式隔离同一个用户或组创建的gear
*    OpenShift支持多租户共享Domain

### OpenShift客户的三种权限类型

*    域和应用的创建者，可以为其他用户分配权限: admin(owner)
*    应用程序的开发人员，可以上传和控制应用: edit
*    信息查看和阅览人员，可以看到域和应用的信息: view

### 创建和管理用户及权限

```
使用oo-admin-ctl-user命令在broker节点上创建用户
[root@server0-a] # oo-admin-ctl-user -c -l kevin
[root@server0-a] # oo-admin-ctl-user -c -l todd
[root@server0-a] # oo-admin-ctl-user -c -l shrek

为用户配置登录密码
[root@server0-a] # htpasswd -b /etc/openshift/htpasswd kevin redhat
[root@server0-a] # htpasswd -b /etc/openshift/htpasswd todd redhat
[root@server0-a] # htpasswd -b /etc/openshift/htpasswd shrek redhat
```

### 为不同的用户配置不同权限

```
在broker节点上创建对应名称用户
[root@server0-a] # useradd kevin
[root@server0-a] # useradd todd
[root@server0-a] # useradd shrek

为用户设置密码，并用不同终端登录
[root@server0-a] # echo redhat | passwd --stdin kevin
[root@server0-a] # echo redhat | passwd --stdin todd
[root@server0-a] # echo redhat | passwd --stdin shrek
```

### 基于域的用户和权限管理

```
使用不同用户登录broker节点（开发节点）
# ssh kevin@server0-a
# ssh todd@server0-a
# ssh shrek@server0-a

使用kevin用户创建环境和域
[kevin@server0-a] $ rhc setup
[kevin@server0-a] $ rhc domain create osedev

将todd用户和shrek用户添加到osedev域中
[kevin@server0-a] $ rhc member add todd -n osedev
[kevin@server0-a] $ rhc member add shrek -n osedev

配置域中的用户权限
1. 查看当前权限
[kevin@server0-a] $ rhc member list -a osedev
2. 设置shrek权限为view
[kevin@server0-a] $ rhc member update --role view shrek -n osedev
```

### 分别使用不同用户连接OpenShift服务说明权限

###### rhc setup 命令运行状态

*    admin, edit用户权限可以加入域并创建应用
*    view用户权限只能查看信息，无法完全执行setup操作

###### rhc app create 命令运行状态

*    admin, edit用户权限可以创建新的应用
*    view用户权限不可以创建新的应用

###### rhc domain create

*    所有权限均可创建新的域，并在新的域中创建自己的应用

### 使用OpenShift Web Consle登录管理应用

*    所有权限用户均可登录web console
*    admin、edit用户权限可以创建新的应用和已有应用的附加Cartridge
*    view用户权限需要创建新的自有域后才能创建应用，并且不能创建原有域中应用的附加Cartridge

