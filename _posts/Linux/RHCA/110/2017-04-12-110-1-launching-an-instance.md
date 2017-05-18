---
layout: post
title:  "启动实例(110-1)"
categories: Linux
tags: RHCA 110
---

### 登录 Horizon Web 界面

OpenStack 控制面板是用于管理 OpenStack 服务的 Web 型图形用户界面。此控制面板也称为 Horizon。它可以对控制面板的品牌进行自定义。

###### 使用自签名证书

通常，证书授权机构 (CA) 将用于生成 OpenStack 使用的证书，包括适用于 Horizon 的 Web 服务器证书。在课堂中，将使用默认证书机制来生成自签名证书。由于安全原因，默认情况下，Firefox 不接受这些自签名证书。要手动接受证书，请遵循以下步骤。

1. 浏览到 Horizon URL。
2. 展开“我理解风险”菜单。
3. 按“添加例外项...”按钮。
4. 按确认安全例外按钮来永久储存此例外。

如果必须重新安装 OpenStack，自签名证书将保留在浏览器中，并且在尝试连接到新的 OpenStack 安装时，将显示错误：

```
Secure Connection Failed
An error occurred during a connection to demo.example.com. You have received an invalid certificate.
Please contact the server administrator or email correspondent and give them the following information:
Your certificate contains the same serial number as another certificate issued by the certificate authority.
Please get a new certificate containing a unique serial number.
(Error code: sec_error_reused_issuer_and_serial)
```

此错误指出，序列号已经由证书颁发机构复用。要在 Firefox 中移除旧证书和证书颁发机构，请遵循以下步骤：

1. 按 Alt 键打开 Firefox 菜单。
2. 转至 Edit → Preferences。
3. 单击高级图标，然后选择证书选项卡。
4. 按“查看证书”按钮。
5. 在颁发机构选项卡中，向下滚动到 openstack CA（若存在）。高亮显示服务器名称，然后按删除或不信任... 按钮。按“确定”按钮以确认。对 openstack 列表下面列出的任何其他 CA 执行相同操作。
6. 单击服务器选项卡。在 openstack 可折叠标签中，高亮显示服务器名称，然后按删除按钮。按“确定”按钮以确认。针对 openstack 下面列出的任何其他服务器，重复相同步骤。
7. 关闭“首选项”窗口。
8. 按 Alt 键再次打开菜单。转至 History → Clear Recent History...。
9. 在“要清除的时间范围：”中，选择“Everything”。选择“Cache”和“Active Logins”复选框。按“立即清除”按钮。
10. 转至 Horizon 仪表板 URL 并如上所述接受自签名证书。

###### 项目中的用户

身份选项卡为查看和管理项目与用户提供了接口。为创建 OpenStack 云，需要使用多租户项目，以充当由不同用户所有的资源的容器。

在创建新的项目时，就已预先配置了一组资源配额。配额包括实例数量、VCPU 数目、RAM 大小，以及可分配给项目中实例的浮动 IP 数。用户可以分配有多个项目，但只有其中一个可指定为主要项目。主要项目就是用户所关联的第一个项目。

可以使用 Horizon 控制面板或命令行工具创建、修改和删除项目，查看项目使用情况，添加或移除作为项目成员的用户，修改配额，以及设置活动项目。

通过利用 Horizon 控制面板或命令行工具，在管理项目中关联了管理角色的用户可以查看、创建、编辑和删除用户，以及更改用户密码。只有以具有管理特权的用户登录时，才可看到用户选项卡。在创建用户时，除了指定用户名、密码、电子邮件地址和主要项目外，还需要指定角色。OpenStack 附带有两个预定义角色，即 _member_ 和 admin。为用户添加管理员角色，可以使该用户成为超级管理用户。

> 在 Red Hat OpenStack Platform 的早期版本中，命令行实用工具中使用了“租户”这一术语，而“项目”则是在 Horizon 中首次引入的。当前版本的 OpenStack 同时在 Horizon 控制面板和统一命令行界面中使用“项目”这个术语。


### 启动实例所需的组件

要在 OpenStack 中启动实例，首先需要设置下列组件：

1. 映像

映像是包含虚拟磁盘的文件，其上装有可引导的操作系统。可以使用 openstack image create 创建或自定义映像；也可以导入不同软件供应商提供的多种预制映像。在多租户云环境中，用户也可将个人的映像与其他项目共享。这些映像可以使用各种各样的格式（RAW、QCOW2、ISO、VMDK、VHD、ARI、AKI 和 AMI），并可导入到 OpenStack 中。

当前项目的映像可以在 Horizon 控制面板的Images下找到。Images主页中提供创建映像按钮，可以创建新的映像。

2. 类别

OpenStack 中的虚拟硬件模板称为类别。它定义 RAM 最小大小，以及磁盘和映像格式。Horizon 控制面板和命令行工具提供相关功能，可修改和删除现有的类别，以及创建新的类别。

权限最小的用户在启动实例时，可在 Horizon 控制面板中查找当前项目的类别，但只有管理员才有权限编辑或新建类别（这些权限也可委派给其他用户）。管理员用户可以使用Admin选项卡，其中提供了 Flavors 项目。

例如，m1.tiny, m1.small, m1.medium, m1.large, m1.xlarge

3. 安全组

安全组是 IP 过滤规则的集合，这些规则应用于实例的网络。它们根据项目而不同，项目成员可以编辑默认规则，也可添加新的规则集。所有项目都预先创建有默认安全组，它应用到未定义其他安全组的实例。除非经过编辑，否则默认安全组将拒绝所有传入流量。

当前项目的安全组可以在 Horizon 控制面板的Access & Security下找到。Access & Security主页中提供创建安全组按钮，可以创建新的安全组。

4. 密钥对

为提升云中的安全形象，使用 ssh 登录云实例时通过利用公钥和私钥组成的密钥对来进行，而不使用用户名与密码。这些密钥对可以在 Horizon 控制面板的Access & Security下创建或导入。Access & Security主页中提供创建密钥对按钮，可以创建或导入密钥对。

5. 网络

网络定义添加到实例的网络路由。根据实例所属的网络以及网络的路由表，实例可以面向公共网络或面向专用网络。可以将浮动 IP 地址关联到面向专用网络的接口，使它变成面向公共网络，从而能够从 OpenStack 网络外部对它进行访问。管理员可以创建和配置网络和子网，再指示其他 OpenStack 服务将虚拟设备连接到这些网络上的端口，从而配置丰富的网络拓扑。OpenStack 联网支持每个项目拥有多个专用网络，并且允许项目选择自己的 IP 寻址方案，即使 IP 地址与其他项目所用的相重叠也无妨。

网络拓扑可以在 Horizon 控制面板的Project下查看，只需选择Network菜单下的网络拓扑菜单项。


### OpenStack 架构

###### 本课程阐述的服务

1. Horizon（控制面板）

    此 Web 界面可管理各种 OpenStack 服务。它为启动实例、管理网络和设置访问控制等操作提供一个图形用户界面。

    在上一实验中创建实例时用于登录的 Web 界面就是由 Horizon 服务提供的。

2. Keystone（身份）

    此中央化身份服务能够为其他服务提供身份验证和授权。Keystone 也为特定 OpenStack 云中运行的服务及关联的端点提供中央目录。它支持多种形式的身份验证，包括用户名与密码凭据、基于令牌的系统，以及 Amazon Web Services (AWS) 登录。Keystone 充当用户和组件的单一登录 (SSO) 身份验证服务。此服务负责创建和管理用户、角色以及项目/租户。

*    用户：Keystone 服务验证由合法用户发出的传入请求，并可向其分配令牌以根据用户在项目中的角色来访问特定的资源。上一节中曾使用 test_user 用户以 _member_ 角色登录，使用的凭据则是用户名和密码。
*    项目：项目这一术语在 Keystone 中使用，相当于 Red Hat OpenStack Platform 旧版本中所用的租户。项目是用户、映像、实例、网络和卷等各项的组合。它有助于隔离或分组身份对象。根据服务提供商，项目/租户可映射到客户、帐户、组织或环境。在上一实验中，test-project 就是作为项目而使用的。

3. Neutron（网络）

    此软件定义型网络服务可帮助创建网络、子网、路由器和浮动 IP 地址。管理员可以创建由其他 OpenStack 服务管理的接口设备并与网络连接。OpenStack 联网功能随附了各种插件和代理，可用于 Cisco 虚拟和物理交换机，以及 Open vSwitch 等交换机。常见代理为 L3 和 DHCP，后者为实例提供 DHCP IP 地址。OpenStack 联网功能让项目/租户能够创建高级虚拟网络拓扑，这些拓扑中可包含防火墙、负载平衡器和虚拟专用网络 (VPN) 等服务。

    上一实验中使用的公共和专用网络以及浮动 IP 地址就是由 Neutron 服务提供的。

4. Cinder（块存储）

    此服务管理虚拟机的存储卷。这是在 Nova 中运行的实例的永久性块存储。可以通过执行快照来备份数据，快照可用于还原数据，或者用于创建新的块存储卷。这通常在实例中用于存储，如数据库文件。

5. Nova（计算）

    此服务管理节点上所运行的虚拟机，并按需提供虚拟机。Nova 是一种分布式服务，与 Keystone（身份验证）、Glance（映像）和 Horizon（Web 接口）进行交互。Nova 设计为可在标准硬件上水平缩放，根据需要下载映像以启动实例。Nova 计算将 libvirtd、qemu 和 kvm 用于系统管理程序。

    上一实验中使用了 Nova 服务生成 test_instance 实例。

6. Glance（映像）

    此服务充当虚拟机映像的注册表，允许用户复制服务器映像以即时存储。这些映像可在设置新实例时用作模板。

    在上一实验中，我们使用了 test_image 创建实例。

###### 其他支持的服务

本课程中未阐述这些额外服务，但它们受到红帽的支持：

7. Swift（对象存储）

    此服务提供对象存储，供用户存储和检索文件。Swift 架构是分布式的，允许水平缩放，并提供防故障冗余能力。

8. Heat（编配）

    此服务可通过表述性状态转移 (REST) API 和兼容 CloudFormation 的查询 API，编配使用 Amazon Web Services (AWS) CloudFormation 模板格式的多个复合云应用。

9. Ceilometer（遥测）

    OpenStack 遥测服务提供用户级使用情况数据，它们可用于客户记帐、系统监控或警告提醒。它可以从 OpenStack 服务发送的通知中收集数据，如计算使用情况事件，或者通过轮询 OpenStack 基础架构资源来收集。此外，该服务也提供插件系统，可用于添加新的监控指标。

10. Gnocchi（时间序列数据库）

    Gnocchi 是 TDBaaS（时间序列数据库即服务），用于将 Ceilometer 收集的指标和资源元数据存储到此数据库中。它将 Swift 用作其存储。

11. Trove（关系数据库）

    Trove 是数据库即服务 (DBaaS)，以提供可扩展且可靠的关系数据库引擎为使命。此服务以较高的性能提供隔离，同时自动化复杂的数据库管理任务，如部署、配置、修补、备份、恢复和监控。

12. Manila（共享文件存储）

    Manila 是安全文件共享即服务。它使用 NFS 和 CIFS 协议共享文件。它可以配置为在单一节点后端上运行，或跨越多个节点运行。

13. Sahara（数据处理）

    Sahara 旨在为用户提供一个简便的途径，在 OpenStack 上调配数据处理群集（如 Hadoop、Spark 和 Storm）。

14. TripleO

    TripleO，也称为 OpenStack on OpenStack (OOO)，设计为以 OpenStack 自有的服务为基础来安装、升级和运行 OpenStack 云。它使用 Nova、Neutron 和 Heat，以及 Chef 或 Puppet 等其他编配工具自动化队伍管理，包括在数据中心规模上进行缩放。

15. Ironic（裸机调配）

    Ironic 是调配物理硬件而非虚拟机的 OpenStack 项目。它提供 PXE 和 IPMI 等多种驱动程序，涵盖种类繁多的硬件。它也允许添加特定于供应商的驱动程序。

16. Tempest（集成测试）

    Tempest 提供一组可对实时 OpenStack 群集运行的集成测试。它提供用于 OpenStack API 验证的多种测试、情景，以及可在验证 OpenStack 部署时使用的其他具体测试。

###### 预览中的服务

下列服务目前处于预览状态：

1. Designate（DNS 服务）

    Designate 为 OpenStack 提供 DNSaaS 服务。Designate 开箱支持 PowerDNS 和 Bind9。

2. OpenDaylight (SDN)

    OpenDaylight 是一个开源项目，注重于加快对软件定义型网络 (SDN) 的采用。OpenDaylight 控制器可以为物理和虚拟网络提供灵活的管理。在 SDN 控制器一侧，OpenDaylight 具有可与 Neutron 交互的北向 API，也将 OVSDB（Open vSwitch 数据库管理协议）用于计算节点上虚拟交换机的南向配置。

3. Grafana（运营工具）

    Grafana 是一款功能丰富的开源指标控制面板和图形编辑器。Grafana 提供可插拔的面板和数据源，可以轻松扩展。它内置了对 Graphite、InfluxDB 和 Cloudwatch 等最流行时间序列数据源的支持。

4. OVS-DPDK（数据包处理）

    此数据包处理项目包含一组重要的工具，可以加快网络转型软件（即所谓的软件定义型网络 (SDN)）的开发，还包含一项称为网络功能虚拟化 (NFV) 的补充倡议。OVS-DPDK（Open vSwitch 数据平面开发套件）是一个 OpenStack 补丁，可以实现在 OpenStack 云中运用 DPDK vSwitch。

5. FWaaS（防火墙即服务）

    FWaaS（防火墙即服务）是一个 Neutron 扩展，它为安全功能集提供以云为中心的抽象，跨越传统 L2/L3 防火墙，实现更加丰富的应用感知型下一代防火墙。

6. VPNaaS（VPN 服务）

    VPNaaS（VPN 即服务）是一个 Neutron 扩展，引入一个 VPN 功能集。

7. NFV（网络功能虚拟化）

    它定义将通常独立的、用于高级和低级网络功能（如防火墙、网络地址转换、侵入检测、缓存、网关和加速器等）的专用设备替换为一个或一组称为虚拟网络功能 (VNF) 的实例。OpenStack 的 NFV 支持旨在为电信提供商的部署提供尽可能最佳的基础架构，同时遵循 IaaS 云的设计原则。

###### OpenStack 术语

OpenStack 使用下列术语：

*    云控制器：协调管理器。OpenStack 云中的所有机器使用高级消息队列协议 (AMQP) 与云控制器通信。在 Red Hat OpenStack Platform 中，有两个选项可用于 AMQP：Apache Qpid 消息守护进程 (qpidd) 和 RabbitMQ。
*    云类型：云计算可通过三种形式部署，即公共云、私有云和混合云。这些不同的形式根据所处理的数据种类提供不同级别的安全性和管理。
*    云模型：根据所提供的具体服务，云服务可以归类到三种不同的模型：IaaS（基础架构即服务）、PaaS（平台即服务）和 SaaS（软件即服务）。Red Hat OpenStack Platform 基于 IaaS，允许使用者为各种各样的部署构建自己的基础架构；而红帽 OpenShift 则属于 PaaS，因为它为主机应用提供可扩展的平台。
*    计算节点：一种系统管理程序；运行 Nova 计算服务的任何计算机。通常，计算机只运行 Nova 计算服务。
*    卷（块存储）：提供给单个实例并连接到该实例的永久磁盘。卷是永久性的，可以连接到正在运行的实例（或与之断开连接）。Cinder 服务默认将 LVM 用作后端，卷以裸设备的形式出现在实例上。逻辑卷是从此卷组中创建的。可以创建卷快照，类似于正常的逻辑卷快照。
*    临时磁盘：暂时供实例使用的磁盘。创建实例时，在计算节点的 /var/lib/nova/instances/instance-00000000X/disk.local 中创建临时磁盘作为 QCOW2 映像。当实例终止时，在使用 dd 擦除后，将删除此磁盘。第一个临时磁盘通常在实例中显示为 /dev/vdb。
*    实例：虚拟机。一些实用工具可能将它称为服务器。
*    类别：与实例关联的硬件。这包含 RAM、CPU 和磁盘。
*    堆栈：从模板中构建的一组实例。模板文件以 JavaScript 对象表示法 (JSON) 编写，这种数据交换格式旨在以更简单的方式替代可扩展标记语言 (XML) 文件编码。堆栈和模板文件在 Heat 编配服务中使用。
*    OpenStack 联网 (Neutron)：OpenStack 联网 API 使用下列抽象概念来描述网络资源：

        1. 网络：一个隔离的 L2 区段，与物理网络世界中的 VLAN 类似。
        2. 子网：一组 IPv4 或 IPv6 地址及关联的配置状态。
        3. 端口：用于将单个设备（如虚拟服务器的 NIC）连接到虚拟网络的连接点。也用于描述关联的网络配置，如要在该端口上使用的 MAC 和 IP 地址。

*    Open vSwitch：用于提供虚拟交换机的软件。Open vSwitch 提供流量排队和成型以及自动流量控制。Open vSwitch 插件将用于 OpenStack 网络。

### 总结

在本章中，您可以学到：

*    OpenStack 由各种不同服务组成，如 Horizon、Keystone、Nova、Cinder、Swift、Neutron、Glance、Heat 和 Ceilometer。
*    Horizon 控制面板可用于在 Red Hat OpenStack Platform 中启动实例。
*    云计算中有多种服务模型可用：基础架构即服务 (IaaS)、平台即服务 (PaaS) 和软件即服务 (SaaS)。
*    云计算是一种模型，可以实现随需应变地通过网络从可配置计算资源共享池中获取所需的资源，资源能够快速供应并释放，使管理资源的工作量和与服务提供商的交互减小到最低限度。
*    Red Hat OpenStack Platform 包含各种不同的服务，分别提供一组特定的功能。此类架构是模块化的：管理员可以仅部署他们需要的服务，或者将现有基础架构的组成部分重新利用到 OpenStack 环境中。
