---
layout: post
title:  "Managing Snapshots(236-11)"
categories: Linux
tags: RHCA 236
---

### 管理快照

###### 创建快照

红帽  Gluster 存储允许管理员创建卷快照，即内容在某一时点的只读呈现。红帽  Gluster 存储利用组成卷的 brick 的 LVM 快照来实施这些快照；因此，如果需要快照，就必须使用精简配置的 LVM 来创建所有 brick。

要创建卷快照，可以使用 gluster snapshot create <SNAPSHOTNAME> <VOLUME> [no-timestamp] [description DESCRIPTION] [force] 命令。

除非使用了 no-timestamp 选项，否则快照名称上将附加以 GMT 表示的当前时间的时间戳。

> 注意: 如果对要执行快照的卷配置了异地复制，则需要暂停异地复制后才能执行快照。

默认情况下，每个卷允许最多 256 个快照。达到此限制时，无法创建新的快照。管理员可以降低此限制，但不能提高。

###### 挂载快照

默认情况下，创建快照时，它将以 “Stopped” 状态创建。这意味着，客户端无法访问快照。若要使快照能被访问，需要通过命令 gluster snapshot activate SNAPSHOTNAME 激活快照。

在激活快照后，客户端可以借助原生客户端将它挂载为 <SERVER>:/snaps/<SNAPSHOTNAME>/<PARENTVOLUME>。快照在服务器侧实施为只读状态。

###### 删除快照

不再需要某一快照时，可以使用 gluster snapshot delete <SNAPSHOTNAME> 将它删除。

也可以使用 gluster snapshot delete volume <VOLUME> 命令同时删除某个卷的所有快照。要删除所有卷的全部快照，可使用命令 gluster snapshot delete all。

###### 用户可用的快照

为方便用户从快照恢复自己的数据，而无需系统管理员进行干预，可以配置用户可用的快照 (uss)。启用之后，带有快照的卷上的每一目录将具有名为 .snaps 的虚拟目录。此目录不会显示在目录列表中，即使使用 ls 的 -a 选项也如此，但它是可以访问的。此目录中包含每个快照的子目录，这些目录中含有来自快照的父目录的数据。

> 注意: 尝试访问快照中不具有该目录的目录时，将引发 stale file handle 错误消息。

要启用用户可用的快照，应当将卷的卷选项 features.uss 设置为 enable。

> 注意: 只有激活的快照才会显示在 .snaps 目录中。如果卷目前已由任何客户端通过原生客户端挂载，则无法对该卷设置 features.uss 选项。 

###### 从快照恢复

从快照恢复数据的方式主要有三种：

*    从挂载的快照（或用户可用的快照）复制文件到原始位置。
*    通过 gluster snapshot clone <NEWVOLUME> <SNAPSHOTNAME> 将快照克隆到新卷。这将提供空间节约型克隆，仅存储与快照不同的数据。原始快照不受此操作影响。
*    使用 gluster snapshot restore <SNAPSHOTNAME> 命令将快照提升为原始卷。恢复之后，需要通过 gluster volume heal <VOLUME> full 命令为卷触发完整修复。

    以这种方式恢复快照将删除原始卷并把它替换为快照，然后将快照删除。
    
    > 重要: 以这种方式恢复快照会改变该卷中所有 brick 的 brick 路径。如果原始 brick 已通过 /etc/fstab 挂载，则之后需要对它们进行修复。 

###### 查看快照信息

可以通过多种方式来检索快照的相关信息：

*   gluster snapshot list                       显示所有快照的列表。这仅列出其名称。
*   gluster snapshot info [<SNAPSHOTNAME>]      显示所列快照的详细信息；如果命令行中未给定快照，则显示所有快照。
*   gluster snapshot status [<SNAPSHOTNAME>]    显示所请求快照或所有快照中每个 brick 的信息，包括快照中各 brick 使用的空间大小。 

### 引导式练习：管理快照

servera 和 serverb 计算机上的受信存储池正在提供一个名为 snapvol 的卷，它挂载到 workstation 上的 /mnt/snapvol。此卷目前有一个快照，名为 original。

您已被要求执行以下任务：

*    为 snapvol 卷创建一个新快照，名为 safetysnap。此快照不应标记时间戳。重要信息：此任务应当在所有其他任务之前完成。
*    启用用户可用的 snapvol 卷快照。original 和 safetysnap 快照都应当能以这种方式访问。
*    将 original 快照永久挂载到 workstation 上的 /mnt/original。
*    从 original 快照恢复 file02、file04、file08 和 file16 文件到 snapvol 卷。
    
1. 为 snapvol 卷创建一个新快照，名为 safetysnap。

    a. 使用 gluster snapshot create 创建该快照。记住抑制自动时间戳。

    [root@servera ~]# gluster snapshot create safetysnap snapvol no-timestamp

    b. 验证该快照已创建完毕。

    [root@servera ~]# gluster snapshot info safetysnap
    Snapshot                  : safetysnap
    Snap UUID                 : 370a41b9-d54f-4004-8c6e-1eb74315aae4
    Created                   : 2016-05-26 11:09:42
    Snap Volumes:

    	Snap Volume Name          : c135509d9ce54aeaa1e284a71df77ba6
    	Origin Volume name        : snapvol
    	Snaps taken for snapvol      : 2
    	Snaps available for snapvol  : 254
    	Status                    : Stopped

2. 启用用户可用的 snapvol 卷快照。original 和 safetysnap 快照都应当能以这种方式访问。

    a. 从 workstation 卸载该 snapvol 卷。此操作是必要的，因为当卷被客户端挂载后，就无法启用用户可用的快照功能。

    [root@workstation ~]# umount /mnt/snapvol

    b. 启用用户可用的 snapvol 卷快照。

    [root@servera ~]# gluster volume set snapvol features.uss enable

    c. 将 snapvol 卷挂载到 workstation 上的 /mnt/snapvol。

    [root@workstation ~]# mount /mnt/snapvol

    d. 激活 original 和 safetysnap 快照。快照需要激活后才能直接或利用用户可访问的快照进行挂载。

    [root@servera ~]# gluster snapshot activate original
    [root@servera ~]# gluster snapshot activate safetysnap

    e. 在 workstation 上，验证 /mnt/snapvol/.snaps/ 目录现在列出了两个快照。

    [root@workstation ~]# ls /mnt/snapvol/.snaps/
    original safetysnap

3. 将 original 快照永久挂载到 workstation 上的 /mnt/original。

    a. 在 workstation 上，创建 /mnt/original 目录。

    [root@workstation ~]# mkdir /mnt/original

    b. 将下面这一行添加到 workstation 上的 /etc/fstab：

    servera:/snaps/original/snapvol /mnt/original glusterfs _netdev 0 0

    c. 挂载 original 快照。

    [root@workstation ~]# mount /mnt/original

    d. 验证 original 快照的内容现已可用。

    [root@workstation ~]# cat /mnt/original/file00
    This file is original

4. 从 original 快照恢复 file02、file04、file08 和 file16 文件到 snapvol 卷。

    a. 将列出的四个文件从 /mnt/original 复制到 /mnt/snapvol。

    [root@workstation ~]# for FILE in /mnt/original/file{02,04,08,16}
    > do
    >   cp ${FILE} /mnt/snapvol/
    > done


### 调度快照

红帽  Gluster 存储随附一个快照调度程序。通过此调度程序，管理员可以固定间隔为卷调度快照。

快照调度程序可以同时在多个节点上运行；而且，利用锁文件，每一作业仅由一个节点进行处理。这可实现快照创建的一贯性，即使出现一个或多个节点不可用。

###### 配置快照调度

在能够为卷调度快照之前，首先需要在所有参与快照调度的节点上初始化并启用快照调度程序。这需要执行几个步骤：

1. 通过运行 gluster volume set all cluster.enable-shared-storage enable 命令启用共享存储。
2. 通过在所有节点上启用 SELinux Boolean cron_system_cronjob_use_shares，允许 crond 守护进程访问标有 fusefs_t 的文件。(setsebool -P cron_system_cronjob_use_shares 1)
3. 在每个节点上，通过运行命令 snap_scheduler.py init 初始化快照调度程序目录。这将在共享存储上创建所需的文件。
4. 在每个节点上，通过运行命令 snap_scheduler.py enable 启用快照调度程序。这将在系统 cron 作业和快照调度程序创建的作业之间添加链接。

###### 调度快照

在所有节点上初始化并启用快照调度程序后，可以添加作业。作业需要作业名称、使用 cron 语法的计划，以及要为其创建快照的卷。若要添加新作业，可使用以下命令：

    [root@demo ~]# snap_scheduler.py add "<JOBNAME>" "<SCHEDULE>" <VOLUME>

在此命令中，JOBNAME 是这个计划的唯一标识符，SCHEDULE 是使用 cron 语法且包含五个字段的计划，VOLUME 则是应当为其创建快照的卷。

计划中的五个字段从左到右分别为：minutes、hours、day-of-month、month 和 day-of-week。有关计划允许使用的语法的更多信息，请参见 crontab(5) man page。 

###### 管理计划

可以查看、编辑和删除现有的快照。若要查看当前配置的作业列表，可使用命令 snap_scheduler.py list。若要删除作业，可使用命令 snap_scheduler.py delete JOBNAME。

若要修改作业，可使用命令snap_scheduler.py edit "<JOBNAME>" "<SCHEDULE>" <VOLUME>

可以在 /var/log/glusterfs/snap_scheduler.log 和 /var/log/glusterfs/gcron.log 中找到快照调度程序的日志文件。

###### 配置快照行为

可以通过 gluster snapshot config 命令配置快照行为。这种方式可以设置若干个选项：

*    activate-on-create          如果设为 enable，新快照将自动激活。默认为 disable。
*    snap-max-hard-limit         任何一个卷所允许的快照数量上限。默认值为 256。
*    snap-max-soft-limit         在发出警告之前卷上允许的快照数量。如果启用了 auto-delete，则达到 snap-max-soft-limit 时自动删除最旧的快照。此设置以 snap-max-hard-limit 的百分比形式表示，默认值为 90。
*    auto-delete                 如果设为 enable，则达到 snap-max-soft-limit 时自动删除最旧的快照。 


### 引导式练习：调度快照

为 servera 和 serverb 上的 snapvol 卷配置自动快照。这些快照的计划名称应当为 serenity。这些快照及其他所有快照都应当自动激活，方便用户进行访问。每个卷允许的快照数量不能超过 10 个，当一个卷的快照数量超过 5 个时，应当自动删除最旧的快照。 

1. 根据指定的要求配置快照限制：

    每个卷最多 10 个快照。
    当一个卷的快照数量超过 5 个时，自动删除最旧的快照。
    自动激活新快照。 

    a. 配置 snap-max-hard-limit 和 snap-max-soft-limit，使其符合最多 10 个快照并在达到 5 个快照后自动删除的限制要求。记住 snap-max-soft-limit 表示为 snap-max-hard-limit 的百分比。

    [root@servera ~]# gluster snapshot config snap-max-hard-limit 10 \
    > snap-max-soft-limit 50
    Changing snapshot-max-hard-limit will limit the creation of new snapshots if they exceed the new snapshot-max-hard-limit.
    If Auto-delete is enabled, snap-max-soft-limit will trigger deletion of oldest snapshot, on the creation of a new snapshot, when the snap-max-soft-limit is reached
    Do you want to continue> (y/n) y
    snapshot config: snap-max-hard-limit & snap-max-soft-limit for system set successfully

    b. 超过 snap-max-soft-limit 时启用自动删除快照。

    [root@servera ~]# gluster snapshot config auto-delete enable
    snapshot config: auto-delete successfully set

    c. 启用自动激活新快照。

    [root@servera ~]# gluster snapshot config activate-on-create enable
    snapshot config: activate-on-create successfully set

2. 通过启用共享存储，并允许 crond 访问标有 fusefs_t 的文件，准备 servera 和 serverb 以进行快照调度。

    a. 为红帽  Gluster 存储启用共享存储。

    [root@servera ~]# gluster volume set all cluster.enable-shared-storage enable

    b. 在 servera 和 serverb 上，通过永久启用 cron_system_cronjob_use_shares SELinux Boolean，允许 crond 访问标有 fusefs_t 的文件。

    [root@servera ~]# setsebool -P cron_system_cronjob_use_shares 1
    [root@serverb ~]# setsebool -P cron_system_cronjob_use_shares 1

3. 在 servera 和 serverb 上初始化并启用快照调度程序，然后创建名为 serenity 的新计划，该计划将每隔两分钟对 snapvol 卷执行快照。

    a. 在 servera 和 serverb 上，初始化快照调度程序。

    [root@servera ~]# snap_scheduler.py init

    [root@serverb ~]# snap_scheduler.py init

    b. 在 servera 上，启用快照调度程序。

    [root@servera ~]# snap_scheduler.py enable

    c. 创建名为 serenity 的新计划，该计划每隔两分钟为 snapvol 卷执行新快照。

    [root@servera ~]# snap_scheduler.py add serenity "*/2 * * * *" snapvol

4. 验证快照现在已调度为两分钟的间隔。

    a. 查看为快照调度程序配置的计划。

    [root@servera ~]# snap_scheduler.py list
    JOB_NAME         SCHEDULE         OPERATION        VOLUME NAME
    --------------------------------------------------------------------
    serenity         */2 * * * *      Snapshot Create  snapvol

    b. 查看当前快照的列表。

    [root@servera ~]# gluster snapshot list
    Scheduled-serenity-snapvol_GMT-2016.05.27-11.38.01
    Scheduled-serenity-snapvol_GMT-2016.05.27-11.40.01
    Scheduled-serenity-snapvol_GMT-2016.05.27-11.42.01
    Scheduled-serenity-snapvol_GMT-2016.05.27-11.44.02
    Scheduled-serenity-snapvol_GMT-2016.05.27-11.46.01


### 实验：管理快照

有一个受信存储池，它包含两台计算机：serverc 和 serverd。这个受信存储池当前托管了一个名为 zoo 的卷，利用原生客户端挂载到 workstation 上的 /mnt/zoo。

此卷目前有两个快照：elephants 和 tigers。tigers 快照已不再需要，应当将它删除以回收空间。另一项目需要读写 elephants 快照中的数据，因此需要将该快照克隆到名为 otters 的新卷。应当要启动此卷，从而让其他项目能够尽快开始工作。应当由 zoo 卷生成名为 snakes 的无时间戳快照。

您已被要求进行以下配置，从而简化您的用户的工作流：

*    所有新快照都应当自动激活。
*    每个卷应当限制为最多 64 个快照。如果有超过 48 个快照，则应自动删除多余的快照。
*    所有快照应当利用虚拟目录 .snaps 向客户端提供。 

为了能更好地恢复重要的文件，每天凌晨 4 点按照名为 animals 的计划自动从 zoo 卷生成快照。

1. 删除 tigers 快照。

    a. 在 serverc 系统上，删除 tigers 快照。

    [root@serverc ~]# gluster snapshot delete tigers
    Deleting snap will erase all the information about the snap.
    Do you still want to continue (y/n) y

2. 将 elephants 快照克隆到名为 otters 的新卷。启动新的 otters 卷。

    a. 将 elephants 快照克隆到名为 otters 的新卷。

    [root@serverc ~]# gluster snapshot clone otters elephants
    snapshot clone: success: Clone otters created successfully

    b. 启动新的 otters 卷。

    [root@serverc ~]# gluster volume start otters
    volume start: otters: success

3. 启用自动激活所有新快照、自动删除快照，以及允许的快照数上限。

    a. 配置 snap-max-hard-limit 和 snap-max-soft-limit，使其符合最多 64 个快照并在达到 48 个快照后自动删除的限制要求。

    记住 snap-max-soft-limit 表示为 snap-max-hard-limit 的百分比。在本例中，48 个快照就是 64 的 75%。

    [root@serverc ~]# gluster snapshot config snap-max-hard-limit 64 \
    > snap-max-soft-limit 75
    Changing snapshot-max-hard-limit will limit the creation of new snapshots if they exceed the new snapshot-max-hard-limit.
    If Auto-delete is enabled, snap-max-soft-limit will trigger deletion of oldest snapshot, on the creation of a new snapshot, when the snap-max-soft-limit is reached
    Do you want to continue> (y/n) y
    snapshot config: snap-max-hard-limit & snap-max-soft-limit for system set successfully

    b. 超过 snap-max-soft-limit 时启用自动删除快照。

    [root@serverc ~]# gluster snapshot config auto-delete enable
    snapshot config: auto-delete successfully set

    c. 启用自动激活新快照。

    [root@serverc ~]# gluster snapshot config activate-on-create enable
    snapshot config: activate-on-create successfully set

4. 启用通过虚拟目录 .snaps 访问快照。

    a. 从 workstation 卸载 /mnt/zoo，因为不能为已挂载的卷启用用户可用的快照。

    [root@workstation ~]# umount /mnt/zoo

    b. 启用用户可用的 zoo 卷快照。

    [root@serverc ~]# gluster volume set zoo features.uss enable
    volume set: success

    c. 将 /mnt/zoo 挂载到 workstation。

    [root@workstation ~]# mount /mnt/zoo

5. 为 zoo 卷创建一个新快照，名为 snakes。

[root@serverc ~]# gluster snapshot create snakes zoo no-timestamp
snapshot create: success: Snap snakes created successfully

6. 使用名为 animals 的计划，启用 zoo 卷的快照调度计划。每天在凌晨 4 点创建快照。

    a. 为红帽  Gluster 存储启用共享存储。

    [root@serverc ~]# gluster volume set all cluster.enable-shared-storage enable

    b. 在 serverc 和 serverd 上，通过永久启用 cron_system_cronjob_use_shares SELinux Boolean，允许 crond 访问标有 fusefs_t 的文件。

    [root@serverc ~]# setsebool -P cron_system_cronjob_use_shares 1

    [root@serverd ~]# setsebool -P cron_system_cronjob_use_shares 1

    c. 在 serverc 和 serverd 上，初始化快照调度程序。

    [root@serverc ~]# snap_scheduler.py init

    [root@serverd ~]# snap_scheduler.py init

    d. 在 serverc 上，启用快照调度程序。

    [root@serverc ~]# snap_scheduler.py enable

    e. 创建名为 animals 的新计划，该计划于每天凌晨 4 点为 snapvol 卷执行新快照。

    [root@serverc ~]# snap_scheduler.py add animals "0 4 * * *" zoo


### 总结

在本章中，您已学会如何：

*    使用 gluster snapshot create 创建快照。
*    利用原生客户端访问快照。
*    通过 features.uss 选项配置用户可用的快照。
*    通过复制、克隆和恢复等操作，从快照恢复数据。
*    使用 snap_scheduler.py 初始化并启用快照调度。
*    使用 snap_scheduler.py 创建、编辑和删除快照计划。 

