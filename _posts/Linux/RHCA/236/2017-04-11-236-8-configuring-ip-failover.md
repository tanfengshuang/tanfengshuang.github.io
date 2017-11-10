---
layout: post
title:  "Configuring IP Failover(236-8)"
categories: Linux
tags: RHCA 236
---

### 通过 CTDB 进行 IP 故障切换

###### CTDB 功能

在使用 Samba 导出红帽  Gluster 存储卷时，可使用 Samba Clustered Trivial Database(CTDB) 提供自动 IP 故障切换和 Samba 共享锁定恢复。这可用于向客户端提供高可用的 Samba 共享。

在使用本地用户帐户并且不使用 LDAP 或 Active Directory 等中央化用户身份验证方案时，也可以在参与的节点之间自动同步本地用户数据库。

###### 安装 CTDB

若要在红帽  Gluster 存储节点上安装 CTDB，可利用来自 rh-gluster-3-samba-for-rhel-7-server-rpms 频道的 ctdb 软件包。不应当使用 Red Hat Enterprise Linux 7 主存储库中的 ctdb2.5，它不受支持。

如果 CTDB 必须要管理 Samba 共享，则还应当安装同一频道中的 samba 软件包。

###### 配置 CTDB

设置 CTDB 需要执行几个步骤：

1. 在所有加入 CTDB 群集的节点上，启用 rh-gluster-3-samba-for-rhel-7-server-rpms 频道。
2. 在所有节点上安装 samba 和 ctdb 软件包。
3. 在需要加入 IP 故障切换群集的每个节点上，创建带有一个 brick 的（小型）复制卷。此卷用于存储锁定信息。暂时不要启动此卷。
4. 在所有节点上，更新 /var/lib/glusterd/hooks/1/start/post/S29CTDBsetup.sh 和 /var/lib/glusterd/hooks/1/stop/pre/S29CTDB-teardown.sh，使得 META= 变量设置为早前创建的复制卷。这些脚本会将复制卷永久挂载到所有节点上的 /gluster/lock。
5. 在所有节点上，将下面这一行添加到 /etc/samba/smb.conf 的 [global] 部分。这将启用 Samba 的群集功能。

> clustering=yes

6. 启动复制卷，并验证它已挂载到所有节点的 /gluster/lock。
7. 在所有节点上，打开防火墙中的 4379/TCP 端口以及 samba 服务。
8. 在所有节点上，创建 /etc/ctdb/nodes 文件并对它进行编辑，使得每一行包含一个节点的 IP 地址。例如，如果要使用 IP 地址分别为 10.1.2.3 和 10.1.2.4 的两个节点，则该文件应当如下方所示：

```
10.1.2.3
10.1.2.4
```

此文件需要在所有节点上相同，因为各个节点的物理节点号 (PNN) 由这一文件决定。建议使用 Ansible 或 Puppet 等配置管理系统来管理这些文件。

9. 在每个节点上创建 /etc/ctdb/public_addresses 文件，以配置要使用的浮动 IP 地址。此文件中的每一行都使用以下格式：IP/NETMASK INTERFACE。例如，如果需要将 IP 地址 192.168.0.100/24 用作浮动 IP 地址，且使用接口 eth1，该文件应当如下方所示：

```
192.168.0.100/24 eth1
```

10. 在所有节点上启用并启动 ctdb.service 服务。
11. 正常配置卷，以使用 Samba 导出。有关更多信息，请参见第 5 章 配置客户端中关于 Samba 的章节。

###### 管理 CTDB

在 CTDB 运行时，可以通过 ctdb 命令来进行管理。借助此命令，管理员可以触发故障切换，查看分配了特定浮动 IP 的位置，以及执行其他操作。在使用 CTDB 和红帽  Gluster 存储时，可以利用以下几个命令。几乎在所有情形中，都可使用选项 -n PNN_LIST 指定逗号分隔的节点号列表，用于查询，而非 localhost。类似地，也可利用 -d 选项启用调试。

*    ctdb pnn           显示此主机的物理节点号。
*    ctdb status        查看常规状态信息，包括主机、其 PNN 以及状态的列表。
*    ctdb ping -n all   对群集中的所有主机执行 Ping，以验证它们是否正常。
*    ctdb ip            查看对此节点已知的浮动 IP 地址列表，以及哪一节点正在提供这些 IP。添加 -v 选项将提供更为详尽的信息。
*    ctdb ipinfo IP     显示关于指定浮动 IP 地址的详细信息，包括哪一节点正在提供此地址。
*    ctdb ip IP PNN     将指定的 IP 地址移到指定的节点。


### 引导式练习：通过 CTDB 进行 IP 故障切换

servera 和 serverb 计算机上部署了一个包含两个节点的红帽  Gluster 存储。这个群集已配置了两个卷：一个名为 ctdbmeta 的复制卷，以及一个名为 custdata 的复制卷。您已被要求配置此群集，以使用 IP 172.25.250.15/24 提供 IP 故障切换，并且利用 Samba 通过此 IP 导出 custdata 卷。该共享应当可供 Samba 用户 smbuser（密码为 redhat）使用。两个节点上已创建了名为 smbuser 的本地帐户。ctdbmeta 卷应当用于存储此安装需要的任何锁文件以及其他元数据。在完成所有配置后，custdata 卷应当通过此浮动 IP 地址永久挂载到 workstation 上的 /mnt/custdata

1. 在 servera 和 serverb 上安装所需的软件，并在它们的防火墙中打开此设置需要的所有端口。

    a. 在 servera 和 serverb 计算机上，安装 samba 和 ctdb 软件包。

    [root@servera ~]# yum install samba ctdb
    [root@serverb ~]# yum install samba ctdb

    b. 在两个系统上，打开 samba 服务，并在防火墙中打开端口 4379/tcp。

    [root@servera ~]# firewall-cmd --add-service=samba
    [root@servera ~]# firewall-cmd --add-port=4379/tcp
    [root@servera ~]# firewall-cmd --runtime-to-permanent

    [root@serverb ~]# firewall-cmd --add-service=samba
    [root@serverb ~]# firewall-cmd --add-port=4379/tcp
    [root@serverb ~]# firewall-cmd --runtime-to-permanent

2. 停止 ctdbmeta 卷，然后在 servera 和 serverb 上配置相关的启动和停止 hook，从而将 ctdbmeta 卷用于 CTDB。另外，在两个节点上为 Samba 启用群集。

    a. 停止 ctdbmeta 卷。

    [root@servera ~]# gluster volume stop ctdbmeta
    Stopping volume will make its data inaccessible. Do you want to continue? (y/n) y
    volume stop: ctdbmeta: success

    b. 在 servera 和 serverb 上，编辑 /var/lib/glusterd/hooks/1/start/post/S29CTDBsetup.sh 和 /var/lib/glusterd/hooks/1/stop/pre/S29CTDB-teardown.sh，使其 META="all" 行变为 META=ctdbmeta。

    c. 在 servera 和 serverb 上，通过编辑 /etc/samba/smb.conf 将下面这一行包含在 [global] 块中：

    clustering=yes

3. 启动 ctdbmeta 卷，然后配置 CTDB，从而将 servera 和 serverb 系统用于 IP 故障切换并将 172.25.250.15/24 用作浮动 IP 地址。

    a. 启动 ctdbmeta 卷。

    [root@servera ~]# gluster volume start ctdbmeta
    volume start: ctdbmeta: success

    b. 在 servera 和 serverb 上，创建包含以下内容的文件 /etc/ctdb/nodes。确保该文件在两个节点上完全一致。此文件列出了参与通过 CTDB 进行 IP 故障切换的所有主机的 IP 地址。

    172.25.250.10
    172.25.250.11

    c. 在 servera 和 serverb 上，创建包含以下内容的文件 /etc/ctdb/public_addresses。此文件配置应使用的浮动 IP 地址。

    172.25.250.15/24 eth0

    d. 在 servera 和 serverb 上，启动并启用 ctdb.service 服务。

    [root@servera ~]# systemctl enable ctdb
    [root@servera ~]# systemctl start ctdb

    [root@serverb ~]# systemctl enable ctdb
    [root@serverb ~]# systemctl start ctdb

4. 配置 custdata 卷，以使用 Samba 导出。记得为 smbuser 用户设置 Samba 密码 redhat。

    a. 将 smbuser 的 Samba 密码设置为 redhat。由于 ctdb 将此更改传播到所有节点，因此只需在一个主机上执行此步骤一次。

    [root@servera ~]# smbpasswd -a smbuser
    New SMB password: redhat
    Retype new SMB password: redhat

    b. 为 custdata 卷设置以下选项：

    *    stat-prefetch off
    *    server.allow-insecure on
    *    storage.batch-fsync-delay-usec 0 

    [root@servera ~]# gluster volume set custdata stat-prefetch off
    [root@servera ~]# gluster volume set custdata server.allow-insecure on
    [root@servera ~]# gluster volume set custdata storage.batch-fsync-delay-usec 0
                

    c. 在 servera 和 serverb 上，将下面这一行添加到 /etc/glusterfs/glusterd.vol 的 type mgmt 部分内，然后重启 glusterd.service。

    >    option rpc-auth-allow-insecure on

    [root@servera ~]# systemctl restart glusterd
    [root@serverb ~]# systemctl restart glusterd

    d. 重启 custdata 卷。

    [root@servera ~]# gluster volume stop custdata
    Stopping volume will make its data inaccessible. Do you want to continue? (y/n) y
    volume stop: custd: success
    [root@servera ~]# gluster volume start custdata
    volume start: custdata: success

5. 在 workstation 系统上，利用 Samba 并通过浮动 IP 地址将 custdata 卷永久挂载到 /mnt/custdata。

    a. 创建 /mnt/custdata 挂载点。

    [root@workstation ~]# mkdir /mnt/custdata

    b. 在 workstation 上，将下面这一行添加到 /etc/fstab：

    //172.25.250.15/gluster-custdata /mnt/custdata cifs user=smbuser,pass=redhat 0 0

    c. 挂载 custdata 卷。

    [root@workstation ~]# mount /mnt/custdata

6. 测试您的 IP 故障切换，具体为：在 workstation 上的 /mnt/custdata 中创建一些文件，关闭当前分配有浮动 IP 地址的主机，然后在 workstation 上的 /mnt/custdata 创建更多文件。请勿忘记启动您关闭的计算机。

    a. 在 workstation 上，在 /mnt/custdata 中创建一些文件。

    [root@workstation ~]# touch /mnt/custdata/file{00..49}

    b. 确定哪一节点当前具有浮动 IP 地址。

    [root@servera ~]# ctdb ip -n all
    Public IPs on ALL nodes 
    172.25.250.15 0

    172.25.150.15 0 行末尾的 0 表示此 IP 在物理节点编号 0 上运行。您的系统可能报告不同的物理节点编号。

    c. 使用 ctdb status 来确定哪一主机具有指出的物理节点编号。

    [root@servera ~]# ctdb status
    Number of nodes:2
    pnn:0 172.25.250.10    OK (THIS NODE)
    pnn:1 172.25.250.11    OK
    Generation:1099704256
    Size:2
    hash:0 lmaster:0
    hash:1 lmaster:1
    Recovery mode:NORMAL (0)
    Recovery master:0

    在上例中，浮动 IP 正在 servera (pnn:0) 上运行；但在您的实验环境中，浮动 IP 可能在其他物理节点编号上运行，或者可能分配了不同的物理节点编号。

    d. 关闭在上述步骤中找到的主机。

    [root@serverX ~]# poweroff

    e. 在 workstation 系统上，尝试在 /mnt/custdata 中创建更多文件。在短暂超时后，此操作应当会成功。

    [root@workstation ~]# touch /mnt/custdata/file{50..99}

    f. 重要信息：启动您在前面关闭的计算机。 

7. 在仍然运行的节点上，检查 ctdb 的日志。找到发生接管的行。

    a. 在 less 等查看器中打开日志文件 /var/log/log.ctdb。
    b. 搜索文本 takeoverrun。这是 IP 故障切换/锁定恢复进程的第一行。 


### 配置 NFS Ganesha

###### NFS-Ganesha 功能

NFS-Ganesha 是面向 NFS 的用户模式文件服务器。它支持 NFSv3、NFSv4、NFSv4.1 和 pNFS（作为技术预览）。通过利用 Corosync 和 Pacemaker 提供的群集基础架构，让 NFS-Ganesha 具备高可用性。

红帽  Gluster 存储中内置的 NFS 服务器仅支持 NFSv3。如果需要 NFSv4、Kerberos 身份验证或加密或者 IP 故障切换，管理员应当使用 NFS-Ganesha。

> 重要: NFS-Ganesha 无法与内置的 NFSv3 服务器同时运行。运行 NFS-Ganesha 的所有节点上都应禁用 NFS。

###### 安装 NFS-Ganesha

在红帽  Gluster 存储上安装 NFS-Ganesha 需要两个额外的频道：rh-gluster-3-nfs-for-rhel-7-server-rpms（提供 NFS-Ganesha 软件包）和 rhel-ha-for-rhel-7-server-rpms（提供所需的 HA 软件包）。

在节点注册到所需的频道后，可以安装 glusterfs-ganesha 软件包。此软件包具有几个依赖项，如 nfs-ganesha、nfs-ganesha-cluster 和 pacemaker。

> 重要: 如果正在运行任何 glusterfs 进程，则安装 glusterfs-ganesha 软件包将失败。在安装之前，请停止 glusterd 并且中断所有 glusterfs 和 glusterfsd 进程。

###### 配置 NFS-Ganesha

配置 NFS-Ganesha 需要执行几个步骤： 

1. 在所有要加入群集的节点上，打开防火墙中的下述服务：

    *    nfs
    *    mountd
    *    rpc-bind
    *    high-availability

    如果客户端将使用 NFSv3，则还应当为 statd、lockd 和 rquotad 选择和打开端口。

2. 准备 /etc/ganesha/ganesha-ha.conf。

    a. 在要加入群集的一个节点上，将 /etc/ganesha/ganesha-ha.conf.sample 复制到 /etc/ganesha/ganesha-ha.conf。
    b. 使用以下信息编辑 /etc/ganesha/ganesha-ha.conf；确保所有值都用引号括起。

    *    HA_NAME="CLUSTERNAME"
    *    此名称必须在群集将要运行的网络段内唯一。
    *    HA_VOL_SERVER="GLUSTERNODE"
    *    这应当指向红帽  Gluster 存储受信存储池中的一个服务器。
    *    HA_CLUSTER_NODES="COMMA-SEPARATED_LIST_OF_NODES"
    *    这是应当要成为 NFS-Ganesha HA 群集一部分的主机的列表。
    *    VIP_HOSTNAME_WITH_DOTS_REPLACED_WITH_UNDERSCORES="FLOATING_IP"
    *    为群集中的每个节点分配一个虚拟 IP。当一个节点失败时，这些 IP 地址可以移到其他节点。

    c. 将 /etc/ganesha/ganesha-ha.conf 复制到组成群集的其他节点上。

3. 准备所有节点，以成为群集成员。

    a. 在所有节点上，启用并启动 pcsd 服务。
    b. 在所有节点上，启用 pacemaker 服务。
    c. 在所有节点上，设置 hacluster 用户的密码。
    d. 从一个节点，使用 pcs cluster auth LIST_OF_CLUSTER_NODES 命令。这将为 pcsd 服务和 pcs 命令在所有节点之间设置基于证书的身份验证。

4. 创建新的 SSH 密钥对，供 NFS-Ganesha 用于在节点之间复制配置文件。将此密钥对复制到所有节点上，并给予它免密码访问所有节点上 root 帐户的权限。

    a. 使用 ssh-keygen 创建名为 /var/lib/glusterd/nfs/secret.pem{,.pub} 的新密钥对。
    b. 使用 ssh-copy-id 允许此密钥对免密码访问所有节点的 root。
    c. 将密钥对复制到所有其他节点。

5. 在所有节点上启动 glusterd 服务。
6. 使用命令 gluster volume set all cluster.enable-shared-storage enable，为所有节点启用共享存储卷。此卷将用于在节点间跟踪 NFS 卷文件锁定。
7. 在所有节点的 /etc/ganesha/ganesha.conf 中为 NFS 配置辅助服务端口。

    将下面几行添加到 NFS_Core_Param 块中。

    MNT_PORT = 20048;

    这将配置 mountd 服务，使它与前面在防火墙中打开的端口相匹配。

    如果要使用 NFSv3，也请添加 NLM_Port = 32803; 这一行。若要为 lockd 设置固定端口，取消注释 /etc/sysconfig/nfs 中的 STATD_PORT 行，再重启 nfs-config 和 rpc-statd 服务。

8. 运行命令 glusterfs nfs-ganesha enable，以配置 pacemaker 群集。这将配置 Pacemaker 群集，为所有卷禁用内置的 NFS 服务器，以及执行其他操作。
9. 通过以下命令，为所有需要使用 NFS-Ganesha 导出的卷启用 NFS-Ganesha 导出：gluster volume set VOLUME ganesha.enable on。 

###### 修改导出选项

使用 NFS-Ganesha 导出卷后，目录 /etc/ganesha/exports 中将为该卷创建一个导出文件。此文件中可以配置额外的选项，如客户端限制和 Kerberos 选项等。有关可用选项及其语法的更多信息，请参见 /usr/share/doc/ganesha/config_samples/export.txt 文件。

在更新导出文件后，可以通过命令 /usr/libexec/ganesha/ganesha-ha.sh --refresh-config /etc/ganesha VOLUME 将它同步到群集并予以激活。 


### 引导式练习：配置 NFS Ganesha

要求使用 nfs-ganesha 导出 custdata 卷。nfs-ganesha 创建的群集应当取名为 gls-ganesha，它应当具有两个虚拟 IP：172.25.250.16/24 和 172.25.250.17/24。您的 servera 和 serverb 系统应当是此群集的成员。

客户端仅使用 NFSv4 进行连接，所以 NFSv3 特定端口不必永久映射，但客户端应当能够使用 showmount -e 查找可用的卷。导出的 custdata 卷应当永久可读写挂载到 workstation 上的 /mnt/nfs。 

1. 在 servera 和 serverb 上安装所需的软件包。

    a. 在 servera 和 serverb 上，停止 glusterd.service 服务，然后中断任何遗留的 glusterfs 和 glusterfsd 进程，因为它们会干扰 glusterfs-ganesha 软件包安装。

    [root@serverx ~]# systemctl stop glusterd
    [root@serverx ~]# killall glusterfs
    [root@serverx ~]# killall glusterfsd

    b. 在 servera 和 serverb 上，安装 glusterfs-ganesha 软件包。

    [root@serverx ~]# yum install glusterfs-ganesha

2. 更新 servera 和 serverb 上的防火墙，以允许 pacemaker/corosync 流量、NFS 流量和 portmapper 流量，并允许与 mountd 通信。

    a. 在 servera 和 serverb 系统上，打开防火墙中的相关服务：

    [root@serverx ~]# firewall-cmd --add-service=high-availability \
    > --add-service=nfs --add-service=rpc-bind --add-service=mountd

    b. 在两个系统上，保持防火墙配置。

    [root@serverx ~]# firewall-cmd --runtime-to-permanent

3. 在 servera 系统上，将示例文件 ganesha-ha.conf 复制到其正确的位置，然后使用以下信息更新该文件：

```
HA_NAME="gls-ganesha"
HA_VOL_SERVER="servera"
HA_CLUSTER_NODES="servera.lab.example.com,serverb.lab.example.com"
VIP_servera_lab_example_com="172.25.250.16"
VIP_serverb_lab_example_com="172.25.250.17"
```

删除任何未使用的 VIP_ 行，再将该文件复制到 serverb 计算机上。

    a. 在 servera 上，将示例 ganesha-ha.conf 文件复制到其正确的位置。

    [root@servera ~]# cp /etc/ganesha/ganesha-ha.conf{.sample,}

    b. 按照说明编辑 /etc/ganesha/ganesha-ha.conf，确保将所有变量内容正确放在引号内。

    c. 从 servera，将 /etc/ganesha/ganesha-ha.conf 复制到 serverb。

    [root@servera ~]# scp /etc/ganesha/ganesha-ha.conf serverb:/etc/ganesha/

4. 通过启用正确的服务、设置群集用户密码并且互相验证身份，准备 servera 和 serverb 以作为群集成员。

    a. 在这两个系统上，启用 pacemaker 和 pcsd 服务。

    [root@serverx ~]# systemctl enable pacemaker pcsd

    b. 在这两个系统上，启动 pcsd.service 服务。

    [root@serverx ~]# systemctl start pcsd

    c. 在这两个系统上，将 hacluster 用户的密码设为 redhat。

    [root@serverx ~]# echo redhat | passwd --stdin hacluster

    d. 从 servera 系统，对所有节点之间的 pcs 通信进行身份验证。

    [root@servera ~]# pcs cluster auth -u hacluster -p redhat \
    > servera.lab.example.com serverb.lab.example.com
    servera.lab.example.com: Authorized
    serverb.lab.example.com: Authorized

5. 创建 SSH 密钥对，以启用 nfs-ganesha 的免密码 SSH 通信。

    a. 在 servera 上，在 /var/lib/glusterd/nfs/ 中创建新 SSH 密钥对，名为 secret.pem。

    [root@servera ~]# ssh-keygen -f /var/lib/glusterd/nfs/secret.pem -t rsa -N ''

    b. 将密钥对复制到 serverb 上的相同位置。

    [root@servera ~]# scp /var/lib/glusterd/nfs/secret.pem* serverb:/var/lib/glusterd/nfs/

    c. 允许新密钥免密码访问 servera 和 serverb 上的 root 帐户。更改后，需要使用此访问权限在所有节点上刷新导出配置。

    [root@servera ~]# ssh-copy-id -i /var/lib/glusterd/nfs/secret.pem.pub root@servera
    [root@servera ~]# ssh-copy-id -i /var/lib/glusterd/nfs/secret.pem.pub root@serverb

6. 在两者节点上启动 glusterd，然后为 glusterd 启用共享存储。

    a. 在 servera 和 serverb 上启动 glusterd.service。

    [root@serverX ~]# systemctl start glusterd

    b. 为 glusterd 启动共享存储。此共享存储将用于存储状态信息，以在恢复 NFS 锁定时提供帮助。

    [root@servera ~]# gluster volume set all cluster.enable-shared-storage enable

7. 在两者节点上配置 nfs-ganesha，将默认端口 (20048/tcp/20048/UDP) 用于 mountd 进程。

    a. 在 servera 和 serverb 上，将下面这一行添加到 /etc/ganesha/ganesha.conf 中的 NFS_core_Param 块。不要忘记行末的分号：

    MNT_Port = 20048;

8. 为群集启用 nfs-ganesha，然后使用 nfs-ganesha 导出 custdata 卷。

    a. 在您的群集上启用 nfs-ganesha。

    [root@servera ~]# gluster nfs-ganesha enable
    Enabling NFS-Ganesha requires Gluster-NFS to be disabled across the trusted pool. Do you still want to continue? (y/n) y
    This will take a few minutes to complete. Please wait ..
    nfs-ganesha : success

    b. 使用 nfs-ganesha 导出 custdata。

    [root@servera ~]# gluster volume set custdata ganesha.enable on
    volume set: success

9. 将 custdata 卷永久可读写挂载到 workstation 上的 /mnt/nfs。

    a. 验证 custdata 卷已利用配置的虚拟 IP 地址通过 NFS 导出。

    [root@workstation ~]# showmount -e 172.25.250.16
    Export list for 172.25.250.16:
    /custdata (everyone)

    b. 在 workstation 上，创建 /mnt/nfs 挂载点。

    [root@workstation ~]# mkdir /mnt/nfs

    c. 将以下条目添加到 workstation 上的 /etc/fstab 中：

    172.25.250.16:/custdata /mnt/nfs nfs rw,vers=4 0 0

    d. 在 workstation 上挂载 custdata 卷。

    [root@workstation ~]# mount /mnt/nfs


### 实验：通过 CTDB 进行 IP 故障切换

serverc 和 serverd 计算机上部署了一个包含两个节点的红帽  Gluster 存储。这个群集已配置了两个卷：一个名为 ctdbmeta 的复制卷，以及一个名为 labdata 的复制卷。

您已被要求配置此群集，以使用 IP 172.25.250.18/24 提供 IP 故障切换，并且利用 Samba 通过此 IP 导出 labdata 卷。该共享应当可供 Samba 用户 smbuser（密码为 redhat）使用。两个节点上已创建了名为 smbuser 的本地帐户。ctdbmeta 卷应当用于存储此安装需要的任何锁文件以及其他元数据。在完成所有配置后，labdata 卷应当通过此浮动 IP 地址永久挂载到 workstation 上的 /mnt/labdata。 

1. 在 serverc 和 serverd 上安装所需的软件，并在它们的防火墙中打开此设置需要的所有端口。

    a. 在 serverc 和 serverd 计算机上，安装 samba 和 ctdb 软件包。

    [root@serverc ~]# yum install samba ctdb
    [root@serverd ~]# yum install samba ctdb

    b. 在两个系统上，打开 samba 服务，并在防火墙中打开端口 4379/tcp。

    [root@serverc ~]# firewall-cmd --add-service=samba
    [root@serverc ~]# firewall-cmd --add-port=4379/tcp
    [root@serverc ~]# firewall-cmd --runtime-to-permanent

    [root@serverd ~]# firewall-cmd --add-service=samba
    [root@serverd ~]# firewall-cmd --add-port=4379/tcp
    [root@serverd ~]# firewall-cmd --runtime-to-permanent

2. 停止 ctdbmeta 卷，然后在 serverc 和 serverd 上配置相关的启动和停止 hook，从而将 ctdbmeta 卷用于 CTDB。另外，在两个节点上为 Samba 启用群集。

    a. 停止 ctdbmeta 卷。

    [root@serverc ~]# gluster volume stop ctdbmeta
    Stopping volume will make its data inaccessible. Do you want to continue? (y/n) y
    volume stop: ctdbmeta: success

    b. 在 serverc 和 serverd 上，编辑 /var/lib/glusterd/hooks/1/start/post/S29CTDBsetup.sh 和 /var/lib/glusterd/hooks/1/stop/pre/S29CTDB-teardown.sh，使其 META="all" 行变为 META=ctdbmeta。

    c. 在 serverc 和 serverd 上，通过编辑 /etc/samba/smb.conf 将下面这一行包含在 [global] 部分中：

    clustering=yes

3. 启动 ctdbmeta 卷，然后配置 CTDB，从而将 serverc 和 serverd 系统用于 IP 故障切换并将 172.25.250.18/24 用作浮动 IP 地址。

    a. 启动 ctdbmeta 卷。

    [root@serverc ~]# gluster volume start ctdbmeta
    volume start: ctdbmeta: success

    b. 在 serverc 和 serverd 上，创建包含以下内容的文件 /etc/ctdb/nodes。确保该文件在两个节点上完全一致。此文件列出了参与通过 CTDB 进行 IP 故障切换的所有主机的 IP 地址。

    172.25.250.12
    172.25.250.13

    c. 在 serverc 和 serverd 上，创建包含以下内容的文件 /etc/ctdb/public_addresses。此文件配置应使用的浮动 IP 地址。

    172.25.250.18/24 eth0

    d. 在 serverc 和 serverd 上，启动并启用 ctdb.service 服务。

    [root@serverc ~]# systemctl enable ctdb
    [root@serverc ~]# systemctl start ctdb

    [root@serverd ~]# systemctl enable ctdb
    [root@serverd ~]# systemctl start ctdb

4. 配置 labdata 卷，以使用 Samba 导出。不要忘了为 smbuser 用户设置 Samba 密码 redhat。

    a. 将 smbuser 的 Samba 密码设置为 redhat。由于 ctdb 将此更改传播到所有节点，因此只需在一个主机上执行此步骤一次。

    [root@serverc ~]# smbpasswd -a smbuser
    New SMB password: redhat
    Retype new SMB password: redhat

    b. 为 labdata 卷设置以下选项：

    *    stat-prefetch off
    *    server.allow-insecure on
    *    storage.batch-fsync-delay-usec 0 

    [root@serverc ~]# gluster volume set labdata stat-prefetch off
    [root@serverc ~]# gluster volume set labdata server.allow-insecure on
    [root@serverc ~]# gluster volume set labdata storage.batch-fsync-delay-usec 0
                

    c. 在 serverc 和 serverd 上，将下面这一行添加到 /etc/glusterfs/glusterd.vol 的 type mgmt 块内，然后重启 glusterd.service。

    >    option rpc-auth-allow-insecure on

    [root@serverc ~]# systemctl restart glusterd
    [root@serverd ~]# systemctl restart glusterd

    d. 重启 labdata 卷，以使用 SMB 导出它。

    [root@servera ~]# gluster volume stop labdata
    Stopping volume will make its data inaccessible. Do you want to continue? (y/n) y
    volume stop: custd: success
    [root@servera ~]# gluster volume start labdata
    volume start: labdata: success

5. 在 workstation 系统上，利用 Samba 并通过浮动 IP 地址将 labdata 卷永久挂载到/mnt/labdata

    a. 创建 /mnt/labdata 挂载点。

    [root@workstation ~]# mkdir /mnt/labdata

    b. 在 workstation 上，将下面这一行添加到 /etc/fstab：

    //172.25.250.18/gluster-labdata /mnt/labdata cifs user=smbuser,pass=redhat 0 0

    c. 挂载 labdata 卷。

    [root@workstation ~]# mount /mnt/labdata


### 总结

在本章中，您已学会如何：

*    使用 rh-gluster-3-samba-for-rhel-7-server-rpms 频道中的软件包安装 CTDB。
*    使用各个节点上的 brick 创建复制卷，以用于 CTDB 元数据。
*    更新 META 卷的 hook 脚本。
*    使用 clustering=yes 配置 Samba。
*    在 /etc/ctdb/nodes 中配置 PNN。
*    在 /etc/ctdb/public_addresses 中配置 VIP。
*    使用 ctdb 命令。
*    使用 rh-gluster-3-nfs-for-rhel-7-server 和 rhel-ha-for-rhel-7-server-rpms 频道中的软件包安装 glusterfs-ganesha。
*    打开所有需要的防火墙端口，包括 mountd、rpc-bind、nfs 和 high-availability。
*    在 /etc/ganesha/ganesha-ha.conf 中配置 Ganesha HA 群集设置。
*    为 pcs 对节点进行身份验证。
*    创建 SSH 密钥对，以用于更新导出配置。
*    为辅助 NFS 服务配置固定端口。
*    使用 glusterfs nfs-ganesha enable 启用 NFS-Ganesha。
*    使用 gluster volume set VOLUME ganesha.enable on 通过 NFS-Ganesha 导出卷。
*    使用 ganesha-ha.sh 更新导出选项。 
