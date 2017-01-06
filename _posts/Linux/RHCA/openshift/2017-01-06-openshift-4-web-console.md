---
layout: post
title:  "Openshift web console 安装配置测试"
categories: Openshift
tags: openshift
---

### Openshift Web Console

*    作为用户Web工具可以方便管理人员在没有CLI或REST API的时候配置管理Openshift
*    软件包openshift-origin-console
*    默认使用Apache基础认证(basic auth)
*    需要安装配置好了Broker的机器上

```
# yum install -y openshift-origin-console
```

*    配置基础认证（Apache basic auth）

```
# ls /var/www/openshift/
broker      console
# cd /var/www/openshift/console/httpd/conf.d
# ls
ldap-user-sample.ldiff
openshift-origin-auth-remote-user-basic.conf.sample
openshift-origin-auth-remote-user-kerberos.conf.sample
openshift-origin-auth-remote-user-ldap.conf.sample
README-KERB
README-LDAP
# cp openshift-origin-auth-remote-user-basic.conf.sample openshift-origin-auth-remote-user.conf
# vim openshift-origin-auth-remote-user.conf
AuthUserFile    /etc/openshift/htpasswd         -> 使用/etc/openshift/htpasswd来做认证, htpasswd -c /etc/openshift/htpasswd demo
# cat /etc/openshift/htpasswd
demo:...
```

*    为保证登录的安全会话需要配置会话随机数

```
# vim /etc/openshift/console.conf
# echo "SESSION_SECRET="$(openssl rand -hex 64) >> /etc/openshift/console.conf
```

*    启动Openshift Web Console并测试

```
启动服务并保证开机自启动
# chkconfig openshift-console on 
# service openshift-console start

测试web连接
# firefox http://server0-a.example.com
demo:redhat
```
