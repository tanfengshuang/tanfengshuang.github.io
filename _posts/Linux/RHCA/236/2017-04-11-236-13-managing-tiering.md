---
layout: post
title:  "Managing Tiering(236-13)"
categories: Linux
tags: RHCA 236
---

### 分层概念和术语

###### 红帽 Gluster 存储中的分层

红帽 Gluster 存储支持分层，管理员可借助此功能根据数据 I/O 访问要求高效利用正确的硬件。这会根据底层硬件提供的性能，创建不同的分层。分层涉及通过特定的分类定义数据，并根据用户的 I/O 访问来移动相关的数据。在红帽 Gluster 存储中，分层将频繁访问的数据放入性能较高的热层（如固态驱动器 (SSD)），并将不活跃的数据放入性能较低的冷层（如机械硬盘），且不会造成 I/O 中断。确定了数据的活跃度后，分层使用重新平衡逻辑将活跃和不活跃的数据重新放入最适合的存储层。数据根据访问频率定义为热或冷。当文件访问量增多时，其数据被移动到热层。相反，当文件访问量减少时，其数据被移动到冷层。 

1. 分层架构

分层转换器基于 DHT 和重新平衡逻辑，将一个卷分割为两个子卷：hot 和 cold。hot 子卷被视为 cold 子卷的缓存。转换器负责决定将哪一分层用于文件，以及何时在分层之间迁移文件。一个文件可以驻留于任一个卷，但一个文件不能分割到两个子卷上。文件迁移按照以下条件发生：

*    温度：文件被访问的频率。
*    容量：达到了 hot 子卷的容量（或水印）。

###### 数据移动框架

数据移动框架是利用重新平衡逻辑的迁移逻辑一般化。用于迁移文件的触发器由应用驱动。文件被升级或降级到适当的分层。可以配置文件的升级和降级检查频率，以适合具体的 glusterfs 架构。

1. 数据移动触发器

数据移动触发器是触发文件从源存储单元到目标存储单元的机制。源存储单元 (SSU) 和目标存储单元 (DSU) 分别指定从中迁移出数据的 glusterfs brick/卷，以及迁移后数据所驻留的 glusterfs brick/卷。数据移动触发器将创建数据移动请求 (DMR)，该请求将提交至数据移动服务 (DMS)。触发器有两个，即 I/O Path Trigger 和 Scanning Trigger。

*    I/O Path Trigger        当文件符合指定的数据移动规则时，此触发器将启动。
*    Scanner Trigger         根据指定的移动规则，Scanner Trigger 遍历指定的源单元并选择对象/文件。在选择对象/文件后，向数据移动服务提交数据移动请求。

2. 数据移动服务 (DMS)

数据移动服务负责数据的实际移动。此服务接收由数据移动触发器提交的数据移动请求。数据移动请求指定用于处理该请求的适当插件。

> 注意: 文件的升级或降级由数据移动触发器决定。例如，确定哪些文件需要升级或降级的选择逻辑由数据移动插件决定。

### 管理分层

###### 连接热层

红帽 Gluster 存储卷没有预配置为分层卷。若要连接热层到卷，需要额外的 brick。这些额外 brick 基于能提供优于 cold 层性能的硬件。当热层添加到现有的卷时，先调用分发转换器 (DHT) 的固定布局过程（以同步各分层之间的目录），然后启动移动数据的流程。此流程类似于删除或添加 brick 时使用的逻辑。

可使用 gluster volume tier attach 命令将热层连接到卷。卷和用于热层的所有 brick 将作为参数传递给该命令。

    [root@demo-a ~]# gluster volume tier
    demo-vol attach replica 2
    demo-a:/bricks/brick-a2/brick
    demo-b:/bricks/brick-b2/brick
    
连接了热层后，可以通过 gluster volume info 命令检索关于热层和冷层类型以及 brick 的额外信息。

[root@demo-a ~]# gluster volume info demo-vol
Volume Name: prod-vol
Type: Tier
Volume ID: be44e961-c194-4cf2-9e33-418e824a9163
Status: Started
Number of Bricks: 4
Transport-type: tcp
Hot Tier :
Hot Tier Type : Replicate
Number of Bricks: 1 x 2 = 2
Brick1: demo-b:/bricks/brick-b2/brick
Brick2: demo-a:/bricks/brick-a2/brick
Cold Tier:
Cold Tier Type : Replicate
Number of Bricks: 1 x 2 = 2
Brick3: demo-a:/bricks/brick-a1/brick
Brick4: demo-b:/bricks/brick-b1/brick
Options Reconfigured:
cluster.tier-mode: cache
features.ctr-enabled: on
performance.readdir-ahead: on

gluster volume tier status 命令提供关于热层和冷层操作的统计信息，包括两个分层中已升级和已降级的文件。

[root@demo-a ~]# gluster volume tier demo-vol status
Node                 Promoted files       Demoted files        Status
---------            ---------            ---------            ---------
localhost            1                    5                    in progress
demo-b               0                    2                    in progress
Tiering Migration Functionality: demo-vol: success

###### 断开热层

除了连接热层外，也可以从卷断开它们。数据移动流程将更改状态，把所有数据从热层移到冷层上，这与删除 brick 时大致相同。当所有数据都迁移后，通过执行来完成断开操作。gluster volume tier detach 命令将热层从卷断开。

    [root@demo-a ~]# gluster volume tier demo-vol detach start
    
断开卷需要分两步走，先断开热层，然后执行断开操作。若要验证热层断开的状态，可使用 gluster volume tier detach status 命令。该命令应当会将 brick 的断开进程返回为 completed。

[root@demo-a ~]# gluster volume tier demo-vol detach status
Node Rebalanced-files          size       scanned      failures       skipped
--------      -----------   -----------   -----------   -----------   -----------
localhost            0        0Bytes             0             0             0
demo-b               0        0Bytes             0             0             0


   status       run time in secs
------------     --------------
     completed               0.00
     completed               1.00

当所有 brick 的状态都标记为 completed 时，可运行 gluster volume tier commit 命令来执行热层断开。

[root@demo-a ~]# gluster volume tier demo-vol detach commit
Removing tier can result in data loss. Do you want to Continue? (y/n)
y
volume detach-tier commit: success
Check the detached bricks to ensure all files are migrated.
If files with data are found on the brick path, copy them via a gluster mount point before re-purposing the removed brick.

###### 配置升级和降级

文件的升级和降级由文件访问情况以及热层的容量占用情况决定。为卷启用了分层后，可以使用 cluster.watermark-hi 和 cluster.watermark-hi 这两个卷设置来确定文件是否能被升级和/或降级。

只要热层的数据使用量没有达到 cluster.watermark-lo 设置中设定的百分比（默认 75%），文件只能被升级。如果数据使用量超过 cluster.watermark-hi（默认为 90%），文件只能被降级。如果数据使用量在这两个水印文件之间，则文件可以被升级和降级。 

1. 配置升级和降级频率

可以通过以下两项设置来控制文件的升级和降级频率：cluster.tier-promote-frequency 和 cluster.tier-demote-frequency。这两项设置的默认值分别为 120 和 3600 秒。如果某一文件在过去 cluster.tier-promote-frequency 秒内至少被访问一次，它就符合升级条件。类似地，如果热层上的某一文件在过去cluster.tier-demote-frequency 秒内未曾被访问过，则它符合降级条件。

还可以配置将文件标记为 hot 所需的最少读取数和写入数。这可以通过 cluster.read-freq-threshold 和 cluster.write-freq-threshold 设置完成。这两个选项都可取 0 到 1000 范围的任何值，默认值为 0。如果设为 0，则不使用该阈值。1 到 1000 范围的值表示文件若要达到升级条件而必须被访问的次数。

2. 限制数据移动

可以通过两项设置来限制热层和冷层之间移动的数据量。cluster.tier-max-files 设置可用于限制一个周期内可以移动的文件数量。如果未设置，则默认为 10000 个文件。

cluster.tier-max-mb 设置可用于限制一个周期内可以移动的数据量。如果未设置，则默认为 4000 MiB。 


### 引导式练习：管理分层

启用和禁用卷的分层。 

1. 验证 prod-vol 卷，以及 brick-a1、brick-a2、brick-b1 和 brick-b2 这几个 brick。

    a. 从 servera，验证 prod-vol 的卷信息。

    [root@servera ~]$ gluster volume info prod-vol
    Volume Name: prod-vol
    Type: Replicate
    Volume ID: be44e961-c194-4cf2-9e33-418e824a9163
    Status: Started
    Number of Bricks: 1 x 2 = 2
    Transport-type: tcp
    Bricks:
    Brick1: servera:/bricks/brick-a1/brick
    Brick2: serverb:/bricks/brick-b1/brick
    Options Reconfigured:
    performance.readdir-ahead: on

    b. 从 servera，验证为 servera 和 serverb 挂载了哪些 brick。

    [student@workstation ~]$ for I in server{a,b}
    > do
    >   ssh ${I} "mount | grep brick"
    > done
    /dev/mapper/vg_bricks-brick--a1 on /bricks/brick-a1 type xfs (rw,relatime,seclabel,attr2,inode64,logbsize=64k,sunit=128,swidth=128,noquota)
    /dev/mapper/vg_bricks-brick--a2 on /bricks/brick-a2 type xfs (rw,relatime,seclabel,attr2,inode64,logbsize=64k,sunit=128,swidth=128,noquota)

    /dev/mapper/vg_bricks-brick--b1 on /bricks/brick-b1 type xfs (rw,relatime,seclabel,attr2,inode64,logbsize=64k,sunit=128,swidth=128,noquota)
    /dev/mapper/vg_bricks-brick--b2 on /bricks/brick-b2 type xfs (rw,relatime,seclabel,attr2,inode64,logbsize=64k,sunit=128,swidth=128,noquota)

2. 将含有 servera:/bricks/brick-a2/brick 和 serverb:/bricks/brick-b2/brick 的热层连接到 prod-vol 卷。

    a. 从 servera，将含有 servera:/bricks/brick-a2/brick 和 serverb:/bricks/brick-b2/brick 的热层连接到 prod-vol 卷。

    [root@servera ~]$ gluster volume tier prod-vol attach replica 2 \
    > servera:/bricks/brick-a2/brick \
    > serverb:/bricks/brick-b2/brick
    volume attach-tier: success
    Tiering Migration Functionality: prod-vol: success: Attach tier is successful on prod-vol. use tier status to check the status.
    ID: 9e422b41-91fc-428a-b3e1-c95b2df6685f

3. 验证 prod-vol 的卷信息。

    a. 从 servera，验证 prod-vol 的卷信息。

    [root@servera ~]$ gluster volume info prod-vol
    Volume Name: prod-vol
    Type: Tier
    Volume ID: be44e961-c194-4cf2-9e33-418e824a9163
    Status: Started
    Number of Bricks: 4
    Transport-type: tcp
    Hot Tier :
    Hot Tier Type : Replicate
    Number of Bricks: 1 x 2 = 2
    Brick1: serverb:/bricks/brick-b2/brick
    Brick2: servera:/bricks/brick-a2/brick
    Cold Tier:
    Cold Tier Type : Replicate
    Number of Bricks: 1 x 2 = 2
    Brick3: servera:/bricks/brick-a1/brick
    Brick4: serverb:/bricks/brick-b1/brick
    Options Reconfigured:
    cluster.tier-mode: cache
    features.ctr-enabled: on
    performance.readdir-ahead: on

4. 验证 prod-vol 的分层启用状态。

    a. 从 servera，验证 prod-vol 的分层启用状态。

    [root@servera ~]$ gluster volume tier prod-vol status
    Node                 Promoted files       Demoted files        Status
    ---------            ---------            ---------            ---------
    localhost               1                    2                    in progress
    serverb.lab.example.com 0                    1                    in progress
    Tiering Migration Functionality: prod-vol: success


### 扩展分层卷

###### 扩展分层卷

分层卷允许向冷层和热层添加额外的 brick。向冷层添加新 brick 通过 gluster volume add-brick 命令来完成，并且要求先从卷断开其热层。其流程如下所示： 

1. 针对要利用新 brick 扩展冷层的卷，从卷断开其热层。记住这包括两个步骤：先使用 gluster volume tier detach start 命令断开热层，再通过 gluster volume tier detach commit 命令执行断开操作。

[root@demo-a ~]# gluster volume tier demo-vol detach start
[root@demo-a ~]# gluster volume tier demo-vol detach commit

2. 使用 gluster volume add-brick 命令将新 brick 添加到冷层。

[root@demo-a ~]# gluster volume add-brick demo-vol demo2:/bricks/brick1 demo2:/bricks/brick2

3. 使用 gluster volume rebalance 命令对卷进行重新平衡。验证所有节点已标记为 completed，确保重新平衡完成。

[root@demo-a ~]# gluster volume rebalance demo-vol start
[root@demo-a ~]# gluster volume rebalance demo-vol status

4. 通过 gluster volume tier attach 命令将热层重新连接到卷。

[root@demo-a ~]# gluster volume tier demo-vol attach replica 2 demo2:/bricks/brick1 demo2:/bricks/brick2

使用 gluster volume tier attach 命令将更多 brick 添加到热层。记住在开始之前，先通过 gluster volume tier detach 命令从卷断开热层。

向热层添加更多 brick 通过 gluster volume tier attach 命令来完成。与冷层类似，首先需要使用 gluster volume tier detach 命令从卷断开热层。 

1. 使用 gluster volume tier detach start 命令，从卷断开即将扩展的热层。在完成时，通过 gluster volume tier detach commit 命令执行断开操作。

[root@demo-a ~]# gluster volume tier demo-vol detach start
[root@demo-a ~]# gluster volume tier demo-vol detach commit

2. 使用 gluster volume tier attach force 命令，将热层（包括旧的和新的 brick 在内）重新连接到卷。

[root@demo-a ~]# gluster volume tier
demo-vol attach replica 2
demo-a:/bricks/brick-a2/brick demo-a:/bricks/brick-a3/brick
demo-b:/bricks/brick-b2 demo-b:/bricks/brick-b3/brick

###### 缩小分层卷

卷的冷层和热层都可以缩小。可通过 gluster volume remove-brick 命令从冷层删除 brick。在缩小卷之前，也需要先断开热层，然后在删除 brick 后重新连接。其流程如下所示：

1. 使用 gluster volume tier detach start 命令，从卷断开即将缩小的热层。在完成时，通过 gluster volume tier detach commit 命令执行断开操作。

[root@demo-a ~]# gluster volume tier demo-vol detach start
[root@demo-a ~]# gluster volume tier demo-vol detach commit

2. 使用 gluster volume remove-brick 命令删除 brick。在完成 brick 删除后，通过 gluster volume remove-brick commit 命令提交。

[root@demo-a ~]# gluster volume remove-brick demo-vol demo2:/bricks/brick1 start
[root@demo-a ~]# gluster volume remove-brick demo-vol demo2:/bricks/brick1 commit

3. 通过 gluster volume tier attach 命令重新连接热层。

[root@demo-a ~]# gluster volume tier demo-vol attach replica 2 demo2:/bricks/brick1 demo2:/bricks/brick2

若要从热层删除 brick，可通过 gluster volume tier attach 命令断开热层，然后仅利用要继续包含在热层中的 brick 重新连接热层。

1. 通过 gluster volume tier detach start 命令从卷断开热层。在完成时，通过 gluster volume tier detach commit 命令执行热层断开操作。

[root@demo-a ~]# gluster volume tier demo-vol detach start
[root@demo-a ~]# gluster volume tier demo-vol detach commit

2. 通过 gluster volume tier attach 命令，将热层重新连接到卷，并忽略不再属于该热层的 brick。

[root@demo-a ~]# gluster volume tier demo-vol attach replica 2 demo2:/bricks/brick1 demo2:/bricks/brick2


### 引导式练习：扩展分层卷

扩展和缩小分层卷的冷层和热层。 

1. 验证热层已连接至卷 prod-vol。

    a. 从 servera，通过 gluster volume info 验证热层已连接到 prod-vol。

    [root@servera ~]$ gluster volume info prod-vol
    Volume Name: prod-vol
    Type: Tier
    Volume ID: be44e961-c194-4cf2-9e33-418e824a9163
    Status: Started
    Number of Bricks: 4
    Transport-type: tcp
    Hot Tier :
    Hot Tier Type : Replicate
    Number of Bricks: 1 x 2 = 2
    Brick1: serverb:/bricks/brick-b2/brick
    Brick2: servera:/bricks/brick-a2/brick
    Cold Tier:
    Cold Tier Type : Replicate
    Number of Bricks: 1 x 2 = 2
    Brick3: servera:/bricks/brick-a1/brick
    Brick4: serverb:/bricks/brick-b1/brick
    Options Reconfigured:
    cluster.tier-mode: cache
    features.ctr-enabled: on
    performance.readdir-ahead: on

2. 利用brick-a3 和 brick-b3，扩展 prod-vol 的热层。

    a. 从 servera，将热层从 prod-vol 断开。

    [root@servera ~]$ gluster volume  tier prod-vol detach start
    volume detach-tier start: success
    ID: 171d000e-b2f3-4623-9697-d96623a01226         

    b. 验证层断开操作的完成状态。

    [root@servera ~]$ gluster volume tier prod-vol detach status
                                    Node Rebalanced-files          size       scanned      failures       skipped
           status   run time in secs
                                   ---------      -----------   -----------   -----------
          ------------          --------------
                                   localhost                0        0Bytes            20
           completed               1.00
                     serverb.lab.example.com                0        0Bytes             0
           completed               1.00

    c. 执行热层的断开操作。

    [root@servera ~]$  gluster volume tier prod-vol detach commit
    Removing tier can result in data loss. Do you want to Continue? (y/n) y
    volume detach-tier commit: success
    Check the detached bricks to ensure all files are migrated.
    If files with data are found on the brick path, copy them via a gluster mount
    point before re-purposing the removed brick. 

    d. 在 servera 上删除并重新创建 /bricks/brick-a2/brick。在 serverb 上删除并重新创建 /bricks/brick-b2/brick。

    [root@servera ~]$ rm -rf /bricks/brick-a2/brick
    [root@servera ~]# mkdir /bricks/brick-a2/brick

    [root@serverb ~]$ rm -rf /bricks/brick-b2/brick
    [root@serverb ~]# mkdir /bricks/brick-b2/brick

    e. 从 servera，重新连接包含旧 brick brick-a2 和 brick-b2 以及新 brick brick-a3 和 brick-b3 的热层。

    [root@servera ~]$ gluster volume tier prod-vol attach replica 2 \
    > servera:/bricks/brick-a2/brick \
    > serverb:/bricks/brick-b2/brick \
    > servera:/bricks/brick-a3/brick  \
    > serverb:/bricks/brick-b3/brick
    volume attach-tier: success
    Tiering Migration Functionality: prod-vol: success: Attach tier is successful on prod-vol. use tier status to check the status.
    ID: 9e422b41-91fc-428a-b3e1-c95b2df6685f

3. 验证新扩展的热层。

    a. 从 servera，验证卷 prod-vol 中的热层。

    [root@servera ~]$ gluster volume info prod-vol
    Volume Name: prod-vol
    Type: Tier
    Volume ID: be44e961-c194-4cf2-9e33-418e824a9163
    Status: Started
    Number of Bricks: 6
    Transport-type: tcp
    Hot Tier :
    Hot Tier Type : Distributed-Replicate
    Number of Bricks: 2 x 2 = 4
    Brick1: serverb:/bricks/brick-b3/brick
    Brick2: servera:/bricks/brick-a3/brick
    Brick3: serverb:/bricks/brick-b2/brick
    Brick4: servera:/bricks/brick-a2/brick
    Cold Tier:
    Cold Tier Type : Replicate
    Number of Bricks: 1 x 2 = 2
    Brick5: servera:/bricks/brick-a1/brick
    Brick6: serverb:/bricks/brick-b1/brick
    Options Reconfigured:
    cluster.tier-mode: cache
    features.ctr-enabled: on
    performance.readdir-ahead: on

4. 验证热层已连接至卷 dev-vol。

    从 servera，通过 gluster volume info 验证热层已连接到 dev-vol。

    [root@servera ~]$ gluster volume info dev-vol
    Volume Name: dev-vol
    Type: Tier
    Volume ID: 16627eb3-2650-4a87-b664-d2e893bac3fd
    Status: Started
    Number of Bricks: 6
    Transport-type: tcp
    Hot Tier :
    Hot Tier Type : Distributed-Replicate
    Number of Bricks: 2 x 2 = 4
    Brick1: serverb:/bricks/brick-b6/brick
    Brick2: servera:/bricks/brick-a6/brick
    Brick3: serverb:/bricks/brick-b5/brick
    Brick4: servera:/bricks/brick-a5/brick
    Cold Tier:
    Cold Tier Type : Replicate
    Number of Bricks: 1 x 2 = 2
    Brick5: servera:/bricks/brick-a4/brick
    Brick6: serverb:/bricks/brick-b4/brick
    Options Reconfigured:
    cluster.tier-mode: cache
    features.ctr-enabled: on
    performance.readdir-ahead:

5. 删除 brick-a5 和 brick-b5，以缩小热层。

    a. 从 servera，断开 dev-vol 中的热层。

    [root@servera ~]$ gluster volume tier dev-vol detach start
    volume detach-tier start: success
    ID: 171d000e-b2f3-4623-9697-d96623a00226

    b. 验证层断开操作的完成状态。

    [root@servera ~]$ gluster volume tier dev-vol detach status
                                    Node Rebalanced-files          size       scanned      failures       skipped
           status   run time in secs
                                   ---------      -----------   -----------   -----------
          ------------          --------------
                                   localhost                0        0Bytes            20
           completed               1.00
                     serverb.lab.example.com                0        0Bytes             0
           completed               1.00

    c. 执行热层的断开操作。

    [root@servera ~]$  gluster volume tier prod-vol detach commit
    Removing tier can result in data loss. Do you want to Continue? (y/n) y
    volume detach-tier commit: success
    Check the detached bricks to ensure all files are migrated.
    If files with data are found on the brick path, copy them via a gluster mount point before re-purposing the removed brick.

    d. 在 servera 上删除并重新创建 /bricks/brick-a6/brick。在 serverb 上删除并重新创建 /bricks/brick-b6/brick。

    [root@servera ~]$ rm -rf /bricks/brick-a6/brick
    [root@servera ~]# mkdir /bricks/brick-a6/brick

    [root@serverb ~]$ rm -rf /bricks/brick-b6/brick
    [root@serverb ~]# mkdir /bricks/brick-b6/brick

    e. 将 brick-a6 和 brick-b6 这两个 brick 重新连接到 dev-vol。

    [root@servera ~]$ gluster volume tier dev-vol attach replica 2 \
    > servera:/bricks/brick-a6/brick \
    > serverb:/bricks/brick-b6/brick
    volume attach-tier: success
    Tiering Migration Functionality: dev-vol: success: Attach tier is successful
    on dev-vol. use tier status to check the status.
    ID: 9e422b41-91fc-428a-b3e1-c95b2df6685f

6. 验证热层已连接至卷 dev-vol。

    a. 从 servera，通过 gluster volume info 验证热层已连接到 dev-vol。

    [root@servera ~]$ gluster volume info dev-vol
    Volume Name: dev-vol
    Type: Tier
    Volume ID: 16627eb3-2650-4a87-b664-d2e893bac3fd
    Status: Started
    Number of Bricks: 4
    Transport-type: tcp
    Hot Tier :
    Hot Tier Type : Replicate
    Number of Bricks: 1 x 2 = 2
    Brick1: serverb:/bricks/brick-b6/brick
    Brick2: servera:/bricks/brick-a6/brick
    Cold Tier:
    Cold Tier Type : Replicate
    Number of Bricks: 1 x 2 = 2
    Brick5: servera:/bricks/brick-a4/brick
    Brick6: serverb:/bricks/brick-b4/brick
    Options Reconfigured:
    cluster.tier-mode: cache
    features.ctr-enabled: on
    performance.readdir-ahead:


### 实验：管理分层

连接、断开、扩展和缩小卷的热层。 

1. 验证 prod-vol 卷，以及哪些 brick 已挂载到 serverc 和 serverd 上。

    a. 从 serverc，验证 prod-vol 的卷信息。

    [root@serverc ~]$ gluster volume info prod-vol
    Volume Name: prod-vol
    Type: Replicate
    Volume ID: be44e961-c194-4cf2-9e33-418e824a9163
    Status: Started
    Number of Bricks: 1 x 2 = 2
    Transport-type: tcp
    Bricks:
    Brick1: serverc:/bricks/brick-c1/brick
    Brick2: serverd:/bricks/brick-d1/brick
    Options Reconfigured:
    performance.readdir-ahead: on

    b. 从 serverc，验证为 serverc 和 serverd 挂载了哪些 brick。

    [student@workstation ~]$ for I in server{c,d}
    > do
    >   ssh ${I} "mount | grep brick"
    > done
    /dev/mapper/vg_bricks-brick--c1 on /bricks/brick-c1 type xfs (rw,relatime,seclabel,attr2,inode64,logbsize=64k,sunit=128,swidth=128,noquota)
    /dev/mapper/vg_bricks-brick--c2 on /bricks/brick-c2 type xfs (rw,relatime,seclabel,attr2,inode64,logbsize=64k,sunit=128,swidth=128,noquota)
    /dev/mapper/vg_bricks-brick--c3 on /bricks/brick-c3 type xfs (rw,relatime,seclabel,attr2,inode64,logbsize=64k,sunit=128,swidth=128,noquota)
    /dev/mapper/vg_bricks-brick--c4 on /bricks/brick-c4 type xfs (rw,relatime,seclabel,attr2,inode64,logbsize=64k,sunit=128,swidth=128,noquota)
    /dev/mapper/vg_bricks-brick--c5 on /bricks/brick-c5 type xfs (rw,relatime,seclabel,attr2,inode64,logbsize=64k,sunit=128,swidth=128,noquota)
    /dev/mapper/vg_bricks-brick--c6 on /bricks/brick-c6 type xfs (rw,relatime,seclabel,attr2,inode64,logbsize=64k,sunit=128,swidth=128,noquota)

    /dev/mapper/vg_bricks-brick--d1 on /bricks/brick-d1 type xfs (rw,relatime,seclabel,attr2,inode64,logbsize=64k,sunit=128,swidth=128,noquota)
    /dev/mapper/vg_bricks-brick--d2 on /bricks/brick-d2 type xfs (rw,relatime,seclabel,attr2,inode64,logbsize=64k,sunit=128,swidth=128,noquota)
    /dev/mapper/vg_bricks-brick--d3 on /bricks/brick-d3 type xfs (rw,relatime,seclabel,attr2,inode64,logbsize=64k,sunit=128,swidth=128,noquota)
    /dev/mapper/vg_bricks-brick--d4 on /bricks/brick-d4 type xfs (rw,relatime,seclabel,attr2,inode64,logbsize=64k,sunit=128,swidth=128,noquota)
    /dev/mapper/vg_bricks-brick--d5 on /bricks/brick-d5 type xfs (rw,relatime,seclabel,attr2,inode64,logbsize=64k,sunit=128,swidth=128,noquota)
    /dev/mapper/vg_bricks-brick--d6 on /bricks/brick-d6 type xfs (rw,relatime,seclabel,attr2,inode64,logbsize=64k,sunit=128,swidth=128,noquota)
                
2. 将含有 serverc:/bricks/brick-c2/brick 和 serverd:/bricks/brick-d2/brick 的热层连接到 prod-vol 卷。

    a. 从 serverc，将含有 serverc:/bricks/brick-c2/brick 和 serverd:/bricks/brick-d2/brick 的热层连接到 prod-vol 卷。

    [root@serverc ~]$ gluster volume tier prod-vol attach replica 2 \
    > serverc:/bricks/brick-c2/brick \
    > serverd:/bricks/brick-d2/brick
    volume attach-tier: success
    Tiering Migration Functionality: prod-vol: success: Attach tier is successful on prod-vol. use tier status to check the status.
    ID: 9e422b41-91fc-428a-d3e1-c95d2df6685f

3. 验证 prod-vol 的卷信息。

    a. 从 serverc，验证 prod-vol 的卷信息。

    [root@serverc ~]$ gluster volume info prod-vol
    Volume Name: prod-vol
    Type: Tier
    Volume ID: be44e961-c194-4cf2-9e33-418e824a9163
    Status: Started
    Number of Bricks: 4
    Transport-type: tcp
    Hot Tier :
    Hot Tier Type : Replicate
    Number of Bricks: 1 x 2 = 2
    Brick1: serverd:/bricks/brick-d2/brick
    Brick2: serverc:/bricks/brick-c2/brick
    Cold Tier:
    Cold Tier Type : Replicate
    Number of Bricks: 1 x 2 = 2
    Brick3: serverc:/bricks/brick-c1/brick
    Brick4: serverd:/bricks/brick-d1/brick
    Options Reconfigured:
    cluster.tier-mode: cache
    features.ctr-enabled: on
    performance.readdir-ahead: on

4. 从 serverc，验证 prod-vol 的分层启用状态。

[root@serverc ~]$ gluster volume tier prod-vol status
Node                 Promoted files       Demoted files        Status
---------            ---------            ---------            ---------
localhost               1                    2                    in progress
serverd.lab.example.com 0                    1                    in progress
Tiering Migration Functionality: prod-vol: success

5. 从 serverc，通过 gluster volume info 验证热层已连接到 dev-vol。

[root@serverc ~]$ gluster volume info dev-vol
Volume Name: dev-vol
Type: Tier
Volume ID: 16627ed3-2650-4a87-b664-d2e893bac3fd
Status: Started
Number of Bricks: 6
Transport-type: tcp
Hot Tier :
Hot Tier Type : Distributed-Replicate
Number of Bricks: 2 x 2 = 4
Brick1: serverd:/bricks/brick-d6/brick
Brick2: serverc:/bricks/brick-c6/brick
Brick3: serverd:/bricks/brick-d5/brick
Brick4: serverc:/bricks/brick-c5/brick
Cold Tier:
Cold Tier Type : Replicate
Number of Bricks: 1 x 2 = 2
Brick5: serverc:/bricks/brick-c4/brick
Brick6: serverd:/bricks/brick-d4/brick
Options Reconfigured:
cluster.tier-mode: cache
features.ctr-enabled: on
performance.readdir-ahead: on

6. 删除 brick-c5 和 brick-d5，以缩小热层。

    a. 从 serverc，断开 dev-vol 中的热层。

    [root@serverc ~]$ gluster volume tier dev-vol detach start
    volume detach-tier start: success
    ID: 171d000e-d2f3-4623-9697-d96623a00226

    b. 验证层断开操作的完成状态。

    [root@serverc ~]$ gluster volume tier dev-vol detach status
                                    Node Rebalanced-files          size       scanned      failures       skipped
           status   run time in secs
                                   ---------      -----------   -----------   -----------
          ------------          --------------
                                   localhost                0        0Bytes            20
           completed               1.00
                     serverd.lab.example.com                0        0Bytes             0
           completed               1.00

    c. 执行热层的断开操作。

    [root@serverc ~]$  gluster volume tier dev-vol detach commit
    Removing tier can result in data loss. Do you want to Continue? (y/n) y
    volume detach-tier commit: success
    Check the detached bricks to ensure all files are migrated.
    If files with data are found on the brick path, copy them via a gluster mount point before re-purposing the removed brick.

    d. 清理 brick-c6 和 brick-d6 这两个 brick。

    [root@serverc ~]# rm -rf /bricks/brick-c6/brick
    [root@serverc ~]# mkdir /bricks/brick-c6/brick

    [root@serverd ~]# rm -rf /bricks/brick-d6/brick
    [root@serverd ~]# mkdir /bricks/brick-d6/brick

    e. 仅将 brick-c6 和 brick-d6 这两个 brick 重新连接到 dev-vol。

    [root@serverc ~]$ gluster volume tier dev-vol attach replica 2 \
    > serverc:/bricks/brick-c6/brick \
    > serverd:/bricks/brick-d6/brick
    volume attach-tier: success
    Tiering Migration Functionality: dev-vol: success: Attach tier is successful
    on dev-vol. use tier status to check the status.
    ID: 9e422b41-91fc-428a-d3e1-c95d2df6685f

7. 验证热层已连接至卷 dev-vol。

    a. 从 serverc，通过 gluster volume info 验证热层已连接到 dev-vol。

    [root@serverc ~]$ gluster volume info dev-vol
    Volume Name: dev-vol
    Type: Tier
    Volume ID: 16627ed3-2650-4a87-b664-d2e893bac3fd
    Status: Started
    Number of Bricks: 4
    Transport-type: tcp
    Hot Tier :
    Hot Tier Type : Distributed-Replicate
    Number of Bricks: 1 x 2 = 2
    Brick1: serverd:/bricks/brick-d6/brick
    Brick2: serverc:/bricks/brick-c6/brick
    Cold Tier:
    Cold Tier Type : Replicate
    Number of Bricks: 1 x 2 = 2
    Brick5: serverc:/bricks/brick-c4/brick
    Brick6: serverd:/bricks/brick-d4/brick
    Options Reconfigured:
    cluster.tier-mode: cache
    features.ctr-enabled: on
    performance.readdir-ahead: on

### 总结

在本章中，您学到了：

*    红帽 Gluster 存储中的分层功能支持根据数据的访问频率来放置数据。
*    可以通过从属于热层和冷层这两个分层的 brick 来配置分层卷。
*    热层基于高性能硬件，包含访问频率较高的文件。
*    冷层基于低性能硬件，包含访问频率较低的文件。
*    冷层和热层都可以扩展或缩小。



