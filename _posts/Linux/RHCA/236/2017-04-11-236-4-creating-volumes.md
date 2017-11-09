---
layout: post
title:  "Creating Volumes(236-4)"
categories: Linux
tags: RHCA 236
---

### 创建不同卷类型

###### 卷类型

红帽 Gluster 存储可通过三种主要方式将 brick 组合为卷：分布式、复制式和离散式。这些类型还可组合，形成分布式复制卷和分布式离散卷。

###### 分布式卷

分布式卷是创建过程中未指定任何选项时所创建的默认卷类型。在分布式卷中，任何文件始终存在于一个 brick 上。文件最终驻留的 brick 取决于弹性哈希算法。

这种文件分布方式可以最大利用可用的存储资源，但如果有一个 brick 变为不可用，将失去对该卷上存储的文件的访问权限，直到该 brick 再次可用为止。

当存储空间总量比可用性更为重要时，可以使用分布式卷。

1. 创建分布式卷

若要创建分布式卷，可以运行 gluster volume create 命令但不使用任何额外选项；例如：

```
[root@demoa ~]# gluster volume create distributedvolume <BRICKS>...
```

###### 复制式卷

在复制式卷中，brick 之间互相镜像。这意味着文件写入到某一 brick 时，也会同时写入到另一个或另一些 brick 上。在创建或扩展卷时指定 replica <number of replicas> 选项，即可创建复制式卷。

> 重要: 使用大于三的副本数目当前属于技术预览，可能不会获得全面支持。卷中 brick 的总数应当是所指定副本数的倍数。

复制式卷提供冗余能力和高可用性，但它以减小容存储容量为代价。 

1. 创建复制式卷

若要创建复制式卷，可以在运行 gluster volume create 命令时指定 replica <REPLICACOUNT> 选项；例如：

```
[root@demoa ~]# gluster volume create replicavolume replica 2 <BRICKS>...
```

> 注意: 在创建复制式卷时，指定的 brick 数量必须是请求的副本数的倍数。如果指定的 brick 数超过副本数，则生成的将是分布式复制卷。本节下文中阐述这种卷类型。

###### 离散式卷

离散式卷基于纠删码 (EC)。纠删码是一种数据保护方式，其中数据分割为多个片段，使用冗余数据段进行扩展和编码，并存储在一组不同的位置上。RAID5 和 RAID6 使用类似的技术提供块级别驱动器失败防护，而离散式卷则在文件级别上提供此功能。

与复制式卷相比，离散式卷占用的存储空间比较少。例如，若要在存储 1 TiB 数据时使用一个副本，至少将占用 2 TiB 存储空间，而离散式卷只需 1.5 TiB 即可提供冗余能力。

离散式卷需要 N 个 brick，其中 N 是读取一个文件至少需要的数据 brick 数 (K) 加上允许出现失败的 brick 数 (M)。这便形成以下公式：N = K + M。

M 有时称为冗余级别。

利用这一种纠删码时，只要 N 个 brick 中至少有 K 个可用（即允许 M 个 brick 失败），原始内容始终都能恢复。 

红帽目前支持以下布局的离散式卷：

*    6 个冗余级别 2 的 brick (4 + 2)
*    11 个冗余级别 3 的 brick (8 + 3)
*    12 个冗余级别 4 的 brick (8 + 4)

1. 创建离散式卷

在创建离散式卷时，必须指定两个选项，即 disperse-data K 和 redundancy M。指定的 brick 数必须是 (K + M) 的倍数，例如：

```
[root@demoa ~]# gluster volume create dispersevol disperse-data 4 redundancy 2 brick1 brick2 brick3 brick4 brick5 brick6
```

###### 组合卷类型

如有需要，可以组合卷类型。这样，可以创建 distributed-replicated（分布式复制）和 distributed-dispersed（分布式离散）这两种卷类型。在这些类型中，形成若干个复制式或离散式集合，文件在这些集合之间分布。

当创建组合卷时，要确保指定的 brick 数量是副本数的倍数，或者 dispersed-data + redundancy 数的倍数。

######### 分布式复制卷

在 distributed-replicated（分布式复制）卷中，形成若干个复制式集合，集合数等于卷中 brick 数除以副本数。然后，文件在这些复制式集合之间分布。 

> 注意: 分布式复制卷有时称为 X × Y 卷。这些情况下，X 是分布数，Y 则是副本数。

###### 分布式离散卷

在 distributed-dispersed（分布式离散）卷中，形成若干个离散式集合，集合数等于卷中 brick 数除以 (disperse-data + redundancy) 数。然后，文件在这些离散式集合之间分布。 

> 注意: 分布式离散卷有时称为 X × (K + M) 卷。这些情况下，X 是分布数，K 是 disperse-data 数，而 M 则是 redundancy 数。 


### 引导式练习：创建卷

1. 创建一个复制卷，副本数为 2，名为 replvol。此卷应当使用两个 brick：servera:/bricks/brick-a1/brick 和 serverb:/bricks/brick-b1/brick。

将此卷临时挂载到 workstation 上的 /mnt/replvol。

    a. 创建一个复制卷，副本数为 2，名为 replvol。此卷应当使用两个 brick：servera:/bricks/brick-a1/brick 和 serverb:/bricks/brick-b1/brick。

    [root@servera ~]# gluster volume create replvol replica 2 \
    > servera:/bricks/brick-a1/brick \
    > serverb:/bricks/brick-b1/brick
    volume create: replvol: success: please start the volume to access data

    b. 启动卷。

    [root@servera ~]# gluster volume start replvol
    volume start: replvol: success

    c. 检查 replvol 卷的布局。

    [root@servera ~]# gluster volume info replvol

    Volume Name: replvol
    Type: Replicate
    Volume ID: dc6abb99-388e-4aba-b983-434df80620b1
    Status: Started
    Number of Bricks: 1 x 2 = 2
    Transport-type: tcp
    Bricks:
    Brick1: servera:/bricks/brick-a1/brick
    Brick2: serverb:/bricks/brick-b1/brick
    Options Reconfigured:
    performance.readdir-ahead: on

    d. 在 workstation 上创建挂载点 /mnt/replvol。

    [root@workstation ~]# mkdir /mnt/replvol

    e. 确保 workstation 上已安装了 glusterfs-fuse 软件包。

    [root@workstation ~]# yum -y install glusterfs-fuse

    f. 将 replvol 卷临时挂载到 workstation 上的 /mnt/replvol。

    [root@workstation ~]# mount -t glusterfs servera:/replvol /mnt/replvol

2. 使用 6 个 brick 和冗余级别 2 创建一个离散式卷，名为 dispersevol。此卷应当使用下列 brick：

*    serverc:/bricks/brick-c1/brick
*    serverd:/bricks/brick-d1/brick
*    servera:/bricks/brick-a2/brick
*    serverb:/bricks/brick-b2/brick
*    serverc:/bricks/brick-c2/brick
*    serverd:/bricks/brick-d2/brick 

3. 将此卷挂载到 workstation 上的 /mnt/dispersevol。

    a. 使用上述 brick，创建离散式卷 dispersevol。需要使用 force 标志，因为同一离散式卷的多个 brick 将驻留在同一服务器上。在教室/测试环境之外，这应当要避免。

    [root@servera ~]# gluster volume create dispersevol disperse-data 4 redundancy 2 \
    > serverc:/bricks/brick-c1/brick \
    > serverd:/bricks/brick-d1/brick \
    > servera:/bricks/brick-a2/brick \
    > serverb:/bricks/brick-b2/brick \
    > serverc:/bricks/brick-c2/brick \
    > serverd:/bricks/brick-d2/brick \
    > force
    volume create: dispersevol: success: please start the volume to access data

    b. 启动 dispersevol 卷。

    [root@servera ~]# gluster volume start dispersevol
    volume start: dispersevol: success

    c. 检查 dispersevol 卷的布局。

    [root@servera ~]# gluster volume info dispersevol

    Volume Name: dispersevol
    Type: Disperse
    Volume ID: 9cc62c07-ce92-49c6-995d-686474fbaa8f
    Status: Started
    Number of Bricks: 1 x (4 + 2) = 6
    Transport-type: tcp
    Bricks:
    Brick1: serverc:/bricks/brick-c1/brick
    Brick2: serverd:/bricks/brick-d1/brick
    Brick3: servera:/bricks/brick-a2/brick
    Brick4: serverb:/bricks/brick-b2/brick
    Brick5: serverc:/bricks/brick-c2/brick
    Brick6: serverd:/bricks/brick-d2/brick
    Options Reconfigured:
    performance.readdir-ahead: on

    d. 在 workstation 上，将 dispersevol 卷临时挂载到 /mnt/dispersevol。

    [root@workstation ~]# mkdir /mnt/dispersevol
    [root@workstation ~]# mount -t glusterfs servera:/dispersevol /mnt/dispersevol

3. 在 workstation 上，复制一些文件到这两个卷上，然后检查组成卷的 brick 的内容。

    a. 在 workstation，将 /boot 的内容递归复制到两个卷上。

    [root@workstation ~]# cp -R /boot /mnt/replvol/
    [root@workstation ~]# cp -R /boot /mnt/dispersevol/

    b. 在 servera 上个，检查 /bricks/brick-a1/brick/boot 和/bricks/brick-a2/brick/boot 的内容。您注意到了什么？

    brick-a2 上的文件似乎比较小，大概是该文件实际大小的四分之一。 

### 实验：创建卷

要求在红帽 Gluster 存储群集上创建两个新卷。下表中概述了这两个卷的详细信息。所有 brick 都已为您创建好。应当启动这两个卷，使它们能被客户端访问。

在下表中，brick-a3 指 servera:/bricks/brick-a3/brick，brick-b6 指 serverb:/bricks/brick-b6/brick，以此类推。

*       名称 	            类型 	                Brick
*    distreplvol 	2 x 2（分布式复制卷）           brick-a3、brick-b3、brick-c3 和 brick-d3。
*    distdispvol 	2 x (4 + 2)（分布式离散卷）     brick-a4、brick-b4、brick-c4、brick-d4、brick-a5、brick-b5、brick-c5、brick-d5、brick-a6、brick-b6、brick-c6 和 brick-d6。

> 注意: 若要用于生产环境，distdispvol 布局需要至少 6 个服务器。在教室环境中，我们仅使用四个系统，所以可能需要在创建过程中使用 force 选项。 

1. 如上所述，创建和启动 distreplvol 卷。

    a. 使用副本数 2 和指定的 brick，创建 distreplvol 卷。

    [root@servera ~]# gluster volume create distreplvol replica 2 \
    > servera:/bricks/brick-a3/brick \
    > serverb:/bricks/brick-b3/brick \
    > serverc:/bricks/brick-c3/brick \
    > serverd:/bricks/brick-d3/brick
    volume create: distreplvol: success: please start the volume to access data

    b. 启动 distreplvol 卷。

    [root@servera ~]# gluster volume start distreplvol
    volume start: distreplvol: success

    c. 检查 distreplvol 卷的布局。

    [root@servera ~]# gluster volume info distreplvol

    Volume Name: distreplvol
    Type: Distributed-Replicate
    Volume ID: 291fefbe-210e-41b3-ac93-03ce8dfb7293
    Status: Started
    Number of Bricks: 2 x 2 = 4
    Transport-type: tcp
    Bricks:
    Brick1: servera:/bricks/brick-a3/brick
    Brick2: serverb:/bricks/brick-b3/brick
    Brick3: serverc:/bricks/brick-c3/brick
    Brick4: serverd:/bricks/brick-d3/brick
    Options Reconfigured:
    performance.readdir-ahead: on

2. 如上所述，创建和启动 distdispvol 卷。

    a. 由于 distdispvol 卷使用 12 个 brick，因此请创建包含所有 brick 的文本文件，以避免在 create 命令中出现拼写错误和重复输入。

    [root@servera ~]# for BRICKNUM in {4..6}; do
    >   for NODE in {a..d}; do
    >       echo server${NODE}:/bricks/brick-${NODE}${BRICKNUM}/brick
    >   done
    > done > /tmp/distdispbricks

    b. 创建 distdispvol 卷。记住使用 force 选项，因为我们在教室中使用了不受支持的设置（只有四个节点，而不是六个）。

    ```
    [root@servera ~]# gluster volume create distdispvol 
    > disperse-data 4 redundancy 2 $(</tmp/distdispbricks) force
    volume create: distdispvol: success: please start the volume to access data
    ```
    
    c. 启动 distdispvol 卷。

    [root@servera ~]# gluster volume start distdispvol
    volume start: distdispvol: success

    d. 检查 distdispvol 卷的布局。

    [root@servera ~]# gluster volume info distdispvol
    Volume Name: distdispvol
    Type: Distributed-Disperse
    Volume ID: 72417e4f-5c67-4251-8d1e-9b5501086503
    Status: Started
    Number of Bricks: 2 x (4 + 2) = 12
    Transport-type: tcp
    Bricks:
    Brick1: servera:/bricks/brick-a4/brick
    Brick2: serverb:/bricks/brick-b4/brick
    Brick3: serverc:/bricks/brick-c4/brick
    Brick4: serverd:/bricks/brick-d4/brick
    Brick5: servera:/bricks/brick-a5/brick
    Brick6: serverb:/bricks/brick-b5/brick
    Brick7: serverc:/bricks/brick-c5/brick
    Brick8: serverd:/bricks/brick-d5/brick
    Brick9: servera:/bricks/brick-a6/brick
    Brick10: serverb:/bricks/brick-b6/brick
    Brick11: serverc:/bricks/brick-c6/brick
    Brick12: serverd:/bricks/brick-d6/brick
    Options Reconfigured:
    performance.readdir-ahead: on
    
### 总结

在本章中，您学到了：

*    若不指定其他选项，则创建分布式卷。
*    复制卷中的 brick 数必须正好等于副本计数的倍数。
*    离散卷使用的 brick 数量等于 disperse-data 加上 redundancy。支持的布局为 4+2、8+3 和 8+4。
*    如果使用的 brick 数量是复制卷或离散卷所需数量的倍数，系统将创建分布式复制卷或分布式离散卷。 

