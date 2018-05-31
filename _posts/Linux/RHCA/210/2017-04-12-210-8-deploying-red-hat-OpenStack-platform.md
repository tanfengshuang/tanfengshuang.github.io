---
layout: post
title:  "部署红帽 OpenStack 平台(210-8)"
categories: Linux
tags: RHCA 210
---

### 描述红帽 OpenStack 平台安装方法

###### 描述各种安装方法

Red Hat OpenStack Platform 具有多种安装方法，它们在复杂性、功能和成熟度上各有千秋。这些方法包括如下：

*    PackStack
*    Undercloud

######### PackStack

PackStack 是原始的安装程序，用于将 Red Hat OpenStack Platform 部署到 红帽企业 Linux和其他红帽支持的 Linux 变体上，如 CentOS 和 Fedora 等。这是一个命令行实用程序，利用 Puppet 模块来部署 OpenStack 或通过 SSH 连接所需的主机。该安装程序能够部署单一节点安装，以及更复杂的多节点安装。PackStack 安装程序在 Red Hat OpenStack Platform 8 中依然受到支持，也是概念验证安装的建议安装程序。

使用 PackStack 安装程序有一些优点：

*    Packstack 支持通过应答文件实现非交互式运行。这有助于在不同的主机上复制相同的配置。
*    PackStack 使用非常简单，也允许通过再次运行安装来更改现有的环境。

使用 PackStack 安装程序存在一些限制：

*    无法在裸机硬件上部署 OpenStack 平台。
*    以编程方式使用时，不支持通过 API 访问来实现自动化或自定义。
*    不支持多个控制器节点，因此无法配置 OpenStack 平台高可用性。
*    除了计算节点和控制器节点外，没有配置其他节点的能力。
*    不允许通过复杂拓扑实现网络隔离。
*    不包含用户界面。

OpenStack 平台引导主机

undercloud 是红帽用于 Red Hat OpenStack Platform 部署的全新安装、配置和监控工具集。它是部署生产就绪型 Red Hat OpenStack Platform 8 环境的建议工具集。undercloud 于 Red Hat OpenStack Platform 7 中作为默认安装程序推出。undercloud 是多年辛勤耕耘的结晶，是符合上游 OpenStack 项目要求的同类最佳部署工具。它以 TripleO 项目为基础，提供下列功能：

*    使用 OpenStack API 安装 OpenStack 环境。
*    提供用于缩放 OpenStack 环境容量的工具。undercloud 可以向当前版本的 Red Hat OpenStack Platform 环境应用更新。undercloud 可以和红帽 Ceph 存储、红帽卫星以及其他虚拟化产品集成。
*    提供中央日志记录工具，实现运作可见性。
*    利用 Pacemaker 实现所部署 OpenStack 环境的高可用性。
*    利用 Heat 模板，定义部署的拓扑和配置。

Undercloud 运用两个主要概念：undercloud 和 overcloud。undercloud 安装和配置 overcloud。


### 描述 TripleO 架构

###### TripleO 架构

undercloud由多个不同的组件组成，且以 TripleO 和 Ironic 等上游 OpenStack 部署项目为基础。与它的前辈级部署工具相似，undercloud 的目标如下：

*    发现其上部署 OpenStack 平台的裸机服务器。
*    充当要在这些节点上部署的软件的部署管理器。
*    为部署定义复杂的网络拓扑和配置。
*    向部署的节点发布软件更新和配置。
*    重新配置由 undercloud 部署的现有环境。
*    为 OpenStack 启用高可用性支持。

######### TripleO

TripleO 是 OpenStack On OpenStack 的易记名称，供 undercloud 用于部署、配置和自动化的部署与管理工具。TripleO 创建名为 overcloud 的生产云，以及名为 undercloud 的底层部署云。在能够部署 overcloud 之前，需要先部署 undercloud。 

######### TripleO 架构

TripleO 利用 Nova、Ironic、Neutron、Heat、Glance 和 Ceilometer 等多种 OpenStack 服务，在裸机硬件上部署 overcloud。

*    Ironic：调配物理硬件，并利用 PXE 和 IPMI 等技术将它提供给 overcloud 部署。
*    Nova：调配各种 overcloud 节点，如计算节点、控制器和 Ceph 存储节点。
*    Neutron：提供用于部署 overcloud 的联网环境。
*    Glance：提供映像存储库，供 Ironic 用于裸机调配，也用于部署 overcloud 节点时使用。
*    Heat：提供编配复杂架构的 overcloud 部署的途径。
*    Ceilometer：帮助收集与 overcloud 节点相关的指标。
*    Cinder：提供部署 overcloud 节点所需的卷。
*    Horizon：提供 Web 用户界面。
*    Keystone：为所有 overcloud 节点提供身份验证。

TripleO 架构具有以下一些优点：

*    TripleO 以 OpenStack API 及其组件为基础。
*    使用 OpenStack 组件可以加快 TripleO 的功能开发。TripleO 自动继承 Glance 和 Heat 等引入的所有新功能。

> 注意: 虽然 Ironic 主要用于调配物理计算机，本课程中我们将用它调配虚拟机。

TripleO 中使用的术语

*    术语 	        定义
*    Undercloud 	undercloud 是 Red Hat OpenStack Platform 主节点，用作调配和管理 OpenStack 节点 (overcloud) 的安装程序平台。
*    Overcloud 	    overcloud 是使用 undercloud 为生产和部署而创建的 Red Hat OpenStack Platform 环境。
*    裸机调配 	    安装未装有任何操作系统的节点。undercloud 可以为 overcloud 节点执行裸机调配。
*    内省 	        此过程旨在发现节点的属性并且了解它们是否达到预期的性能基准，由 Ironic 执行。Ironic 通过所用的电源管理接口为节点上电，然后使用发现映像对它们执行 PXE 引导。这些映像而后将数据回馈到 Ironic 数据库，如 CPU、RAM 和 disk 等主机属性。这有助于了解节点的能力，辨别出哪些不符合性能基准的节点。
*    编配 	        undercloud 提供并读取一系列 YAML 模板，来创建 overcloud 节点。借助 Heat 并利用这些 YAML 文件编配 overcloud 节点的部署。它也有助于通过编辑这些 YAML 文件来自定义 Heat 堆栈。
*    高可用性 	    undercloud 具备高可用性，能够提供控制器节点的故障转移。undercloud 在每个控制器节点上安装一套相同的组件，将它们作为一整个服务来进行管理。如果某一个控制器节点出现运行故障，拥有这样的群集便可提供回退机制。
*    部署角色 	    undercloud 提供了多个选项，可确保正确的角色（如 ceph-storage、compute 和 controller 等）etc.) 分配到最适当的节点来执行作业；它们称为部署角色。这些角色可以通过手动标记进行手动分配，或者使用自动化健康状态检查 (AHC) 工具来分配。只有特定的部署角色能够被分配；它们是 controller、compute、ceph-storage、cinder-storage 和 swift-storage。
*    基准测试 	    undercloud 内省过程允许在部署之前对 overcloud 节点进行基准测试，确保所有系统按照预期正常运行。内省和基准测试数据馈入到高级角色匹配流程中，确保只有真正具备履行某一角色的节点被分配到该角色。基准测试流程可以收集 CPU、内存和磁盘性能指标。这是通过自动化健康状态检查 (AHC) 工具来进行的。

### 描述 Undercloud 后端服务

###### Undercloud 的后端服务

Red Hat OpenStack Platform director 利用几种后端服务来运行其自身，并对 overcloud 进行管理。可以利用三种服务来故障排除 overcloud 部署期间可能出现的任何问题，它们分别是 Heat、Ironic 和 RabbitMQ。

###### Heat - 编配服务

编配服务为 Undercloud 提供基于模板的编配引擎，作为可重复运行的环境用于创建和管理资源，如存储、网络、实例和应用程序。

模板用于创建堆栈，后者是资源（如实例、浮动 IP、卷、安全组或用户）的集合。该服务通过单一模块化模板提供对所有 Undercloud 核心服务的访问，其具备额外的编配功能，如自动缩放和基本高可用性。

在配置后，管理员可以启动 Heat 堆栈。Heat 堆栈是指通过同一个界面部署和管理的多个基础架构资源的集合。堆栈提供统一的人类可读格式，可用于标准化和加速交付。尽管 Heat 项目是作为模拟 AWS CloudFormation 而启动（从而使其能够兼容 CloudFormation (CFN) 使用的模板格式），但它还支持自己的原生模板格式，名为 HOT，表示 Heat 编配模板。Undercloud 提供一系列 HOT 模板，用于部署不同的 overcloud 元素。

HOT 模板是使用 YAML 语法编写的，包含三个主要部分：

1. 参数：通过模板部署时提供的输入参数。
2. 资源：要部署的基础架构元素，如虚拟机或网络端口。
3. 输出：输出参数由 Heat 动态生成；例如，通过模板部署的公共 IP 和实例。

heat命令支持堆栈管理，包括下表中所含的子命令。

表 8.2. 使用heat命令进行 Heat 堆栈管理

*    子命令 	                详情
*    stack-create 	        创建堆栈。
*    stack-list 	        列出用户的堆栈。--show-nested 选项在列表中包含嵌套堆栈。
*    stack-show 	        显示堆栈的详细信息。
*    stack-delete 	        删除堆栈。
*    resource-list STACKNAME 	显示堆栈所创建的资源列表。-n 选项用于为所要显示的资源指定嵌套堆栈的深度。
*    deployment-list 	    列出部署的软件，及其部署 ID。
*    deployment-show ID 	显示被部署的软件组件的详细信息。

对 Heat 编配服务进行故障排除要求管理员理解底层基础架构的配置方式，因为 Heat 利用这些资源来创建堆栈。例如，在创建实例时，将通过 Keystone，以用户调用 Nova API 的相同方式来调用 Heat。在 Neutron 请求网络端口时，还将通过 Keystone 来进行对 API 的请求。这意味基础架构需要已配置且正在运行；管理员必须确保通过 Heat 请求的资源也可以手动请求。Heat 故障排除包括：

*    确保模板引用的所有 Undercloud 服务均已配置且正在运行。
*    确保资源（如映像或密钥对）存在。
*    确保基础架构具有部署堆栈的功能。

完成此故障排除后，管理员可以查看 Heat 服务的配置：

*    Heat API 配置。
*    Keystone 配置。

###### Ironic - 裸机调配服务

Red Hat OpenStack Platform 裸机调配服务 Ironic 支持调配虚拟和物理计算机供 overcloud 部署使用。 

> 注意: 在本课程中，Ironic 用于部署虚拟而非物理计算机供 overcloud 使用。

与节点相关的所有信息通过一个称为“内省”的过程来进行检索。节点被内省之后，它便准备好用于在其上部署 overcloud 服务。Ironic 利用 Undercloud 中包含的不同服务来部署 overcloud 服务。Ironic 支持不同的驱动程序来运行内省过程，具体根据环境硬件的支持，如 IPMI 和 DRAC。

下表提供了 ironic 命令的选项，在 Red Hat OpenStack Platform Director 中调配新节点时最常用到它们。

使用 ironic 命令进行裸机管理

*    子命令 	        详情
*    node-list 	    列出通过 Ironic 注册的节点
*    node-show 	    显示节点详细信息
*    node-update 	更新节点信息
*    node-set-provision-state 	更改节点的调配状态

###### RabbitMQ - 消息传递服务

Undercloud 基于 Red Hat OpenStack Platform 服务，它们利用了多种后端服务，其中包括用于支持在各项服务的组件之间进行通信的消息代理。Red Hat OpenStack Platform 中包含 RabbitMQ，作为在其 OpenStack 架构上使用的消息代理，因为提供了可用于设置高级配置的企业级功能。

消息代理实现了生产者和消费者应用之间的消息收发。在内部，此通信由 RabbitMQ 利用这两方之间的交换、排队和绑定来执行。当某一应用产生想要发送到一个或多个消费者应用的消息时，它将该消息放到绑定了一个或多个队列的交换上。消费者可以订阅这些队列，从而接收来自生产者的消息。

RabbitMQ 通过 RabbitMQ 工具提供基于 CLI 的工具集。这些工具可用于管理 RabbitMQ 的所有配置，并提供对当前状态和代理使用情况的概览。rabbitmqctl 命令提供了多个选项，来检查 RabbitMQ 服务的状态，以及它上面的队列/消息。它们包含在下表中。

使用rabbitmqctl命令进行 RabbitMQ 状态检查

*    子命令 	            详情
*    report 	        显示 RabbitMQ 守护进程的当前状态摘要
*    list_exchanges 	列出 RabbitMQ 守护进程上的交换
*    list_queues 	    列出 RabbitMQ 守护进程上的队列
*    list_bindings 	    列出 RabbitMQ 守护进程上的绑定
*    list_consumers 	列出 RabbitMQ 守护进程上的消费者

为了使用 OpenStack 服务对问题进行故障排除，理解请求在此服务的不同组件中遵循的工作流程非常重要。大多数 OpenStack 服务架构提供一个组件来提供该服务支持的 API；例如，在 Cinder 中，这由 cinder-api 服务来管理。此组件是该服务的其余组件架构的入口点，因此，要隔离问题，这是第一个要检查的服务。

一旦检查了 API 组件（特别是日志文件上不存在任何错误），下一步是检查其他服务组件是否能够毫无问题地通信。相关服务配置文件中与 RabbitMQ 消息代理或其配置相关的任何错误都应显示在服务的日志文件中。例如，对于 Cinder 服务，一旦 openstack-cinder-api 已通过此服务支持的 Cinder API 处理了请求，此请求将由 openstack-cinder-volume 和 openstack-cinder-scheduler 组件来处理。这些组件使用 RabbitMQ 消息代理来处理自身之间的通信，以在最切合实际的存储后端位置创建卷。大部分组件（例如，openstack-cinder-scheduler）都无法很好地回应无法运行的 RabbitMQ 后端，从而毫无原因地崩溃。要调试此情况，需要检查组件相关日志（例如 /var/log/cinder/scheduler.log）并检查作为 RabbitMQ 消息代理的客户端的组件上有哪些问题。由于 RabbitMQ 相关问题而导致组件崩溃的最常见问题通常与相关组件配置文件（例如，Cinder 组件的 cinder.conf）中缺少授权或加密的正确配置有关，或者与 Cinder 服务由于次要原因（与服务组件行为无关）而不可用有关。 


### 部署 Red Hat OpenStack Platform Director (Undercloud)

###### 部署 Red Hat OpenStack Platform Director

Red Hat OpenStack Platform Director 的部署将 TripleO 用于进行部署、管理和自动化。要部署生产云（即 overcloud），需要先部署 undercloud。为提供这一功能，TripleO 或 OpenStack Director 使用了多个原生 OpenStack API。TripleO 提供了各种各样的工具、定义 overcloud 的模板，以及用于部署 overcloud 的相关映像。

###### OpenStack Director 部署步骤

要部署 OpenStack Director (undercloud)，必须要执行三个基本步骤：

1. 使用必要的软件包安装主机。
2. 配置要用于 undercloud 部署的配置参数。
3. 部署 undercloud。
4. 验证部署。

######### 安装 undercloud 软件包

安装 python-tripleoclient 软件包，以提供 undercloud 安装所需的软件包以及统一 CLI。要使用 Horizon 控制面板访问 undercloud，需要安装 openstack-dashboard 软件包。

如果想要对部署的 overcloud 节点执行集成测试，也可安装 openstack-tempest。

[root@director ~]# yum -y install python-tripleoclient openstack-dashboard keystone openstack-tempest

######### 配置 undercloud 部署参数

OpenStack Platform Director 使用发出部署命令的本地目录中的 undercloud.conf 文件，作为其配置使用的文件。

在继续安装之前，需要先创建具有 sudo 特权的非 root 用户。

[root@director ~]# useradd stack
[root@director ~]# echo "redhat" | passwd stack --stdin
[root@director ~]# echo "stack ALL=(root) NOPASSWD:ALL" | tee -a /etc/sudoers.d/stack
[root@director ~]# chmod 0440 /etc/sudoers.d/stack

若要进行更改，可从 复制 示例文件到非 root 用户主目录中，并命名为 。undercloud.conf.sample/usr/share/instack-undercloud/undercloud.conf.sampleundercloud.conf

[stack@director ~]$ cp /usr/share/instack-undercloud/undercloud.conf.sample ~/undercloud.conf

可以使用 openstack-config 来更改 undercloud.conf。

[stack@director ~]$ sudo yum -y install openstack-utils

如下为要定义的Undercloud参数：

*    参数 	                            详情
*    local_ip 	                        为 undercloud 的调配 NIC 定义的 IP 地址。
*    undercloud_public_vip 	            为 undercloud 的公共 API 定义的 IP 地址。请使用与调配网络中任何其他 IP 都无冲突的 IP 地址。
*    undercloud_admin_vip 	            为 undercloud 的管理 API 定义的 IP 地址。请使用与调配网络中任何其他 IP 都无冲突的 IP 地址。
*    undercloud_service_certificate 	用于 OpenStack SSL 通信的证书的文件位置。
*    local_interface 	                用作 undercloud 的调配 NIC 的接口。
*    masquerade_network 	            为外部网络进行伪装的 CIDR 范围。这有助于通过使用 undercloud 上的 NAT 为调配网络提供外部访问。
*    dhcp_start,dhcp_end 	            overcloud 节点的 DHCP 分配范围的起点和终点。
*    network_cidr 	                    undercloud 用于 overcloud 节点的网络。这是调配网络 CIDR。
*    network_gateway 	                供 overcloud 实例使用的网关。这是将数据包转发到外部网络的 undercloud 主机 IP 地址。
*    inspection_iprange 	            undercloud 用于通过 PXE 引导发现裸机节点的 IP 地址范围。

```
[stack@director ~]$ openstack-config --set undercloud.conf DEFAULT local_ip 172.25.250.10/24
[stack@director ~]$ openstack-config --set undercloud.conf DEFAULT undercloud_public_vip 172.25.250.110
[stack@director ~]$ openstack-config --set undercloud.conf DEFAULT undercloud_admin_vip 172.25.250.111
[stack@director ~]$ openstack-config --set undercloud.conf DEFAULT local_interface eth0
[stack@director ~]$ openstack-config --set undercloud.conf DEFAULT masquerade_network 172.25.250.0/24
[stack@director ~]$ openstack-config --set undercloud.conf DEFAULT dhcp_start 172.25.250.20
[stack@director ~]$ openstack-config --set undercloud.conf DEFAULT dhcp_end 172.25.250.30
[stack@director ~]$ openstack-config --set undercloud.conf DEFAULT network_cidr 172.25.250.0/24
[stack@director ~]$ openstack-config --set undercloud.conf DEFAULT network_gateway 172.25.250.10
[stack@director ~]$ openstack-config --set undercloud.conf DEFAULT inspection_iprange 172.25.250.150,172.25.250.180
```

######### 部署 undercloud

Undercloud 部署是完全自动化的，使用由 TripleO 提供的 Puppet 清单。执行openstack undercloud install命令时，它利用 Python 脚本/usr/lib/python2.7/site-packages/instack_undercloud/undercloud.py开始执行。

undercloud 安装分成两大阶段：

*    Instack：在本地部署 diskimage-builder 元素。这些元素描述要在 undercloud 节点上进行的软件包安装或系统配置。所有可运行脚本位于/usr/share/instack_undercloud/puppet-stack-config下的install.d目录中。
*    os-refresh-config：提供分阶段部署系统配置的机制，并且使用 os-apply-config 应用配置。它管理响应配置参数更改的实例内流程，有助于针对部署后进行的任何更改来刷新配置。

######### 验证 undercloud 安装

成功安装 OpenStack Director 后将生成 stackrc 文件（包含用于访问 undercloud 的环境变量）和 undercloud-passwords.conf 文件（包含所有自动生成的密码，如果 undercloud 安装期间未提供密码参数）。

[stack@director ~]$ ls -l ~
total 16
-rw-------. 1 stack stack  227 Jan 20 04:04 stackrc
-rw-r--r--. 1 stack stack 5532 Jan 20 03:47 undercloud.conf
-rw-rw-r--. 1 stack stack 1398 Jan 20 03:48 undercloud-passwords.conf
[stack@director ~]$ cat ~/stackrc
export NOVA_VERSION=1.1
export OS_PASSWORD=$(sudo hiera admin_password)
export OS_AUTH_URL=http://172.25.250.10:5000/v2.0
export OS_USERNAME=admin
export OS_TENANT_NAME=admin
export COMPUTE_API_VERSION=1.1
export OS_NO_CACHE=True
export OS_CLOUDNAME=undercloud
export OS_IMAGE_API_VERSION=1

使用openstack endpoint list验证 OpenStack undercloud 安装所配置的端点。

[stack@director ~]$ source ~/stackrc
[stack@director ~]$ openstack endpoint list
+----------------------------------+-----------+--------------+---------------+
| ID                               | Region    | Service Name | Service Type  |
+----------------------------------+-----------+--------------+---------------+
| 3a825a3adce84748bf69c49140b1114b | regionOne | ceilometer   | metering      |
| 14003d5a5e8b4ce58eed34dbd5d06da3 | regionOne | nova         | compute       |
| 2ca47a5535884fdb9e9d41e96e7ae8ed | regionOne | swift        | object-store  |
| bc9b17a890a94e3ba078d4091ee018ba | regionOne | glance       | image         |
| e601183f7faf48679827f4ea94ff5da1 | regionOne | novav3       | computev3     |
| b12b9bb3731c40c684fd33f4110bb37d | regionOne | heat         | orchestration |
| 1ea6e9c157b144c1b2030baf7ed113ed | regionOne | ironic       | baremetal     |
| 8ebf0c925d084bcea3261608808c42ad | regionOne | neutron      | network       |
| 53a3fcf926b14781b105557009dff107 | regionOne | keystone     | identity      |
+----------------------------------+-----------+--------------+---------------+


### 部署 Overcloud

###### 准备 overcloud 映像

Undercloud 中的下列默认节点类型是可以在 overcloud 中部署的节点：

*    控制器
*    计算
*    Ceph 存储
*    Cinder 存储
*    Swift 存储

调配 overcloud 节点时需要以下预构建映像：

*    发现内核和 ramdisk：在裸机发现和内省期间使用。
*    部署内核和 ramdisk：在调配和部署的第一阶段中使用。
*    overcloud 内核、ramdisk 和完整映像：每个创建的 overcloud 的基础。

这些基础映像可以从红帽客户门户下载。 

###### 上传 overcloud 映像

使用 openstack overcloud image upload 将 overcloud 未打包映像上传到 undercloud。

[stack@director images]$ source ~/stackrc
[stack@director ~]$ openstack overcloud image upload --imagepath ~/images

上传的映像有：

*    bm-deploy-kernel
*    bm-deploy-ramdisk
*    overcloud-full-vmlinuz
*    overcloud-full
*    overcloud-full-initrd

bm-deploy-kernel 和 bm-deploy-ramdisk 都用于裸机部署供 overcloud 使用的计算机，而 overcloud-full-vmlinuz、overcloud-full 和 overcloud-full-initrd 则支持 Red Hat OpenStack Platform 环境本身的部署。发现映像也安装到 PXE 服务器。要查看映像列表，可使用命令 openstack image list；它不会显示发现映像。

```
[stack@director ~]$ openstack image list
+--------------------------------------+------------------------+
| ID                                   | Name                   |
+--------------------------------------+------------------------+
| cf16ce61-71dd-4846-bfda-f63d803b4410 | bm-deploy-kernel       |
| 9dc1b4ee-7f2a-4764-99c5-1403bd68fa93 | bm-deploy-ramdisk      |
| b013d447-4cc5-4f3d-b38e-d9d428253e21 | overcloud-full-vmlinuz |
| 629da664-bb0c-455f-9027-f32d73c5768a | overcloud-full         |
| 29c25169-c0c3-4c9a-86f2-1a526a5113ad | overcloud-full-initrd  |
+--------------------------------------+------------------------+
```

###### overcloud 节点部署的阶段

1. 注册:

*    stack 用户上传关于所提议的 overcloud 节点的状态。
*    此信息包含用于电源管理的凭据。
*    此信息存储在 Ironic 数据库中，在内省阶段使用。

2. 内省：

*    Ironic 连接已注册的节点，收集更多关于硬件资源的详细信息。
*    此过程中上发现内核和 ramdisk 映像。

3. 部署:

*    stack 用户通过分配内省阶段中发现的资源和节点，部署 overcloud 节点。
*    此阶段中使用硬件配置文件和 Heat 模板。

######### 注册 overcloud 节点

注册 overcloud 节点包括将它添加到可能用于 overcloud 的节点的 Ironic 列表中。undercloud 需要下列信息来注册节点：

*    所使用的电源管理类型，如 IPMI 或 PXE over SSH。可以使用 ironic driver-list 列出 Ironic 支持的各种电源管理驱动程序。
*    节点在电源管理网络上的 IP 地址。
*    用于电源管理接口的凭据。
*    PXE/调配网络上 NIC 的 MAC 地址。
*    将供内省使用的内核和 ramdisk。

所有这些信息可以使用 JSON（JavaScript 对象表示法）文件或 CSV 文件进行传递。openstack baremetal import命令可将此文件导入到 Ironic 数据库。

```
[stack@director ~]$ openstack baremetal import --json instackenv.json
``` 
  
######### overcloud 节点内省

对于 overcloud 节点的内省/发现，Ironic 使用 undercloud 提供的 PXE（预启动执行环境）。dnsmasq 用于向 Ironic 服务提供 DHCP 和 PXE 功能。PXE 发现映像通过 HTTP 交付。在内省之前，注册的节点分配有有效的内核和 ramdisk；每一要内省的节点：

*    Power State 应当为 power off。
*    Provision State 应当为 available。
*    Maintenance 应当为 False。
*    实例 UUID 应当设置为 None。

openstack baremetal introspection命令用于启动内省，而 bulk start 则可用于继续内省所有节点。要检查的两个节点是 controller 和 compute 节点。

```
[stack@director ~]$ openstack baremetal introspection bulk start
Setting available nodes to manageable...
Starting introspection of node: e7409a60-c28e-40e4-bab8-61f06a5b44c4
Starting introspection of node: b6eecdcd-63e7-4fd4-a336-8f5eb184329c
Waiting for introspection to finish...
```

###### overcloud 部署角色

在内省后，undercloud 知道了用于部署 overcloud 的节点，但可能还不知道要部署的 overcloud 节点类型是什么。类别用于分配节点部署角色，它们对应于 overcloud 节点类型：

*    control：控制器节点
*    compute：计算节点
*    ceph-storage：Ceph 存储节点
*    block-storage：Cinder 存储节点
*    object-storage：Swift 存储节点

undercloud 使用裸机硬编码类别，必须设置为任何未用角色的默认类别；否则，将使用基于角色的类别。

```
[stack@director ~]$ openstack flavor create --id auto --ram 6144 --disk 38 --vcpus 2 baremetal
[stack@director ~]$ openstack flavor create --id auto --ram 6144 --disk 38 --vcpus 2 compute
```

undercloud 执行自动化角色匹配，为每一节点类别应用适当的硬件。如果节点位于完全相同的硬件上，而且也没有创建类别，则为每一节点随机选择部署角色。也可以通过手动标记向节点绑定部署角色。

为使用这些部署配置文件，需要利用 capabilities:profile 属性关联到对应的类别。需要 capabilities:boot_option 属性来设置类别的引导模式。

```
[stack@director ~]$ openstack flavor set
> --property "cpu_arch"="x86_64"
> --property "capabilities:boot_option"="local"
> --property "capabilities:profile"="compute" compute
```

###### 自定义 Heat 模板

Heat 模板提供了自定义 overcloud 部署来适合不同架构方案的途径。/usr/share/openstack-tripleo-heat-templates中提供了示例 Heat 模板。这些模板可以自定义并按需要使用。

######### 部署 overcloud 节点

要部署 overcloud 节点，可使用 openstack overcloud deploy 命令并为要调配的不同节点指定缩放值。如果需要使用不同的模板集合，可以使用 --templates 选项进行指定，-e 或 --environment-file 选项则用于使用环境文件中指定的值来覆盖默认的变量值。要部署的控制器和计算节点的数量通过 --control-scale 和 --compute-scale 选项进行控制。

```
[stack@director ~]$ openstack overcloud deploy --templates \
> --ntp-server 172.25.250.254 --control-scale 1 --compute-scale 1 \
> --neutron-tunnel-types vxlan --neutron-network-type vxlan \
> --neutron-public-interface nic1
Deploying templates in the directory /usr/share/openstack-tripleo-heat-templates/
```


### 总结

在本章中，您可以学到：

*    Undercloud 支持 overcloud 节点的高可用性、缩放和升级。
*    首先创建一个 undercloud（面向操作员的部署云），它包含必要的 OpenStack 组件以部署和管理 overcloud（面向实际租户的工作负载云）。
*    TripleO 利用 OpenStack 的现有核心组件，如 Nova、Ironic、Neutron、Heat、Glance 和 Ceilometer，可以在裸机硬件上部署 OpenStack。
*    将 overcloud 部署到物理服务器通过结合使用 Heat、Nova、Neutron、Glance 和 Ironic 来进行。
*    在部署 overcloud 之前，首先要下载或构建将要安装到 overcloud 的各个节点上的映像。
*    内省 ramdisk 探查节点上的硬件并收集相关数据，如 CPU 核心数、本地磁盘大小和 RAM 数量等。
*    在 OpenStack 云中创建实例时，其类别指定了应当创建的虚拟机的容量。
*    Undercloud 利用 Pacemaker 来实现高可用性。
