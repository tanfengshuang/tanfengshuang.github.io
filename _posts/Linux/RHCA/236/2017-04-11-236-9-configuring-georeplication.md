---
layout: post
title:  "Configuring Georeplication(236-9)"
categories: Linux
tags: RHCA 236
---

### 配置异地复制

###### 异地复制功能

异地复制在两个卷之间提供异步的单向增量复制。这表示对主卷进行的更改将同步到从卷，但这种同步不是即刻执行的。通过运行修改版本的 rsync （称为 gsyncd），在主卷和从卷之间使用 SSH 连接。

异地复制可以在同一主机上的卷之间配置，也可以在本地卷和远程主机上的卷之间配置。这一远程主机可以使用同一数据中心的 LAN、WAN 或者互联网进行连接。

异地复制和复制卷并不相同。复制卷镜像同一受信存储池中 brick 之间的数据，而异地复制则镜像（地理上分散的）受信存储池之间的数据。复制卷用于实现高可用性，而异地复制则用于备份和灾难恢复。还有一个区别，复制卷是同步的，任何更改都会立即写入所有 brick；而异地复制是异步的，更改会以某种间隔批量同步。

异地复制还能以级联的方式配置。单个主卷可以同步到多个从卷，这些从卷又可以各自同步到一个或多个从卷。这可用于实现数据同步到全球各地多个数据中心的设置。 

在出现灾难性故障时，从卷可以被提升，客户端可以切换为使用该从卷来访问数据。当主卷恢复在线时，任何新的或更改的数据可以重新同步回该主卷。

###### 异地复制前提条件

在可以配置异地复制前，必须先满足几个前提条件。

*    主卷和从卷必须在使用相同版本红帽  Gluster 存储的实例上。
*    从节点不可是主受信存储池中任何节点的对等点。
*    主卷的某一节点（即运行 geo-replication create 命令的节点）的 root 帐户和从节点上用于异地复制的帐户之间需要免密码 SSH 访问。 

###### 配置异地复制

异地复制可以通过两种方式来配置：第一种方式使用从节点上的 root 帐户，第二种使用非特权用户帐户。第二种方式被视为更加安全，因为不授予主节点 root 访问权限。在本课程中，我们将配置使用非特权用户帐户的更安全方式。 

使用从节点上非特权帐户配置异地复制，需要执行几个步骤。 

1. 在从节点上，创建用于单个卷异地复制的非特权用户，如 geoaccount，以及用于所有异地复制用户的组，如 geogroup。将新用户添加到新组中。
2. 在从节点上，配置 mountbroker。mountbroker 是一个守护进程，非特权用户可通过它挂载文件系统。

    a. 为 mountbroker 创建目录 (/var/mountbroker-root)，权限设为 0711，再确保 SELinux 将该目录标记为 home_root_t，与 /home 相同。
    b. 在从节点上运行以下命令：

        a). 配置 mountbroker 的根目录：

        [root@slave ~]# gluster system:: execute mountbroker \
        > opt mountbroker-root /var/mountbroker-root

        b). 设置用于从卷的用户帐户：

        [root@slave ~]# gluster system:: execute mountbroker \
        > user GEOACCOUNT SLAVE-VOLUME

        > 注意: 可以将同一帐户用于多个卷。在这种情况下，可以将从卷指定为逗号分隔列表。

        c). 配置要用于日志记录的组：

        [root@slave ~]# gluster system:: execute mountbroker \
        > opt geo-replication-log-group GEOGROUP

        d). 允许来自端口号 1024 以上端口的 RPC 调用：

        [root@slave ~]# gluster system:: execute mountbroker \
        > opt rpc-auth-allow-insecure on

    这些命令将更新 /etc/glusterfs/glusterd.vol，以在受信存储池中的所有节点上反映更改。

    可以通过命令 gluster system:: execute mountbroker info 查看完整的 mountbroker 配置。
    
    > 注意: gluster system:: execute mountbroker是用于在所有节点上调用脚本 /usr/libexec/glusterfs/peer_mountbroker 的打包程序。

    c. 在所有从节点上重启 glusterd。 

3. 在其中一个主节点上的 root 和从节点上的 geoaccount 帐户之间设置免密码 SSH 身份验证，然后使用该身份验证来创建和分发用于异地复制的 SSH 密钥对。

    a. 在其中一个主节点上，以 root 用户身份并使用 ssh-keygen 创建新的密钥对。
    b. 使用 ssh-copy-id 将新密钥对的公钥复制到从节点上的 geoaccount 帐户。
    c. 在主节点上，使用命令 gluster system:: execute gsec_create 在受信存储池中所有节点的 /var/lib/glusterd/geo-replication/ 中创建 SSH 密钥对。

4. 在主节点上，创建新的异地复制协议，再将为异地复制创建的 SSH 密钥对推送到从节点。

```
[root@master ~]# gluster volume geo-replication MASTER-VOLUME \
> GEOACCOUNT@SLAVE-NODE::SLAVE-VOLUME create push-pem
```

5. 在从节点上，执行 /usr/libexec/glusterfs/set_geo_rep_pem_keys.sh GEOACCOUNT MASTER-VOLUME SLAVE-VOLUME。这会将前面步骤中推送的 SSH 密钥对复制到正确的位置，并将它们复制到从属受信存储池中的其他节点上。 

6. 对主卷进行异地复制准备，具体为：添加共享存储，再启用元数据卷以用于异地复制协议。此元数据卷将存储共享存储上关于异地复制状态的信息。

    a. 在其中一个主节点上，启用共享存储：

    [root@master ~]# gluster volume set all cluster.enable-shared-storage enable

    b. 启用将元数据卷用于异地复制协议。

    [root@master ~]# gluster volume geo-replication MASTER-VOLUME \
    > GEOACCOUNT@SLAVE-NODE::SLAVE-VOLUME config use_meta_volume true

7. 启动异地复制。

[root@master ~]# gluster volume geo-replication MASTER-VOLUME \
> GEOACCOUNT@SLAVE-NODE::SLAVE-VOLUME start

8. 查看异地复制的状态：

[root@master ~]# gluster volume geo-replication MASTER-VOLUME \
> GEOACCOUNT@SLAVE-NODE::SLAVE-VOLUME status


### 引导式练习：配置异地复制

将 servera 和 serverb 上的复制卷 mastervol 异地复制到 servere 上的 slavevol 卷。

为限制对 servere 上 root 帐户的访问，必须对异地复制进行配置，以使用非特权用户 geoaccount 进行通信。此用户必须属于 geogroup 组，并且该组必须有权限访问 servere 上与此复制相关的所有日志文件。已经为您创建了此用户和组。geoaccount 用户的密码为 redhat。

1. 在提供 mastervol 卷的受信存储池上启用共享存储。此共享存储供异地复制守护进程用于在节点消失时触发故障切换。

    a. 启用共享存储：

    [root@servera ~]# gluster volume set all cluster.enable-shared-storage enable

2. 设置从 servera 上 root 帐户到 servere 上 geoaccount 帐户的免密码 SSH 访问权限。稍后配置异地复制密钥时，需要此访问权限。

    a. 在 servera 上，创建免密码的新 SSH 密钥对。

    [root@servera ~]# ssh-keygen -f ~/.ssh/id_rsa -N ''

    b. 允许新 SSH 密钥免密码 SSH 访问 servere 上的 geoaccount 帐户。

    [root@servera ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub geoaccount@servere
    The authenticity of host 'servere (172.25.250.14)' can't be established.
    ECDSA key fingerprint is f3:3a:20:c9:5a:cc:cc:f0:44:f7:00:90:03:18:b1:8d.
    Are you sure you want to continue connecting (yes/no)? yes
    /usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
    /usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
    geoaccount@servere's password: redhat

    Number of key(s) added: 1

    Now try logging into the machine, with:   "ssh 'geoaccount@servere'"
    and check to make sure that only the key(s) you wanted were added.

3. 在 servere 上，创建名为 /var/mountbroker-root 的新目录。此目录必须创建为具有权限 0711，并且其 SELinux 上下文与 /home 相同。此目录将用于在异地复制期间承载（非特权）挂载。

    a. 在 servere 上创建 /var/mountbroker-root 目录，权限设为 0711。

    [root@servere ~]# mkdir -m 0711 /var/mountbroker-root

    b. 配置 SELinux，让 /var/mountbroker-root 的上下文与 /home 相同。

    [root@servere ~]# semanage fcontext -a -e /home /var/mountbroker-root

    c. 在 /var/mountbroker-root 上恢复（新的）SELinux 上下文。

    [root@servere ~]# restorecon -Rv /var/mountbroker-root

4. 在 servere 上，配置下列选项，然后重启 glusterd 服务。

*    将 mountbroker-root 目录设置为 /var/mountbroker-root。
*    将 mastervol 的 mountbroker 用户设置为 geoaccount。
*    将 geo-replication-log-group 组设置为 geogroup。
*    允许来自非特权端口的 RPC 连接。 

    a. 将 mountbroker-root 目录设置为 /var/mountbroker-root。

    [root@servere ~]# gluster system:: execute mountbroker \
    > opt mountbroker-root /var/mountbroker-root

    b. 将 slavevol 卷的 mountbroker 用户设置为 geoaccount。

    [root@servere ~]# gluster system:: execute mountbroker \
    > user geoaccount slavevol

    c. 将 geo-replication-log-group 组设置为 geogroup。

    [root@servere ~]# gluster system:: execute mountbroker \
    > opt geo-replication-log-group geogroup

    d. 允许来自非特权端口的 RPC 连接。

    [root@servere ~]# gluster system:: execute mountbroker \
    > opt rpc-auth-allow-insecure on

    e. 在 servere 上重新启动 glusterd 服务。

    [root@servere ~]# systemctl restart glusterd

5. 使用 servere 上的 geoaccount 帐户，配置并启动 servera 和 serverb 上的 mastervol 卷和 servere 上的 slavevol 卷之间的异地复制。

    a. 在 servera 上，创建用于各个节点的异地复制守护进程的 SSH 密钥对。

    [root@servera ~]# gluster system:: execute gsec_create

    b. 在 servera 上，创建并推送用于异地复制的 SSH 密钥。

    [root@servera ~]# gluster volume geo-replication mastervol \
    > geoaccount@servere::slavevol create push-pem

    c. 在 servere 上，将上一步中推送的密钥复制到正确的位置。

    [root@servere ~]# /usr/libexec/glusterfs/set_geo_rep_pem_keys.sh \
    > geoaccount mastervol slavevol

    d. 在 servera 上，配置 mastervol 和 slavevol 之间的异地复制链接，以使用共享存储满足跟踪更改等需要。

    [root@servera ~]# gluster volume geo-replication mastervol \
    > geoaccount@servere::slavevol config use_meta_volume true

    e. 启动 mastervol 和 slavevol 之间的异地复制。

    [root@servera ~]# gluster volume geo-replication mastervol \
    > geoaccount@servere::slavevol start

6. 验证异地复制现在正常工作。

    a. 在 servera 上，查看所有异地复制链接的状态。

    [root@servera ~]# gluster volume geo-replication status

    MASTER NODE                MASTER VOL    MASTER BRICK              SLAVE USER    SLAVE                                 SLAVE NODE    STATUS     CRAWL STATUS       LAST_SYNCED
    -------------------------------------------------------------------------------
    -------------------------------------------------------------------------------
    -----------------------
    servera.lab.example.com    mastervol     /bricks/brick-a1/brick    geoaccount    ssh://geoaccount@servere::slavevol    servere       Active     Changelog Crawl    2016-05-17 16:10:04
    serverb.lab.example.com    mastervol     /bricks/brick-b1/brick    geoaccount    ssh://geoaccount@servere::slavevol    servere       Passive    N/A                N/A

    如果有任何 brick 报告为 Initializing，请等待几秒后重新运行该命令。如果有任何 brick 报告为 Faulty，请检查您的配置。

    b. 在 servere 上，验证文件正在复制到承载 slavevol 卷的 brick (/brick/brick-e1/brick/)。

    [root@servere ~]# ls /bricks/brick-e1/brick
    file00
    file01
    ...
    file98
    file99


### 管理异地复制

###### 调优异地复制选项

可以修改异地复制配置的不同选项。这包括日志文件的位置设置，以及是否应当在从卷上删除文件等。要查看异地复制协议的所有可用选项及其当前设置，可使用以下命令：

[root@master ~]# gluster volume geo-replication MASTERVOL GEOACCOUNT@SLAVENODE::SLAVEVOL config

可以使用以下语法来更新选项：

[root@master ~]# gluster volume geo-replication MASTERVOL GEOACCOUNT@SLAVENODE::SLAVEVOL config NAME VALUE

如下为可以使用的部分选项：

1. ignore-deletes 
    默认情况下，此设置设为 false。如果设为 true，主卷上删除的文件不会同时在从卷上删除。

2. checkpoint

*    通过设置检查点，可以轻松查看特定日期和时间之前的所有更改是否已经同步。此选项可取两个可能值：now，将检查点设置为当前日期和时间；或者，以时期 (date +%s) 起秒数表示的时间。
*    在设置检查点后，可以在异地复制协议的 status detail 输出中查看检查点状态。
*    要删除检查点，请在设置该选项时使用不带值的 '!checkpoint' 名称。

3. 也可以直接在主卷上设置影响异地复制的选项。其中一个选项是 changelog.rollover-time，它决定以什么频率检查更改日志中要同步到从卷的更改。此设置的默认值为 15 秒，但也可配置其他时间。对于一般的操作，建议设置 10 到 15 秒之间的时间。例如，若要将翻转时间设为 5 秒，可以使用以下命令：

    [root@master ~]# gluster volume set MASTERVOL changelog.rollover-time 5

###### 添加新节点或 brick

添加新 brick 到启用异地复制的卷时，已经配置了异地复制的节点上不需要执行任何操作。红帽  Gluster 存储将自动为受影响的卷重启异地复制守护进程。

如果在尚不属于异地复制协议的节点上添加 brick，需要执行一些额外的步骤。

1. 从可以免密码 SSH 访问已配置从属节点的节点，运行命令 gluster system:: execute gsec_create。这将为尚未配置异地复制的任何主机创建 SSH 密钥对。
2. 从可以免密码 SSH 访问已配置从属节点的节点，运行命令 gluster volume geo-replication MASTERVOL GEOACCOUNT@SLAVENODE::SLAVEVOL create push-pem force。这会将新密钥对推送到所有从节点。
3. 如果使用元数据卷，则启用 gluster_shared_storage 卷以挂载到新节点的 /var/run/gluster/shared_storage。
4. 停止并启动异地复制。
5. 验证异地复制会话的状态。 
    
###### 提升从卷

当主卷出现故障时，从卷可以用作客户端的新卷。在将客户端指向从卷时前，先在该卷上设置以下两个卷选项。这有助于在主卷再次可用时将更改同步回主卷。

[root@slave ~]# gluster volume set SLAVEVOL geo-replication.indexing on
[root@slave ~]# gluster volume set SLAVEVOL changelog on

当主卷再次可用时，可通过下列步骤将更改同步到主卷上：

1. 创建自从卷到主卷的新异地复制会话，但不要启动它。
2. 将新会话的 special-sync-mode 选项设置为 recover。
3. 停止所有访问从卷的 I/O，再为新复制协议设置检查点 now。
4. 启动新会话，再监控其状态，直至检查点被标记为已完成。
5. 当所有数据都同步回主卷后，停止新的复制协议。
6. 重置之前在从卷上设置的选项。

    [root@slave ~]# gluster volume reset SLAVEVOL geo-replication.indexing force
    [root@slave ~]# gluster volume reset SLAVEVOL changelog

7. 将客户端重新指向原始主卷。

###### 异地复制日志文件

异地复制的日志存储在多个位置上。在充当主节点的节点上，可在 /var/log/glusterfs/geo-replication/ 中找到异地复制相关的文件，每个正在异地复制的卷都有一个子目录。

主节点和从节点都将日志文件存储在 /var/log/glusterfs/geo-replication-slaves/ 中。主节点存储一个名为 slave.log 的文件，而从节点则为每个从卷存储一个单独的文件，并在 mbr 子目录中为每个从卷存储一个文件。 


### 引导式练习：管理异地复制

您已被要求通过以下设置更新此异地复制协议：

*    应当每 5 秒解析更改日志，以查看是否有需要同步的新更改。
*    mastervol 上删除的文件不应同时从 slavevol 删除。
*    应当使用当前日期和时间创建一个新检查点。

1. 将 mastervol 卷的 changelog.rollover-time 设置更新为 5 秒。

    a. 通过原生客户端挂载卷时，无法更新 rollover-time 设置。从 workstation 卸载 /mnt/mastervol。

    [root@workstation ~]# umount /mnt/mastervol

    b. 在 servera 上，将 mastervol 的 changelog.rollover-time 设置设为 5 秒。

    [root@servera ~]# gluster volume set mastervol changelog.rollover-time 5

    c. 在 workstation 系统上，重新挂载 /mnt/mastervol。

    [root@workstation ~]# mount /mnt/mastervol

2. 配置异地复制协议，以在 slavevol 上保留从 mastervol 删除的文件。

    a. 对于 mastervol 和 geoaccount@servere::slavevol 之间的异地复制协议，将 ignore-deletes 配置选项设为 true。

    [root@servera ~]# gluster volume geo-replication mastervol \
    > geoaccount@servere::slavevol config ignore-deletes true

    b. 在 workstation 上删除 /mnt/mastervol/importantfile 文件，以测试此设置。

    [root@workstation ~]# rm /mnt/mastervol/importantfile

    c. 在 servera 上，使用 gluster volume geo-replication status 的输出来观察是否发生了同步。重复此命令，直到执行了最新的同步。相关的卷是 LAST_SYNCED。

    [root@servera ~]# gluster volume geo-replication status

    d. 验证 importantfile 仍然位于 servere 上的 /bricks/brick-e1/brick 中。

    [root@servere ~]# ls -l /bricks/brick-e1/brick/importantfile

3. 使用当前的日期和时间，为异地复制协议创建检查点。

    a. 创建检查点 now。

    [root@servera ~]# gluster volume geo-replication mastervol \
    > geoaccount@servere::slavevol config checkpoint now

    b. 验证是否已创建了检查点：

    [root@servera ~]# gluster volume geo-replication mastervol \
    > geoaccount@servere::slavevol status detail


### 实验：配置异地复制

将 serverc 和 serverd 上的复制卷 datavol 异地复制到 servere 上的 backupvol 卷。

为限制对 servere 上 root 帐户的访问，必须对异地复制进行配置，以使用 geouser 非特权用户进行通信。此用户必须属于 geogroup 组，并且该组必须有权限访问 servere 上与此复制相关的所有日志文件。已经为您创建了此用户和组。geouser 用户的密码为 redhat。应当每 5 秒在更改日志中扫描要同步的数据，并且从 datavol 删除的文件不应同时从 backupvol 删除。 

1. 在提供 datavol 卷的受信存储池上启用共享存储。

    a. 启用共享存储：

    [root@serverc ~]# gluster volume set all cluster.enable-shared-storage enable

2. 设置从 serverc 上 root 帐户到 servere 上 geouser 帐户的免密码 SSH 访问权限。稍后配置异地复制密钥时，需要此访问权限。

    a. 在 serverc 上，创建免密码的新 SSH 密钥对。

    [root@serverc ~]# ssh-keygen -f ~/.ssh/id_rsa -N ''

    b. 允许新 SSH 密钥免密码 SSH 访问 servere 上的 geouser 帐户。

    [root@serverc ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub geouser@servere
    The authenticity of host 'servere (172.25.250.14)' can't be established.
    ECDSA key fingerprint is f3:3a:20:c9:5a:cc:cc:f0:44:f7:00:90:03:18:b1:8d.
    Are you sure you want to continue connecting (yes/no)? yes
    /usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
    /usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
    geouser@servere's password: redhat

    Number of key(s) added: 1

    Now try logging into the machine, with:   "ssh 'geouser@servere'"
    and check to make sure that only the key(s) you wanted were added.

3. 在 servere 上，创建名为 /var/mountbroker-root 的新目录。此目录必须创建为具有权限 0711，并且其 SELinux 上下文与 /home 相同。此目录将用于在异地复制期间承载（非特权）挂载。

    a. 在 servere 上创建 /var/mountbroker-root 目录，权限设为 0711。

    [root@servere ~]# mkdir -m 0711 /var/mountbroker-root

    b. 配置 SELinux，让 /var/mountbroker-root 的上下文与 /home 相同。

    [root@servere ~]# semanage fcontext -a -e /home /var/mountbroker-root

    c. 在 /var/mountbroker-root 上恢复（新的）SELinux 上下文。

    [root@servere ~]# restorecon -Rv /var/mountbroker-root

4. 在 servere 上，配置下列选项，然后重启 glusterd 服务。

*    将 mountbroker-root 目录设置为 /var/mountbroker-root。
*    将 backupvol 的 mountbroker 用户设置为 geouser。
*    将 geo-replication-log-group 组设置为 geogroup。
*    允许来自非特权端口的 RPC 连接。 

    a. 将 mountbroker-root 目录设置为 /var/mountbroker-root。

    [root@servere ~]# gluster system:: execute mountbroker \
    > opt mountbroker-root /var/mountbroker-root

    b. 将 backupvol 卷的 mountbroker 用户设置为 geouser。

    [root@servere ~]# gluster system:: execute mountbroker \
    > user geouser backupvol

    c. 将 geo-replication-log-group 组设置为 geogroup。

    [root@servere ~]# gluster system:: execute mountbroker \
    > opt geo-replication-log-group geogroup

    d. 允许来自非特权端口的 RPC 连接。

    [root@servere ~]# gluster system:: execute mountbroker \
    > opt rpc-auth-allow-insecure on

    e. 在 servere 上重新启动 glusterd 服务。

    [root@servere ~]# systemctl restart glusterd

5. 使用 servere 上的 geouser 帐户，配置并启动 serverc 和 serverd 上的 datavol 卷和 servere 上的 backupvol 卷之间的异地复制。

    a. 在 serverc 上，创建用于各个节点的异地复制守护进程的 SSH 密钥对。

    [root@serverc ~]# gluster system:: execute gsec_create

    b. 在 serverc 上，创建并推送用于异地复制的 SSH 密钥。

    [root@serverc ~]# gluster volume geo-replication datavol \
    > geouser@servere::backupvol create push-pem

    c. 在 servere 上，将上一步中推送的密钥复制到正确的位置。

    [root@servere ~]# /usr/libexec/glusterfs/set_geo_rep_pem_keys.sh \
    > geouser datavol backupvol

    d. 在 serverc 上，配置 datavol 和 backupvol 之间的异地复制链接，以使用共享存储满足跟踪更改等需要。

    [root@serverc ~]# gluster volume geo-replication datavol \
    > geouser@servere::backupvol config use_meta_volume true

    e. 启动 datavol 和 backupvol 之间的异地复制。

    [root@serverc ~]# gluster volume geo-replication datavol \
    > geouser@servere::backupvol start

6. 验证异地复制现在正常工作。

    a. 在 serverc 上，查看所有异地复制链接的状态。

    [root@serverc ~]# gluster volume geo-replication status

    MASTER NODE                MASTER VOL    MASTER BRICK              SLAVE USER    SLAVE                                 SLAVE NODE    STATUS     CRAWL STATUS       LAST_SYNCED
    -------------------------------------------------------------------------------
    -------------------------------------------------------------------------------
    -----------------------
    serverc.lab.example.com    datavol       /bricks/brick-c1/brick    geouser       ssh://geouser@servere::backupvol    servere       Active     Changelog Crawl    2016-05-17 16:10:04
    serverd.lab.example.com    datavol       /bricks/brick-d1/brick    geouser       ssh://geouser@servere::backupvol    servere       Passive    N/A                N/A

    如果有任何 brick 报告为 Initializing，请等待几秒后重新运行该命令。如果有任何 brick 报告为 Faulty，请检查您的配置。

    b. 在 servere 上，验证文件正在复制到承载 backupvol 卷的 brick (/brick/brick-e1/brick/)。

    [root@servere ~]# ls /bricks/brick-e1/brick
    file00
    file01
    ...
    file98
    file99

7. 将 datavolume 卷的 changelog.rollover-time 设置更新为 5 秒。

    a. 通过原生客户端挂载卷时，无法更新 rollover-time 设置。从 workstation 卸载 /mnt/datavol。

    [root@workstation ~]# umount /mnt/datavol

    b. 在 serverc 上，将 datavol 的 changelog.rollover-time 设置设为 5 秒。

    [root@serverc ~]# gluster volume set datavol changelog.rollover-time 5

    c. 在 workstation 系统上，重新挂载 /mnt/datavol。

    [root@workstation ~]# mount /mnt/datavol

8. 配置异地复制协议，以在 backupvol 上保留从 datavol 删除的文件。

    a. 对于 datavol 和 geouser@servere::backupvol 之间的异地复制协议，将 ignore-deletes 配置选项设为 true。

    [root@serverc ~]# gluster volume geo-replication datavol \
    > geouser@servere::backupvol config ignore-deletes true

### 总结

在本章中，您已学会如何：

*    描述异地复制。
*    准备异地复制的主节点和从节点。
*    配置 mountbroker，以将非特权用户用于异地复制。
*    配置异地复制对的选项。
*    向异地复制的卷添加新节点和 brick。
*    提升从卷供客户端使用。
*    自从卷恢复为主卷。
*    查找异地复制日志文件。 
