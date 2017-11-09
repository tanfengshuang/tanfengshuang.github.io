---
layout: post
title:  "Installing Red Hat Gluster Storage(236-2)"
categories: Linux
tags: RHCA 236
---

### 安装 Red Hat Storage Server On-premise

###### 部署 Red Hat Gluster Storage On-Premise

Red Hat Gluster Storage On-Premise 可以从 DVD 映像安装，该映像可从红帽客户门户 (http://access.redhat.com) 下载。此映像可刻录为 DVD，或者可利用 PXE 环境来使用。

也提供基于订阅存储库的红帽 Gluster 存储安装。如果要在红帽企业 Linux 7.2 基础上部署红帽 Gluster 存储，则需要在系统上连接并启用以下存储库：

*    rhel-7-server-rpms
*    rh-gluster-3-for-rhel-7-server-rpms
*    rh-gluster-3-nfs-for-rhel-7-server-rpms
*    rh-gluster-3-samba-for-rhel-7-server-rpms

红帽 Gluster 存储也可以部署到红帽企业 Linux 6.7 上。若要提供此支持，必须在系统上连接并启用以下存储库：

*    rhel-6-server-rpms
*    rhel-scalefs-for-rhel-6-server-rpms
*    rhs-3-for-rhel-6-server-rpms
*    rh-gluster-3-nfs-for-rhel-6-server-rpms
*    rh-gluster-3-samba-for-rhel-6-server-rpms

只有需要 NFS-Ganesha 支持时，才需要 rh-gluster-3-nfs-* 存储库。类似地，若要提供 Samba 或 CTDB 支持，则应选择 rh-gluster-3-samba 存储库。

###### 安装红帽 Gluster 存储

1. 从 https://access.redhat.com 获取红帽 Gluster 存储的安装 ISO。
2. 使此安装介质可供安装使用。将它刻录到可写 DVD 上，上传到服务器管理卡的虚拟 DVD 驱动器上，或者通过 PXE 引导来提供它。
3. 引导服务器，以便从准备好的安装介质进行安装。这时显示类似于如下的屏幕：
4. 选择 Install Red Hat Gluster Storage 3.1.2，再单击 Enter 来开始安装。
5. 第一个屏幕将要求选择安装过程中使用的语言。语言选择 English，键盘选择 English (United States)。单击 Continue 以进入下一屏幕。
6. 这时将显示 INSTALLATION SUMMARY 中心。此屏幕中显示可进一步选择配置的图标。如果图标旁边显示警告符号，这表示必须完成选择后 Begin Installation 按钮才会激活。
   INSTALLATION SUMMARY 中心的第一个图标是 DATE & TIME 图标。单击它，为所要安装的红帽 Gluster 服务器设置时区。这时显示一张世界地图。选择最能代表当地时区的城市，然后单击 Done 以返回到 INSTALLATION SUMMARY 中心。
7. 单击 INSTALLATION DESTINATION 图标，以选择要用于安装的存储设备类型。必须至少选择一个目标磁盘设备。用户可以选择其他选项，从而全面控制此屏幕中配置存储的方式。单击 Done，以返回到 INSTALLATION SUMMARY 中心。
   
   如果选中标有 I would like to make additional space available 的复选框，用户可以回收之前安装的系统上的磁盘空间。这时将显示以下屏幕，供用户选择要将哪些分区重新用于新安装。
   
   进行完选择后，必须单击 Reclaim space 按钮来确认选择并且继续安装。

8. 可以通过单击 INSTALLATION SUMMARY 中心的 NETWORK & HOSTNAME 图标来设定网络设置。此图标位于 INSTALLATION DESTINATION 图标的下方。当网络接口已激活并且 DHCP 配置用于网络时，网络设置及主机名称字段将由 DHCP 提供的信息来决定。
   
   可以通过单击 Configure... 按钮来设置静态网络信息。这时将出现一个对话框，其中提供多个选项卡供用户设置 IPv4 和 IPv6 网络设置。若要手动设置主机名称，可在 NETWORK & HOSTNAME 主屏幕中 Host name 字段中提供信息。单击 Done 按钮，以返回到 INSTALLATION SUMMARY 中心。

> 注意: 只有执行 PXE 安装或者安装过程中指定了 NTP 时间服务时，才需要网络连接。从 CDROM 映像安装时，可以在安装后配置网络。

9. 在解决了 INSTALLATION SUMMARY 中心的所有警告后，Begin Installation 按钮激活。单击 Begin Installation 以开始安装。
10. 在格式化磁盘并且开始安装软件包期间，如果 CONFIGURATION 屏幕中的图标旁边出现警告符号，这表示应当创建 root 密码和用户帐户。单击 ROOT PASSWORD 图标，以设置 root 密码。这时显示 ROOT PASSWORD 屏幕。
    
    在用户键入 root 密码时，将通过一个刻度表指示所指定的密码的相对强度。必须在 Confirm 字段中重新键入密码，以避免拼写错误。尽管强烈建议不要使用弱密码，但再次单击 Done 可强制安装程序接受弱密码。

11. 安装程序完成时，单击 Reboot 以结束安装。


### 在公共云上安装红帽存储服务器

###### 在 Amazon EC2 上安装红帽 Gluster 存储

在 Amazon EC2 公共云上安装红帽 Gluster 存储与在企业内部安装红帽 Gluster 存储稍有不同。尽管也支持从基于 Red Hat Enterprise Linux 的预存在映像（称为 AMI，即 Amazon Machine Image）安装，但更简单的安装选项是从预装有红帽 Gluster 存储的 AMI 开始。

逐步完成在 EC2 上调配机器的一般步骤。确保选择大型实例和红帽 Gluster 存储 AMI。如果启用适用于 OpenStack Swift 的红帽 Gluster 存储，请记得打开端口 22/TCP、6000/TCP、6001/TCP、6002/TCP、443/TCP 和 8080/TCP。

###### 在 EC2 上调配存储

由于普通 EBS 卷上的 I/O 性能可能不一致，因此红帽建议配置 RAID 0 条带，其包含用作存储 brick 的八个大小相等的 EBS 卷。有关如何完成此操作的更多信息，请参见位于 https://access.redhat.com 的《红帽 Gluster 存储管理指南》。

###### 在 Azure 上安装红帽 Gluster 存储

Microsoft Azure 上的红帽 Gluster 存储可以利用 Azure 可用性设置在计划或非计划停机期间帮助维持数据可用性。参考资源部分中提供了关于如何在 Microsoft Azure 上设置红帽 Gluster 存储的详细指南链接。

###### 在 Google 云平台安装红帽 Gluster 存储

红帽 Gluster 存储也可以部署到 Google 云平台上。access.redhat.com 上提供了关于如何对此进行配置的详细说明。


### 实验：安装红帽 Gluster 存储

1. 要求在 servera 计算机上安装红帽存储服务器。该系统应当仅安装在主磁盘上，回收任何未使用的空间，并且应当配置为使用本地时区。root 用户密码应设为 redhat。
2. 第一网络接口应当配置为在引导时通过 DHCP 激活，其主机名称应当设为 servera.lab.example.com。 

1. 打开 servera 控制台。引导加载程序显示基于文本的菜单，倒计时直到它引导至测试模式。使用箭头键选择 Install Red Hat Gluster Storage 3.1.2 选项，再按 Enter。
2. 启动了图形安装程序后，选择 English (United States) 作为在安装过程中使用的语言。单击 Continue 以确认选择。
3. 如果您使用非美国英语键盘，请使用 KEYBOARD 图标选择键盘，然后单击 Done 返回到安装程序主屏幕。
4. 这时将显示 INSTALLATION SUMMARY 屏幕。图标旁边出现警告符号表示需要更多信息才能继续安装。等待几秒钟，其中一些警告符号将消失，因为安装程序会收集与系统相关的信息。

选择安装到 /dev/vda 设备并回收任何未使用的空间。

*    单击 INSTALLATION DESTINATION 图标，以指定要在其上安装该软件的磁盘。
*    选择与 vda 对应的磁盘图标。
*    选中 I would like to make additional space available 复选框。这将从设备清除任何现有的分区（若必要）。
*    单击 Done 以确认选择。这时显示 RECLAIM DISK SPACE 屏幕。
*    单击 Reclaim space，以返回到 INSTALLATION SUMMARY 屏幕。 

5. 配置此系统的网络设置，以使用主机名称 servera.lab.example.com 并将 DHCP 用于网络配置。

*    单击标有 NETWORK & HOST NAME 的图标，以配置此存储服务器的网络。
*    单击 Ethernet (eth0) 设备的滑块按钮，将它移到 ON。
*    单击 Configure，再单击 General 选项卡，然后勾选 Automatically connect to this network when it is available。单击 Save 以保存更改。
*    Host name 字段应当会自动填充 servera.lab.example.com；如果没有，请手动输入。
*    单击 Done 以确认选择，再返回到 INSTALLATION SUMMARY 屏幕。 

6. 将 servera 系统配置为使用本地时区。

*    单击 DATE & TIME 图标，以输入日期和时间设置。
*    在地图上选择距您最近的城市，然后单击 Done 以返回到 INSTALLATION SUMMARY 屏幕。 

7. 因为安装程序已获得需要的所有信息，Begin installation 按钮现在已激活。单击该按钮以开始安装过程。这时显示 CONFIGURATION 屏幕。系统将格式化所选的磁盘，并且开始安装软件包。

8. ROOT PASSWORD 图标旁边有一个警告符号。将 root 用户密码设置为 redhat。

*    单击 ROOT PASSWORD 图标。
*    在 Root password: 和 Confirm: 框中键入 redhat。
*    单击 Done 以接受选择。
*    因为 redhat 是弱密码，所以屏幕底部会显示警告。再次单击 Done 以忽略该警告。 

9. 等待安装完成，然后单击 Reboot。

10. 等待 servera 计算机启动，然后以 root 用户身份并使用您在安装过程开头指定的密码 (redhat) 登录。

11. 确认 glusterd 服务已自动启动。

```
    [root@servera ~]# systemctl status glusterd
    ● glusterd.service - GlusterFS, a clustered file-system server
       Loaded: loaded (/usr/lib/systemd/system/glusterd.service; enabled;
        vendor preset: disabled)
       Active: active (running) since Thu 2016-03-17 22:04:31 EDT; 1min 7s ago
      Process: 1343 ExecStart=/usr/sbin/glusterd -p /var/run/glusterd.pid
       --log-level $LOG_LEVEL $GLUSTERD_OPTIONS (code=exited, status=0/SUCCESS)
     Main PID: 1351 (glusterd)
       CGroup: /system.slice/glusterd.service
               └─1351 /usr/sbin/glusterd -p /var/run/glusterd.pid --log-level INFO

    Mar 17 22:04:29 servera.lab.example.com systemd[1]: Starting GlusterFS,
     a clustered file-system server...
    Mar 17 22:04:31 servera.lab.example.com systemd[1]: Started GlusterFS,
     a clustered file-system server.
```

12. 在 servera 上，使用 curl 拉取代码，以便为评测准备系统。将输出定向到 bash 以便其执行并配置 SSH 密钥。

```
[root@servera ~]# curl http://materials.example.com/finish-install | bash
```

13. 完成时，通过从 workstation 计算机运行 lab install-rhs grade 命令来评测您的工作。

```
[student@workstation ~]$ lab install-rhs grade
```

14. 重要清理：通过重置您的虚拟机，将 servera 重置到开始实验之前的状态。 


### 总结

在本章中，您学到了：

*    用于部署红帽 Gluster 存储的不同方式，包括从 ISO/PXE 以及预定义的虚拟映像安装等。
*    如果对 EBS 映像分条以在 Amazon EC2 云中使用。
*    从何处查找在 Microsoft Azure 上部署红帽 Gluster 存储的文档 (https://access.redhat.com/articles/using-gluster-with-azure) 


