---
layout: post
title:  "使用Watchman获得运行指标"
categories: Openshift
tags: openshift
---

*    Watchman是一个Node节点上的应用
*    Watchman是在OpenShift Enterprise 2.1中引入的
*    Watchman通过Plugin的方式支持Syslog、JBoss、Gear State、Throttler、Metric和Out-of-Memory(OOM)
*    Watchman默认将指标数据写入messages文件


### 安装Watchman

*    默认安装node节点的时候，Watchman就已经安装了
*    Watchman属于openshift-origin-node-util软件包

```
允许Watchman开机运行
chkconfig openshift-watchman on

开启Watchman服务
service openshift-watchman start
```

### Watchman目前支持的插件

*    插件由Ruby写成，保存在Node节点/etc/openshift/watchman/plugins.d/目录下（后缀为rb为运行使用，后缀disabled为未开启使用）
*    修改插件后，需要重新启动openshift-watchman服务 - service openshift-watchman restart

```
# ssh root@server0-b
[root@server0-b] # cd /etc/openshift/watchman/plugins.d/
[root@server0-b] # ls
env_plugin.rb           metrics_plugin.rb   syslog_plugin.rb.disabled   
gear_state_plugin.rb    monitored_gear.rb   throttler_plugin.rb
jboss_plugin.rb         oom_plugin.rb
[root@server0-b] # service openshift-watchman status
Watchman is not running
[root@server0-b] # grep watchman /var/log/messages          -> 没有任何关于watchman的log
[root@server0-b] # service openshift-watchman start
[root@server0-b] # grep watchman /var/log/messages
```

### 允许Watchman监控Node节点相关系统

*    设置node.conf配置文件使watchman有效

```
[root@server0-b] # vim /etc/openshift/node.conf
WATCHMAN_METRICS_ENABLED=true
```

*    默认日志保存在文件 /var/log/openshift/node/platform.log 中
*    将watchman日志保持在Syslog中，type为metric

```
[root@server0-b] # mv syslog_plugin.rb.disabled syslog_plugin.rb

```

### 配置Watchman

```
# ssh root@server0-b
[root@server0-b] # chkconfig openshift-watchman on
[root@server0-b] # service openshift-watchman start

[root@server0-b] # vim /etc/openshift/node.conf
WATCHMAN_METRICS_ENABLED=true
WATCHMAN_METRICS_INTERVAL=30

METRICS_METADATA="appName:OPENSHIFT_APP_NAME,gearUuid:OPENSHIFT_GEAR_UUID"
CGROUPS_METRICS_KEYS="cpu.stat,cpuacct.stat,memory.usage_in_bytes"
MAX_CGROUPS_METRICS_MESSAGE_LENGTH=1024
METRICS_PER_GEAR_TIMEOUT=3
METRICS_PER_SCRIPT_TIMEOUT=1
METRICS_MAX_LINE_LENGTH=2000

[root@server0-b] # vim /etc/sysconfig/watchman
RETRY_DELAY=60
RETRY_PERIOD=60
GEAR_RETRIES=10
STATE_CHAGE_DELAY=60

[root@server0-b] # cd /etc/openshift/watchman/plugins.d/
[root@server0-b] # mv syslog_plugin.rb.disabled syslog_plugin.rb
[root@server0-b] # service openshift-watchman restart
[root@server0-b] # grep --color -i watchman /var/log/message
```
