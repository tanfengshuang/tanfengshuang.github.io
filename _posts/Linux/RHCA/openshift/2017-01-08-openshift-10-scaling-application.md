---
layout: post
title:  "创建支持可扩展的应用"
categories: Openshift
tags: openshift
---

*    默认创建的应用都是不能扩展的
*    默认应用是使用单一的gear服务所有的请求
*    要创建可扩展的应用需要在创建应用时加上参数-s
*    OpenShift中可扩展应用前端是使用Haproxy作为分发代理后端多个gear相应请求的
*    默认Openshift根据请求的多少而自动增加可扩展应用中gear的数量
*    用户可以设置相应参数，手动调整可扩展应用中gear的数量
*    可扩展应用中的gear之间是不会同步数据或保持持久化数据的
*    如果需要在多个gear之间同步数据或保持除数据库之外的一致性访问数据，请使用类似Swift的分布式文件系统和REST API
*    可以通过http://app-namespace-CLOUD_DOMAIN/haproxy-status查看HAProxy的运行状态


### 增加负载能力的可扩展应用（Scaling Application）

###### 可扩展应用Haproxy-status页面安全

```
配置Haproxy-status页面访问权限

登录应用
# rhc ssh AppName

编辑配置文件
# vim haproxy/conf/haproxy.conf
stats auth username:password        -> 在'list stats'下加入

重启Haproxy Cartridge使配置生效
# ctl_app restart
选择haproxy
```

###### 手动扩展和收缩可扩展应用的gear数量

```
创建配置文件，将扩展性调至手动模式
touch .openshift/makers/disable_auto_scaling

ssh登录应用
shc ssh AppName

为应用增加gear
add-gear -a $OPNSHIFT_APP_NAME -u $USER -s namespace

为应用减少gear
remove-gear -a $OPNSHIFT_APP_NAME -u $USER -s namespace
```

### 创建并配置可扩展应用

###### 创建支持可扩展的应用

```
使用student用户登录server0-a
# ssh student@server0-a

创建可扩展的应用
[student@server0-a] $ cd ~/ose
[student@server0-a] $ rhc app create scaledapp -t php-5.3 -s

查看可扩展的应用
[student@server0-a] $ rhc app show -a scaledapp

登录http://server0-a.example.com，查看可扩展应用的状态 - demo:redhat

将扩展方式调为手动
[student@server0-a] $ cd ~student/ose/scaleapp
[student@server0-a] $ touch .openshift/markers/disable_auto_scaling
[student@server0-a] $ git add .
[student@server0-a] $ git commit -m ".."
[student@server0-a] $ git push

###### 手动增加应用gear个数

```
查看增加gear前应用状态
# rhc app show scaledapp

登录应用
# rhc ssh scaledapp

add-gear增加gear个数
[student@server0-a] $ add-gear -a $OPNSHIFT_APP_NAME -u $USER -n ose

查看增加gear后应用状态
[student@server0-a] $ rhc app scaledapp
```



###### 查看可扩展应用的Haproxy状态，并配置安全访问

```
查看Haproxy运行状态
# firefox http://scaledapp-ose.apps0.example.com/haproxy-status

配置Haproxy-status页面访问权限
登录应用
[student@server0-a] $ rhc ssh scaledapp

编辑配置文件
[student@server0-a] $ vim haproxy/conf/haproxy.conf
stats auth username:password        在"listen stats"下加入

重启Haproxy Cartridge，使配置生效
[student@server0-a] $ ctl_app restart
选择haproxy



```








