---
layout: post
title:  "服务与计划任务"
categories: Linux
tags: RHCSA systemd systemctl at crontab ssh
---

关于linux的基本命令的详细使用可以通过网站 [http://man.linuxde.net/](http://man.linuxde.net/)查看

### systemd

*    RHEL7使用systemd替换了sysV, systemd目的是要取代Unix时代以来一直在使用的init系统，兼容SysV和LSB的启动脚本，而且能够在进程启动过程中更有效的引导加载服务
*    支持并行化服务
*    同时采用socket与D-Bus总线式激活服务
*    按需启动守护进程(daemon)
*    利用Linux的cgroups监视进程
*    支持快照和系统恢复
*    维护挂载点和自动挂载点
*    各服务间基于依赖关系进行精密控制

##### 单元

*    系统服务(.service)
*    挂载点(.mount)
*    sockets(.sockets)
*    系统设备(.device)
*    交换分区(.swap)
*    文件路径(.path)
*    启动目标(.target)
*    由systemd管理的计时器(.timer)

##### 目标(target)

0 关机                      poweroff.target     runlevel0.target
1 单用户                    rescue.target       runlevel1.target
2 字符多用户界面，无网络    multi-user.target   runlevel2.target
3 字符多用户界面，生产模式  multi-user.target   runlevel3.target
4 保留模式                  multi-user.target   runlevel4.target
5 图形化多用户模式          graphical.target    runlevel5.target
6 重启                      reboot.target       runlevel6.target


```
# systemctl start httpd
# systemctl stop httpd
# systemctl restart httpd
# systemctl reload httpd
# systemctl enable httpd
# systemctl disable httpd
# systemctl is-enable httpd
# systemctl mask httpd
# systemctl unmask httpd
# systemctl list-dependencies httpd
# systemctl list-unit-files --type service | grep httpd
```

```
查看默认的开机模式
# systemctl get-default
graphical.target

设置默认的开机模式
# systemctl set-fault graphical.target

切换到其他模式
# systemctl isolate multi-user.target
```

### ssh

##### 公钥私钥

```
# ssh-keygen
# ssh-copy-id
```

##### 配置文件

/etc/ssh/ssh_config     ssh客户端配置文件
/etc/ssh/sshd_config    ssh服务器配置文件

```
# vim /etc/ssh/sshd_config
PermitRootLogin yes     是否允许root登录
PasswordAuthentication yes  是否允许密码登录

密钥文件
# ll /etc/ssh/ssh_host*
-rw-r-----. 1 root ssh_keys  227 Oct 26 10:01 /etc/ssh/ssh_host_ecdsa_key
-rw-r--r--. 1 root root      162 Oct 26 10:01 /etc/ssh/ssh_host_ecdsa_key.pub
-rw-r-----. 1 root ssh_keys  387 Oct 26 10:01 /etc/ssh/ssh_host_ed25519_key
-rw-r--r--. 1 root root       82 Oct 26 10:01 /etc/ssh/ssh_host_ed25519_key.pub
-rw-r-----. 1 root ssh_keys 1675 Oct 26 10:01 /etc/ssh/ssh_host_rsa_key
-rw-r--r--. 1 root root      382 Oct 26 10:01 /etc/ssh/ssh_host_rsa_key.pub
```

### sz/rz

sz/rz 是基于ZModem传输协议的命令。对传输的数据会进行核查，并且有很好的传输性能。使用起来更是非常方便，但前提是window端需要有能够支持ZModem的telnet或者SSH客户端，例如secureCRT
```
# rpm -qf /usr/bin/rz
lrzsz-0.12.20-39.fc24.x86_64

# rpm -qf /usr/bin/sz
lrzsz-0.12.20-39.fc24.x86_64
```

### 计划任务

##### at

at <时间描述>
at> <任务描述>
at> <ctrl+d>

atq 查询当前用户正在等待的计划任务
atrm <任务号>   删除一个正在等待的计划任务

可以在配置文件/etc/at.deny中添加禁止执行at命令的用户

```
# at 6pm Monday
# at now + 5minutes
# at 13:50 05/12/2016   (mm/dd/yy（月/日/年）或dd.mm.yy（日.月.年）)

# atq
5   Tue Nov 29 17:37:00 2016 a root
6   Tue Nov 29 17:37:00 2016 a root

# atrm 6

# atq
5   Tue Nov 29 17:37:00 2016 a root 
```

##### crontab

*    crontab -e  编辑当前用户的计划任务时间表
*    crontab -l  列出当前的计划任务时间表
*    crontab -r  删除当前的计划任务时间表
*    crontab -u username <-e|-l|-r>  以某一个用户的身份管理
*    用户时间表文件: /var/spool/cron/username
*    可以在配置文件/etc/corn.deny中添加禁止执行crontab命令的用户
*    /etc下面的cron.daily/   cron.hourly/  cron.monthly/ cron.weekly/ 是系统的定时任务
*    man 5 crontab   查看crontab的时间配置帮助信息

```
时间配置格式:
  *       *       *      *          *         指令
每分钟  每小时  每天    每月    每周星期几  任务描述

时间数值的特殊表示方法
*   表示该范围内的任意时间
，  表示间隔的多个不连续时间
-   表示一个连续的时间范围
/   指定间隔的时间频率

例如，
0 17 * * 1-5    周一到周五每天17:00
30 8 * * 1,3,5  每周一，三，五的8点30分
0 8-18/2 * * *  8点到18点之间每隔2小时
0 * */3 * *     每隔三天

```





