---
layout: post
title:  "管理类别(110-4)"
categories: Linux
tags: RHCA 110
---


### 管理类别

###### 类别

类别是资源模板，决定实例的 RAM 和磁盘的大小，以及核心数量（即容量）。类别也可以指定辅助临时存储、交换磁盘、用于限制使用的元数据，或者特殊项目访问权限。OpenStack 默认安装中提供五种类别。不过，在一些用例中可能需要创建和管理专门的类别，如更改默认内存和容量来满足底层硬件需求，或者更改或添加元数据以强制实例使用特定的 I/O 速率。

默认类别

*    名称 	VCPU 	RAM 	根磁盘大小
*    m1.tiny 	1 	512 MB 	1 GB
*    m1.small 	1 	2048 MB 	20 GB
*    m1.medium 	2 	4096 MB 	40 GB
*    m1.large 	4 	8192 MB 	80 GB
*    m1.xlarge 	8 	16384 MB 	160 GB

###### 创建类别

创建类别由 admin 用户执行，但也可通过重新定义访问控制来委派给其他用户。要允许所有用户配置类别，可以在 /etc/nova/policy.json 文件中指定策略： "compute_extension:flavormanage": "",

1. 以管理员用户身份，在控制面板中选择管理>系统>类别。
2. 单击创建类别，再指定参数。
3. 单击创建类别来创建该类别。

###### 创建类别所用的参数

类别允许用户定义参数，使得用户可以选择要运行的实例类型。除了提供 RAM 大小、VCPU 和磁盘相关的灵活选择外，它还允许通过元数据定义 NUMA 拓扑等自由特征；例如，允许 RAM 用于视频设备，以及设置磁盘读/写 IOPS 配额限制等。

类别参数

*    参数 	        说明
*    名称 	        类别的名称
*    ID 	        类别的 ID。默认值为 auto，可生成 UUID 值，但也可以手动指定为整数 或 UUID 值。
*    VCPU 	        虚拟 CPU 的数量
*    RAM (MB) 	    内存（以 MB 为单位）
*    根磁盘 (GB) 	临时磁盘的大小（以 GB 为单位）；若要使用原生映像大小，可指定 0。如果在启动实例时 Instance Boot Source=Boot from Volume，将忽略此参数。
*    临时磁盘 (GB) 	辅助临时磁盘大小（以 GB 为单位）
*    交换磁盘 (MB) 	交换磁盘大小（以 MB 为单位）

> 类别访问权限允许用户指定选定项目，或可以使用该类别的项目。如果不选择项目，则所有类别都具有公共访问权限。

###### 编辑类别

1. 以管理员用户身份，在控制面板中选择管理>系统>类别。
2. 在选择了类别后，单击编辑类别，再修改所需的参数。
3. 单击保存以完成编辑。

###### 删除类别

1. 以管理员用户身份，在控制面板中选择管理>系统>类别。
2. 在选择了类别后，单击删除类别。
3. 单击删除类别以确认删除。


### 验证类别功能

用户可能需要一个为项目专门调节的自定义类别。例如，用户可能要求内存为 128 GB 的类别，因此可创建一个新的自定义类别。但是，如果所有其他项目对此类别进行不必要的访问，那么云基础架构可能会非常快地达到满容量。为预防此类事件，需要对类别进行限制，只允许与必要的项目共享。在各种不同的用例中，使用某些创建的类别可能会导致云基础架构中启动实例时耗尽资源，因此在创建类别时应当要注意这些问题。

###### 使用定义资源超过项目配额的类别

每个项目在可使用的 VCPU、RAM 和根磁盘等的数量上定义有配额限值。这些限值的目的是使用户和组受到它们约束，并阻止对云基础架构的超额订阅。如果创建的类别所要求的资源超过为项目指定的配额，实例启动将失败。因此在创建类别时，应当要注意为可访问该类别的项目所指定的配额。在这样的情形中，超出项目限值的资源将在 Horizon 控制面板中用红色表示。

###### 使用定义资源超过计算主机的类别

如果底层计算主机没有实例所需的资源（基于所选类别），实例启动将失败。因此在创建类别时，应当要注意各个计算主机的底层容量。Nova 服务使用了一个调度程序，它可帮助查找相应的计算主机来启动按照所选类别需要指定资源的实例。如果 Nova 服务找不到计算主机，它将引发错误，实例启动便失败。这些错误记录在 nova-conductor.log 文件中。

```
[root@servera ~]# tail /var/log/nova/nova-conductor.log
NoValidHost: No valid host was found. There are not enough hosts available.
2015-10-09 13:06:25.552 2083 WARNING nova.scheduler.utils [req-83e4bc2e-5119-40ff-b720-5695321c2a8a 5dc3829dd7c54dca8a1da478e1944e8d 
ad2e1477597e456694f7c39c8c58e3cf- - -] [instance: 0bd71d01-3b0c-4dbb-924f-ba795d805774] Setting instance to ERROR state.
```

###### 使用定义根磁盘大小比映像根磁盘小的类别

用于创建实例的映像是实例模板，它们定义调配该实例时要使用的操作系统和软件。这些映像定义有根磁盘，用于存放操作系统文件，可以是 QCOW2、RAW 和 VMDK 等不同格式。在调配实例时，Nova 服务从 Glance 存储复制映像，将它转换为原始稀疏映像，并使它成为在 /var/lib/nova/instances/_base 提供的基础映像。根据所使用的类别，实例的根磁盘创建在 /var/lib/nova/instances/instance-id/disk 下。如果实例根磁盘大小超过映像根磁盘，它将扩大基础映像磁盘的大小，并将它用作覆盖来引导实例。如果类别根磁盘小于映像根磁盘，将导致无法启动实例，因为 Nova 服务无法收缩文件系统以容纳在较小的根磁盘中。

###### 使用根磁盘大小和 RAM 小于映像根磁盘和 RAM 最小大小的类别

在创建映像时，可以使用根磁盘和/或 RAM 最小大小来定义根磁盘和/或 RAM 的最低要求。这有助于在存在较大映像时选择符合这些要求的类别。在选择所需映像的类别时，应当要注意此要求。在 Horizon 控制面板中，不满足映像最低要求的类别将被灰显。

###### 验证临时磁盘和交换磁盘

*    临时磁盘大小（GB 为单位）在创建类别时定义。若未指定，则默认为 0 值。临时磁盘提供与实例生命周期衔接的实例本地磁盘存储。当实例终止时，临时磁盘上的所有数据都会丢失。该磁盘在格式化后连接，并自动挂载到 /mnt 下。临时磁盘不包含在任何快照中。临时磁盘位于 /var/lib/nova/instances/instance-id/disk.local。
*    交换磁盘大小（MB 为单位）在创建类别时指定。在调配实例后，将创建交换磁盘并连接到该实例。交换磁盘创建在 /var/lib/nova/instance-id/disk.swap 下。

### 总结

在本章中，您可以学到：

*    类别是资源模板，用于针对 RAM、磁盘、VCPU 和临时磁盘定义实例大小。
*    类别应当在计算主机的定义容量和项目的定义配额之内创建。
*    类别可以由管理员编辑，类别访问权限则可用于提供特定项目的访问权限。

















