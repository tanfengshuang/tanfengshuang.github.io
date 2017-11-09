---
layout: post
title:  "Configuring Red Hat Gluster Storage(236-3)"
categories: Linux
tags: RHCA 236
---

### 构建受信存储池

###### 准备节点

安装红帽 Gluster 存储后，需要执行几个步骤，然后才能组建受信存储池。其中一些步骤（如调优）为可选，但能为最终性能带来明显变化。

1. 应用调优配置文件

红帽 Gluster 存储附带两个已调优的调优配置文件，可以根据预期的工作负载来应用它们；rhgs-random-io 适用于数量多、规模小的读取和写入，rhgs-sequential-io 则适用于大型文件传输。它们都基于 throughput-performance 配置文件。若要应用其中一个配置文件，可执行命令 tuned-adm profile <PROFILE>。例如：

```
[root@demo ~]# tuned-adm profile rhgs-random-io
```

2. 打开防火墙

默认情况下，系统防火墙由 firewalld 管理，将拦截所有进入 glusterd 守护进程的流量。glusterfs-server 软件包添加了一个新的防火墙服务守护进程，名为 glusterfs，它可用于开放防火墙上所有相关的端口。

```
[root@demo ~]# firewall-cmd --add-service=glusterfs
[root@demo ~]# firewall-cmd --runtime-to-permanent
```

###### 配置受信存储池

1. 安装并准备好服务器后，可以将它们关联到一起，组成受信存储池。此过程可通过 gluster peer probe <server> 命令完成。例如：

```
[root@demo ~]# gluster peer probe seconddemo.example.com
```

一个服务器在一个时间上只能从属于一个受信存储池。

> 注意1: 几乎所有对受信存储池和卷执行的操作都是通过 gluster 命令执行的。gluster 命令的基本语法为 gluster <object> <action> [options and arguments]...。

> 注意2: 运行 gluster help 以获取所有对象和操作的概述；或者，运行 gluster <object> help 以获取有关能对具体对象类型执行的操作的帮助。 

2. 查看受信存储池状态

配置了受信存储池后，可以通过两种方式查看受信存储池的状态；gluster peer status 显示每个对等点的详细信息，gluster pool list 命令则显示更精炼的概述。

```
[root@demo ~]# gluster peer status
Number of Peers: 1

Hostname: seconddemo.example.com
Uuid: 992f5057-ed8d-4ab9-99ac-eefb93856ad2
State: Peer in Cluster (Connected)

[root@demo ~]# gluster pool list
UUID					Hostname               	State
992f5057-ed8d-4ab9-99ac-eefb93856ad2	seconddemo.example.com Connected
e1db6528-69c8-4b23-8848-e31d889eaf7c	localhost              Connected
```

> 注意 gluster peer status 不会列出 localhost，而 gluster pool list 会列出。

3. 从受信存储池删除状态

若要从受信存储池删除节点，可使用命令 gluster peer detach <server>。

```
[root@demo ~]# gluster peer detach seconddemo.example.com
```

###### 读取日志文件

受信存储池中的每一节点都保留详细的日志。这些日志可以在 /var/log/glusterfs 中找到。此目录中包含对应于具体 brick、异地复制和快照的日志的子目录。其中含有对应于不同子系统的单独文件，如 NFS、卷再平衡和自我修复等。

主要的日志文件为 /var/log/glusterfs/etc-glusterfs-glusterd.vol.log。此文件显示所添加的对等点、对 glusterd 执行的命令，以及所创建的卷等。

所有日志文件中的消息都带有时间戳，其使用的是 UTC 时间而非本地时区。这可以更加轻松地比较散布于多个时区的服务器之间的日志。


### 引导式练习：构建受信存储池

1. 在 servera 和 serverb 计算机上，验证 glusterd.service 服务正在运行。

    a. 验证 servera 上正在运行 glusterd.service。

    [root@servera ~]# systemctl is-active glusterd
    active

    b. 验证 serverb 上正在运行 glusterd.service。

    [root@serverb ~]# systemctl is-active glusterd
    active

2. 在 servera 和 serverb 上，将防火墙配置为允许 glusterfs 流量。

    a. 在 servera 上，允许 glusterfs 流量通过防火墙。

    [root@servera ~]# firewall-cmd --add-service=glusterfs
    [root@servera ~]# firewall-cmd --permanent --add-service=glusterfs

    b. 在 serverb 上，允许 glusterfs 流量通过防火墙。

    [root@serverb ~]# firewall-cmd --add-service=glusterfs
    [root@serverb ~]# firewall-cmd --permanent --add-service=glusterfs

3. 从 servera，将 serverb.lab.example.com 添加到受信存储池，然后在两个节点上验证受信存储池的状态。

    a. 从 servera 系统，将 serverb.lab.example.com 添加到受信存储池。

    [root@servera ~]# gluster peer probe serverb.lab.example.com
    peer probe: success.

    b. 在 servera 上，验证受信存储池的状态。

    [root@servera ~]# gluster peer status
    Number of Peers: 1

    Hostname: serverb.lab.example.com
    Uuid: da5cdd9d-a432-4ad3-9296-87cc19dd7234
    State: Peer in Cluster (Connected)

    c. 在 serverb 上，验证受信存储池的状态。

    [root@serverb ~]# gluster peer status
    Number of Peers: 1

    Hostname: servera.lab.example.com
    Uuid: 6367f231-909f-48ba-bf7c-3b02a1c58f91
    State: Peer in Cluster (Connected)

    d. 从 servera，查看受信存储池中节点的完整列表。

    [root@servera ~]# gluster pool list
    UUID                                    Hostname                State
    da5cdd9d-a432-4ad3-9296-87cc19dd7234    serverb.lab.example.com Connected
    6367f231-909f-48ba-bf7c-3b02a1c58f91    localhost               Connected

4. 在 servera，检查 /var/log/glusterfs/etc-glusterfs-glusterd.vol.log 文件的最后两行，查找关于新对等点的信息。

```
[root@servera ~]# tail /var/log/glusterfs/etc-glusterfs-glusterd.vol.log
```

5. 在您的 workstation 系统上，运行命令 lab setup-pool grade 以验证您的工作。 


### 创建 Brick（存储块）

###### Brick 要求

若要开始构建卷 (volume)，首先需要创建 brick。brick 是一种 XFS 文件系统（具有 512 字节索引节点），挂载于其中一个存储服务器。

在物理部署中，建议在精简配置逻辑卷的基础上创建 brick。这样，管理员可以超量使用可用的存储，在必要时增加额外的物理存储。利用精简配置的存储确实要求管理员密切监控剩余的物理存储量。

建议为 brick 提供描述性强的挂载点，从而能轻松识别它们。转的常见位置包括 /exports、/bricks 和 /rhgs 下的子目录，如 /exports/largedata-brick-A 或 /bricks/brick1。挂载点也应该在授信存储池范围内唯一。用于 brick 的设备应当为基于 LVM。LVM 可增强红帽 Gluster 存储中的快照管理，在创建期间对 LVM 存储后端系统造成的影响非常小。在使用红帽 Gluster 存储池时，brick 挂载到 /rhgs/<BRICKNAME>，brick 子目录的名称与 brick 名称相同。

在生产系统中，物理卷应该在由 12 个驱动器组成的 RAID 6 阵列上创建。RAID6 条带大小应当设置为与平均文件大小相符，从而获得最优的性能。

> 重要: 在创建 XFS 文件系统时，请勿忘记将索引节点大小设为 512 字节。红帽 Gluster 存储将元数据存储在索引节点的扩展属性区域，其默认大小 256 字节在大部分情形中都不够。如果计划使用统一文件和对象存储，则索引节点大小应当设为 1024 字节。

如需在云部署中创建 brick 的详细信息，请参见《红帽 Gluster 存储管理指南》（网址为 https://access.redhat.com）以及相关的知识库文章。 

###### 在精简配置逻辑卷上创建 brick

在精简配置逻辑卷上创建 brick 需要执行几个步骤。在大型部署中，这些步骤（以及其他管理功能）可以通过 gdeploy（基于 ansible）或 Heketi（目前在技术预览中提供）进行自动化。有关使用 gdeploy 或 heketi 的更多信息，请参见《红帽 Gluster 存储管理指南》（网址为 https://access.redhat.com）中的相关章节。

若要手动创建 brick，请使用以下步骤：

1. 创建用于存放数据的 LVM 卷组。在教室中，所有节点上都已创建了名为vg_bricks 的卷组。
2. 创建 LVM 精简池。LVM 精简池的创建与逻辑卷相似，但标志设置为将它标记为供精简卷使用的存储的池。

    例如，若要在 vg_bricks 卷组中创建名为 examplepool 的 10 GiB 精简池，可使用以下命令：

    [root@demo ~]# lvcreate -L 10G -T vg_bricks/examplepool

3. 创建由 LVM 精简池支持的逻辑卷。精简卷没有物理大小，但有虚拟大小。与一般的逻辑卷不同，其范围不是在卷创建时分配的，而是在写入操作发生时动态分配。当精简卷基础上的文件系统支持“丢弃”功能时，不再使用的范围将被自动取消分配。

    例如，若要使用 vg_bricks/examplepool 精简池创建名为 thinvol 的 2 GiB 精简卷，可使用以下命令：

    [root@demo ~]# lvcreate -V 2G -T vg_bricks/examplepool -n thinvol
              
4. 使用 512 字节索引节点的 XFS 文件系统格式化新逻辑卷。

    [root@demo ~]# mkfs -t xfs -i size=512 /dev/vg_bricks/thinvol
              
5. 挂载新文件系统。这要求创建挂载点（必须在受信存储池中唯一），并在 /etc/fstab 中创建一个条目。

    下例将 /dev/vg_bricks/thinvol 卷挂载到 /bricks/thinvol。

    [root@demo ~]# mkdir -p /bricks/thinvol
    [root@demo ~]# echo "/dev/vg_bricks/thinvol /bricks/thinvol xfs defaults 1 2" >> /etc/fstab
    [root@demo ~]# mount /bricks/thinvol

6. 在新文件系统中创建用于存储 brick 的子目录。这将防止在 brick 未挂载时意外删除根文件系统上存储的数据，以免造成数据丢失。

    [root@demo ~]# mkdir /bricks/thinvol/brick
              
7. 在新 brick 目录上设置 SELinux 上下文以允许访问。此上下文应当不受自动重新标记的影响。为此，可使用 semanage 添加新的 fcontext 匹配，从而将 glusterd_brick_t 上下文添加到 brick 目录。

    [root@demo ~]# semanage fcontext -a -t glusterd_brick_t /bricks/thinvol/brick
    [root@demo ~]# restorecon -Rv /bricks/thinvol/brick


### 引导式练习：创建 Brick（存储块）

1. 在 servera 和 serverb 系统上，分别在 vg_bricks 卷组中创建一个 10 GiB LVM 精简池，名为 thinpool。

    a. 在 servera 上，在 vg_bricks 卷组内创建一个 10 GiB LVM 精简池，名为 thinpool。

    [root@servera ~]# lvcreate -L 10G -T vg_bricks/thinpool

    b. 在 serverb 上，在 vg_bricks 卷组内创建一个 10 GiB LVM 精简池，名为 thinpool。

    [root@serverb ~]# lvcreate -L 10G -T vg_bricks/thinpool

    c. 在 servera 和 serverb 上，验证 thinpool 池已创建为精简池。

    [root@servera ~]# lvdisplay
     --- Logical volume ---
     ...
     LV Pool metadata     thinpool_meta
     LV Pool data         thinpool_tdata
     ...
     LV Size              10.00 GiB
     ...

2. 在 servera 上，使用 vg_bricks/thinpool LVM 精简池，创建虚拟大小为 2 GiB、名为 brick-a1 的逻辑卷。

   在 serverb 上，使用 vg_bricks/thinpool LVM 精简池，创建虚拟大小为 2 GiB、名为 brick-b1 的逻辑卷。

    a. 在 servera 上，使用 vg_bricks/thinpool LVM 精简池，创建虚拟大小为 2 GiB、名为 brick-a1 的逻辑卷。

    [root@servera ~]# lvcreate -V 2G -T vg_bricks/thinpool -n brick-a1

    b. 在 serverb 上，使用 vg_bricks/thinpool LVM 精简池，创建虚拟大小为 2 GiB、名为 brick-b1 的逻辑卷。

    [root@serverb ~]# lvcreate -V 2G -T vg_bricks/thinpool -n brick-b1

3. 在 brick-a1 和 brick-b1 逻辑卷上，创建索引节点大小为 512 字节的 XFS 文件系统。

    a. 在 servera 的 brick-a1 brick 上，创建索引节点大小为 512 字节的 XFS 文件系统。

    [root@servera ~]# mkfs -t xfs -i size=512 /dev/vg_bricks/brick-a1
    meta-data=/dev/vg_bricks/brick-a1 isize=512    agcount=8, agsize=65520 blks
             =                       sectsz=512   attr=2, projid32bit=1
             =                       crc=0        finobt=0
    data     =                       bsize=4096   blocks=524160, imaxpct=25
             =                       sunit=16     swidth=16 blks
    naming   =version 2              bsize=4096   ascii-ci=0 ftype=0
    log      =internal log           bsize=4096   blocks=2560, version=2
             =                       sectsz=512   sunit=16 blks, lazy-count=1
    realtime =none                   extsz=4096   blocks=0, rtextents=0

    b. 在 serverb 的 brick-b1 brick 上，创建索引节点大小为 512 字节的 XFS 文件系统。

    [root@serverb ~]# mkfs -t xfs -i size=512 /dev/vg_bricks/brick-b1
    meta-data=/dev/vg_bricks/brick-b1 isize=512    agcount=8, agsize=65520 blks
             =                       sectsz=512   attr=2, projid32bit=1
             =                       crc=0        finobt=0
    data     =                       bsize=4096   blocks=524160, imaxpct=25
             =                       sunit=16     swidth=16 blks
    naming   =version 2              bsize=4096   ascii-ci=0 ftype=0
    log      =internal log           bsize=4096   blocks=2560, version=2
             =                       sectsz=512   sunit=16 blks, lazy-count=1
    realtime =none                   extsz=4096   blocks=0, rtextents=0

4. 在 servera 和 serverb 系统上，分别创建目录 /bricks/brick-a1 和 /bricks/brick-b1，然后永久挂载您的 brick。

    a. 在 servera 上，创建目录 /bricks/brick-a1。

    [root@servera ~]# mkdir -p /bricks/brick-a1

    b. 在 serverb 上，创建目录 /bricks/brick-b1。

    [root@serverb ~]# mkdir -p /bricks/brick-b1

    c. 在 servera 上，将下面这一行添加到 /etc/fstab。

    /dev/vg_bricks/brick-a1 /bricks/brick-a1 xfs defaults 1 2

    d. 在 serverb 上，将下面这一行添加到 /etc/fstab。

    /dev/vg_bricks/brick-b1 /bricks/brick-b1 xfs defaults 1 2

    e. 在 servera 和 serverb 上，运行命令 mount -a 以挂载您的新文件系统。

    [root@servera ~]# mount -a

    [root@serverb ~]# mount -a

5. 在 servera 和 serverb 上，在新 brick 上创建 /brick 子目录。

    a. 在 servera 上，创建 /bricks/brick-a1/brick 目录。

    [root@servera ~]# mkdir /bricks/brick-a1/brick

    b. 在 serverb 上，创建 /bricks/brick-b1/brick 目录。

    [root@servera ~]# mkdir /bricks/brick-b1/brick

6. 在 servera 和 serverb 上，将您刚才创建的 brick 目录的默认 SELinux 标签配置为 glusterd_brick_t，然后递归重新标记 /bricks 目录。

    在 servera 和 serverb 上，将您刚才创建的 brick 目录的默认 SELinux 标签配置为 glusterd_brick_t。

    [root@servera ~]# semanage fcontext -a -t glusterd_brick_t /bricks/brick-a1/brick

    [root@serverb ~]# semanage fcontext -a -t glusterd_brick_t /bricks/brick-b1/brick

    在 servera 和 serverb 上，对 /bricks 递归应用新的默认上下文。

    [root@servera ~]# restorecon -Rv /bricks

    [root@serverb ~]# restorecon -Rv /bricks

7. 在 workstation 系统上，运行命令 lab setup-bricks grade 以验证您的工作。


### 构建卷

###### 创建卷

现在已经有了一个受信存储池和一些 brick，可以构建第一个卷了。使用 gluster volume create <VOLUME-NAME> [stripe and replica options] <BRICK>... 命令构建卷。

该卷将是分布式的，只要没有指定 stripe或 replica 选项。第 4 章 创建卷中详细介绍了各种不同的卷类型。

> 重要: 新卷在创建后不会自动启动，因此管理员必须手动启动它们，以提供给客户端。 

如，若要使用 demoa 上的 /bricks/brick-a1/brick 和 demob 上的 /bricks/brick-b1/brick 这两个 brick 创建名为 firstvol 的新卷，管理员可以使用以下命令。

```
[root@demoa ~]# gluster volume create firstvol demoa:/bricks/brick-a1/brick demob:/bricks/brick-b1/brick
volume create: firstvol: success: please start the volume to access data
```

###### 检查卷

创建了卷后，可以使用两个命令来检查它的状态；gluster volume status <VOLUME> 可查看卷状态的简略报告，gluster volume info <VOLUME> 则可显示更为详细的视图。

```
[root@demoa ~]# gluster volume status firstvol
Volume firstvol is not started

[root@demoa ~]# gluster volume info firstvol
Volume Name: firstvol
Type: Distribute
Volume ID: 44d71c80-910c-4869-9029-604829b01c99
Status: Created
Number of Bricks: 2
Transport-type: tcp
Bricks:
Brick1: demoa:/bricks/brick-a1/brick
Brick2: demob:/bricks/brick-b1/brick
Options Reconfigured:
performance.readdir-ahead: on
```

可以通过 gluster volume list 命令请求群集中所有卷的概览。 

###### 启动和停止卷。

卷必须启动后，才能供客户端访问。同样，卷停止后就不再能被客户端访问。 

gluster volume start <VOLUME> 命令可以启动卷：

```
[root@demoa ~]# gluster volume start firstvol
volume start: firstvol: success
```

gluster volume stop <VOLUME> 命令可以通知卷：

```
[root@demoa ~]# gluster volume stop firstvol
Stopping a volume will make its data inaccessible. Do you want to continue? (y/n) y
volume stop: firstvol: success
```

###### 删除卷

可以使用命令 gluster volume delete <VOLUME> 来删除卷：

```
[root@demoa ~]# gluster volume delete firstvol
Deleting volume will erase all information about the volume. Do you want to continue? (y/n) y
volume delete: firstvol: success
```

> 重要: 删除卷后，组成该卷的 brick 将无法即刻使用。与原有卷相关的元数据仍然存储在该卷上文件的扩展属性中。这些数据可以通过 getfattr -d -m'.*' <BRICK-DIRECTORY> 命令查看。虽然可通过 setfattr -x 命令删除这些属性，但实际应用中，更快的做法是删除并重新创建 brick 子目录。 

###### 测试卷

卷启动后，它将提供给客户端。若要测试新卷是否正常工作，管理员可以使用 glusterfs 文件系统类型将它挂载到不同的系统，安装 glsuterfs-fuse 软件包后便可使用该文件系统。卷的设备名称将是 <SERVER>:<VOLUME>，其中 <SERVER> 是受信存储池中的任何服务器。

```
[root@demoworkstation ~]# yum -y install glusterfs-fuse
[root@demoworkstation ~]# mkdir /mnt/firstvol
[root@demoworkstation ~]# mount -t glusterfs demoa:firstvol /mnt/firstvol
[root@demoworkstation ~]# touch /mnt/firstvol/file{0..099}
[root@demoworkstation ~]# umount /mnt/firstvol
```

第 5 章 配置客户端中提供了关于客户端及其选项的更多信息。

### 引导式练习：构建卷

1. 从 servera 系统，利用 brick servera:/bricks/brick-a1/brick 和 serverb:/bricks/brick-b1/brick 创建一个名为 firstvol 的新 gluster 卷，无需使用任何额外选项。不要忘记启动该卷。

    a. 从 servera 系统，利用 brick servera:/bricks/brick-a1/brick 和 serverb:/bricks/brick-b1/brick 创建一个名为 firstvol 的新 gluster 卷，无需使用任何额外选项。

    [root@servera ~]# gluster volume create firstvol servera:/bricks/brick-a1/brick serverb:/bricks/brick-b1/brick
    volume create: firstvol: success: please start the volume to access data

    b. 在 servera 计算机上，检查 firstvol 卷的配置。

    [root@servera ~]# gluster volume info firstvol
    Volume Name: firstvol
    Type: Distribute
    Volume ID: 74545dfe-b5ee-4ae1-9b22-a9516d067c7a
    Status: Created
    Number of Bricks: 2
    Transport-type: tcp
    Bricks:
    Brick1: servera:/bricks/brick-a1/brick
    Brick2: serverb:/bricks/brick-b1/brick
    Options Reconfigured:
    performance.readdir-ahead: on

    注意状态：Created 这一行。

    c. 从 servera，启动您的新 firstvol 卷。

    [root@servera ~]# gluster volume start firstvol

    d. 在 servera 计算机上，再次检查 firstvol 卷的配置。

    [root@servera ~]# gluster volume info firstvol
    Volume Name: firstvol
    Type: Distribute
    Volume ID: 74545dfe-b5ee-4ae1-9b22-a9516d067c7a
    Status: Started
    Number of Bricks: 2
    Transport-type: tcp
    Bricks:
    Brick1: servera:/bricks/brick-a1/brick
    Brick2: serverb:/bricks/brick-b1/brick
    Options Reconfigured:
    performance.readdir-ahead: on

    注意状态：Started 这一行。 

2. 在 workstation 系统上，安装 glusterfs-fuse 软件包，然后使用文件系统类型 glusterfs 将 servera:firstvol 挂载到 /mnt 上。在该卷上创建几个文件，再卸载它。

    a. 在 workstation 系统上，安装 glusterfs-fuse 软件包。

    [root@workstation ~]# yum install glusterfs-fuse

    b. 在 workstation 系统上，使用文件系统类型 glusterfs 将 servera:firstvol 挂载到 /mnt 上。

    [root@workstation ~]# mount -t glusterfs servera:firstvol /mnt

    c. 在 workstation上，在 /mnt 中创建几个文件。

    [root@workstation ~]# touch /mnt/file{0..0100}

    d. 从 workstation 卸载该卷。

    [root@workstation ~]# umount /mnt

3. 检查 servera 和 serverb 上 brick 的内容。您注意到了什么？

    a. 检查 servera 和 serverb 上 brick 的内容。

    [root@servera ~]# ls -l /bricks/brick-a1/brick

    [root@serverb ~]# ls -l /bricks/brick-b1/brick

    b. 对于创建的文件您有什么发现？

    创建的文件已大致均匀地分布到两个 brick 上。 


### 实验：配置红帽 Gluster 存储

利用 serverc 和 serverd 计算机提供红帽 Gluster 存储的概念验证。这些计算机应当组成一个受信存储池，并导出一个名为 labvol 的卷。labvol 卷应当包含两个 brick，它们由位于精简配置池上的 LVM 卷提供支持。

下方表格中概述了 brick 的要求：

表 3.1. serverc 资源

*    资源	            描述
*    LVM 精简池名称 	labpool
*    LVM 精简池大小 	10 GiB
*    LVM 卷组 	    vg_bricks
*    Brick LV 名称 	brick-c1
*    Brick 大小 	    2 GiB
*    Brick 挂载点 	/bricks/brick-c1
*    Brick 子目录 	/bricks/brick-c1/brick

表 3.2. serverd 资源

*    资源	            描述
*    LVM 精简池名称 	labpool
*    LVM 精简池大小 	10 GiB
*    LVM 卷组 	    vg_bricks
*    Brick LV 名称 	brick-d1
*    Brick 大小 	    2 GiB
*    Brick 挂载点 	/bricks/brick-d1
*    Brick 子目录 	/bricks/brick-d1/brick

1. 将 serverc 和 serverd 组合为一个受信存储池。

    a. 在 serverc 和 serverd 系统上，允许 glusterfs 流量通过防火墙。

    [root@serverc ~]# firewall-cmd --add-service=glusterfs
    [root@serverc ~]# firewall-cmd --permanent --add-service=glusterfs

    [root@serverd ~]# firewall-cmd --add-service=glusterfs
    [root@serverd ~]# firewall-cmd --permanent --add-service=glusterfs

    b. 从 serverc 系统，将 serverd 添加到受信存储池。

    [root@serverc ~]# gluster peer probe serverd.lab.example.com
    peer probe: success.

2. 按照说明中指定的要求，创建您的卷所需的两个 brick。

    a. 在 serverc 和 serverd 系统上，分别在 vg_bricks 卷组中创建一个 10 GiB LVM 精简池，名为 labpool。

    [root@serverc ~]# lvcreate -L 10G -T vg_bricks/labpool
    [root@serverd ~]# lvcreate -L 10G -T vg_bricks/labpool

    b. 在 serverc 系统上，在 vg_bricks/labpool 中创建一个 2 GiB 逻辑卷，名为 brick-c1。

    [root@serverc ~]# lvcreate -V 2G -T vg_bricks/labpool -n brick-c1

    c. 在 serverd 系统上，在 vg_bricks/labpool 中创建一个 2 GiB 逻辑卷，名为 brick-d1。

    [root@serverd ~]# lvcreate -V 2G -T vg_bricks/labpool -n brick-d1

    d. 使用 512 字节索引节点的 XFS 文件系统格式化这两个新逻辑卷。

    [root@serverc ~]# mkfs -t xfs -i size=512 /dev/vg_bricks/brick-c1
    [root@serverd ~]# mkfs -t xfs -i size=512 /dev/vg_bricks/brick-d1

    e. 分别将 serverc 和 serverd 上的新逻辑卷永久挂载到 /bricks/brick-c1 和 /bricks/brick-d1。

    [root@serverc ~]# mkdir -p /bricks/brick-c1
    [root@serverc ~]# echo "/dev/vg_bricks/brick-c1 /bricks/brick-c1 xfs defaults 1 2" >> /etc/fstab
    [root@serverc ~]# mount /bricks/brick-c1

    [root@serverd ~]# mkdir -p /bricks/brick-d1
    [root@serverd ~]# echo "/dev/vg_bricks/brick-d1 /bricks/brick-d1 xfs defaults 1 2" >> /etc/fstab
    [root@serverd ~]# mount /bricks/brick-d1

    f. 在您的新 brick 上创建 /brick 子目录，然后为这些目录设置正确的默认 SELinux 上下文。

    [root@serverc ~]# mkdir /bricks/brick-c1/brick
    [root@serverc ~]# semanage fcontext -a -t glusterd_brick_t /bricks/brick-c1/brick
    [root@serverc ~]# restorecon -Rv /bricks/brick-c1

    [root@serverd ~]# mkdir /bricks/brick-d1/brick
    [root@serverd ~]# semanage fcontext -a -t glusterd_brick_t /bricks/brick-d1/brick
    [root@serverd ~]# restorecon -Rv /bricks/brick-d1

3. 使用 serverc:/bricks/brick-c1/brick 和 serverd:/bricks/brick-d1/brick 这两个 brick，创建并启动您的 labvol 卷。

    a. 将两个新 brick 组合为名为 labvol 的卷，不使用任何额外选项。

    [root@serverc ~]# gluster volume create labvol serverc:/bricks/brick-c1/brick serverd:/bricks/brick-d1/brick

    b. 启动新的 labvol 卷。

    [root@serverc ~]# gluster volume start labvol

4. 从 workstation系统运行命令 lab basicconfig grade，以验证您的工作。

    [student@workstation ~]$ lab basicconfig grade

5. 重要清理：将 workstation、servera、serverb、serverc 和 serverd 系统重置为初始状态，以清理您的实验环境，为后续练习做好准备。 


### 总结

在本章中，您学到了：

*    如何利用 tuned-adm profile 选择红帽 Gluster 存储节点的调优配置文件。
*    如何利用 glusterfs 服务为红帽 Gluster 存储打开正确的防火墙端口。
*    如何使用 gluster peer probe 添加节点到受信存储池。
*    如何创建精简配置的 LVM 池。
*    如何创建精简配置的逻辑卷。
*    使用 XFS 文件系统格式化 brick。
*    将逻辑卷上的子目录用作 brick。
*    将 glusterd_brick_t SELinux 上下文应用到 brick 子目录。
*    如何使用 gluster volume create 创建卷。
*    如何使用 gluster volume start 启动卷。
*    如何使用 glusterfs-fuse 测试卷。 


