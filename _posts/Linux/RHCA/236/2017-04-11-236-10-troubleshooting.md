---
layout: post
title:  "Troubleshooting(236-10)"
categories: Linux
tags: RHCA 236
---

### 管理失效 Brick

###### 管理自我修复

对于复制卷，有 brick 离线时需要执行自我修复后才能重新同步所有副本。这由自我修复守护进程 (glusterfshd) 进行透明处理，该守护进程在带有复制 brick 的所有节点上运行。

自我修复守护进程将识别（可能）需要修复的文件，然后在不同的复制 brick 之间同步这些文件。若要查看目前需要修复的文件列表，可以使用 gluster volume heal <VOLUME> info 命令。如果未列出文件，则所有文件都已修复，或者不需要修复。

有关已修复文件数量的详细信息，可使用 gluster volume heal <VOLUME> statistics 命令找到。此命令将显示每一循环每个 brick 修复的文件数量。

1. 触发自我修复

在需要时，管理员可以运行 gluster volume heal <VOLUME> heal [full] 命令触发自我修复。

若不使用 full 选项，将根据更新时间执行普通修复。若使用 full 选项，将执行完整修复。

2. 管理脑裂文件

在某些情况中，文件可能会出现脑裂状况；例如，如果与两个不同 brick 交互的两个客户端同时更新了某一文件，但与其他副本的通信存在错误。在这样的情形中，这一文件将被标记为脑裂 (split-brain)，此状况无法由红帽 Gluster 存储自动解决。

可以使用 gluster volume heal <VOLUME> info split-brain 命令查找处于脑裂状态的文件列表。

若要手动恢复处于脑裂状态的文件，需要使用 setfattr 和 getfattr 命令手动编辑 brick 上文件的扩展属性。《红帽 Gluster 管理指南》中关于手动恢复脑裂文件的附录部分提供了详细的步骤。 

###### 更换 brick

如果某一 brick 或支持该 brick 的物理存储出现了故障，它需要被更换掉。更换 brick 的方式取决于该 brick 所属卷的类型。

1. 更换分布式卷中的 brick

在纯粹的分布式卷上，可以通过添加新 brick 再删除旧 brick 来更换 brick。删除旧 brick 时触发重新平衡操作，将所有文件移动到新 brick。

请参见第 7 章 扩展卷了解为卷添加和删除 brick 的步骤。

> 警告: 如果需要更换的 brick 已离线，可以使用与复制卷相同的步骤，但这会导致数据丢失。

2. 更换复制卷中的 brick

对于具有数据冗余能力的卷，可以使用 gluster volume replace-brick <VOLUME> OLD_BRICK NEW_BRICK commit force 命令来更换 brick。以这种方式更换 brick 时，OLD_BRICK 可以处于离线状态。

这将立即删除旧 brick 并添加新 brick，而这会触发自我修复操作。修复操作的状态可以通过 gluster volume heal <VOLUME> info 命令监控。

> 注意: 在以前版本的红帽  Gluster 存储中，可以通过迁移数据来启动 brick 更换操作，然后在所有数据迁移后执行。当前版本中不再提供这些选项，因为自我修复和添加/删除操作可以执行相同的功能。 


### 引导式练习：管理失效 Brick

servera 系统已停机。唯一从此系统提供的卷名为 replvol，属于复制卷。使 servera 恢复在线，然后查看该系统的自我修复信息。在验证所有文件已正确修复后，将 servera 上的 brick 更换为 brick servera:/bricks/brick-a2/brick。 

1. 在 serverb 系统上，检查 replvol 卷上需要修复的文件列表。

    [root@serverb ~]# gluster volume heal replvol info

2. 使 servera 系统恢复在线。不要重置此系统，只需启动它。
3. 验证 replvol 卷的修复状态。等待所有文件完成修复。

    a. 检查 replvol 卷的自修复进程状态。

    [root@servera ~]# gluster volume heal replvol info

    b. 重复上一命令，直到报告零文件的所有 brick 已修复。 

4. 将 servera 上 replvol 中的 brick 更换为 brick servera:/bricks/brick-a2/brick。

    a. 确定 servera 上正在使用中的 brick。

    [root@servera ~]# gluster volume info replvol

    Volume Name: replvol
    Type: Replicate
    Volume ID: 291fefbe-210e-41b3-ac93-03ce8dfb7293
    Status: Started
    Number of Bricks: 1 x 2 = 2
    Transport-type: tcp
    Bricks:
    Brick1: servera:/bricks/brick-a1/brick
    Brick2: serverb:/bricks/brick-b1/brick
    Options Reconfigured:
    performance.readdir-ahead: on

    b. 将 servera:/bricks/brick-a1/brick 更换为 servera:/bricks/brick-a2/brick。

    [root@servera ~]# gluster volume replace-brick replvol \
    > servera:/bricks/brick-a1/brick \
    > servera:/bricks/brick-a2/brick commit force
    volume replace-brick: success: replace-brick commit force operation successful


### 配置 BitRot 检测

###### 启用 BitRot 检测

红帽 Gluster 存储提供一项功能，其名为 BitRot 检测。启用之后，卷上的所有文件将以固定的间隔被清理，同时计算校验和。此校验和将存储在 brick 上文件的扩展属性中。如果文件自上次校验和存储后有了更改（根据更改日志），将计算和存储新的校验和。如果文件根据更改日志没有更改，但计算的校验和与存储的校验和不符，则向日志文件 /var/log/glusterfs/bitd.log 和 /var/log/glusterfs/scrub.log 写入错误。

这样，管理员可以在数据因故障存储介质而隐性损坏时检测到。此功能最常用于使用存储在 JBOD 上 brick 的卷，而不用于存储在 RAID6 等已启用内置错误检测（和纠正）的 brick 上的卷。

若要为卷启用 BitRot 检测，可使用 gluster volume bitrot <VOLUME> enable 命令。若要禁用 BitRot 检测，则可使用 gluster volume bitrot <VOLUME> disable 命令。 

###### 配置 BitRot 检测

为所有文件计算校验和可能对卷的性能造成负面影响，尤其是较大的卷，因此可以配置清理的频率以及可以同时清理的文件数量。

以下命令可以影响和查询 BitRot 检测守护进程及相关清理进程的设置。

1. gluster volume bitrot <VOLUME> scrub status
    显示指定卷的 BitRot 检测的状态和设置。同时也列出检测到错误的所有文件。

2. gluster volume bitrot <VOLUME> scrub pause
    暂停指定卷的清理。这不会停止 BitRot 检测，只是暂停清理流程。当存储带宽需要用于其他用途时，可以利用此功能。

3. gluster volume bitrot <VOLUME> scrub resume
    恢复指定卷的清理。

4. gluster volume bitrot <VOLUME> scrub-throttle <RATE>
    配置每个节点可以同时清理的文件数量。值可以是 lazy、normal 和 aggressive。
    lazy 允许一次清理一个文件，normal 允许一次清理两个或 (CPUS - 4) / 2 个（取较大者）文件，而 aggressive 允许一次清理四个或 (CPUS - 4) / 2 个（取较大者）文件。
    默认值为 lazy。

5. gluster volume bitrot <VOLUME> scrub-frequency <FREQUENCY>
    配置对校验和清理所有文件的频率。值可以是 hourly、daily、weekly、biweekly 和 monthly。
    默认值为 biweekly，即每两周一次。 

###### 恢复损坏的文件

当 BitRot 检测到损坏文件时，它将根据文件的 GFID来报告文件；GFID 是以 32 位十六进制数表示的 128 位编号。若要定位该文件，可使用 find 命令。请勿忘记从报告的 GFID 中删除破折号。

[root@demoa ~]# find /path/to/brick/.glusterfs -name GFID
/path/to/brick/.glusterfs/XX/YY/GFID
[root@demoa ~]# find /path/to/brick/ -samefile  /path/to/brick/.glusterfs/XX/YY/GFID
/path/to/brick/.glusterfs/XX/YY/GFID
/path/to/brick/corruptedfile

在找到损坏的文件后，可以删除该损坏文件，以及 brick 上 .glusterfs 中的相关文件。这两个文件都可以通过上例中最后的 find 命令找到。如果损坏的文件设有任何其他硬链接，也需要将它们删除。

如果含有损坏文件的 brick 是复制卷的一部分，可以通过触发修复（使用 gluster volume heal <VOLUME> 或等待自我修复）来恢复损坏文件。

如果损坏文件不在复制卷上，则需要从最近的备份中进行恢复。 


### 引导式练习：配置 BitRot 检测

要求为从 servera 和 serverb 提供的 replvol 卷配置 BitRot 检测。BitRot 守护进程应当每小时检查文件，并且它应当同时处理尽可能多的文件。 

1. 使用给定的规格，为 replvol 启用和配置 BitRot 检测。

    a. 为 replvol 启用 BitRot 检测。

    [root@servera ~]# gluster volume bitrot replvol enable

    b. 将 replvol 的 BitRot 检测配置为每小时扫描一次所有文件。

    [root@servera ~]# gluster volume bitrot replvol scrub-frequency hourly

    c. 将 replvol 的 BitRot 检测配置为同时扫描最多数量的文件。

    [root@servera ~]# gluster volume bitrot replvol scrub-throttle aggressive

### 实验：故障排除

serverc 系统最近遇到了一些问题，目前已关机。您已被要求将它打开，确保 datavol 卷上的所有文件都已成功修复，然后将 brick serverc:/bricks/brick-c1/brick 更换为该卷中的 brick serverc:/bricks/brick-c2/brick。

您也被要求为 datavol 卷配置 BitRot 检测，设置为每天检查文件并且同时扫描 normal 数量的文件。 

1. 验证 datavol 卷的修复状态。打开 serverc 计算机，等待所有文件完成修复，然后继续。

    a. 检查 datavol 卷的自修复进程状态。

    [root@serverd ~]# gluster volume heal datavol info

    b. 启动 serverc 计算机。不要重置，只需启动它。

    c. 检查 datavol 卷的自修复进程状态。

    [root@serverd ~]# gluster volume heal datavol info

    d. 重复上一命令，直到报告零文件的所有 brick 已修复。 

2. 将 serverc 上 datavol 中的 brick 更换为 brick serverc:/bricks/brick-c2/brick。

    a. 确定 serverc 上正在使用中的 brick。

    [root@serverc ~]# gluster volume info datavol
    Volume Name: datavol
    Type: Replicate
    Volume ID: 291fefbe-210e-41b3-ac93-03ce8dfb7293
    Status: Started
    Number of Bricks: 1 x 2 = 2
    Transport-type: tcp
    Bricks:
    Brick1: serverc:/bricks/brick-c1/brick
    Brick2: serverd:/bricks/brick-d1/brick
    Options Reconfigured:
    performance.readdir-ahead: on

    b. 将 serverc:/bricks/brick-c1/brick 更换为 serverc:/bricks/brick-c2/brick。

    [root@serverc ~]# gluster volume replace-brick datavol \
    > serverc:/bricks/brick-c1/brick \
    > serverc:/bricks/brick-c2/brick commit force
    volume replace-brick: success: replace-brick commit force operation successful

3. 使用给定的规格，为 datavol 启用和配置 BitRot 检测。

    a. 为 datavol 启用 BitRot 检测。

    [root@serverc ~]# gluster volume bitrot datavol enable

    b. 将 datavol 的 BitRot 检测配置为每天扫描一次所有文件。

    [root@serverc ~]# gluster volume bitrot datavol scrub-frequency daily

    c. 将 datavol 的 BitRot 检测配置文件同时扫描 normal 数量的文件。

    [root@serverc ~]# gluster volume bitrot datavol scrub-throttle normal


### 总结

在本章中，您学到了：

*    自我修复通过 gluster volume heal 命令进行管理。
*    处于脑裂状态的文件可以通过修改正确的扩展属性来恢复。
*    在分布式卷中，可以通过添加新 brick 再删除旧 brick 来更换 brick。
*    可以通过 gluster volume replace-brick 命令更换复制卷中的 brick。
*    可以为卷启用 BitRot 检测守护进程，以提醒管理员隐形数据损坏。 
