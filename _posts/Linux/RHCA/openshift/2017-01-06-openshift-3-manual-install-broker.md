---
layout: post
title:  "手动安装单机Openshift环境"
categories: Openshift
tags: openshift
---

### 为手动安装OpenShift作准备

###### RedHat OpenShift安装频道

*    Broker

Broker 是所有应用管理活动的入口。它主要负责管理用户登录、DNS、应用状态以及应用服务编排（服务分发）。用户和 Broker 交互主要是通过 Web 管理控制台、CLI 工具、JBoss 工具或者是 REST API

```
rhel-6-server-ose-2.1-infra-rpms
rhel-6-server-ose-2.1-rhc-rpms
rhel-6-server-rpms
rhel-6-server-rhscl-6-rpms
```

*    Node

```
rhel-6-server-ose-2.1-jbosseap-rpms
rhel-6-server-ose-2.1-node-rpms
rhel-6-server-rpms
rhel-6-server-rhscl-6-rpms
jb-eap-6-for-rhel-6-server-rpms
jb-ews-6-for-rhel-6-server-rpms
```

*    Developer workstations

```
rhel-6-server-ose-2.1-rhc-rpms
```

###### RedHat OpenShift官方文档位置

*    https://access.redhat.com/documentation/en-US/OpenShift_Enterprise/
*    Administration Guide
*    Deployment Guide
*    Troubleshooting Guide
*    User Guide

### 安装配置Broker基础消息服务

*    安装配置Mongodb服务
*    安装配置ActiveMQ服务
*    安装配置Mcollective服务
*    保证httpd和ntpd服务开机启动

###### 安装配置Mongodb服务

```
# rht-vmctl status all
# rht-vmctl start servera

安装配置Mongodb服务和客户端
# yum install -y mongodb-server mongodb

修改配置文件
# vim /etc/mongodb.conf     -> 在文件最后添加两行
auth=true
smallfiles=true

配置mongodb开机启动
# chkconfig mongod on; service mongod start

配置mongodb环境，添加broker所需要的用户
# mongo
> show dbs
admin   (empty)
local   0.03125GB
> user admin
> db.addUser("admin", "redhat")
> db.auth("admin", "redhat")
> use openshift_broker
> db.addUser("openshift", "redhat")
> db.auth("openshift", "redhat")
> exit
```


###### 安装配置ActiveMQ服务

```
安装ActiveMQ服务和客户端
# yum install -y activemq activemq-client

配置ActiveMQ服务
# rm -rf /etc/activemq/activemq.xml
# wget -P /etc/activemq/ http://classroom.example.com/pub/materials/activemq.xml
# vim /etc/activemq/activemq.xml
brokerName=server0-a.example.com
将username是mcollective的密码修改为"redhat"
将username是admine的密码修改为"redhat"

配置开机启动ActiveMQ服务
# chkconfig activemq on; service activemq start

查看日志确认启动正常
# tail /var/log/activemq/activemq.log

配置ActiveMQ web console
# vim /etc/activemq/jetty-realm.properties      -> 添加用户及权限配置到jetty-realm.properties文件
<import resource="jetty.xml"/>      -> 有这一行就表示可以web访问
admin: redhat, admin                -> 定义可以访问网页的用户 username: password, rolename
# service activemq restart          -> 重启ActiveMQ服务
浏览器访问http://127.0.0.1:8161/admin

```

###### 安装配置Mcollective服务


```
打开61613/tcp端口访问权限
# iptables -L -n > iptables.out
# lokkit --port=61613:tcp
# iptables -L -n > iptables.out.1

安装Mcollective软件包
# yum install -y ruby193-mcollective-client

下载配置模板文件并设置其权限
# rm -rf /opt/rh/ruby193/root/etc/mcollective/client.cfg
# wget -p /opt/rh/ruby193/root/etc/mcollective/ http://classroom.example.com/pub/materials/client.cfg
# chown apache.apache /opt/rh/ruby193/root/etc/mcollective/client.cfg
# chmod 640 /opt/rh/ruby193/root/etc/mcollective/client.cfg

修改client.cfg文件
# vim /opt/rh/ruby193/root/etc/mcollective/client.cfg
#logger_type=console        -> 注释掉这一行
logfile=/var/log/mcollective-client.cfg     -> 添加这一行
loglevel=debug              -> 将log改为debug级别，方便定位问题
plugin.activemq.pool.1.host=server0-a.example.com
plugin.activemq.pool.1.password="redhat"
factsource=yaml              -> 取消这行的注释
plugin.yaml=/opt/rh/ruby193/root/etc/mcollective/facts.yaml             -> 取消这行的注释

Mcollective只是一个客户端软件，不是一个服务，所以不用启动服务，只是当有人调用这个包时，就会启动
```

###### 保证httpd和ntpd服务开机启动

```
保证httpd和ntpd服务开机启动
# chkconfig --list httpd
# chkconfig httpd on
# service httpd status
# service httpd start
# chkconfig --list ntpd
# chkconfig ntpd on
```


### 安装Broker

*    安装软件包
*    配置Apache代理的ServerName
*    打开防火墙对应的网络端口
*    创建需要的RSA key
*    设置SELinux相关项目
*    设置Mongodb调用相关属性
*    设置动态域名注册更新相关配置
*    设置用户认证配置
*    允许RubyGems

```
确保mongod activemq httpd服务已经启动
# service mongod status
# service activemq status
# service httpd status

Mcollective只是一个客户端软件，不是一个服务，所以不用启动服务，只是当有人调用这个包时，就会启动
```


###### 安装Broker软件包

```
# yum install -y openshift-origin-broker\
openshift-origin-broker-util\
rubygem-openshift-origin-remote-user\
rubygem-openshift-origin-msg-broker-mcollective\
rubygem-openshift-origin-dns-update\
v8314\
http://classroom.exapmle.com/pub/materials/crudini-0.3-2.el6.noarch.rpm

```

###### 配置Apache代理Broker的访问

*    配置主机名为server0-a
*    配置文件： /etc/httpd/conf.d/000002_openshift_origin_broker_servername.conf
*    配置项：ServerName server0-a.example.com

```
# cd /etc/httpd/conf.d/
# ls
000002_openshift_origin_broker_proxy.conf
000002_openshift_origin_broker_servername.conf
mod_dnssd.conf
proxy_ajp.conf
README
ruby193-passenger.conf
ssl.conf
welcome.conf
# vim 000002_openshift_origin_broker_servername.conf
ServerName server0-a.example.com
```

###### 需要运行打开对应端口的访问权限

*    需要打开的访问端口： http(80) https(443) ssh(22)
*    打开防火墙对应端口命令 - lokkit --service=http --service=https --service=ssh

```
# lokkit --service=http --service=https --service=ssh
# cat /etc/sysconfig/iptables
# iptables -L -n
```

###### 为OpenShift创建SSL密钥对

*    后面的服务认证会用到

```
# openssl genrsa -out /etc/openshift/server_priv.pem 2048
# ls -l /etc/openshift/server_priv.pem
-rw-r--r--. 1   root    root    1675    Jan 06 08:48    /etc/openshift/server_priv.pem
# openssl rsa -in /etc/openshift/server_priv.pem -pubout > /etc/openshift/server_pub.pem        -> 通过私钥，提取出公钥
Writing RSA key
# ls -l /etc/openshift/server_p*
-rw-r--r--. 1   root    root    1675    Jan 06 08:48    /etc/openshift/server_priv.pem
-rw-r--r--. 1   root    root    1675    Jan 06 08:58    /etc/openshift/server_pub.pem
# chown apache.apache /etc/openshift/server_pub.pem         -> 公钥需要提供给别人使用，所以需要修改公钥，私钥自己本地root使用，所以不用修改
# chmod 640 /etc/openshift/server_pub.pem
```

###### 为Broker配置认证和会话随机密钥

```
# echo "SESSION_SECRET="$(openssl rand -hex 64) >> /etc/openshift/broker.conf
# echo "AUTH_SALT="$(openssl rand -base64 64) >> /etc/openshift/broker.conf
# vim /etc/openshift/broker.conf
SESSION_SECRET=...
AUTH_SALT=...
```

###### 创建Broker与Node通信的RSA密钥

```
# ssh-genkey -t rsa -b 2048 -N '' -f /root/.ssh/rsync_id_rsa
# ls /root/.ssh/
authorized_keys     rsync_id_rsa    rsync_id_rsa.pub
# cp /root/.ssh/rsync_id_rsa*  /etc/openshift/
```

###### 为Broker正常运行设置SELinux Boolean值

需要设置打开的值

*    httpd_unified
*    httpd_execmem
*    httpd_can_network_connect
*    httpd_can_network_relay
*    httpd_run_stickshift
*    named_write_master_zones
*    allow_ypbind


```
# setsebool -P httpd_unified=on \
httpd_execmem=on \
httpd_can_network_connect=on \
httpd_can_network_relay=on \
httpd_run_stickshift=on \
named_write_master_zones=on \
allow_ypbind=on \
```

###### 为Broker正常运行设置SELinux安全上下文

通过fixfiles工具调整预设

```
# fixfiles -R ruby193-rubygem-passenger restore
# fixfiles -R ruby193-mod_passenger restore
# restorecon -rv /var/run
# restorecon -rv /opt
```

###### 设置Mongodb调用相关属性

```
# vim /etc/openshift/broker.conf
CLOUD_DOMAIN=apps0.example.com
MONGO_USER=openshift
MONGO_PASSWORD=redhat
MONGO_DB=openshift_broker
```

###### 配置Broker的plugin(/etc/openshift/plugin.d)

*   远程用户认证plugin - openshift-origin-auth-remote-user.conf(.example)
*   远程消息调用plugin - openshift-origin-msg-broker-mcollective.conf(.example)
*   动态域名更新plugin - openshift-origin-dns-bind.conf(.example)

###### 配置openshift使用远程DNS提交和解析动态域名

*    配置文件位置：/etc/openshift/plugin.d
*    配置文件名：openshift-origin-dns-nsupdate.conf
*    模板配置文件名：openshift-origin-dns-nsupdate.conf.example

```
BIND_SERVER=动态更新DNS服务ip
BIND_PORT=动态更新DNS服务端口
BIND_ZONE=动态更新的基础域
BIND_KEYNAME=动态更新key信息名称
BIND_KEYVALUE=动态更新key信息内容
BIND_KEYALGORITHM=动态更新key加密方式
```

```
# cd  /etc/openshift/plugins.d
# ls
openshift-origin-auth-remote-user.conf.example
openshift-origin-dns-nsupdate.conf.example
openshift-origin-msg-broker-mcollective.conf.example
# 将conf.example文件改为conf
# ls
openshift-origin-auth-remote-user.conf
openshift-origin-dns-nsupdate.conf          -> 只需要修改dns文件
openshift-origin-msg-broker-mcollective.conf

获得实验环境预创建的BIND_KEYVALUE
# wget http://classroom.example.com/pub/ddnskeys/kapps0.example.com*.key    -> 复制文件最后一部分

# vim openshift-origin-dns-nsupdate.conf          -> 更新为使用classroom为动态更新域名服务器
BIND_SERVER="172.25.254.254"
BIND_PORT="53"
BIND_ZONE="apps0.example.com"
BIND_KEYNAME="apps0.example.com"
BIND_KEYVALUE="前面复制出来的key字段"
BIND_KEYALGORITHM="HMAC-MD5"

```


###### 配置基础认证（Apache basic auth）并添加测试用户

```
# cd /var/www/openshift/broker/httpd/conf.d
# ls
openshift-origin-auth-remote-user-basic.conf.sample
openshift-origin-auth-remote-user-kerberos.conf.sample
openshift-origin-auth-remote-user-ldap.conf.sample

# cp openshift-origin-auth-remote-user.conf.sample openshift-origin-auth-remote-user.conf

# htpasswd -c /etc/openshift/htpasswd demo      -> -c 创建文件
New password:
Re-type new password:
Adding password for user demo 
```


###### 允许RubyGems

*    Broker程序中使用了大量的RubyGems，我们需要建立缓冲告知Broker他们都在哪里

```
# oo-admin-broker-cache -c              -> 重新创建broker cache
Clearing broker cache
```

###### 启动服务

*    Broker启动基于两个服务httpd和openshift-broker

```
配置Broker开机启动
# chkconfig httpd on
# chkconfig openshift-broker on

启动Broker服务
# service httpd start
# service openshift-broker start

# service httpd restart
# service openshift-broker restart
```

###### 检测Broker安装状态

*    测试Broker配置策略  - oo-accept-broker -v
*    测试Broker REST API 工作状态   - curl -Ik https://localhost/broker/rest/api

```
# oo-accept-broker -v

# curl -Ik https://localhost/broker/rest/api
```
