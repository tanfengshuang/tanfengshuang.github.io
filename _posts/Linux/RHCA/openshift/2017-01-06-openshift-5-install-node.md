---
layout: post
title:  "Openshift安装Node结点"
categories: Openshift
tags: openshift
---

*    在新的服务器上部署Node节点（server0-b)
*    准备Node节点消息队列通信环境
*    安装Node节点所需软件包，并打开相应服务及防火墙配置
*    设置Node节点上PAM、Cgroups和资源配额
*    设置Node节点上SELinux和系统内核参数相关配置
*    配置Node节点上SSH服务、Openshift端口代理和其他必要服务
*    为Node节点添加Cartridge


### 在新的服务器上部署Node节点（server0-b)

*    首先启动serverb主机 - rht-vmctl start serverb
*    登录serverb主机 - ssh root@server0-b

### 准备Node节点消息队列通信环境

```
将broker上设置的SSH认证公钥同步到node上
[server0-a] # ssh-copy-id -i /etc/openshift/rsync_id_rsa.pub root@server0-b

安装node节点消息队列软件包
[server0-b] # yum install -y openshift-origin-msg-node-mcollective

导入node节点Mcollective配置文件
[server0-b] # cd /opt/rh/ruby193/root/etc/mcollective/
[server0-b] # rm -rf server.cfg
[server0-b] # wget http://classroom.example.com/pub/materials/server.cfg
[server0-b] # vim server.cfg
plugin.activemq.pool.1.host = server0-a.example.com
plugin.activemq.pool.1.password = redhat

启动并测试Node节点消息队列通信环境
1. 保证Node节点消息队列服务开机启动
[server0-b] # chkconfig ruby193-mcollective on
[server0-b] # service ruby193-mcollective start
2. 测试Node节点消息队列服务
[server0-a] # oo-mco ping
server0-b.example.com       time=XXXX ms
```

### 安装Node节点所需软件包，并打开相应服务及防火墙配置

需要在Node节点上安装的软件包

*    rubygem-openshift-origin-node
*    ruby193-rubygem-passenger-native
*    policycoreutils-python
*    openshift-origin-node-util
*    rubygem-openshift-origin-container-selinux
*    rubygem-openshift-origin-frontend-nodejs-websocket
*    rubygem-openshift-origin-frontend-apache-mod-rewrite

```
安装软件包
[server0-b] # yum install -y rubygem-openshift-origin-node \
ruby193-rubygem-passenger-native \
policycoreutils-python \
openshift-origin-node-util \
rubygem-openshift-origin-container-selinux \
rubygem-openshift-origin-frontend-nodejs-websocket \
rubygem-openshift-origin-frontend-apache-mod-rewrite \
http://classroom.example.com/pub/materials/crudini-0.3-2.el6.noarch.rpm

查找软件源中的Cartridge
[server0-b] # yum search openshift-origin-cartridge

安装后续实验需要的Cartridge
[server0-b] # yum install -y openshift-origin-cartridge-cron \
openshift-origin-cartridge-haproxy \
openshift-origin-cartridge-php \
openshift-origin-cartridge-mysql \
openshift-origin-cartridge-dependencies-recommended-php

配置防火墙规则，打开服务端口
[server0-b] # lokkit --service=ssh \
--service=http \
--service=https \
--port=8000:tcp \
--port=8443:tcp
[server0-b] # iptables -L -n

保证相关服务开机自动启动
[server0-b] # chkconfig httpd on 
[server0-b] # chkconfig --list network
[server0-b] # chkconfig network on
[server0-b] # chkconfig --list ntpd
[server0-b] # chkconfig ntpd on
[server0-b] # chkconfig --list sshd
[server0-b] # chkconfig sshd on
[server0-b] # chkconfig oddjobd on
[server0-b] # chkconfig openshift-node-web-proxy on

启动httpd服务   -> 其他服务暂时不要启动，需要配置完成后才能成功启动
[server0-b] # service httpd start
```


###  设置Node节点上PAM、Cgroups和资源配额

*    PAM(Pluggable Authentication Module)系统，执行用户鉴别和账户维护的服务
*    Cgroup(Linux Kernel Control Group), Linux内核提供的一种可以限制、记录、隔离进程组所使用的物理资源的机制
*    磁盘限额（Quota），Linux基于文件系统的磁盘限制，可以限制用户使用磁盘空间及创建文件的个数

###### 为Node节点设置PAM

*    将/etc/pam.d/ssh配置文件中的pam_selinux.so替换为pam_openshift.so
*    将pam_namespace相关设置加入到runuser、runuser-l、su、system-auth-ac和sshd的pam配置中

```
session [default=1 success=ignore]  pam_succeed_if.so   quiet   shell=/usr/bin/oo-trap-user
session required    pam_namespace.so    no_unmounts_on_close
```

###### pam_namespace相关设置

*    使用多实例化提高安全性的方法
*    文档位置/usr/share/doc/pam-*/txts/README.pam_namespace
*    在/etc/security/namespace.d/tmp.conf中加入

```
/tmp    $HOME/tmp   user:iscript=/usr/sbin/oo-namespace-init    root,adm
```

*    在/etc/security/namespace.d/shm.conf中加入

```
/dev/shm    tmpfs   tmpfs:mntopts=size=5M:iscript=/usr/sbin/oo-namespace-init    root,adm

格式说明：
多实例化目录    新的实例化基础目录或位置        用户/上下文/级别        非实例化用户列表
```


```
[server0-b] # sed -i 's/pam_selinux/pam_openshift/g' /etc/pam.d/sshd
[server0-b] # vim /etc/pam.d/runuser
[server0-b] # vim /etc/pam.d/runuser-l
[server0-b] # vim /etc/pam.d/su
[server0-b] # vim /etc/pam.d/system-auth-ac
[server0-b] # vim /etc/pam.d/sshd
session [default=1 success=ignore]  pam_succeed_if.so   quiet   shell=/usr/bin/oo-trap-user
session required    pam_namespace.so    no_unmounts_on_close

[server0-b] # vim /usr/share/doc/pam-1.1.1/txts/README.pam_namespace
[server0-b] # vim /etc/security/namespace.d/tmp.conf
/tmp    $HOME/tmp   user:iscript=/usr/sbin/oo-namespace-init    root,adm
[server0-b] # vim /etc/security/namespace.d/shm.conf
/dev/shm    tmpfs   tmpfs:mntopts=size=5M:iscript=/usr/sbin/oo-namespace-init    root,adm
```

###### 设置Cgroup相关配置

*    Openshift中是通过Cgroup来限制gear中的资源使用的，默认在rubygem-openshift-origin-node包中有配置cgroup的例子文件

```
[server0-b] # cp /opt/rh/ruby193/root/usr/share/gems/doc/openshift-origin-node-*/cgconfig.conf /etc/cgconfig.conf
```

*    更新PAM配置，添加pam_cgroup支持

```
在runuser、runuser-l、system-auth-ac和sshd的pam配置文件中加入
session     optional    pam_cgroup.so
```

```
[server0-b] # cp /opt/rh/ruby193/root/usr/share/gems/doc/openshift-origin-node-1.23.9.21/cgconfig.conf /etc/
[server0-b] # vim /etc/pam.d/runuser
[server0-b] # vim /etc/pam.d/runuser-l
[server0-b] # vim /etc/pam.d/system-auth-ac
[server0-b] # vim /etc/pam.d/sshd
session     optional    pam_cgroup.so
```


*    重置Cgroup相关文件和目录的SELinux权限

```
[server0-b] # restorecon -v /etc/cgconfig.conf
[server0-b] # restorecon -v /etc/cgrules.conf
[server0-b] # restorecon -v /cgroup
```

*    配置Cgroup相关服务开启启动

```
[server0-b] # chkconfig cgconfig on
[server0-b] # chkconfig cgred on
[server0-b] # service cgconfig start
[server0-b] # service cgred start
```

###### 为Node节点设置磁盘限额（Quota）

*    Openshift通过/etc/openshift/resource_limits.conf文件配置其对gear的资源限制，其中quota_files设置打开文件数(inode), quota_blocks设置数据容量存储大小（KiB）
*    Node端的存储资源限制就由本地Quota提供

###### 配置Node本地Quota

```
编辑/etc/fstab文件添加usrquota文件系统参数
[server0-b] # vim /etc/fstab
UUID=...    ext4    defaults,usrquota   1   1

重新挂接磁盘使usrquota文件生效
[server0-b] # mount | grep '/dev/vda1'
/dev/vda1   on  /   type    ext4    (rw)
[server0-b] # mount -o remount /
[server0-b] # mount | grep '/dev/vda1'
/dev/vda1   on  /   type    ext4    (rw,usrquota)

检查并生成quota统计文件
[server0-b] # quotacheck -cugm /        -> -c 创建，-u 用户，-g 组，-m 自动,必须加参数m，因为根目录不能umount

确认aquota.user文件SELinux安全上下文正确
[server0-b] # ls -lZ /aquota.user
[server0-b] # restorecon /aquota.user
[server0-b] # ls -lZ /aquota.user

使磁盘限额生效
[server0-b] # quotaon /

[server0-b] # repquota      -> 查看
```


### 设置Node节点上SELinux和系统内核参数相关配置

*    打开Node节点上SELinux的Boolean
*    设置Node节点上SELinux的安装上下文
*    设置Node节点上的内核网络及内存参数


```
打开Node节点上SELinux的Boolean
[server0-b] # setsebool -P httpd_unified=on \
httpd_can_network_connect=on \
httpd_can_network_delay=on \
httpd_run_stickshift=on \
httpd_enable_homedirs=on \      -> openshift上的虚拟用户能查看自己的家目录
allow_polyinstantion=on

设置Node节点上SELinux的安装上下文
[server0-b] # restorecon -rv /var/run
[server0-b] # restorecon -rv /var/lib/openshift /etc/openshift/node.conf /etc/httpd/conf.d/openshift

设置Node节点上的内核网络及内存参数
[server0-b] # vim /etc/sysctl.conf
kernel.sem = 250 32000 32 4096  -> 每个进程上最多信号量个数   系统总信号量个数    。    。
net.ipv4.ip_local_port_range = 15000 35530
net.netfilter.nf_conntrac_max = 1048576
net.ipv4.ip_forward = 1
net.ipv4.conf.all.route_localnet = 1
[server0-b] # sysctl -p         -> 让参数生效

```


### 配置Node节点上SSH服务、Openshift端口代理和其他必要服务

*    配置SSH服务使其运行Git访问，并设置最大连接数和最大尝试连接数
*    配置Openshift端口代理相关iptables配置，并启动相应服务
*    配置Node节点相关的配置(/etc/openshift/node.conf)
*    配置上传Facter数据到broker

###### 配置SSH服务使其运行Git访问, 并设置最大连接数和最大尝试连接数

```
添加GIT_SSH项到Node节点的SSH服务端配置文件中
[server0-b] # echo -e "\nAcceptEnv GIT_SSH" >> /etc/ssh/sshd_config

设置Node节点的SSH服务最大在线连接数和最大尝试连接数
[server0-b] # echo -e "MaxSession 40"   >> /etc/ssh/sshd_config
[server0-b] # echo -e "MaxStartups 40"   >> /etc/ssh/sshd_config
```

###### 配置Openshift端口代理相关iptables配置，并启动相应服务

```
创建自定义链，并将接收的数据包导向自定义链
[server0-b] # vim /etc/sysconfig/iptables
:rhc-app-comm   -   [0:0]
-A INPUT -j rhc-app-comm
[server0-b] # service iptables restart
[server0-b] # chkconfig iptables on

设置开机启动Node节点上的相关服务
1. 配置并启动openshift-iptalbes-port-proxy服务
[server0-b] # chkconfig openshift-iptalbes-port-proxy on
[server0-b] # service openshift-iptalbes-port-proxy start
2. 配置并启动openshift-gear服务
[server0-b] # chkconfig openshift-gear on
[server0-b] # service openshift-gear start

配置Node节点相关配置(/etc/openshift/node.conf)
[server0-b] # vim /etc/openshift/node.conf
PUBLIC_HOSTNAME = server0-b.example.com
PUBLIC_IP = 172.25.0.11
BROKER_HOST = server0-a.example.com
CLOUD_DOMAIN = apps0.example.com

配置Node节点相关的env参数文件
[server0-b] # vim /etc/openshift/env/OPENSHIFT_BROKER_HOST
server0-a.example.com
[server0-b] # vim /etc/openshift/env/OPENSHIFT_CLOUD_DOMAIN
apps0.example.com

配置Node节点Web服务主机名称
[server0-b] # vim /etc/httpd/conf.d/00001_openshift_origin_node_servername.conf
ServerName  server0-b.example.com

确认配置上传Facter数据到broker
1. 配置文件位置
[server0-b] # vim /opt/rh/ruby193/root/etc/mcollective/facts.yaml
2. 确认计划人物更新脚本
[server0-b] # ls /etc/cron.minutely/openshift-facts
3. 执行更新上传Facter数据到broker
[server0-b] # /etc/cron.minutely/openshift-facts

重启并测试Node节点
1. 重启Node节点使相关服务和配置脚本有序启动
[server0-b] # reboot
2. 测试Node节点安装运行是否正确
[server0-b] # oo-accept-node -v
```

### 添加新的Cartridge

*    Cartridge就是Openshift的应用资源库
*    当Broker和Node分开部署时，Cartridge安装在Node节点上
*    当Node节点为多机状态时，每台Node节点上的Cartridge需要保持一致（包括数量和版本）
*    Node节点上新增Cartridge需要确认，并在Broker上激活确认
*    Broker上的Cartridge信息缓冲默认为6小时，你需要手动刷新以求更早的可用

###### 在Node节点上安装并更新Cartridge信息

*    安装diy和postgresql两个Cartridge
*    更新Node节点本地Cartridge信息

```
[server0-b] # yum search openshift-origin-cartridge
[server0-b] # yum list openshift-origin-cartridge*
[server0-b] # yum install -y openshift-origin-cartridge-{diy,postgresql}
[server0-b] # ls /usr/libexec/openshift/cartridge/
cron    diy     haproxy     mysql   php     postgresql
[server0-b] # oo-admin-cartridge -a install -s /usr/libexec/openshift/cartridge/diy
succeeded
[server0-b] # oo-admin-cartridge -a install -s /usr/libexec/openshift/cartridge/postgresql
succeeded
```

###### 在Broker节点上确认Node节点上Cartridge的更新

*    查看目前Broker节点上cartridge信息
*    更新Broker节点上cartridge信息

```
[server0-a] # oo-admin-ctl-cartridge -c list
[server0-a] # oo-admin-ctl-cartridge -c import-node --activate
```

###### 手动更新Broker信息缓冲

*    操作可能会根据机器性能延迟几分钟

```
[server0-a] # oo-admin-console-cartridge --clear
Clearing clonsole cache.
```

###### 在Openshift Web Console中验证更新

*    打开浏览器，访问http://server0-a.example.com

```
# ssh -X root@server0-a
[server0-a] # firefox http://server0-a.example.com
demo:redhat
```

###### 重启并测试Node节点

*    重启Node节点使相关服务和配置脚本有序启动
*    测试Node节点安装运行是否正常

```
[server0-b] # reboot
[server0-b] # oo-accept-node -v
find district uuid: NONE
1 ERRORS            -> 配置完后面一章这个错误即可消
```









