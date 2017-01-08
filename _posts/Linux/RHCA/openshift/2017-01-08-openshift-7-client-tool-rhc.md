---
layout: post
title:  "Openshift客户端应用"
categories: Openshift
tags: openshift
---

*    OpenShift命令行客户端工具的安装频道 - rhel-6-server-ose-2.1-rhc-rpms
*    OpenShift命令行客户端工具的软件包名 - rhc(rubygem-rhc)
*    OpenShift命令行客户端工具的本地配置文件 - ~/.openshift/express.conf

### 完成本地OpenShift客户端环境部署

*    安装RHC客户端工具
*    使用普通用户部署客户端环境
*    使用OpenShift用户demo登录
*    查看客户端配置文件 ~/.openshift/express.conf

```

[server0-a] # yum install -y rhc

[student@server0-a] $ rhc setup
定义域名的namespace,  such as ose

apps0.example.com CLOUD_DOMAIN
AppName-Namespace.CLOUD_DOMAIN

[student@server0-a] $ cd ~/.openshift/
[student@server0-a] $ ls
express.conf token_XXXXX
[student@server0-a] $ cat token_XXXXX
XXXXXXXXXXXXXXXXXXXXXX

[student@server0-a] $ cat express.conf
default_rhlogin=demo
```

### RHC工具常用命令

*    显示用户状态信息   -   rhc account
*    设置应用独立域名   -   rhc alias
*    列出所有已创建应用 -   rhc apps
*    创建和管理应用     -   rhc app
*    配置认证token      -   rhc authorization
*    管理应用cartridge  -   rhc cartridge
*    管理部署           -   rhc deployment
*    配置应用域名       -   rhc domain
*    应用环境变量配置   -   rhc env
*    远端应用导入本地   -   rhc git-clone
*    登出应用管理       -   rhc logout
*    域权限管理         -   rhc member/members/member-list
*    查看服务状态       -   rhc server
*    管理应用快照       -   rhc snapshot
*    SSH到远端应用      -   rhc ssh
*    实时查看应用日志   -   rhc tail


```
[student@server0-a] $ rhc account

kevinphp.kissingwolf.com    CNAME   kevinphp-ose.apps0.example.com


[student@server0-a] $ rhc members ose
```

