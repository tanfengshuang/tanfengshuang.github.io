---
layout: post
title:  "Configuring Clients(236-5)"
categories: Linux
tags: RHCA 236
---

### 使用原生客户端挂载卷

###### 红帽 Gluster 存储原生客户端

若要访问红帽 Gluster 存储卷，推荐使用的工具为原生客户端。原生客户端的构建基础是 FUSE （用户空间文件系统）技术。原生客户端支持 POSIX ACL 和自动故障切换。

和其他用于挂载红帽 Gluster 存储卷的选项不同，原生客户端不依赖于任何单一主机处于用状态。在挂载过程中，从指定的服务器或指定的任何备份服务器检索关于卷的信息；此后，原生客户端将直接与组成卷的 brick 交互。这与其他用于挂载卷的选项相比，可以实现更高的吞吐量和更强的可靠性。

###### 安装原生客户端

原生客户端默认安装在红帽  Gluster 存储服务器上。在 Red Hat Enterprise Linux 7 系统上，可从位于 rhel-x86_64-server-7-rh-gluster-3-client 频道和 rhel-x86_64-server-rh-common-7 频道的 glusterfs-fuse 软件包获取该客户端，但后一个频道可能会在未来弃用。

Red Hat Enterprise Linux6 客户端可以利用 rhel-x86_64-server-rhsclient-6 频道中的 glusterfs-fuse 软件包安装该客户端。Red Hat Enterprise Linux5 客户端应当使用 rhel-x86_64-server-rhsclient-5 频道。

> 重要: 所有客户端应当使用同一版本的原生客户端。在升级过程中，建议先升级所有服务器，然后再升级客户端。

在向此频道订阅计算机后，使用 yum 安装 glusterfs-fuse 软件包：

```
[root@demo ~]# yum install glusterfs-fuse
```

###### 更新原生客户端

更新原生客户端要求在更新过程中卸载所有已挂载的卷。例如，如果服务器 demo 具有两个用原生客户端挂载的卷（位于 /mnt/vol1 和 /mnt/vol2），应当采用以下步骤。

1. 使用原生客户端卸载所有卷。

    [root@demo ~]# umount /mnt/vol1
    [root@demo ~]# umount /mnt/vol2

2. 更新 glusterfs 和 glusterfs-fuse 软件包。

    [root@demo ~]# yum update glusterfs glusterfs-fuse
                
3. 重新挂载两个卷。

    [root@demo ~]# mount /mnt/vol1
    [root@demo ~]# mount /mnt/vol2

###### 使用原生客户端

使用 mount 命令手动挂载红帽存储卷，或者从 /etc/fstab 自动挂载。指定 glusterfs 作为文件系统类型，并将卷指代为 <STORAGE-SERVER>:<VOLUME>，其中 <STORAGE-SERVER> 是受信存储池中的一个存储服务器，而 <VOLUME> 则是要挂载的卷的名称。

> 重要: 原生客户端直接与卷中的不同 brick 通信；因此，添加服务器到存储池时所用的和添加 brick 时所使用的服务器主机名称和 IP 地址务必都要能够解析，并且能被客户端访问到。

使用原生客户端挂载卷时，可以指定多个不同的挂载选项。下表重点列出了一些比较常用的选项：

*    选项 	                                        用途
*    backup-volfile-servers=<SERVER1>:<SERVER2> 	如果卷地址中指定的服务器不可访问，则回退到使用此列表中指定的一个服务器来检索与卷相关的信息。可以通过冒号 (:) 来分隔多个服务器。
*    ro 	                                        将卷挂载为只读卷。
*    acl 	                                        在此卷上启用 POSIX ACL。使用原生客户端时，可以实施和设置 ACL。
*    _netdev 	                                    此选项并非由原生客户端本身使用，而是由系统启动逻辑使用。如果对于 /etc/fstab 中的某一个卷，此选项存在于其他选项的前面，该卷将等待网络启动之后再挂载。

在 Red Hat Enterprise Linux 7 上，此选项的使用是可选的，因为系统启动脚本知道 glusterfs 是网络文件系统。在 Red Hat Enterprise Linux 6 客户端上，需要使用此选项，才能在系统启动期间不会出现停滞。 

###### 挂载示例

下方为使用原生客户端挂载红帽  Gluster 存储卷的一些示例。

1. 使用 POSIX ACL 手动挂载卷

本例使用 POSIX ACL 将 volume 卷挂载到 /mnt/volume。

```
[root@demo ~]# mount -t glusterfs -o acl storage-server:/volume /mnt/volume
```

2. 当指定的主机不可访问时仍然挂载卷

本例使用 /etc/fstab 中的一个条目，将 volume 卷挂载到 /mnt/volume。它将尝试与主机 storage-server-1 通信来检索卷的相关信息，但会在该主机无法访问时回退到使用 storage-server-2 或 storage-server-3。

storage-server-1:/volume /mnt/volume glusterfs _netdev,backup-volfile-servers=storage-server-2:storage-server-3 0 0

> 重要: 请在 /etc/fstab 中指定 _netdev 挂载选项，因为需要正常工作的网络连接才能访问该卷。若无此选项，系统启动脚本将尝试在网络启动之前挂载此文件系统，这会导致失败。

###### 原生客户端挂载故障排除

原生客户端将每个卷与客户端相关的日志信息存储在 /var/log/glusterfs/mnt-<VOLNAME>.log 文件中。这些日志可用于辨别潜在的问题，或者在故障排除过程中利用。

这些日志文件将显示变为不可用状态的服务器、将使用哪些替代 backup-volfile-servers，以及与个别 brick 的通信问题等。


### 引导式练习：使用原生客户端挂载卷

将卷 servera:/custdata 永久挂载到 workstation 计算机的 /mnt/custdata 目录。即使 servera 或存储群集中的任何其他系统不可用，此挂载也应当要成功。在配置此永久挂载后，您应当测试卷在 servera 变为不可用状态时是否仍然保持可用。

1. 准备 workstation 系统，以使用原生客户端。

    在 workstation 系统上安装 glusterfs-fuse 软件包。

    [root@workstation ~]# yum -y install glusterfs-fuse

2. 将卷 servera:/custdata 永久挂载到 workstation 系统。确保即使 servera 或存储群集中的任何其他系统不可用，该卷也能挂载。

    a. 在 workstation 上，创建挂载点 /mnt/custdata。

    [root@workstation ~]# mkdir /mnt/custdata

    b. 将下面这一行添加到 workstation 上的 /etc/fstab。

    servera:/custdata /mnt/custdata glusterfs _netdev,backup-volfile-servers=serverb:serverc:serverd 0 0

    c. 在 workstation 上挂载 custdata 卷。

    [root@workstation ~]# mount /mnt/custdata

3. 测试在 servera 变为不可用时该卷是否仍然可以访问。

    a. 在 workstation，在 /mnt/custdata 中创建几个测试文件。

    [root@workstation ~]# touch /mnt/custdata/file{00..39}

    b. 关闭 servera 系统。

    c. 在 workstation 上，验证您仍然可以看到 /mnt/custdata 下的所有文件。

    [root@workstation ~]# ls -l /mnt/custdata

    在短暂停顿后（原生客户端会在此过程中遇到与 servera 服务器通信超时），其内容应当能正常列出。

    d. 在原生客户端的日志中检查 custdata 卷。您应该会看到 No route to host 消息。

    [root@workstation ~]# tail /var/log/glusterfs/mnt-custdata.log

4. 在 workstation 系统上，卸载 /mnt/custdata，然后尝试重新加载它。

    a. 在 workstation 系统上，卸载 /mnt/custdata。

    [root@workstation ~]# umount /mnt/custdata

    b. 在 workstation 系统上，挂载 /mnt/custdata。

    [root@workstation ~]# mount /mnt/custdata

    在短暂超时后（期间原生客户端将判断 servera 不可用），将使用 backup-volfile-servers 选项中的一个系统来检索与卷相关的信息，然后挂载该卷。

    c. 再次打开 servera 系统。 

### 使用 NFS 客户端挂载卷

###### 红帽  Gluster 存储卷和 NFSv3

默认情况下，所有新的红帽  Gluster 卷将通过 NFSv3 导出，并且会启用 ACL。这将使得不能运行原生客户端的客户端能够访问红帽  Gluster 存储卷上存储的数据，而不必添加专用的服务器来重新导出这些卷。

NFSv3 导出不使用 Linux 内核中的 NFSv3 服务器。相反，它们将使用为红帽  Gluster 存储专门编写的专用 NFSv3 服务器，它仅通过 TCP 导出。

与原生客户端不同，使用 NFSv3 的客户端无法在连接的服务器变为不可用状态时自动切换为使用其他服务器。这样的故障切换和 NFSv4 支持可以使用 fs-ganesha, 进行配置，具体如第 8 章 配置 IP 故障切换中所述。

###### 准备服务器以使用 NFSv3 导出。

除非为某一卷特地禁用了 NFSv3，否则在卷启动时就已通过 NFSv3（从所有主机）自动导出。要从某一客户端访问卷，用作 NFSv3 服务器的主机上的服务器必须修改为允许这一流量。

在使用 firewalld 时，必须允许以下两个服务通过防火墙，以便能启用 NFSv3：rpc-bind 和 nfs。rpc-bind 服务允许连接端口 111/TCP 和 111/UDP（portmapper 服务所需），而 nfs 服务则允许连接端口 2049/TCP（NFSv3 服务本身所需）。

```
[root@demo ~]# firewall-cmd --add-service=rpc-bind --add-service=nfs
[root@demo ~]# firewall-cmd --runtime-to-permanent
```

###### 配置客户端

希望利用 NFSv3 挂载卷的客户端应当安装有 nfs-utils 软件包。卷导出为 /<VOLNAME>，可以像一般的 NFSv3 文件系统一样挂载。

> 注意: Red Hat Enterprise Linux7 可以利用 NFSv3 挂载红帽  Gluster 存储卷，不需要任何额外的挂载选项，但仍然建议将 vers=3 选项添加到挂载，这样可避免不必要地先尝试协议版本 4。如果客户端希望先通过 UDP 尝试挂载，也可指定 proto=tcp 选项。

> 重要: 虽然也可以将红帽  Gluster 存储卷配置为通过 UDP 导出（借助 nfs.mount-udp 选项），但 UDP 导出不支持子目录挂载，也不遵守卷上所设的客户端限制。


### 引导式练习：使用 NFS 客户端挂载卷

要求将卷 servera:/mediadata 永久挂载到 workstation 计算机的 /mnt/mediadata 目录。 此挂载应当使用 NFSv3 客户端，并且应当要同时可读和可写。 

1. 准备 servera，以通过 NFSv3 提供卷。

    a. 在 servera 上，将 rpc-bind 和 nfs 服务添加到运行时防火墙配置。

    [root@servera ~]# firewall-cmd --add-service=rpc-bind --add-service=nfs

    b. 在 servera 上，将当前的防火墙配置存储为永久防火墙配置。

    [root@servera ~]# firewall-cmd --runtime-to-permanent

2. 将卷 servera:/mediadata 永久挂载到您的 workstation 系统。

    a. 在 workstation 上，创建挂载点 /mnt/mediadata。

    [root@workstation ~]# mkdir /mnt/mediadata

    b. 将下面这一行添加到 workstation 上的 /etc/fstab。

    servera:/mediadata /mnt/mediadata nfs rw 0 0

    c. 在 workstation 上挂载 mediadata 卷。

    [root@workstation ~]# mount /mnt/mediadata


### 使用 CIFS 导出功能挂载卷

###### 使用 SMB 导出红帽  Gluster 存储卷

红帽  Gluster 存储支持利用 Samba 4 通过 SMB 协议导出卷。在使用 samba-vfs-glusterfs 插件时，卷不需要挂载到导出 samba 共享的服务器上，因为 vfs 创建使用 GlusterFS API 来直接访问数据。

###### 配置服务器以使用 Samba 导出卷

在可以使用红帽  Gluster 存储服务器通过 SMB 导出卷前，需要做些准备工作。下列步骤将准备红帽  Gluster 存储服务器，以使用 SMB 导出卷。

1. 将服务器订阅到 rh-gluster-3-samba-for-rhel-7-server-rpms 频道。此频道提供了更新版本的 samba，以及直接访问卷所需的 VFS 插件。
2. 安装 samba 软件包。这将安装 Samba 4 主服务器，以及 samba-vfs-glusterfs 插件。
3. 启动并启用 smb.service 服务。
4. 在本地防火墙上打开 samba 服务。 

###### 配置要使用 Samba 导出的卷

在配置了服务器本身后，还需要对卷进行配置，从而能使用 Samba 来导出。

1. 禁用需要导出的卷的 stat-prefetch。

    [root@demo ~]# gluster volume set <VOLNAME> stat-prefetch off
          
2. 允许从无特权端口（端口号为 1024 或以上）与组成此卷的 brick 通信。

    [root@demo ~]# gluster volume set <VOLNAME> server.allow-insecure on

3. 将卷的 fsync 延时设置为零毫秒，从而确保适当的锁定和 I/O 一致性。

    [root@demo ~]# gluster volume set <VOLNAME> storage.batch-fsync-delay-usec 0

4. 将 glusterd 守护进程配置为允许来自无特权端口的通信，方法是将下面这一行添加到 /etc/glusterfs/gluster.vol 中的 volume management 部分。然后重新启动 glusterd.service。

    > option rpc-auth-allow-insecure on

    [root@demo ~]# systemctl restart glusterd.service

5. 启动或重启该卷。这将触发 hook 脚本 /var/lib/glusterd/hooks/1/post/S30samba-start.sh，向卷的 /etc/samba/smb.conf 添加导出。

6. 如果 samba 没有配置中央化用户信息和身份验证，则应当创建本地用户，并为该帐户创建 samba 密码，以便在身份验证期间使用。

例如，如要添加本地用户 smbuser 并将 Samba 密码设为 redhat：

```
[root@demo ~]# adduser -s /sbin/nologin smbuser
[root@demo ~]# smbpasswd -a smbuser
New SMB password: redhat
Retype new SMB password: redhat
```

###### 挂载红帽  Gluster 存储 Samba 卷

若要挂载使用 samba 导出的红帽  Gluster 存储卷，客户端需要安装 cifs-utils 软件包。此软件包提供 mount.cifs 命令和其他各种实用程序。

使用 samba 导出的红帽  Gluster 存储卷具有名称 gluster-<VOLNAME>，可以和任何其他 samba 共享一样使用。这意味着指定 cifs 文件系统类型，以及指定 user= 和 pass= 挂载选项或 credentials= 挂载选项。如果不使用这些选项，mount 命令将在挂载过程中要求提供凭据，而这极有可能并不是从 /etc/fstab 自动挂载时所希望的。

###### 禁用卷的自动共享

如果一个卷需要 samba 共享，而其他卷不需要，可通过将 user.cifs 或 user.smb 设置为 disable 来禁用卷的自动共享。用于在卷启动时自动添加共享的 hook 脚本将检查这些选项，存在其中之一时将停止添加。

例如，若要为 linuxonly 卷禁用通过 samba 自动共享：

    [root@demo ~]# gluster volume set linuxonly user.cifs disable


### 引导式练习：使用 CIFS 导出功能挂载卷

要求在为 servera 上的红帽  Gluster 存储卷配置 SMB 导出。目前，只有一个主机将用于导出 SMB 共享，也只有 mediadata 卷需要导出。 导出的 mediadata 卷应当永久挂载到 workstation 上的 /mnt/smbdata，其使用用户名 smbuser 和密码 redhat。 

1. 准备 servera 以通过 SMB 提供卷，具体操作为：安装必要的软件包，在防火墙上打开所有相关的端口，并且启动所有相关的服务。

    创建名为 smbuser 的用户，并将 Samba 密码设为 redhat。

    a. 在 servera 上永久开放防火墙，以允许 SMB 流量。

    [root@servera ~]# firewall-cmd --add-service=samba
    [root@servera ~]# firewall-cmd --runtime-to-permanent

    b. 安装所需的软件包，以使用 SMB 在 servera 上导出卷。

    [root@servera ~]# yum -y install samba

    c. 在 servera 上启动并启用 smb.service 服务。

    [root@servera ~]# systemctl enable smb.service
    [root@servera ~]# systemctl start smb.service

    d. 创建名为 smbuser 的用户，并将 samba 密码设为 redhat。

    [root@servera ~]# adduser smbuser
    [root@servera ~]# smbpasswd -a smbuser
    New SMB password: redhat
    Retype new SMB password: redhat

2. 通过 SMB 导出 mediadata。

    a. 设置所需的选项，以通过 SMB 在 mediadata 上导出卷。（stat-prefetch off、server.allow-insecure on 和 storage.batch-fsync-delay-usec 0）

    [root@servera ~]# gluster volume set mediadata stat-prefetch off
    [root@servera ~]# gluster volume set mediadata server.allow-insecure on
    [root@servera ~]# gluster volume set mediadata storage.batch-fsync-delay-usec 0

    b. 将 glusterd 守护进程配置为允许 Samba 使用非安全端口与 brick 通信，具体操作是将下面这一行添加到 servera 上 /etc/glusterfs/glusterd.vol 的 volume management 部分中。完成此更改后，重启 glusterd.service 服务。

    > option rpc-auth-allow-insecure on

    [root@servera ~]# systemctl restart glusterd.service

    c. 重启 mediadata 卷，以通过 SMB 导出它。

    [root@servera ~]# gluster volume stop mediadata
    Stopping volume will make its data inaccessible. Do you want to continue? (y/n) y
    volume stop: mediadata: success
    [root@servera ~]# gluster volume start mediadata
    volume start: mediadata: success

3. 使用 SMB，将 mediadata 卷永久挂载到 workstation 上的 /mnt/smbdata。

    a. 在 workstation，验证该共享可供 smbuser 用户访问。

    [root@workstation ~]# smbclient -L servera -U smbuser%redhat
    ...
      gluster-mediadata Disk  For samba share of volume mediadata
    ...

    b. 在 workstation 上创建 /mnt/smbdata 挂载点。

    [root@workstation ~]# mkdir /mnt/smbdata

    c. 向 workstation 上的 /etc/fstab 添加条目，以使用 SMB 将 mediadata 卷永久挂载到 /mnt/mediadata。使用用户名 smbuser 和密码 redhat。与普通的安全实践不同，该密码应当在 fstab 文件中指定。在生产系统中，该密码应当藏匿于只能由 root 读取的文件中，或者应当使用 AD/Kerberos 机器验证。

    将下面这一行添加到 /etc/fstab：

    //servera/gluster-mediadata /mnt/smbdata cifs user=smbuser,pass=redhat 0 0

    d. 在 workstation 上挂载该卷。

    [root@workstation ~]# mount /mnt/smbdata


### 设置卷选项

使用红帽  Gluster 存储时，可以设置特定于卷的选项。这些选项的范围包括卷的访问控制和低级调优选项等。设置选项的基本语法是 gluster volume set <VOLUME NAME> <OPTION> <VALUE>。

*    gluster volume reset <VOLUME NAME> <OPTION>: 要将选项重置为其默认设置
*    gluster volume info <VOLUME NAME>: 查询为卷设置的所有选项
*    gluster volume set help: 请求所有内置选项的（几乎）完整列表，包括选项的默认值和简短说明
*    gluster volume get <VOLUME NAME> all: 查看已为卷激活的所有选项，将 all 替换为某一选项的名称，即可仅显示该选项。

下方列出了一些实用的选项及其说明：

*    选项 	                默认值 	描述
*    auth.allow 	        * 	    以逗号分隔的客户端列表，这些客户端被允许使用原生客户端连接此卷。可以使用通配符，如 10.0.0.*。
*    auth.reject 		            以逗号分隔的客户端列表，这些客户端绝不应被允许使用原生客户端连接此卷。如果某一客户端同时列在 auth.allow 和 auth.reject 中，该客户端将被拒绝。允许使用通配符。
*    nfs.rpc-auth-allow 	* 	    以逗号分隔的客户端列表，这些客户端被允许使用 NFSv3 客户端连接此卷。允许以 * 的形式使用通配符。
*    nfs.rpc-auth-reject 		    以逗号分隔的客户端列表，这些客户端绝不应被允许使用 NFSv3 客户端连接此卷。允许使用通配符。如果某一客户端同时列在 reject 和 allow 中，它将被拒绝。
*    nfs.disable 	  	            如果设置为 on，此卷不通过 NFSv3 导出。
*    features.read-only 	关闭 	此选项决定卷是否导出为对所有访问它的客户端只读。
*    server.root-squash 	关闭 	如果设置为 on，客户端上的 root 用户将映射为 nfsnobody 用户，这与在 NFS 导出时使用 rootsquash 选项相同。

您也可以为卷设置用户不定义的选项。这些选项将以 user. 开头。与内置选项不同，用户定义的选项无法重置。

用户定义的选项可以在您自己的 (hook-) 脚本中使用。例如，samba hook 脚本查询 user.cifs 选项，如果选项设为 disable，则不通过 CIFS 导出卷。

> 重要: 在更改访问规则时，偶而会需要停止再启动卷，以便让更改生效。 


### 引导式练习：设置卷选项

要求为 galactica 卷启用 server.root-squash 选项。此卷应当利用 NFSv3 挂载到 workstation 上的挂载点 /mnt/galactica，并设有可读写权限。

1. 配置 servera，使其允许 NFS 流量通过防火墙。

    a. 在防火墙上打开 rpc-bind 和 nfs 服务。

    [root@servera ~]# firewall-cmd --add-service=rpc-bind --add-service=nfs

    b. 将 servera 上的当前防火墙配置保存为永久防火墙配置。

    [root@servera ~]# firewall-cmd --runtime-to-permanent

2. 在 servera 上，将相关的卷选项应用到 galactica 卷。

    a. 将 galactica 卷的 server.root-squash 选项设为 on。

    [root@servera ~]# gluster volume set galactica server.root-squash on

3. 配置 workstation，以利用 NFSv3 将 servera:/galactica 卷永久挂载到 /mnt/galactica，并设有可读写权限。

    a. 创建 /mnt/galactica 挂载点。

    [root@workstation ~]# mkdir /mnt/galactica

    b. 将以下条目添加到 workstation 上的 /etc/fstab：

    servera:/galactica /mnt/galactica nfs rw 0 0

    c. 挂载卷。

    [root@workstation ~]# mount /mnt/galactica


### 实验：配置客户端

要求配置两个卷，以限制客户端访问。您还必须配置原生客户端和 NFS 客户端。下表列出了各个卷及其客户端的要求。

*     	                wallace	                                    gromit
*    NFS	            禁用	                                        启用
*    允许的 NFS 客户端 	不适用	                                    172.25.250.*
*    禁止的 NFS 客户端 	不适用	                                    172.25.250.254
*    允许的原生客户端 	    172.25.250.254	                            不适用
*    禁止的原生客户端 	    除允许以外全部	                                不适用
*    挂载于	            /mnt/wallaceworkstation，利用原生客户端。 	    /mnt/gromitservere，读/写访问，使用 NFSv3。

1. 配置 servera，使其允许 NFS 流量通过防火墙。

    a. 在防火墙上打开 rpc-bind 和 nfs 服务。

    [root@servera ~]# firewall-cmd --add-service=rpc-bind --add-service=nfs

    b. 将 servera 上的当前防火墙配置保存为永久防火墙配置。

    [root@servera ~]# firewall-cmd --runtime-to-permanent

2. 在 servera 上，将所有相关的卷选项应用到 wallace 卷。

    a. 禁用 wallace 卷的 NFS。

    [root@servera ~]# gluster volume set wallace nfs.disable on

    b. 将 172.25.250.254 (workstation) 配置为 wallace 卷唯一允许的客户端。

    [root@servera ~]# gluster volume set wallace auth.allow 172.25.250.254

    c. 重启 wallace 卷，以确保所有验证更改都已激活。

    [root@servera ~]# gluster volume stop wallace
    [root@servera ~]# gluster volume start wallace

3. 在 servera 上，为 gromit 卷配置所有相关的选项。

    a. 确保已经为 gromit 卷启用 NFS。此步为可选；NFS 默认为启用，但显式启用它可以提示其他系统管理员它已在使用中。

    [root@servera ~]# gluster volume set gromit nfs.disable off

    b. 允许 172.25.250.0/24 (lab.example.com) 中的所有客户端通过 NFS 访问 gromit 卷。

    [root@servera ~]# gluster volume set gromit nfs.rpc-auth-allow '172.25.250.*'

    c. 显式拒绝 172.25.250.254 (workstation) 通过 NFS 访问 gromit 卷。

    [root@servera ~]# gluster volume set gromit nfs.rpc-auth-reject 172.25.250.254

4. 配置 workstation，以使用原生客户端将 servera:/wallace 卷永久挂载到 /mnt/wallace。

    a. 安装所需的客户端软件。

    [root@workstation ~]# yum -y install glusterfs-fuse

    b. 创建 /mnt/wallace 挂载点。

    [root@workstation ~]# mkdir /mnt/wallace

    c. 将以下条目添加到 workstation 上的 /etc/fstab：

    servera:/wallace /mnt/wallace glusterfs defaults 0 0

    d. 挂载卷。

    [root@workstation ~]# mount /mnt/wallace

5. 配置 servere，以使用 NFS 将 servera:/gromit 卷永久挂载到 /mnt/gromit.

    a. 创建 /mnt/gromit 挂载点。

    [root@servere ~]# mkdir /mnt/gromit

    b. 将以下条目添加到 servere 上的 /etc/fstab：

    servera:/gromit /mnt/gromit nfs rw 0 0

    c. 挂载卷。

    [root@servere ~]# mount /mnt/gromit


### 总结

在本章中，您已学会如何：

*    安装原生客户端。
*    使用原生客户端挂载卷。
*    通过先卸载所有卷来更新原生客户端。
*    为使用 NFSv3 导出卷，打开防火墙上的相关端口。
*    配置客户端以使用 NFSv3 挂载卷。
*    通过打开正确的防火墙端口并安装正确的防火墙，对服务器进行准备，以使用 SMB 导出卷。
*    配置卷以使用 SMB 导出，包括允许来自无特权端口的连接。
*    使用 SMB 挂载卷。
*    如何通过添加 user.cifs disable 选项来禁用卷的 SMB 共享。 
