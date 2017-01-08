---
layout: post
title:  "管理Openshift资源"
categories: Openshift
tags: openshift
---

*    OpenShift可以通过对gear或用户限制的方式管理资源
*    我们将对gear size和district做相应的配置完成资源管理的配置
*    对district的设置将解决之前oo-accept-node测试时出现的ERROR

### 管理Gear Size

*    Gear就是拥有资源的容器，应用在这个容器里运行
*    Gear Size由Broker节点设置并由Node节点支持
*    每个Gear所拥有的资源是受限的，而且Gear之间彼此隔绝
*    我们可以设置一个用户拥有Gear的个数


###### Broker规范了Gear的范围并且配置Gear的默认大小

*    配置文件 - /etc/openshift/broker.conf
*    配置项

```
[server0-a] # vim /etc/openshift/broker.conf
VALID_GEAR_SIZE="small,medium,large"
DEFAULT_MAX_GEARS="100"
DEFAULT_GEARS_CAPABILITES="small,medium"
DEFAULT_GEARS_SIZE="small"

[server0-b] #
resource_limits.conf        -> 默认是small
resource_limits.conf.large.m3.xlarge
resource_limits.conf.medium.m3.xlarge
resource_limits.conf.small.m3.xlarge
resource_limits.conf.xpass.m3.xlarge
```

###### Node支持的Gear size定义

*    配置文件   - /etc/openshift
*    需要将节点Gear size和District关联起来 - 在District部分具体介绍

###### 设置用户默认支持的Gear size

*    配置/etc/openshift/broker.conf
*    重启openshift-broker系统服务
*    使用oo-admin-ctl-user添加和删除gear size


```
为每个用户设置gear相关资源配额
[server0-a] # oo-admin-ctl-user -l demo

[server0-a] # oo-admin-ctl-user -l demo | grep --color "max gears"
max gears: 100
[server0-a] # oo-admin-ctl-user -l demo --setmaxgears 25
[server0-a] # oo-admin-ctl-user -l demo | grep --color "max gears"
max gears: 25

[server0-a] # oo-admin-ctl-user -l demo | grep --color "small"
gear sizes: small
[server0-a] # vim /etc/openshift/broker.conf
VALID_GEAR_SIZE="small,medium,large"
[server0-a] # service openshift-broker restart
[server0-a] # oo-admin-ctl-user -l demo --addgearsize medium        -> 我认为修改的是DEFAULT_GEARS_CAPABILITES的值，可以后面验证
[server0-a] # oo-admin-ctl-user -l demo | grep --color "small"
gear sizes: small, medium
[server0-a] # oo-admin-ctl-user -l demo --addgearsize large
[server0-a] # oo-admin-ctl-user -l demo | grep --color "small"
gear sizes: small, medium, large

[server0-a] # oo-admin-ctl-user -l demo --removegearsize large

对于多个Node节点
node1安装了small
node2安装了medium
node3安装了large
当用户设置了gear size为large时，会用node3

```


### 管理District相关配置

*    Openshift以District的形式定义资源限制
*    在多个Node节点间迁移gear的时候，District可以保证使用相同的UUID和IP
*    由Broker节点创建并定义District
*    需要将Node节点添加入创建的District中

###### District信息由Mcollective传递


```
确认配置文件允许实现District
[server0-a] # cd /etc/openshift/plugins.d/
[server0-a] # ls
openshift-origin-auth-remote-user.conf
openshift-origin-auth-remote-user.conf.example
openshift-origin-dns-nsupdate.conf
openshift-origin-dns-nsupdate.conf.example
openshift-origin-msg-broker-mcollective.conf
openshift-origin-msg-broker-mcollective.conf.example
[server0-a] # vim /etc/openshift/plugins.d/openshift-origin-msg-broker-mcollective.conf
DISTRICT_ENABLED = true
NODE_PROFILE_ENABLED = true

```

###### 创建新的District并将Node节点加入

*    使用命令oo-admin-ctl-district创建新的District
*    使用命令oo-admin-ctl-district将server0-b加入small_district

```
[server0-a] # oo-admin-ctl-district -c list-available
Node in profile: small
        server0-b.example.com
[server0-a] # oo-admin-ctl-district -c create -n small_district -p small
[server0-a] # oo-admin-ctl-district -c add-node -n small_district -i server0-b.example.com

[server0-a] # oo-accept-node -v
INFO: find district uuid: ...
PASS
```
