---
layout: post
title:  "Monitoring Red Hat Gluster Storage(236-14)"
categories: Linux
tags: RHCA 236
---

### 通过 Nagios 监控红帽 Gluster 存储

###### 通过 Nagios 监控红帽 Gluster 存储

红帽 Gluster 控制台实施可对红帽 Gluster 存储池进行监控的功能。红帽控制台通过利用 Gluster 插件，扩展 Nagios 平台的功能。安装 Nagios 且配置为监控 Gluster 基础架构时，可以在趋势标签页中查看 CPU、内存、交换内存、磁盘、网络、卷、brick 和群集的状态。在配置 Nagios 监控前，必须将 Gluster 节点添加到红帽 Gluster 控制台中。 

1. Nagios

Nagios 是一款开源的网络、计算机系统和基础架构监控软件应用。在出现故障时，Nagios 可以向管理员提供问题警告，让他们能够在停机故障影响到业务流程、最终用户或客户之前开始补救流程。Nagios 利用对被监控资源执行的主动和被动检查来实现上述目标。主动检查是通过 shell 脚本定义的检查，这些脚本由 Nagios 直接执行。它们按照定义的间隔调度，并放入调度程序中。只有线程可用时，才会执行检查。被动检查是由 Nagios 或其他外部机制触发的检查，但不在 Nagios 服务器上主动运行。不论是主动检查还是被动检查，Nagios 都会分析检查结果，确定是否有任何事件满足警告定义，在 Nagios 服务器上标记警告，并相应地发出配置的通知（电子邮件、SNMP 陷阱）。 

2. Nagios Remote Plugin Executor (NRPE)

借助 Nagios Remote Plugin Executor (NRPE)，可以在远程 Linux 系统上执行插件。这使得 Nagios 能够监控远程系统上内存等本地资源的使用情况。任何要被监控的系统上都必须配置并运行 NRPE 服务。NRPE 由两个部分组成：

* NRPE 守护进程，它在远程 Gluster 存储节点上运行。
* check_nrpe 插件，它驻留于红帽 Gluster 控制台。

Nagios 通过以下流程监控远程系统的资源。

*    Nagios 首先执行 check_nrpe，并告知它需要检查什么服务。
*    check_nrpe 插件联系远程系统上的 NRPE 守护进程。
*    NRPE 守护进程使用适当的 Nagios 插件检查资源或服务。
*    check_nrpe 插件收到来自 NRPE 守护进程的服务检查结果，并将检查结果发送到 Nagios 服务器。 

3. nrpe.cfg

nrpe.cfg 文件含有多个广泛使用的命令定义，以监控远程系统。这些命令定义将在远程系统上运行。创建的命令定义用于定义 NRPE 将在远程系统上用于监控本地资源和服务的命令。NRPE 文件也包含访问权限指令。指令 allowed_hosts 用于定义监控服务器的 IP 地址或主机名称。必须启用此访问权限，然后才能运行自动发现。若无此访问权限，NRPE 将无法使用插件，系统也将无法由 Nagios 监控。在修改该文件后，需要重启相关的服务。

[root@demo-a ~]$ grep 'allowed_hosts' /etc/nagios/nrpe.cfg
allowed_hosts=127.0.0.1,NagiosServerIP or Hostname
  
可以通过手动运行 check_nrpe，获取 Nagios 服务器和运行 NRPE 的节点之间的通信确认。该命令使用选项 -H 指向远程系统，并使用 -c 指定要运行的命令。必须使用二进制文件的完整路径。例如，若要验证 Nagios 服务器和运行 NRPE 的节点之间的通信，可按以下操作：

[root@demo-a ~]$ /usr/lib64/nagios/plugins/check_nrpe -H RemoteNodeIP -c 'check_procs'
PROCS OK: 147 processes

###### 配置 Nagios

Nagios 提供的 Python 脚本可以发现群集中的所有节点、卷和 brick，不必手动导入现有的红帽 Gluster 存储基础架构。运行 configure-gluster-nagios 即可调用该脚本，它将创建监控所发现资源需要的配置文件。默认情况下，该脚本每 24 小时运行一次，以同步来自红帽 Gluster 存储池配置中的 Nagios 配置。必须使用群集名称和 Nagios 服务器主机地址手动运行一次命令 configure-gluster-nagios。

[root@demo-a ~]$ configure-gluster-nagios -c cluster-name -H Hostname or IP address
  

该命令以交互方式提示您：

*    确认配置。
*    输入 Nagios 服务器主机名称或 IP 地址。
*    确认重启 Nagios 服务器以开始监控。

1. 对象

在 Nagios 中，对象被视为监控和通知逻辑中涉及的要素。配置文件和目录用来定义对象。主机、服务、主机组、联系人、联系人组和命令是对象，它们在对象定义文件中定义。Nagios 使用许多必须专门为红帽 Gluster 存储配置的文件，运行用于自动发现的 configure-gluster-nagios 可进行此配置。尽管文件有许多，本课程着重介绍自动发现过程中配置的文件。

2. 命令定义

    命令定义文件用于定义命令。常见的定义包括服务检查、服务通知、服务事件处理程序、主机检查、主机通知和主机事件处理程序。在下例中，command_name 是用于标识命令的短名称。一个定义中的指令可以在另一定义中引用。通常，command_name 在联系人、主机和服务定义中引用。指令 command_line 指定 Nagios 为服务检查、主机检查、通知和事件处理程序执行的确切命令。

    [root@demo-a ~]$ cat /etc/nagios/gluster/gluster-commands.cfg
            define command{
    command_name    check_nrpe
    command_line     $USER1$/check_nrpe -H $HOSTADDRESS$ -c $ARG1$
    }
            

3. 主机定义

    主机定义可以定义设备、工作站和物理服务器。在下例中，use gluster-host 允许主机定义继承 gluster-host 中包含的值。hostgroups 指令定义主机组的短名称。alias 指令通常用于定义长名称或描述。host_name 指令是用于标识服务器的短名称。_HOST_UUID 指令是主机的 128 位唯一标识符。address 指令指定主机的 IP 地址。

    [root@demo-a ~]$
      cat /etc/nagios/gluster/gluster-cluster/demo-a
    define host {
    use               gluster-host
    hostgroups        gluster_hosts,gluster-cluster
    alias             servera.lab.example.com
    host_name         servera.lab.example.com
    _HOST_UUID        e8198e53-03f9-4ec7-9dfc-135c1bd40d98
    address           172.25.250.10

    }

4. 服务定义

    主机上运行的服务通过服务定义进行定义。服务可以是 POP、SMTP 和 HTTP 等典型服务。服务也可以是与某一主机关联的特定指标。例如，对 PING 的响应被视为一个服务，甚至一个卷。在下例输出中，use brick-service 允许主机定义继承 brick-service 中包含的值。_VOL_NAME 指定卷名称。指令的“notes”定义与服务相关的备注字符串，此备注在查看设备信息时显示。_BRICK_DIR 指定主机的 brick 的路径。

    [root@demo-a ~]$ cat /etc/nagios/gluster/demo-cluster/demo-a.cfg
           define service {
             use                            brick-service
             _VOL_NAME                      demo-vol
             __GENERATED_BY_AUTOCONFIG      1
             notes                          Volume : demo-vol
             host_name                      demo-a.lab.example.com
             _BRICK_DIR                     /bricks/brick-a1/brick
             service_description            Brick Utilization - /bricks/brick-a1/brick

###### 配置 Nagios 服务器 Sendmail 通知

Nagios 提供多种警报通知发送方式，如电子邮件、寻呼机、电话、即时消息和音频等。所选的通知方式应当适用于环境。通知的发送方式取决于对象定义文件中定义的通知命令。contact-group 为每一主机和服务进行定义，它包含一个或多个联系人。contact-group 指令指定哪些组将接收某特定服务或主机的通知。当 Nagios 发送服务和主机通知时，联系人组中的成员将收到通知。

1. 联系人定义

    联系人定义标识在发生问题时应当要获得通知的个人。在下例中，指令 service-notification_period 定义可以发送服务通知的时间段。service_notification_options 指令定义服务必须处于什么状态，以确保通知发送给联系人。service-notification-commands 指令用于列出通知联系人的命令的短名称列表。后面三个前缀为 host 的指令遵循与前缀为 service 的指令相同的逻辑，但它们影响主机通知。

    [root@demo-a ~]$ cat /etc/nagios/gluster/gluster-contacts.cfg
              define contact {
           contact_name                  student
           alias                         rh236
           email                         student@demo.lab.example.com
           service_notification_period   24x7
           service_notification_options  w,u,c,r,f,s
           service_notification_commands notify-service-by-email
           host_notification_period      24x7
           host_notification_options     d,u,r,f,s
           host_notification_commands    notify-host-by-email
    }
            
2. 联系人组定义

    联系人组定义用于将一个或多个联系人分组到一起，以向其发送警报和恢复通知。在下例中，contactgroup_name 标识联系人组。alias 指令指定用于标识组的长名称或描述。members 指令用于定义属于该组的联系人列表。

    [root@demo-a ~]$ cat /etc/nagios/gluster/gluster-contacts.cfg
          define contactgroup{
    	contactgroup_name		novell-admins
    	alias			Red Hat Administrators
    	members			james,john,sam
    	}
            
3. 通过电子邮件通知服务或主机。

    在 /etc/nagios/objects/commands.cfg 中定义 notify-service-by-email 和 notify-host-by-email 并指定要运行的命令，以分别用于服务通知和主机通知。若要在每次自动配置脚本运行时收到更新通知，需要将 $NOTIFICATIONCOMMENT$\n 添加到这两个定义的 | /bin/mail 后面。

    [root@demo-a ~]$ cat /etc/nagios/objects/commands.cfg
        define command{
            command_name    notify-host-by-email
            command_line    /usr/bin/printf "%b" "***** Nagios
            *****\n\nNotification Type: $NOTIFICATIONTYPE$\nHost: $HOSTNAME$\nState: $HOSTSTATE$\nAddress: $HOSTADDRESS$\nInfo: $HOSTOUTPUT$\n\nDate/Time: $LONGDATETIME$\n" $NOTIFICATIONCOMMENT$\n | /bin/mail -s "** $NOTIFICATIONTYPE$ Host Alert: $HOSTNAME$ is $HOSTSTATE$ **" $CONTACTEMAIL$
            }

    # 'notify-service-by-email' command definition
    define command{
            command_name    notify-service-by-email
            command_line    /usr/bin/printf "%b" "***** Nagios
            *****\n\nNotification Type: $NOTIFICATIONTYPE$\n\nService: $SERVICEDESC$\nHost: $HOSTALIAS$\nAddress: $HOSTADDRESS$\nState: $SERVICESTATE$\n\nDate/Time: $LONGDATETIME$\n\nAdditional Info:\n\n$SERVICEOUTPUT$\n" $NOTIFICATIONCOMMENT$\n | /bin/mail -s "** $NOTIFICATIONTYPE$ Service Alert: $HOSTALIAS$/$SERVICEDESC$ is $SERVICESTATE$ **" $CONTACTEMAIL$
            }
            

    1. 配置和验证 NRPE。
    2. 通过 configure-gluster-nagios 执行自动发现。
    3. 配置 Gluster 联系人。
    4. 配置电子邮件通知。
    5. 重启 Nagios 服务。


### 引导式练习：通过 Nagios 监控红帽 Gluster 存储

监控 servera、serverb 和受信存储池。

1. 在 servera 上为 NRPE 配置防火墙。

    a. 从 servera，打开防火墙端口 5666 TCP，并且使它在重新引导后保持打开。

    [root@servera ~]$ firewall-cmd --permanent --add-port=5666/tcp
    [root@servera ~]$ firewall-cmd --reload

2. 在 serverb 上为 NRPE 配置防火墙。

    a. 从 serverb，打开防火墙端口 5666 TCP，并且使它在重新引导后保持打开。

    [root@serverb ~]$ firewall-cmd --permanent --add-port=5666/tcp
    [root@serverb ~]$ firewall-cmd --reload

3. 配置 servera 和 serverb 以允许访问 NRPE。

    a. 从 servera ，将 manager.lab.example.com 添加到 /etc/nagios/nrpe.cfg 中的指令 allowed_hosts。

    [root@servera ~]$ cat /etc/nagios/nrpe.cfg
    ...
    allowed_hosts=127.0.0.1,manager.lab.example.com
    ...

    b. 在 servera 上重启 nrpe。

    [root@servera ~]$ systemctl restart nrpe

    c. 从 serverb，将 manager.lab.example.com 添加到 /etc/nagios/nrpe.cfg 中的指令 allowed_hosts。

    [root@serverb ~]$ cat /etc/nagios/nrpe.cfg
    ...
    allowed_hosts=127.0.0.1,manager.lab.example.com
    ...

    d. 在 serverb 上重启 nrpe。

    [root@serverb ~]$ systemctl restart nrpe

4. 配置 Nagios 以监控 gluster-cluster。

    a. 从 workstation，使用 SSH 连接 manager，并运行命令 configure-gluster-nagios 将群集导入到 Nagios。

    [root@manager ~]# configure-gluster-nagios -c gluster-cluster -H servera.lab.example.com
    Cluster configurations changed

    Changes :
    Hostgroup cluster - ADD
    Host cluster - ADD
             Service - Volume Utilization - prod-vol -ADD
             Service - Volume Self-Heal - prod-vol -ADD
             Service - Volume Status - prod-vol -ADD
             Service - Cluster Utilization -ADD
             Service - Cluster - Quorum -ADD
             Service - Cluster Auto Config -ADD
    Host servera.lab.example.com - ADD
             Service - Brick Utilization - /bricks/brick-a1/brick -ADD
             Service - Brick - /bricks/brick-a1/brick -ADD
    Host serverb.lab..example.com - ADD
             Service - Brick Utilization - /bricks/brick-b1/brick -ADD
             Service - Brick - /bricks/brick-b1/brick -ADD
    Are you sure, you want to commit the changes? (Yes, No) [Yes]: Enter
    Enter Nagios server address [manager.lab.example.com]: Enter
    Cluster configurations synced successfully from host servera.lab.example.com
    Do you want to restart Nagios to start monitoring newly discovered entities? (Yes, No) [Yes]: Enter
    Nagios re-started successfully.

5. 验证 Nagios 配置。

    a. 使用 -v 选项对 /etc/nagios/nagios.cfg 执行命令 nagios，以验证 nagios 配置。

    [root@manager ~]$ nagios -v /etc/nagios/nagios.cfg
    	Nagios Core 3.5.1
    Copyright (c) 2009-2011 Nagios Core Development Team and Community Contributors
    Copyright (c) 1999-2009 Ethan Galstad
    Last Modified: 08-30-2013
    License: GPL

    ...
    Total Warnings: 13
    Total Errors:   0

    Things look okay - No serious problems were detected during the pre-flight check.

6. 配置 nagios，从而发送 glusterd 的电子邮件通知到用户 student。

    a. 修改 /etc/nagios/gluster/gluster-contacts.cfg 中的 contact_name、alias 和 email 指令，使它们分别反映 student、student 和 student@manager.lab.example.com。

    [root@manager ~]$ cat /etc/nagios/gluster/gluster-contacts.cfg
    define contact {
           contact_name                  student
           alias                         student
           email                         student@manager.lab.example.com
           service_notification_period   24x7
           service_notification_options  w,u,c,r,f,s
           service_notification_commands notify-service-by-email
           host_notification_period      24x7
           host_notification_options     d,u,r,f,s
           host_notification_commands    notify-host-by-email
    }
                

    b. 在 /etc/nagios/gluster/gluster-templates.cfg 中，为 gluster-service 和 gluster-generic-host 添加联系人名称 student。

    [root@manager ~]$ cat /etc/nagios/gluster/gluster-templates.cfg
    define host{
       name                         gluster-generic-host
       use                          linux-server
       notifications_enabled        1
       notification_period          24x7
       notification_interval        120
       notification_options         d,u,r,f,s
       register                     0
       contacts                     +snmp,student
    }

    define service {
       name                         gluster-service
       use                          generic-service
       notifications_enabled       1
       notification_period          24x7
       notification_options         w,u,c,r,f,s
       notification_interval        120
       register                     0
       contacts                     +snmp,student
       _gluster_entity              Service
    }

    c. 将 $NOTIFICATIONCOMMENT$\n 直接添加到 /etc/nagios/objects/commands.cfg 中 | /bin/mail -s 的前面，以用于 notify-service-by-email 和 notify-host-by-email 定义。

    [root@manager ~]$ cat /etc/nagios/objects/commands.cfg
    define command{
            command_name    notify-host-by-email
            command_line    /usr/bin/printf "%b" "***** Nagios *****\n\nNotification Type: $NOTIFICATIONTYPE$\nHost: $HOSTNAME$\nState: $HOSTSTATE$\nAddress: $HOSTADDRESS$\nInfo: $HOSTOUTPUT$\n\nDate/Time: $LONGDATETIME$\n $NOTIFICATIONCOMMENT$\n" | /bin/mail -s "** $NOTIFICATIONTYPE$ Host Alert: $HOSTNAME$ is $HOSTSTATE$ **" $CONTACTEMAIL$
            }

    # 'notify-service-by-email' command definition
    define command{
            command_name    notify-service-by-email
            command_line    /usr/bin/printf "%b" "***** Nagios *****\n\nNotification Type: $NOTIFICATIONTYPE$\n\nService: $SERVICEDESC$\nHost: $HOSTALIAS$\nAddress: $HOSTADDRESS$\nState: $SERVICESTATE$\n\nDate/Time: $LONGDATETIME$\n\nAdditional Info:\n\n$SERVICEOUTPUT$\n $NOTIFICATIONCOMMENT$\n" | /bin/mail -s "** $NOTIFICATIONTYPE$ Service Alert: $HOSTALIAS$/$SERVICEDESC$ is $SERVICESTATE$ **" $CONTACTEMAIL$
            }

    d. 重启 nagios 服务，以激活您的更改。

    [root@manager ~]# service nagios restart

7. 将 MTA 配置为侦听远程连接。

    a. 通过注释掉 DAEMON_OPTIONS(`Port=smtp,Addr=127.0.0.1, Name=MTA')dnl 修改 sendmail.mc，以侦远程连接。

    [root@manager ~]$ cat /etc/mail/sendmail.mc
    dnl# DAEMON_OPTIONS(`Port=smtp,Addr=127.0.0.1, Name=MTA')dnl

    b. 启用并重启 manager 上的 sendmail 服务。

    [root@manager ~]# chkconfig sendmail on
    [root@manager ~]# service sendmail restart

    c. 关闭 serverb。

    [root@serverb ~]# poweroff

    d. 在 manager 上，安装 mutt 软件包，然后以 student用户身份使用命令 mutt 检查发送自 Nagios 的消息。

    [root@manager ~]# yum -y install mutt

    [student@manager ~]$ mutt

    e. 在收到警报后，重新打开 serverb 计算机。 


### 监控红帽 Gluster 存储工作负载

###### 红帽 Gluster 存储性能分析

红帽 Gluster 存储卷的性能分析特别重要，它可用于收集性能调优和容量规划的信息。拥有这些信息才能诊断和发现瓶颈。红帽 Gluster 存储随附了性能分析功能，利用 Gluster CLI 执行卷和 brick 的性能分析。命令 volume profile 和 volume top 可用于查看关键的性能信息，它们分别可协助诊断卷和 brick 的瓶颈。性能分析可以划分成两个不同的路径，即 server-side 和 client-side。

1. 服务器侧性能分析

通过服务器侧性能分析，可以在指定数量的定期样本中了解卷的活动。然后，这些信息可用于确定 brick 之间统计数据数据变化。这种统计上的变化有助于确定系统中负载分布不均匀的问题区域。服务器侧性能分析中可以查看以下信息：

*    每个卷的读取和写入 MB/s
*    每个 brick 的读取和写入 MB/s
*    每个卷每个 FOP 延迟统计 + 调用速率
*    每个 brick 每个 FOP（文件操作） 延迟统计 + 调用速率

2. 卷性能分析

存储管理员可以借助红帽 Gluster 存储性能分析监控卷的性能。存储管理员可以查看卷的各个 brick 进程不同文件操作 (FOP) 之间发生的延迟。profile volume name info 输出提供下列信息：

*    文件操作 FOP 列表。
*    各个 FOP 的调用总数。
*    各个 FOP 的延迟平均值、最大值、最小值和百分比。

在查看性能分析信息之前，必须使用以下命令启用性能分析：

[root@demo-a ~]$ gluster volume profile profile-name start
  

在启动性能分析后，系统将跟踪 FOP 及其完成时间。跟踪信息提供两种不同的统计数据，即 cumulative 和 interval。累积统计数据包含 FOP 数、各 FOP 的调用数，以及为该卷启用性能分析起的延迟。而间隔统计数据则包含 FOP 数、各 FOP 的调用数，以及自上次运行性能分析信息起的延迟。可以通过运行以下命令来获取 brick 和服务器的性能分析详细信息。

[root@demo-a ~]$ gluster volume profile profile-name info
           Cumulative Stats:
  %-latency  Avg-latency  Min-Latency  Max-Latency  No. of calls  FOP
---------   -----------  -----------  -----------  ------------  ---------
  0.00      0.00 us      0.00 us      0.00 us      1476          RELEASEDIR
  0.00      0.00 us      0.00 us      0.00 us      52            OPENDIR
  12.06     36.03 us     7.00 us      82.00 us     104           READDIR
  13.12     120.00 us    13.00 us     923.00 us    41            STATFS
  26.26     87.69 us     8.00 us      543.00 us    93            GETATTR
  48.27     85.67 us     9.00 us      359.00 us    175           LOOKUP

Duration: 223771 seconds
Data Read: 0 bytes
Data Written: 0 bytes
  

下表列出了 FOP 定义：

*    RELEASEDIR          释放目录句柄（类似于关闭）。
*    OPENDIR             打开目录（准备 READDIR）。
*    READDIR             从目录读取目录条目。
*    GETXATTR            获取指定扩展属性的值。
*    STATFS              获取文件系统的元数据。
*    LOOKUP              在目录内查找文件。

###### Volume top

命令 volume top 提供 brick 的性能指标，如读取、写入、文件打开调用、文件读取调用、文件写入调用、目录打开调用和目录读取调用。默认情况，该 top 命令显示最多 100 条结果。不过，可以在 list-cnt 选项后传递要显示的结果数（整数）来指定所显示的结果数。

1. 查看 Open 和 Maximum fd Count

open fd count 是当前 brick 上最近打开的文件列表和计数。maximum fd count 是当前打开的文件计数，以及自服务器运行起任何给定时间上打开的最大文件数。可以通过传递参数 open 到 volume top 命令来查看 open 和 maximum fd count。在运行此命令时，如果不指定 brick，则显示所有 brick 的 open fd 指标。例如，以下命令列出了 demo-volume 的 brick demo-a:/bricks/brick-a1/brick 的 open 和 maximum fd count，同时列出了前 10 个打开调用：

[root@demo-a ~]$ gluster volume top demo-volopen brick demo-a:/bricks/brick-a1/brick list-cnt 10
Brick: demo-a:/bricks/brick-a1/brick
Current open fds: 0, Max open fds: 2, Max openfd time: 2016-05-01
19:10:56.204745
Count filename
=======================
1 /file003.bin
1 /file020.bin
1 /file019.bin
1 /file018.bin
1 /file017.bin
1 /file016.bin
1 /file015.bin
1 /file014.bin
1 /file013.bin
1 /file012.bin
  
2. 查看最高文件读取调用数

可以将 read 选项用于 volume top 来查看最高读取调用数。默认情况下，如果没有指定 brick 名称，该命令将显示最多 100 条结果。例如，若要查看 brick demo-a:/bricks/brick-b1/brick 的最高读取调用数的 10 条结果，可使用以下命令。

[root@demo-a ~]$ gluster volume top prod-vol read brick demo-a:/bricks/brick-a1/brick list-cnt 10
Brick: demo-a:/bricks/brick-a1/brick
Count filename
=======================
1 /file9.odp
    
3. 查看最高文件写入调用数

可以将 write 选项用于 volume top 来查看最高写入调用数。默认情况下，如果没有指定 brick 名称，该命令将显示最多 100 条结果。例如，若要查看 brick demo-a:/bricks/brick-b1/brick 的最高写入调用数，可使用以下命令。

[root@demo-a ~]$ gluster volume top prod-vol write brick demo-a:/bricks/brick-a1/brick
Brick: demo-a:/bricks/brick-a1/brick
Count filename
=======================
1 /file9.odp
    

4. 查看最高目录读取调用数

可以将 readdir 选项用于 volume top 来查看目录的最高读取调用数。默认情况下，如果没有指定 brick 名称，将显示从属于该卷的所有 brick 的指标。例如，若要查看 brick demo-a:/bricks/brick-b1/brick 目录的最高读取调用数，可使用以下命令。

[root@demo-a ~]$ gluster volume top prod-vol readdir brick demo-a:/bricks/brick-a1/brick
Brick: demo-a:/bricks/brick-a1/brick
Count filename
=======================
2   /2008
    
5. 查看最高目录打开调用数

可以将 opendir 选项用于 volume top 来查看目录的最高打开调用数。默认情况下，如果没有指定 brick 名称，将显示从属于该卷的所有 brick 的指标。例如，若要查看 brick demo-a:/bricks/brick-b1/brick 的最高打开调用数，可使用以下命令。

[root@demo-a ~]$ gluster volume top prod-vol opendir brick demo-a:/bricks/brick-a1/brick
Brick: demo-a:/bricks/brick-a1/brick
Count filename
=======================
1 /2010
1 /2009
1 /2008
    
6. 查看各个 brick 的读取性能

可以将 read-perf 选项用于 volume top 来查看各个 brick 的文件读取吞吐量。read-perf 选项启动对指定计数和块大小的 dd，然后继续测量对应的吞吐量。默认情况下，如果没有指定 brick 名称，该命令将显示最多 100 条结果。例如，若要查看 demo-vol 的 brick demo-a:/bricks/brick-a1/brick 的读取性能，并使用块大小 256、计数 1 和结果列表 10，可运行以下命令。

[root@demo-a ~]$ gluster volume top prod-vol read-perf bs 256 count 1 brick demo-a:/bricks/brick-a1/brick
Brick: servera:/bricks/brick-a1/brick
Throughput 21.33 MBps time 0.0000 secs

7. 查看各个 brick 的写入性能

可以将 write-perf 选项用于 volume top 来查看各个 brick 的文件写入性能。write-perf 选项启动对指定计数和块大小的 dd，然后继续测量对应的吞吐量。默认情况下，如果没有指定 brick 名称，该命令将显示最多 100 条结果。例如，若要查看 demo-vol 的 brick demo-a:/bricks/brick-a1/brick 的写入性能，并使用块大小 256、计数 1 和结果数 10，可运行以下命令。

[root@demo-a ~]$ gluster volume top prod-vol write-perf bs 256 count 1 brick demo-a:/bricks/brick-a1/brick
Brick: demo-a:/bricks/brick-a1/brick
Throughput 113.78 MBps time 0.0000 secs

###### 客户端侧性能分析

客户端侧性能分析用于识别应用中与 glusterfs 相关的延迟问题。存储管理员可通过它查看与应用最相关的活动。例如，复制可以导致单一应用 WRITE FOP 转换为卷内该数据所驻留的 brick 的多个 WRITE FOP。应用的 WRITE 请求响应时间可能与 brick 级别的 WRITE FOP 延迟不同。这是因为它融入了网络响应时间，只有在 brick 级别的 WRITE FOP 完成后才能完成。例如，若要执行客户端侧性能分析，可使用 setfattr -n trusted.io-status-dump -v 加上保存状态转储的文件的路径和挂载卷的路径。

[root@demo-a ~]$ setfattr -n trusted.io-status-dump -v /root/dumpfile.txt
    /mnt
[root@demo-a ~]$ cat /root/dumpfile.txt
=== Cumulative stats ===
      Duration : 789 secs
      BytesRead : 0
      BytesWritten : 142

Block Size   :               4B+
Read Count   :                 0
Write Count  :                 4

Fop           Call Count    Avg-Latency    Min-Latency    Max-Latency
---           ----------    -----------    -----------    -----------
WRITE                  5       17.00 us       19.00 us       23.00 us
OPENDIR                9      269.00 us      190.00 us      222.00 us
LOOKUP                80      423.06 us        2.00 us     2677.00 us
RELEASE                5           0 us           0 us           0 us
RELEASEDIR             9           0 us           0 us           0 us
------ ----- ----- ----- ----- ----- ----- -----  ----- ----- ----- -----
Current open fd's: 0 Max open fd's: 1 time 2016-05-10 18:30:02.865457

==========Open File Stats========

COUNT:        FILE NAME

==========Read File Stats========

COUNT:        FILE NAME

==========Write File Stats========

COUNT:        FILE NAME
1            /resolv.conf
...

==========Directory open stats========

COUNT:        DIRECTORY NAME
2            /dir

========Directory readdirp Stats=======

COUNT:        DIRECTORY NAME
4            /dir

========Read Throughput File Stats=====

TIMESTAMP            THROUGHPUT(KBPS)   FILE NAME

======Write Throughput File Stats======

TIMESTAMP            THROUGHPUT(KBPS)   FILE NAME
2016-05-10 18:30:21.000000   4.00             /resolv.conf
...

=== Interval 1 stats ===
      Duration : 423 secs
     BytesRead : 0
  BytesWritten : 0
  

通过 gluster volume profile 命令输出的 brick 信息和 setfattr 命令输出的客户端信息，再比较客户端性能分析信息中的 FOP 延迟和 brick 性能分析信息中的 FOP 延迟，性能分析即可利用这些信息。

### 引导式练习：监控红帽 Gluster 存储 Gluster 工作负载

对卷和受信存储池工作负载进行性能分析。 

1. 为 prod-vol 卷启用性能分析。

    a. 从 servera，为 prod-vol 启用 diagnostics.count-fop-hits 和 diagnostics.latency-measurement。

    [root@servera ~]$ gluster volume  profile prod-vol start
    Starting volume profile on prod-vol has been successful.

2. 验证已经为 prod-vol 启用了性能分析。

    a. 从 servera，验证已经为 prod-vol 启用了 diagnostics.count-fop-hits 和 diagnostics.latency-measurement。

    [root@servera ~]$ gluster volume info prod-vol
    Volume Name: prod-vol
    Type: Replicate
    Volume ID: c5532183-8225-431e-b358-0e81271676af
    Status: Started
    Number of Bricks: 1 x 2 = 2
    Transport-type: tcp
    Bricks:
    Brick1: servera:/bricks/brick-a1/brick
    Brick2: serverb:/bricks/brick-b1/brick
    Options Reconfigured:
    diagnostics.count-fop-hits: on
    diagnostics.latency-measurement: on
    performance.readdir-ahead: on

3. 检查积累的 prod-vol 统计数据。

    a. 从 servera，检查积累的 prod-vol 统计数据。

    [root@servera ~]$ gluster volume profile prod-vol info cumulative
    Cumulative Stats:
     %-latency   Avg-latency   Min-Latency   Max-Latency   No. of calls         Fop
     ---------   -----------   -----------   -----------   ------------        ----
          0.00       0.00 us       0.00 us       0.00 us           1476  RELEASEDIR
          0.18       1.08 us       0.00 us       2.00 us             52     OPENDIR
         12.06      36.03 us       7.00 us      82.00 us            104     READDIR
         13.23     100.20 us      13.00 us     923.00 us             41      STATFS
         26.26      87.69 us       8.00 us     543.00 us             93    GETXATTR
         48.27      85.67 us       9.00 us     359.00 us            175      LOOKUP

        Duration: 223771 seconds
       Data Read: 0 bytes
    Data Written: 0 bytes

    Interval 9 Stats:

        Duration: 28 seconds
       Data Read: 0 bytes
    Data Written: 0 bytes

    Brick: serverb:/bricks/brick-b1/brick
    -------------------------------------
    Cumulative Stats:
     %-latency   Avg-latency   Min-Latency   Max-Latency   No. of calls         Fop
     ---------   -----------   -----------   -----------   ------------        ----
          0.00       0.00 us       0.00 us       0.00 us           1400  RELEASEDIR
          0.17       0.98 us       1.00 us       2.00 us             52     OPENDIR
          6.92      20.38 us       5.00 us      73.00 us            104     READDIR
         21.56      71.00 us       7.00 us     300.00 us             93    GETXATTR
         26.55     198.27 us      13.00 us    5005.00 us             41      STATFS
         44.80      78.39 us       9.00 us     295.00 us            175      LOOKUP

        Duration: 210408 seconds
       Data Read: 0 bytes
    Data Written: 0 bytes

    Interval 9 Stats:

        Duration: 28 seconds
       Data Read: 0 bytes
    Data Written: 0 bytes

4. 禁用 prod-vol 的性能分析。

    a. 从 servera，为 prod-vol 禁用 diagnostics.count-fop-hits 和 diagnostics.latency-measurement。

    [root@servera ~]$ gluster volume  profile prod-vol stop
    Stopping volume profile on prod-vol has been successful.

5. 通过 glusterfs volume top 选项，查看 servera 和 serverb 的 brick 性能指标。

    a. 查看 servera 和 serverb 的经常打开文件列表。

    [root@servera ~]$ gluster volume top prod-vol open
    Brick: servera:/bricks/brick-a1/brick
    Current open fds: 0, Max open fds: 2, Max openfd time: 2016-05-01 19:10:56.204745
    Count		filename
    =======================
    1		/file003.bin
    1		/file020.bin
    1		/file019.bin
    1		/file018.bin
    1		/file017.bin
    1		/file016.bin
    1		/file015.bin
    1		/file014.bin
    1		/file013.bin
    1		/file012.bin
    ...
    Brick: serverb:/bricks/brick-b1/brick
    Current open fds: 0, Max open fds: 2, Max openfd time: 2016-05-01 19:10:56.204764
    Count		filename
    =======================
    1		/file003.bin
    1		/file020.bin
    1		/file019.bin
    1		/file018.bin
    1		/file017.bin
    1		/file016.bin
    1		/file015.bin
    1		/file014.bin
    1		/file013.bin
    1		/file012.bin
    ...

    b. 查看在 servera:/bricks/brick-a1/brick 上读取操作的经常打开文件列表。

    [root@servera ~]$ gluster volume top prod-vol read  brick servera:/bricks/brick-a1/brick
    Brick: servera:/bricks/brick-a1/brick
    Count		filename
    =======================
    1		/file9.odp

    c. 查看在 servera:/bricks/brick-a1/brick 上写入操作的经常打开文件列表。

    [root@servera ~]$ gluster volume top prod-vol write brick servera:/bricks/brick-a1/brick
    Brick: servera:/bricks/brick-a1/brick
    Count		filename
    =======================
    13185		/file007.bin
    11076		/file005.bin
    9579		/file018.bin
    9541		/file012.bin
    9080		/file016.bin
    8633		/file009.bin
    8155		/file010.bin
    8143		/file017.bin
    8043		/file011.bin
    ...

    d. 查看在 servera:/bricks/brick-a1/brick 上 bs 为 256、count 为 1 的读取吞吐量。

    [root@servera ~]$ gluster volume top prod-vol read-perf bs 256 count 1 brick servera:/bricks/brick-a1/brick
    Brick: servera:/bricks/brick-a1/brick
    Throughput 21.33 MBps time 0.0000 secs
              
    e. 查看在 servera:/bricks/brick-a1/brick 上 bs 为 512、count 为 2 的写入吞吐量。

    [root@servera ~]$ gluster volume top prod-vol write-perf bs 512 count 2 brick servera:/bricks/brick-a1/brick
    Brick: servera:/bricks/brick-a1/brick
    Throughput 113.78 MBps time 0.0000 secs

    f. 查看 servera:/bricks/brick-a1/brick 的各个目录的打开调用列表。

    [root@servera ~]$ gluster volume top prod-vol opendir servera:/bricks/brick-a1/brick
    Brick: servera:/bricks/brick-a1/brick
    Count		filename
    =======================
    1		/2010
    1		/2009
    1		/2008

    g. 查看在 servera:/bricks/brick-a1/brick 上目录读取调用数最高的列表。

    [root@servera ~]$ gluster volume top prod-vol readdir brick servera:/bricks/brick-a1/brick
    Brick: servera:/bricks/brick-a1/brick
    Count		filename
    =======================
    2		/2008


### 实验：监控红帽 Gluster 存储

配置 Nagios，以监控红帽  Gluster 存储群集并且对卷进行性能分析。

1. 在 serverc 上为 NRPE 配置防火墙。

    a. 从 serverc，打开防火墙端口 5666 TCP，并且使它在重新引导时保持打开。

    [root@serverc ~]# firewalld --permanent --add-port=5666/tcp
    [root@serverc ~]# firewall-cmd --reload

2. 在 serverd 上为 NRPE 配置防火墙。

    b. 从 serverd，打开防火墙端口 5666 TCP，并且使它在重新引导时保持打开。

    [root@serverd ~]# firewalld --permanent --add-port=5666/tcp
    [root@serverd ~]#  firewall-cmd --reload

3. 配置 serverc 和 serverd 以允许访问 NRPE。

    a. 在 serverc 上，将 manager.lab.example.com 添加到 /etc/nagios/nrpe.cfg 中的 allowed_hosts 指令。

    [root@serverc ~]# cat /etc/nagios/nrpe.cfg
    ...
    allowed_hosts=127.0.0.1,manager.lab.example.com
    ...

    b. 在 serverc 上重启 nrpe。

    [root@serverc ~]# systemctl
    restart nrpe

    c. 从 serverd 上，将 manager.lab.example.com 添加到 /etc/nagios/nrpe.cfg 中的 allowed_hosts 指令。

    [root@serverd ~]# cat /etc/nagios/nrpe.cfg
    ...
    allowed_hosts=127.0.0.1,manager.lab.example.com
    ...

    d. 在 serverd 上重启 nrpe。

    [root@serverd ~]# systemctl
    restart nrpe

4. 配置 Nagios 以监控 gluster-cluster。

    a. 从 workstation，使用 SSH 连接 manager，并运行命令 configure-gluster-nagios 将群集导入到 nagios。

    [root@manager ~]# configure-gluster-nagios -c gluster-cluster -H serverc.lab.example.com
    Cluster configurations changed

    Changes :
    Hostgroup cluster - ADD
    Host cluster - ADD
             Service - Volume Utilization - prod-vol -ADD
             Service - Volume Self-Heal - prod-vol -ADD
             Service - Volume Status - prod-vol -ADD
             Service - Cluster Utilization -ADD
             Service - Cluster - Quorum -ADD
             Service - Cluster Auto Config -ADD
    Host serverc.lab.example.com - ADD
             Service - Brick Utilization - /bricks/brick-a1/brick -ADD
             Service - Brick - /bricks/brick-a1/brick -ADD
    Host serverd.lab..example.com - ADD
             Service - Brick Utilization - /bricks/brick-b1/brick -ADD
             Service - Brick - /bricks/brick-b1/brick -ADD
    Are you sure, you want to commit the changes? (Yes, No) [Yes]: Enter
    Enter Nagios server address [manager.lab.example.com]: Enter
    Cluster configurations synced successfully from host serverc.lab.example.com
    Do you want to restart Nagios to start monitoring newly discovered entities? (Yes, No) [Yes]: Enter
    Nagios re-started successfully.

5. 验证 nagios 配置。

    a. 使用 -v 选项对 /etc/nagios/nagios.cfg 执行命令 nagios，以验证 nagios 配置。

    [root@manager ~]# nagios -v /etc/nagios/nagios.cfg
    	Nagios Core 3.5.1
    Copyright (c) 2009-2011 Nagios Core Development Team and Community Contributors
    Copyright (c) 1999-2009 Ethan Galstad
    Last Modified: 08-30-2013
    License: GPL

    ...
    Total Warnings: 13
    Total Errors:   0

    Things look okay - No serious problems were detected during the pre-flight check.

6. 配置 nagios，从而发送 glusterd 的电子邮件通知到用户 student。

    a. 修改 /etc/nagios/gluster/gluster-contacts.cfg 中的 contact_name、alias 和 email 指令，使它们分别反映 student、student 和 student@manager.lab.example.com。

    [root@manager ~]# cat /etc/nagios/gluster/gluster-contacts.cfg
    define contact {
           contact_name                  student
           alias                         student
           email                         student@manager.lab.example.com
           service_notification_period   24x7
           service_notification_options  w,u,c,r,f,s
           service_notification_commands notify-service-by-email
           host_notification_period      24x7
           host_notification_options     d,u,r,f,s
           host_notification_commands    notify-host-by-email
    }

    b. 在 /etc/nagios/gluster/gluster-templates.cfg 中，为 gluster-service 和 gluster-generic-host 添加联系人名称 student。

    [root@manager ~]# cat /etc/nagios/gluster/gluster-templates.cfg
    define host{
       name                         gluster-generic-host
       use                          linux-server
       notifications_enabled        1
       notification_period          24x7
       notification_interval        120
       notification_options         d,u,r,f,s
       register                     0
       contacts                     +snmp,student
    }

    define service {
       name                         gluster-service
       use                          generic-service
       notifications_enabled       1
       notification_period          24x7
       notification_options         w,u,c,r,f,s
       notification_interval        120
       register                     0
       contacts                     +snmp,student
       _gluster_entity              Service
    }
                

    c. 将 $NOTIFICATIONCOMMENT$\n 直接添加到 /etc/nagios/objects/commands.cfg 中 | /bin/mail -s 的前面，以用于 notify-service-by-email 和 notify-host-by-email 定义。

    [root@manager ~]$ cat /etc/nagios/objects/commands.cfg.
    define command{
            command_name    notify-host-by-email
            command_line    /usr/bin/printf "%b" "***** Nagios *****\n\nNotification Type: $NOTIFICATIONTYPE$\nHost: $HOSTNAME$\nState: $HOSTSTATE$\nAddress: $HOSTADDRESS$\nInfo: $HOSTOUTPUT$\n\nDate/Time: $LONGDATETIME$\n $NOTIFICATIONCOMMENT$\n" | /bin/mail -s "** $NOTIFICATIONTYPE$ Host Alert: $HOSTNAME$ is $HOSTSTATE$ **" $CONTACTEMAIL$
            }

    # 'notify-service-by-email' command definition
    define command{
            command_name    notify-service-by-email
            command_line    /usr/bin/printf "%b" "***** Nagios *****\n\nNotification Type: $NOTIFICATIONTYPE$\n\nService: $SERVICEDESC$\nHost: $HOSTALIAS$\nAddress: $HOSTADDRESS$\nState: $SERVICESTATE$\n\nDate/Time: $LONGDATETIME$\n\nAdditional Info:\n\n$SERVICEOUTPUT$\n $NOTIFICATIONCOMMENT$\n" | /bin/mail -s "** $NOTIFICATIONTYPE$ Service Alert: $HOSTALIAS$/$SERVICEDESC$ is $SERVICESTATE$ **" $CONTACTEMAIL$
            }

    d. 重启 nagios 服务，以激活您的更改。

    [root@manager ~]# service nagios restart

7. 将 MTA 配置为侦听远程连接。

    a. 通过注释掉 DAEMON_OPTIONS(`Port=smtp,Addr=127.0.0.1, Name=MTA')dnl 修改 sendmail.mc，以侦远程连接。

    [root@manager ~]# cat /etc/mail/sendmail.mc
    dnl# DAEMON_OPTIONS(`Port=smtp,Addr=127.0.0.1, Name=MTA')dnl
            
    b. 启用并重启 manager 上的 sendmail 服务。

    [root@manager ~]# chkconfig sendmail on
    [root@manager ~]# service sendmail restart

    c. 关闭 serverd。

    [root@serverd ~]# poweroff

    d. 在 manager 上安装 mutt 软件包，然后在 workstation 上以 student 用户身份使用命令 mutt 来查收电子邮件。

    [root@manager ~]# yum install -y mutt
    [student@manager ~]$ mutt

8. 启用 prod-vol 的性能分析。

    a. 从 serverc，为 prod-vol 启用 diagnostics.count-fop-hits 和 diagnostics.latency-measurement。

    [root@serverc ~]# gluster volume  profile prod-vol start
    Starting volume profile on prod-vol has been successful.

9. 验证已经为 prod-vol 启用了性能分析。

    a. 从 serverc，验证已经为 prod-vol 启用了 diagnostics.count-fop-hits 和 diagnostics.latency-measurement。

    [root@serverc ~]# gluster volume info prod-vol
    Volume Name: prod-vol
    Type: Replicate
    Volume ID: c5532183-8225-431e-b358-0e81271676af
    Status: Started
    Number of Bricks: 1 x 2 = 2
    Transport-type: tcp
    Bricks:
    Brick1: serverc:/bricks/brick-a1/brick
    Brick2: serverd:/bricks/brick-b1/brick
    Options Reconfigured:
    diagnostics.count-fop-hits: on
    diagnostics.latency-measurement: on
    performance.readdir-ahead: on

10. 检查积累的 prod-vol 统计数据。

    a. 从 serverc，检查积累的 prod-vol 统计数据。

    [root@serverc ~]# gluster volume profile prod-vol info cumulative
    Cumulative Stats:
     %-latency   Avg-latency   Min-Latency   Max-Latency   No. of calls         Fop
     ---------   -----------   -----------   -----------   ------------        ----
          0.00       0.00 us       0.00 us       0.00 us           1476  RELEASEDIR
          0.18       1.08 us       0.00 us       2.00 us             52     OPENDIR
         12.06      36.03 us       7.00 us      82.00 us            104     READDIR
         13.23     100.20 us      13.00 us     923.00 us             41      STATFS
         26.26      87.69 us       8.00 us     543.00 us             93    GETXATTR
         48.27      85.67 us       9.00 us     359.00 us            175      LOOKUP

        Duration: 223771 seconds
       Data Read: 0 bytes
    Data Written: 0 bytes

    Interval 9 Stats:

        Duration: 28 seconds
       Data Read: 0 bytes
    Data Written: 0 bytes

    Brick: serverd:/bricks/brick-d1/brick
    -------------------------------------
    Cumulative Stats:
     %-latency   Avg-latency   Min-Latency   Max-Latency   No. of calls         Fop
     ---------   -----------   -----------   -----------   ------------        ----
          0.00       0.00 us       0.00 us       0.00 us           1400  RELEASEDIR
          0.17       0.98 us       1.00 us       2.00 us             52     OPENDIR
          6.92      20.38 us       5.00 us      73.00 us            104     READDIR
         21.56      71.00 us       7.00 us     300.00 us             93    GETXATTR
         26.55     198.27 us      13.00 us    5005.00 us             41      STATFS
         44.80      78.39 us       9.00 us     295.00 us            175      LOOKUP

        Duration: 210408 seconds
       Data Read: 0 bytes
    Data Written: 0 bytes

    Interval 9 Stats:

        Duration: 28 seconds
       Data Read: 0 bytes
    Data Written: 0 bytes

11. 从 serverc，为 prod-vol 禁用 diagnostics.count-fop-hits 和 diagnostics.latency-measurement。

[root@serverc ~]# gluster volume  profile prod-vol stop
Stopping volume profile on prod-vol has been successful.

12. 通过 glusterfs volume top 选项，查看 serverc 和 serverd 的 brick 性能指标。

    a. 查看 serverc 和 serverd 的经常打开文件列表。

    [root@serverc ~]# gluster volume top prod-vol open
    Brick: serverc:/bricks/brick-c1/brick
    Current open fds: 0, Max open fds: 2, Max openfd time: 2016-05-01 19:10:56.204745
    Count		filename
    =======================
    1		/file003.bin
    1		/file020.bin
    1		/file019.bin
    1		/file018.bin
    1		/file017.bin
    1		/file016.bin
    1		/file015.bin
    1		/file014.bin
    1		/file013.bin
    1		/file012.bin
    ...
    Brick: serverd:/bricks/brick-d1/brick
    Current open fds: 0, Max open fds: 2, Max openfd time: 2016-05-01 19:10:56.204764
    Count		filename
    =======================
    1		/file003.bin
    1		/file020.bin
    1		/file019.bin
    1		/file018.bin
    1		/file017.bin
    1		/file016.bin
    1		/file015.bin
    1		/file014.bin
    1		/file013.bin
    1		/file012.bin
    ...

    b. 查看在 serverc:/bricks/brick-c1/brick 上读取操作的经常打开文件列表。

    [root@serverc ~]# gluster volume top prod-vol read  brick serverc:/bricks/brick-c1/brick
    Brick: serverc:/bricks/brick-c1/brick
    Count		filename
    =======================
    1		/file9.odp

    c. 查看在 serverc:/bricks/brick-c1/brick 上写入操作的经常打开文件列表。

    [root@serverc ~]# gluster volume top prod-vol write brick serverc:/bricks/brick-c1/brick
    Brick: serverc:/bricks/brick-c1/brick
    Count		filename
    =======================
    13185		/file007.bin
    11076		/file005.bin
    9579		/file018.bin
    9541		/file012.bin
    9080		/file016.bin
    8633		/file009.bin
    8155		/file010.bin
    8143		/file017.bin
    8043		/file011.bin
    ...
              

    d. 查看在 serverc:/bricks/brick-a1/brick 上 bs 为 256、count 为 1 的读取吞吐量。

    [root@serverc ~]# gluster volume top prod-vol read-perf bs 256 count 1 brick serverc:/bricks/brick-c1/brick
    Brick: serverc:/bricks/brick-c1/brick
    Throughput 21.33 MBps time 0.0000 secs

    e. 查看在 serverc:/bricks/brick-c1/brick 上 bs 为 512、count 为 2 的写入吞吐量。

    [root@serverc ~]# gluster volume top prod-vol write-perf bs 512 count 2 brick serverc:/bricks/brick-c1/brick
    Brick: serverc:/bricks/brick-c1/brick
    Throughput 113.78 MBps time 0.0000 secs
              

    f. 查看 serverc:/bricks/brick-a1/brick 的各个目录的打开调用列表。

    [root@serverc ~]# gluster volume top prod-vol opendir serverc:/bricks/brick-c1/brick
    Brick: serverc:/bricks/brick-c1/brick
    Count		filename
    =======================
    1		/2010
    1		/2009
    1		/2008

    g. 查看在 serverc:/bricks/brick-c1/brick 上目录读取调用数最高的列表。

    [root@serverc ~]# gluster volume top prod-vol readdir brick serverc:/bricks/brick-c1/brick
    Brick: serverc:/bricks/brick-c1/brick
    Count		filename
    =======================
    2		/2008


### 总结

在本章中，您学到了：

*    必须配置 nrpe.cfg 文件，以允许远程监控。
*    可以使用 check_nrpe 命令来验证 Nagios 服务器和远程系统之间的通信。
*    通过 configure-gluster-nagios 自动配置可以简化为监控红帽 Gluster 存储而对 Nagios 进行的配置。
*    通过创建服务、主机、命令、联系人和联系人组定义，配置 Nagios 警告和通知。 
