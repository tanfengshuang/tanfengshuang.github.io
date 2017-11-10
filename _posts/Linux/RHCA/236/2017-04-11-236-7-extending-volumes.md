---
layout: post
title:  "Extending Volumes(236-7)"
categories: Linux
tags: RHCA 236
---

### 扩大卷

###### 扩大卷

可以向运行中的卷添加 brick 来扩展它，而不造成任何停机时间。在添加 brick 时，必须始终添加数量充足的 brick，以此保持当前的布局。例如，在扩展副本设为 2 的卷时，必须以 2 的倍数来添加 brick。类似地，在扩展 4x2 分布式复制的卷时，必须添加至少两个或其倍数的 brick。

也可以在扩展卷时增加副本的数量。为此，可在扩展时指定 replica <NEW-NUMBER-OF-REPLICAs> 选项。添加的 brick 数量必须与所需的布局匹配。

gluster volume add-brick <VOLUME> [replica <NEW-REPLICA-COUNT>] BRICK... 命令用于扩展卷。

###### 重新平衡卷

在扩展或缩小卷后，通常需要重新平衡卷。重新平衡卷会将文件在 brick 之间移动，以匹配新的布局。这可能需要将文件移动到不同的 brick 以匹配新的分布式布局，复制文件以同步新的副本，或者这两者的结合。

> 注意: 在将分布式卷扩展为分布式复制卷时，brick 的次序将会改变。可以使用 gluster volume info <VOLUME> 命令来查看新的布局。

要启动重新平衡操作，可使用命令 gluster volume rebalance <VOLUME> start。启动了重新平衡后，可以使用 gluster volume rebalance <VOLUME> status 命令跟踪其进度。 

###### 调整重新平衡操作

重新平衡操作可能会给卷的性能带来负面影响。为防止对卷性能造成任何负面影响，可以对每个卷的重新平衡操作进行调整。这种调整可借助 cluster.rebal-throttle 卷选项来控制。

此选项可取以下三个值之一：

*    lazy           在设为 lazy 时，每个节点仅允许一次迁移一个文件。
*    normal         这是默认的设置。这允许每个节点一次迁移两个文件，或 (NUMBER-OF-LOGICAL_CPUS - 4 ) / 2 个文件，以较大者为准。在具有四个逻辑 CPU 的系统上，这可能表示一次两个文件；在具有八个逻辑 CPU 的系统上，一次两个文件；而在具有十六个逻辑 CPU 的系统上，一次可移动六个文件。
*    aggressive     这允许每个节点一次迁移四个文件，或 (NUMBER-OF-LOGICAL_CPUS - 4 ) / 2 个文件，以较大者为准。在具有四个逻辑 CPU 的系统上，这可能表示一次四个文件；在具有八个逻辑 CPU 的系统上，一次四个文件；而在具有十六个逻辑 CPU 的系统上，一次可移动六个文件。

例如，若要将卷 bigvol 配置为允许激进的重新平衡，可以使用以下命令。

```
[root@demo ~]# gluster volume set bigvol cluster.rebal-throttle aggressive
```


### 引导式练习：扩大卷

受信存储池中有一个复制卷，名为 extendme，它使用两个 brick。该卷上的存储空间已不多，您已被要求将它扩展为 2 x 2 分布式复制卷。为此，已留出了两个 brick：serverc:/bricks/brick-c1/brick 和 serverd:/bricks/brick-d1/brick。两个 brick 都已为您准备好。 

1. 在 servera 上，将这两个新 brick 添加到 extendme 卷，再检查生成的卷。

    a. 将两个新 brick 添加到 extendme 卷。

    [root@servera ~]# gluster volume add-brick extendme serverc:/bricks/brick-c1/brick serverd:/bricks/brick-d1/brick
    volume add-brick: success

    b. 检查生成的卷。

    [root@servera ~]# gluster volume info extendme
    Volume Name: extendme
    Type: Distributed-Replicate
    Volume ID: e72bafb8-32f3-4413-89e2-a5569e322706
    Status: Started
    Number of Bricks: 2 x 2 = 4
    Transport-type: tcp
    Bricks:
    Brick1: servera:/bricks/brick-a1/brick
    Brick2: serverb:/bricks/brick-b1/brick
    Brick3: serverc:/bricks/brick-c1/brick
    Brick4: serverd:/bricks/brick-d1/brick
    Options Reconfigured:
    performance.readdir-ahead: on

2. 重新平衡 extendme 卷。

    a. 启动重新平衡操作。

    [root@servera ~]# gluster volume rebalance extendme start
    volume rebalance: extendme success: Rebalance on extendme has been started succ
    essfully. Use rebalance status command to check status of the rebalance process.
    ID: d24c4a0c-0d39-4d56-b2d0-d3cef19a54e8

    b. 查看重新平衡操作的状态。

    [root@servera ~]# gluster volume rebalance extendme status
                                        Node Rebalanced-files          size       scanned      failures       skipped               status   run time in secs
                                   ---------      -----------   -----------   -----------   -----------   -----------         ------------     --------------
                                   localhost               49      980Bytes           100             0             0            completed               3.00
                     serverb.lab.example.com                0        0Bytes             0             0             0            completed               0.00
                     serverc.lab.example.com                0        0Bytes             2             0             0            completed               0.00
                     serverd.lab.example.com                0        0Bytes             0             0             0            completed               0.00
    volume rebalance: extendme: success


### 缩小卷

可以通过移除一个或多个 brick，在线缩小红帽  Gluster 存储卷。在移除过程中，还可以调整副本计数。

在缩小分布式复制卷时，移除的 brick 数量必须是副本计数的倍数。例如，若要缩小副本计数为 2 的分布式复制卷，需要以 2 的倍数来移除 brick。此外，被移除的 brick 必须来自同一子卷（同一个副本集）。在非复制卷中，所有 brick 必须都可用，才能迁移数据并执行移除 brick 操作。在复制卷中，副本中必须至少有一个 brick 可用。

在大部分情形中，移除一个或多个 brick 后需要开始执行重新平衡操作。 

要移除一个或多个 brick，可按照下列步骤操作： 
1. 使用 remove-brick 命令将 brick 标记为不供新数据使用，并将这些 brick 上的现有数据移到新 brick 上。

    [root@demoa ~]# gluster volume remove-brick demovol demoa:/bricks/demobrick/brick start

2. 查看移除操作的状态；重复查看，直至所有 brick 都报告 completed。

    [root@demoa ~]# gluster volume remove-brick demovol demoa:/bricks/demobrick/brick status
          
3. 在所有 brick 都完成数据迁移后，可以将更改提交至卷。

    [root@demoa ~]# gluster volume remove-brick demovol demoa:/bricks/demobrick/brick commit
          
4. 验证被移除的 brick 上没有遗留数据，再作他用。使用以下命令重新平衡卷：
    
    gluster volume rebalance


### 引导式练习：缩小卷

受信存储池目前有一个 2 x 2 分布式复制卷，名为 shrinkme。由于此卷未充分利用，您已被要求从此卷移除两个 brick。 

1. 在 servera 系统上，检查 shrinkme 卷的布局，再确定可以移除哪两个 brick。

    a. 检查 shrinkme 卷的布局。

    [root@servera ~]# gluster volume info shrinkme

    Volume Name: shrinkme
    Type: Distributed-Replicate
    Volume ID: 9d603f5d-9e36-4965-9fbd-fa84b6a9bc6e
    Status: Started
    Number of Bricks: 2 x 2 = 4
    Transport-type: tcp
    Bricks:
    Brick1: servera:/bricks/brick-a2/brick
    Brick2: serverb:/bricks/brick-b2/brick
    Brick3: serverc:/bricks/brick-c2/brick
    Brick4: serverd:/bricks/brick-d2/brick
    Options Reconfigured:
    performance.readdir-ahead: on

    b. 确定可以移除哪两个 brick。

    由于这是 2 x 2 分布式复制集，需要一次性移除一个完整的复制子卷。这表示 brick-a2 和 brick-b2，或者 brick-c2 和 brick-d2。 

2. 从 shrinkme 卷移除 brick-c2 和 brick-d2。

    a. 启动移除操作。

    [root@servera ~]# gluster volume remove-brick shrinkme
    > serverc:/bricks/brick-c2/brick \
    > serverd:/bricks/brick-d2/brick start
    volume remove-brick start: success
    ID: b3ed6b95-dbf4-4eb1-967b-80d1dc92353d

    b. 监控移除操作；重复查看直至所有被移除的 brick 都报告 completed。

    [root@servera ~]# gluster volume remove-brick shrinkme
    > serverc:/bricks/brick-c2/brick \
    > serverd:/bricks/brick-d2/brick status
                                        Node Rebalanced-files          size       scanned      failures       skipped               status   run time in secs
                                   ---------      -----------   -----------   -----------   -----------   -----------         ------------     --------------
                     serverc.lab.example.com               51         2.0KB            51             0             0            completed               7.00
                     serverd.lab.example.com                0        0Bytes             0             0             0            completed               0.00

    c. 将更改提交至卷。

    [root@servera ~]# gluster volume remove-brick shrinkme
    > serverc:/bricks/brick-c2/brick \
    > serverd:/bricks/brick-d2/brick commit
    Removing brick(s) can result in data loss. Do you want to Continue? (y/n) y
    volume remove-brick commit: success
    Check the removed bricks to ensure all files are migrated.
    If files with data are found on the brick path, copy them via a gluster mount
    point before re-purposing the removed brick.


### 实验：扩展卷

受信存储池具有一个名为 important 的分布式卷，它由两个 brick 组成。最近的一次审计发现，此卷上的文件对业务非常重要，因此您已被要求为此卷增加一些弹性。

为完成此任务，您已获得了两个新 brick：serverc:/bricks/brick-c3/brick 和 serverd:/bricks/brick-d3/brick。这些 brick 应当添加到 important 卷，并且 important 卷的副本计数应当在此过程中设置为 2。 

1. 按照所给的要求，扩展 important 卷。

    a. 检查 important 卷的当前布局。

    [root@servera ~]# gluster volume info important

    Volume Name: important
    Type: Distributed
    Volume ID: b696084d-32ab-4e73-9e71-080303ffbbe0
    Status: Started
    Number of Bricks: 2
    Transport-type: tcp
    Bricks:
    Brick1: servera:/bricks/brick-a3/brick
    Brick2: serverc:/bricks/brick-c3/brick
    Options Reconfigured:
    performance.readdir-ahead: on

    b. 将两个新 brick（serverc:/bricks/brick-c3/brick 和 serverd:/bricks/brick-d3/brick）添加到 important 卷，并将副本计数设置为 2。

    [root@servera ~]# gluster volume add-brick important replica 2 \
    > serverc:/bricks/brick-c3/brick \
    > serverd:/bricks/brick-d3/brick
    volume add-brick: success

    c. 对 important 卷启动重新平衡操作，在各 brick 间重新分布文件。

    [root@servera ~]# gluster volume rebalance important start
    volume rebalance: important: success: Rebalance on important has been started successfully. Use rebalance status command to check status of the rebalance process.
    ID: 32ca380d-a5d2-436a-99ba-b5672a233814

    d. 监控重新平衡操作的状态，直至完成。

    [root@servera ~]# gluster volume rebalance important status
                                        Node Rebalanced-files          size       scanned      failures       skipped               status   run time in secs
                                   ---------      -----------   -----------   -----------   -----------   -----------         ------------     --------------
                                   localhost                0        0Bytes            49             0             0            completed               1.00
                     serverb.lab.example.com                0        0Bytes            51             0             0            completed               1.00
                     serverc.lab.example.com                0        0Bytes             0             0             0            completed               0.00
                     serverd.lab.example.com                0        0Bytes             0             0             0            completed               0.00
    volume rebalance: important: success


### 总结

在本章中，您已学会如何：

*    使用 gluster volume add-brick 命令在线添加额外 brick 到现有的卷。
*    在使用该命令扩展后，重新平衡卷。gluster volume rebalance <VOLUME> start|status
*    使用 cluster.rebal-throttle 选项，对重新平衡操作进行调整。
*    使用 gluster volume remove-brick <VOLUME> <BRICK>... start 命令启动 brick 移除过程。
*    使用 gluster volume remove-brick <VOLUME> <BRICK>... status 命令监控 brick 移除过程。
*    使用 gluster volume remove-brick <VOLUME> <BRICK>... commit 命令结束 brick 移除过程。
*    在 brick 移除过程后重新平衡。 
