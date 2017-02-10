---
layout: post
title:  "Creating and Maintaining RHEV Storage(318-5)"
categories: Linux
tags: RHCA 318
---

### 创建存储域

###### 存储域

存储域是磁盘映像和元数据的集中访问存储库；它们保留虚拟机数据、元数据、ISO 映像、快照以及其他数据，可供数据重心内的所有主机访问。设置存储是新建数据中心的先决条件：除非已连接并激活存储域，否则无法初始化数据中心。要添加新存储域（并非安装期间添加的 ISO 域），管理员必须至少在基础架构中具有一个标记为运行状态的主机。

可以创建三种类型的存储域：

*    数据存储域：存储所有虚拟机的硬盘映像。磁盘映像可能包含已安装的操作系统；或者由虚拟机存储或生成的数据。
*    导出存储域：为在数据中心之间传输的硬盘映像和虚拟机模板提供过渡存储。此外，导出存储域中还存储虚拟机的备份副本。
*    ISO 存储域：存储 ISO 文件，也称为映像。ISO 文件是物理 CD 或 DVD 的表现形式。

RHEV 可以使用各种类型的存储，这包括：

*    网络文件系统 (NFS)。
*    其他兼容 POSIX 的文件系统。
*    Gluster FS。
*    因特网小型计算机系统接口 (iSCSI)。
*    直接连接到虚拟主机的本地存储。
*    光纤通道协议 (FCP)。

配置 NFS、iSCSI 或光纤通道存储后，可通过 RHEV-M 界面将其添加至 RHEV。按照先前所述，必须有至少一个配置的主机用来添加存储域。

###### 存储池管理器 (SPM)

RHEV 使用元数据来描述存储域的内部结构，并且主机以只读模式访问存储域元数据，同时仅有一个写程序。可以对数据域的结构进行更改的主机称为存储池管理器 (SPM)。SPM 协调数据中心内的所有元数据更改。所有其他主机只能读取存储域结构元数据。每个主机可以被授予 SPM 优先级，并且可以手动选择为 SPM；否则，这将由 RHEV 选择。 

###### 虚拟磁盘

有两种类型的虚拟磁盘：稀疏和预分配，各自的工作方式不同。存储域的可用格式为 raw 或 qcow2。

*    预分配或稀疏：预分配虚拟机具有保留的存储（大小与虚拟磁盘自身相同）。这将提高性能，因为在运行时期间不需要存储分配。在 SAN（如 iSCSI 或 FCP）上，通过创建一个大小与虚拟磁盘相同的块设备来实现此操作。在 NFS 上，这通过填充零大小的备份文件并假定备份存储不是 qcow2 来实现，并且不会对零进行去重。

    对于稀疏格式，不会保留虚拟磁盘备份存储，并且在运行时期间根据需要分配。这允许存储过量使用，并假定大部分磁盘没有完全利用且可以更好地利用存储容量。这需要备份存储监控写请求，并且可能导致某些性能问题。在 NFS 备用存储上，可以使用文件来实现。在 SAN 设备上，可以通过创建比虚拟磁盘的定义大小更小的块设备并与系统管理程序进行通信来监控必要分配，从而实现此目的。这不需要底层存储设备的支持。

*    原始：对于原始虚拟磁盘，备份存储设备（文件/块设备）按原样显示给虚拟机，两者之间没有额外层级。这提高了性能，但具有一些限制。对于存储类型和格式的可能组合，请考虑下表。 

|存储 	|格式 	|类型 	|描述
|NFS 	|原始 	|预分配 	|此文件的初始大小是针对虚拟磁盘定义的大小，并且没有格式设置。|
|NFS 	|原始 	|稀疏 	|此文件的初始大小接近于零，并且没有格式设置。|
|NFS 	|Qcow2 	|预分配 	|此文件的初始大小是针对虚拟磁盘定义的大小，并且具有 qcow2 格式设置。|
|NFS 	|Qcow2 	|稀疏 	|此文件的初始大小接近于零，并且具有 qcow2 格式设置。|
|SAN 	|原始 	|预分配 	|此块设备的初始大小是针对虚拟磁盘定义的大小，并且没有格式设置。|
|SAN 	|Qcow2 	|预分配 	|此块设备的初始大小是针对虚拟磁盘定义的大小，并且具有 qcow2 格式设置。没有用处，但可能。|
|SAN 	|Qcow2 	|稀疏 	|此类块设备的初始大小小于针对 VDisk 定义的大小（目前是 1 GB），并且具有 qcow2 格式设置，其空间根据需要分配（目前是以 1GB 为增量）。|

###### 准备 NFS 存储

对于 NFS ISO 域和任何 NFS 数据域而言，尝试将其添加作为 RHEV-M 中的存储域之前，管理员必须在导出相应的文件系统时确保其拥有正确的文件所有权。

> 存储服务器不应是 RHEV 节点中的一个。理想情况下，该服务器应该是高度可靠的，因为它是自己所附属群的群集的一个单点故障。

在存储服务器上，管理员需要创建或确认将导出的以用作存储域的文件系统。如果需要，管理员可以在存储服务器上更新文件 /etc/exports 以确保文件系统以读写方式导出到所有 RHEV 主机。例如，如果 /exports/iso0 将导出到全部位于 172.25.0.0/24 网络上的节点，- /exports/iso0  172.25.0.0/255.255.255.0(rw)

管理员还需要确保所需的守护程序（取决于 NFS 版本）正在运行，比如 rpcbind、nfs 和 nfslock。需要运行命令 exportfs -r 以激活更改。

关于正在导出的目录，用户 vdsm 和群集 kvm 必须看起来可拥有且是可写的；该用户和群集可能不存在于存储服务器（它们作为 rhevm 软件包脚本创建）上。它们具有 UID 36 和 GID 36；可以通过运行以下命令来配置权限：
  
```
# chown 36:36 /exports/iso0
```
    
为确保新创建的目录具有 kvm 组所有权，管理员需要设置用户 ID：

```
# chmod g+s /exports/iso0
```

> engine-setup 命令无法创建 ISO 存储域

###### 准备 iSCSI 存储

RHEV 可在两个级别支持 iSCSI 存储：

*    VG 级别：卷组级别将一个预定的逻辑单元编号 (LUN) 与分配至存储域的卷组相连接。卷组无法在存储域之间共享。
*    LUN 级别：LUN 级别确保管理员为纯粹与域分配 LUN 组。已连接到一个存储域的 LUN 不能再连接到其他存储域。

> 对于有关红帽企业 Linux 上 iSCSI 的设置和配置的信息，请参阅包括在 targetcli 软件包中的存储管理指南和文档文件。 

###### 演示：创建新的 iSCSI 数据存储域

1. 使用 rhevadmin 用户名和 redhat 密码在域 example.com 中登录 RHEV 管理门户。
2. 导航至 RHEV-M 管理门户中的存储 选项卡。
3. 单击新域按钮，显示新域对话框。添加数据存储域 - iSCSI 

*    名称: 域的唯一名称，最多 40 个字符。例如，data0
*    数据中心： 该域将会注册到的数据中心的名称。例如，dcenter0
*    域功能/存储类型： 域功能将是数据、ISO 或者加上一个 NFS、iSCSI、FCP 或本地存储类型的导出，这种选择是用于数据中心的指定存储类型的结果。例如，Data/iSCSI
*    使用主机： 使用要求的活动主机来执行与存储的通信。例如，servera.podX.example.com

4. 利用 iSCSI，首先发现节点中的可用目标，以便在填写 IP 地址：和端口：字段的情况下，单击发现目标按钮。

iSCSI 设置

*    发现目标 - 地址： iSCSI 目标主机的 IP 地址。例如，172.25.X.15
*    发现目标 - 端口： iSCSI 目标主机监听的端口通常为 3260 个。例如，3260
*    目标名称： 运行 iSCSI 发现之后，将提供一个可用目标的列表。例如，iqn.2011-10.com.example.rhevm.pod0:iscsi
*    LUN ID： 登录目标之后，将提供一个可用 LUN 的列表。例如，Will vary

5. 单击新近可用的登录按钮，添加该目标。
6. 最后，以单击目标附近新近可用的 +（“加号”）的方式扩展 LUN 列表，并检查可用的 LUN 附近的框。

添加数据存储域 - 所选 iSCSI

7. 如果这是第一个数据存储域，它将作为主项来用且自动激活。正常情况下，存储域应出现在 RHEV-M 控制台中存储选项卡下面的一行，状态为“未连接”，左边是一个破碎的连接图标。只需单击连接域按钮，在特定的数据数据中心、存储选项卡内将其激活。

等待数据存储域变为激活状态。在第一个数据域变为活动状态之前，无法创建另一个存储域。 


###### ISO 库

ISO 域存储用于虚拟机安装或救援的 ISO 映像（逻辑 CD-ROM 和 DVD-ROM）和 VFD（虚拟软驱映像）。ISO 域可在不同的数据中心之间共享。ISO 存储域只能基于 NFS。 


###### 演示：连接和填充新的 ISO 存储域

1. 配置 NFS 服务器，在本地网通过写入访问共享创建的 ISO 目录。整个 echo 命令应位于一行。请小心使用一对 > “大于”号附加到文件，或使用编辑器在带引号的文本内添加数据。

```
# echo "/exports/rhevisos   172.25.0.0/255.255.0.0(rw,sync,no_root_squash)" >> /etc/exports
# exportfs -r
```

2. 使用 rhevadmin 用户名和 redhat 密码在域 example.com 中登录 RHEV 管理门户。 
3. 导航至存储选项卡。
4. 单击现有 iso0 存储域并选择下方选项卡数据中心。
5. 单击下方窗格中的连接，在连接到数据中心对话框中选中 dcenter0 数据中心，然后单击确定。
6. ISO Uploader 是一个 Linux 命令行工具，能够自动将文件放置在正确位置，且获得正确的权限。ISO Uploader 在上传较大的 ISO 时相对较慢，因为在通过 NFS 复制映像文件（它加载 NFS 共享，并将此文件复制进该共享）。默认情况下，ISO 上传程序使用内部域中的 admin 用户。 列出可用的 ISO 域（尽管仅创建了一个要使用的域）：

```
# engine-iso-uploader list
Please provide the REST API password for the admin@internal RHEV-M user (CTRL+D to abort): redhat 
ISO Storage Domain Name   | Datacenter                | ISO Domain Status 
iso0                      | dcenter0                  | active
```

> /etc/rhevm/isouploader.conf 配置文件用以包括用户名、密码或其他数据。可在 engine-iso-uploader(8) 手册页面找到更多信息。

7. 通过 http://classroom.example.com/materials 将 ISO 从目录上传到 ISO 存储域： rhel-server-7.1-x86_64-boot.iso

1). 使用 engine-iso-uploader 上传 ISO：

```
# wget http://classroom.example.com/materials/rhel-server-7.1-x86_64-boot.iso
...
# engine-iso-uploader -i iso0 upload rhel-server-7.1-x86_64-boot.iso
... output omitted ...
```

2). 除了使用 ISO Uploader，另外一个更快的技术是直接手动上传 ISO 映像至 NFS export。使此方法具有挑战性的部分在于，每个 ISO 域由一个唯一的 128 位 UUID 标识，该 UUID 嵌入到 NFS export 上 ISO 文件所存储在的路径中。正确的 ISO export 顶层目录将会像这样：unique-UUID/images/11111111-1111-1111-1111-111111111111/。决定唯一 UUID 最容易的方法是在复制之前简单查看一下 ISO 域。例如，如果 NFS export 是 /exports/rhevisos/，以下命令表明独特的 UUID（对于其他安装，可能有所不同）和映像目录的路径：

```
# ls /exports/rhevisos/
56a00180-8301-4673-a1b5-b31c25686de6
# ls /exports/rhevisos/56a00180-8301-4673-a1b5-b31c25686de6/images/11111111-1111-1111-1111-111111111111/
rhel-server-7.1-x86_64-boot.iso  virtio-win_amd64.vfd  virtio-win_x86.vfd
rhev-tools-setup.iso             virtio-win.iso
```

3). 将文件直接上传到该目录。进入后，文件必须由用户 vdsm 和组 kvm 拥有 (UID 36/GID 36)。复制完成后，文件应自动显示在 RHEV-M 界面中。

```
# cd /exports/rhevisos/UUID/images/11111111-1111-1111-1111-111111111111/
# wget http://content.example.com/rhel7.1/x86_64/isos/rhel-server-7.1-x86_64-dvd.iso
--2015-04-08 13:39:33--  http://content.example.com/rhel7.1/x86_64/isos/rhel-server-7.1-x86_64-dvd.iso
Resolving content.example.com... 172.25.0.254
Connecting to content.example.com|172.25.0.254|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3890216960 (3.6G) [application/octet-stream]
Saving to: “rhel-server-7.1-x86_64-dvd.iso”

100%[====================================>] 3,890,216,960 64.5M/s   in 53s     

2015-04-08 13:40:26 (64.6 MB/s) - “rhel-server-7.1-x86_64-dvd.iso” saved [3890216960/3890216960]
```

4). 检查内容，然后使用 chown 来更新文件所有权：

```
# ls -l
... output omitted ...
# chown 36:36 *.iso
```

###### 演示：创建新的导出存储域

1. 配置 NFS 服务器，在本地网通过写入访问共享创建的导出目录。整个 echo 命令应位于一行。请小心使用一对 > “大于”号附加到文件，或使用编辑器在带引号的文本内添加数据。

```
[root@rhevm ~]# mkdir /exports/exp
[root@rhevm ~]# chown 36:36 /exports/exp
[root@rhevm ~]# echo "/exports/exp   172.25.0.0/255.255.0.0(rw,sync,no_root_squash)" >> /etc/exports
[root@rhevm ~]# exportfs -r
```

2. 使用 rhevadmin 用户名和 redhat 密码在域 example.com 中登录 RHEV 管理门户。
3. 导航至存储选项卡。
4. 单击新域按钮，显示新域对话框。

*    名称: 域的唯一名称，最多 40 个字符。例如，exp0
*    数据中心： 该域将会在其中注册的数据中心的名称。例如，dcenter0
*    域功能/存储类型： 域功能将是数据、ISO 或者加上一个 NFS、iSCSI、FCP 或本地存储类型的导出，这种选择是用于数据中心的指定存储类型的结果。例如，Export/NFS
*    使用主机： 使用要求的活动主机来执行与存储的通信。例如，servera.podX.example.com
*    导出路径： 对于 NFS 存储，输入 NFS 服务器（IP 地址或可解析的主机名）和所连接的导出路径。例如，rhevm.podX.example.com:/exports/exp


###### 练习：创建数据存储域

在本实验中，您将连接一个新存储域。

成果：

*    您应该能够在数据中心 testdcX 中使用 NFS 建立一个名为 testdataX 的数据存储域（通过指向 rhevm.podX.example.com:/exports/testdata 的 servera）。
*    您应该能够在数据中心 testdcX 中使用 NFS 建立一个名为 testexportX 的导出存储域（通过指向 rhevm.podX.example.com:/exports/testexport 的 servera）。 

此操作需要之前练习中的下列组件：

*    rhevm 上安装的 RHEV Manager。
*    与 rhevm 连接的管理门户。
*    已安装的 RHEV-H 节点。
*    名为 testdcX 的数据中心。
*    名为 testcluX 且包含单个 RHEV-H 主机的群集。

请执行以下步骤：

1. 创建将用于 RHEV 存储域的 NFS 导出。

在 rhevm 上以 root 身份，创建名为 /exports/testdata 的目录，以及名为 /exports/testexport 的目录（所有权 UID:GID 为 36:36）。

```
        [root@rhevm ~]# mkdir /exports/testdata
        [root@rhevm ~]# mkdir /exports/testexport
        [root@rhevm ~]# chown 36:36 /exports/test*
```

2. 配置 NFS 服务器，以向局域网共享新目录并提供写入访问权限。

整个 echo 命令应位于一行。请小心使用一对 > “大于”号附加到文件，或使用编辑器在带引号的文本内添加数据。

```
        [root@rhevm ~]# echo "/exports/testdata   172.25.0.0/255.255.0.0(rw,sync,no_root_squash)" >> /etc/exports
        [root@rhevm ~]# echo "/exports/testexport   172.25.0.0/255.255.0.0(rw,sync,no_root_squash)" >> /etc/exports
        [root@rhevm ~]# exportfs -r
```

3. 在您的数据中心内创建数据存储域。

使用 rhevadmin 用户名和 redhat 密码在域 example.com 中登录 RHEV 管理门户。导航到系统，然后单击存储选项卡。创建带有以下值的新数据存储域：

*    名称：testdataX（其中 X 是您的 pod 号）。
*    数据中心：testdcX（其中 X 为您的 pod 号）。
*    域功能/存储类型：Data / NFS
*    使用主机：servera.podX.example.com
*    导出路径：rhevm.podX.example.com:/exports/testdata
            
单击新域按钮。“新域”对话框显示。使用上述值填写相关信息。单击确定。耐心等待存储过渡至“活动”，因为这是第一个存储域和“主存储域”。

4. 在您的数据中心内创建导出存储域。

仍在系统中，单击存储选项卡。使用以下值创建新的导出存储域：

*    名称：testexportX（其中 X 是您的 pod 号）。
*    数据中心：testdcX（其中 X 为您的 pod 号）。
*    域功能/存储类型：Export / NFS
*    使用主机：servera.podX.example.com
*    导出路径：rhevm.podX.example.com:/exports/testexport

单击新域按钮。“新域”对话框显示。使用上述值填写相关信息。单击确定。耐心等待存储变为“活动”。

5. 确保存储域对群集可用。

单击数据中心选项卡。选择数据中心 testdcX。单击下方存储选项卡并确认 testdataX 以及 testexportX 显示在列表中，且状态为活动



### 将存储与 OpenStack 集成

###### 集成 Glance 映像服务

在红帽企业虚拟化 3.5 中，管理员可以连接外部 OpenStack 环境作为外部提供者。OpenStack Glance 映像服务作为 OpenStack 映像提供者，并且可以配置为外部存储池。Glance 与 RHEV 接口后，便可以检索到活动的 Glance 映像并且可以导入到 RHEV 数据中心。导入时，映像便可用，并且可以通过导入的映像来实例化新虚拟机，或者转换为模板。红帽企业虚拟化引擎还能够将映像导入到 Glance，使管理员可以使用 Glance 的映像管理功能。最新版本现已支持通过 Keystone 服务进行身份验证和令牌管理。

以下过程说明如何添加 OpenStack 映像外部提供者：

1. 在树形窗格中，选择外部提供者条目。
2. 单击添加按钮，以打开添加提供者窗口。
3. 输入名称和描述，然后选择 OpenStack Image 作为提供者类型。
4. 在提供者 URL 文本字段中，输入安装 Glance 实例的机器的 URL 或全限定域名。
5. （可选）选择需要身份验证复选框，然后输入 Glance 的凭据。管理员必须使用 Keystone 中注册的 Glance 用户的用户名和密码。

###### Glance 当前限制

此技术预览仍有一些约束和限制，包括：

*    仅支持 raw 和 qcow2 映像。
*    不支持通过 SSL 的加密通信。
*    目前无法将包含多个卷的映像导出到 Glance。
*    不支持实时导出；在导出之前，需要关闭使用磁盘的虚拟机。
*    在文件域中，raw格式不支持稀疏映像；导入的映像始终需要预分配。
*    不能暂停导入过程又在之后将其恢复。
*    无法删除 Glance 中存在的映像。

###### 集成存储卷服务

目前作为技术预览提供，管理员可以将 RHEV 与 OpenStack Volume 项目（代码名称 Cinder）接口。Cinder 作为 OpenStack Volume 提供者类型下面的外部提供者来提供。集成后，管理员可以检索 Cinder 卷，并可在 Cinder 域中创建新的虚拟磁盘。创建的磁盘可以与虚拟机连接和分离，并且支持本机 RHEV 功能，比如将磁盘标记为可引导或可共享。此最新版本只能管理 Ceph 存储调配的卷。

###### 未来工作

目前的实现正在积极开发中。其中的一些新功能将包含：

*    在创建后更改卷类型。
*    将卷从 Cinder 上传到 Glance。
*    针对卷类型运行 CRUD 操作。
*    管理 Cinder 配额。
*    导入和导出虚拟机和模板。
*    创建实时快照。
*    运行实时存储迁移。
*    将 Cinder 数据与 RHEV 引擎数据库同步。
*    监控 Cinder 存储域。
*    支持多个 Cinder 后端，如 LVM。


### 实验：创建和维护 RHEV 存储

在本实验中，您将在现有数据中心中创建存储域 iscsidataX。您还将连接和填充在安装期间创建的 ISO 域。成果. 您应该能够在数据中心 testdcX 中建立名为 iscsidataX 的数据存储域（使用 iSCSI 作为存储类型）并使用 ISO 域。

根据以下规格来配置 iSCSI 存储域：

*    名称：iscsidataX（其中 X 是您的 pod 编号）。
*    数据中心：testdcX（其中 X 为您的 pod 号）。
*    域功能/存储类型：Data / iSCSI
*    使用主机：servera.podX.example.com
*    目标地址：rhevm.podX.example.com
*    用户身份验证：Disabled

1. 从 workstation 中登录 RHEV 管理器。打开 Firefox，然后转至 rhevm.podX.example.com。在域 example.com 中使用 rhevadmin 并使用 redhat 作为密码。
2. 在数据中心内创建 iscsidataX 存储域。

1). 使用以下值来创建数据存储域：

*    名称：iscsidataX（其中 X 是您的 pod 编号）。
*    数据中心：testdcX（其中 X 为您的 pod 号）。
*    域功能/存储类型：Data / iSCSI
*    使用主机：servera.podX.example.com
*    目标地址：rhevm.podX.example.com
*    用户身份验证：Disabled

单击新域按钮。“新域”对话框显示。使用上述值填写相关信息。

2). 单击发现，以发现现有 iSCSI 目标。此时将显示一个新条目：iqn.2014-10.com.example.rhevm.podX:iscsi, 单击该条目同一行中的箭头以进行登录。

3). 登录后，单击 LUN > 目标选项卡以列出所有可用的目标。检查唯一可用的条目。

4). 单击确定以添加 iSCSI 存储。耐心等待存储变为“活动”。

3. 确保存储域对群集可用。

1). 单击数据中心选项卡。
2). 选择数据中心 testdcX。
3). 单击下方的存储选项卡并确认 iscsidataX 显示在列表中，且状态为活动而类型为数据（主）。

4. 更正 isoX 的 NFS 访问权限，以便可以从课堂环境中访问。 

调整 /etc/exports 并更新正在运行的 NFS 服务器。

```
# echo "/exports/rhevisos   172.25.0.0/255.255.0.0(rw,sync,no_root_squash)" >> /etc/exports
# exportfs -r
```

5. 将 isoX 域连接到数据中心。 

1). 单击域 isoX。在下方窗格中，选择数据中心选项卡。
2). 单击连接链接，以将域连接到数据中心。在显示的窗口中，检查数据中心 testdcX。单击确定以保存更改。

6. 将 http://classroom.example.com/materials/ 中提供的 RHEL 7.1 引导 iso 添加到 ISO 域。 

1). 使用 wget 检索 ISO： - wget http://classroom.example.com/materials/rhel-server-7.1-x86_64-boot.iso

2). 在 isoX 数据存储中上传该映像：

```
# engine-iso-uploader -i isoX upload rhel-server-7.1-x86_64-boot.iso
Please provide the REST API password for the admin@internal oVirt Engine user (CTRL+D to abort): redhat
Uploading, please wait...
INFO: Start uploading rhel-server-7.1-x86_64-boot.iso
...
INFO: rhel-server-7.1-x86_64-boot.iso uploaded successfully
```






