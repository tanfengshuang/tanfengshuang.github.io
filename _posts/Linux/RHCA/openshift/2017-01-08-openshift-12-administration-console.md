---
layout: post
title:  "安装配置和管理Administration Console"
categories: Openshift
tags: openshift
---

*    Administration Console是OpenShift Enterprise 2.1后新推出的OpenShift管理应用工具
*    Administration Console默认没有安装在OpenShift的节点上，需要手工安装
*    Administration Console默认启动后，仅允许Broker节点本地登录
*    Administration Console默认使用Apache httpd代理访问，认证也使用htpasswd方式

### 安装Administration Console

*    软件包名为rubygem-openshift-origin-admin-console，安装在broker节点上

### 启动Administration Console

*    Administration Console和两个服务有关(openshift-broker, httpd)

```
[root@server0-a] # yum install rubygem-openshift-origin-admin-console
[root@server0-a] # service openshift-broker restart
[root@server0-a] # service httpd restart
```

### 设置Administration Console认证

*    默认使用Apache Basic Auth

```
[root@server0-a] # htpasswd -b /etc/openshift/htpasswd kevin redhat
```

*    访问方式为http web方式，默认只允许broker节点

```
[root@server0-a] # firefox http://broker-node-name:8080/admin-console
```


### REST API

*    客户端或其他开发用户可以通过REST API方式得到Administration Console API的信息响应
*    常用的Administration Console信息获取

```
获得Admin stats所有配置文件汇总表
# curl http://broker-node-name:8080/admin-console/capacity/profiles.json

获得用户运行gear的统计表
# curl http://broker-node-name:8080/admin-console/stats/gear_per_user.json

获得域运行的应用的统计表
# curl http://broker-node-name:8080/admin-console/stats/apps_per_domain.json

获得域中用户数的统计表
# curl http://broker-node-name:8080/admin-console/stats/domain_per_use.json
```

### Administration Console访问控制

*    默认仅允许broker本地访问
*    访问端口8080
*    允许客户端或其他主机通过网络访问，需要设置

```
在broker节点上设置apache http proxy
[root@server0-a] # vim /etc/httpd/conf.d/00002_openshift_origin_broker_proxy.conf
ProxyPass /admin-console http://127.0.0.1:8080/admin-console
ProxyPass /assets http://127.0.0.1:8080/assets
```
