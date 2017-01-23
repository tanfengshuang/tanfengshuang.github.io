---
layout: post
title:  "Configuring System Auditing(413-14)"
categories: Linux
tags: 413 
---

### Configuring System Auditing

###### About auditd

Linux内核有用日志记录事件的能力，比如记录系统调用和文件访问。然后管理员可以评审这些日志，确定可能存在的安全漏洞，比如失败的登录尝试，或者用户对系统文件不成功的访问, 这种功能称为Linux用户空间审计系统，在Red Hat Enterprise Linux 5及其之后版本中已经直接可用。当然老版本的Linux 也可以手工添加软件使用。

Linux在用户空间审计系统由auditd、audispd、auditctl、autrace、ausearch和aureport等应用程序组成。

审计后台auditd应用程序通过netlink机制从内核中接收审计消息，然后通过一个工作线程将审计消息写入到审计日志文件中，Linux的审核系统提供了一种记录系统安全信息的方法，为系统管理员在用户违反系统安全规则时提供及时的警告信息。内核其他线程通过内核审计API写入套接字缓冲区队列audit_skb_queue中，内核线程kauditd通过netlink机制将审计消息定向发送给用户控件的审计后台auditd的主线程，auditd主线程再通过事件队列将审计消息传给审计后台的写log文件线程，写入log文件。另一方面，审计后台还通过一个与套接字绑定的管道将审计消息发送给dispatcher应用程序。

图1是Linux audit架构示意图(实线代表数据流，虚线代表组件关之间的控制关系):

![sss](./audit.jpg)
![audit.jpg](http://images.51cto.com/files/uploadimg/20120521/1708200.jpg)

*    auditd： audit守护进程负责把内核产生的信息写入到硬盘上，这些信息是由应用程序和系统活动所触发产生的。audit守护进程如何启动取决于它的配置文件(/etc/sysconfig/auditd)。audit系统函数的启动受文件/etc/audit/auditd.conf的控制。在用户空间审计系统通过auditd后台进程接收内核审计系统传送来的审计信息，将信息写入到/var/log/audit/audit.log 中，audit.log的路径可在/etc/auditd.conf中指定。当auditd没有运行时，内核将审计信息传送给syslog，这些消息通常保存在/var/log/messages文件中，可以用dmesg命令查看。
*    auditctl： auditctl功能用来控制audit系统，它控制着生成日志的各种变量，以及内核审计的各种接口，还有决定跟踪哪些事件的规则。
*    audit.rules： 在/etc/audit/audit.rules中包含了一连串auditctl命令，这些命令在audit系统被启用的时候被立即加载。
*    aureport： aureport的功能是能够从审计日志里面提取并产生一个个性化的报告，这些日志报告很容易被脚本化，并能应用于各种应用程序之中，如去描述结果。
*    ausearch ： ausearch用于查询审计后台的日志，它能基于不同搜索规则的事件查询审计后台日志。每个系统调用进入内核空间运行时有个唯一的事件id，系统调用在进入内核后的运行过程的审计事件共享这个id。
*    audispd： audispd是消息分发的后台进程，用于将auditd后台发过来的一些消息通过syslog写入日志系统。这是一个审计调度进程，它可以将审计的信息转发给其它应用程序，而不是只能将审计日志写入硬盘上的审计日志文件之中。
*    autrace: 这个功能更总类似于strace，跟踪某一个进程，并将跟踪的结果写入日志文件之中。


要使用安全审计系统可采用下面的步骤：

*    安装软件包
*    了解配置文件
*    了解配置命令
*    添加审计规则和观察器来收集所需的数据
*    启用了内核中的audit并开始进行日志记录
*    通过生成审计报表和搜索日志来周期性地分析数据



###### Configuring auditd

*    auditd is the user-space component of the Linux auditing subsystem
*    auditctl can add auditing rules

*    /etc/sysconfig/auditd: startup options, normally parsed by the init script
*    /etc/audit/auditd.conf: main configuration file
*    /etc/audit/audit.rules: persistent auditing rules

*    service auditd status

```
# service auditd status
auditd (pid  1454) is running...
# chkconfig --list auditd
auditd         	0:off	1:off	2:on	3:on	4:on	5:on	6:off
# grep audit.log /etc/audit/auditd.conf 
log_file = /var/log/audit/audit.log
# tail -f /var/log/audit/audit.log
```

###### Open Lab: Verify auditd Operation


### Audit Reporting

###### Reading Audit Messages

> /var/log/audit/audit.log

###### Searching for Events

> ausearch

###### Reporting on Audit Messages

> aureport

```
# 
```

###### Remote Logging with auditd

There are 2 main ways to send audit messages to a remote system.both methods use custom Audit Dispatching with audispd. audispd is configured in /etc/audisp/audispd.conf, with plug-ins configured in /etc/audisp/plugins.d/*.conf


###### Open Lab: Audit Reporting


### Writing Custom Audit Rules

```
# auditctl -w /etc/passwd -p wa -k user-edit
# auditctl -w /bin -p x
# auditctl -l
```

###### Adding Rules

> auditctl -w /etc/passwd -p wa -k user-edit
> auditctl -w /bin -p x

###### Removing Rules

> auditctl -D

###### Inspecting Rules

> auditctl -l 

###### Immutable Rules

auditctl -s 

###### Persistent Rules

> /etc/audit/audit.rules

###### Using Predefined Audit Sets

/usr/share/doc/audit-*/*.rules
Copy to /etc/audit/audit.rules

### Log Ratation

*    service auditd rotate

A sample script that you can drop into /etc/cron.d/* is in /usr/share/doc/audit-*/audit.cron

### Unit Test: Implementing a Custom Audit Policy



