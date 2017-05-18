---
layout: post
title:  "安装 OpenStack(110-11)"
categories: Linux
tags: RHCA 110
---

### 使用 PackStack 安装 OpenStack

###### 使用 PackStack 部署 Red Hat OpenStack Platform

借助 PackStack，用户可以将 Red Hat OpenStack Platform 环境轻松地部署到一台或多台计算机上。它可以通过应答文件进行自定义，文件中包含的一组参数可用于对底层 Red Hat OpenStack Platform 服务进行自定义配置。默认情况下，此工具将使用 VXLAN 配置 Neutron，并将 LVM 设置为 Cinder 的后端，而且还提供其他常见配置。此外，也支持一体化部署。

> VXLAN 配置要求设置 VXLAN 组，后者将定义 VXLAN 型网络所要关联的多播组地址；还要求设置可以使用的 VXLAN 网络标识符 (VNI) 的范围。每个专用网络将分配一个 VNI，此网络中实例之间的流量将标记有该 VNI。

###### 应答文件

默认情况下，PackStack 提供一个应答文件模板，不必自定义便可部署一体化环境。此类应答文件中包含的选项几乎可用于细调 Red Hat OpenStack Platform 环境的每个方面，包括架构布局、转为多计算节点型部署，或者细调要用于 Cinder 和 Neutron 服务的后端。下表中显示了用于细调环境的部分参数。

> PackStack 支持更改已部署环境的配置；只需在关联的应答文件中更改相应的参数，然后重新运行 packstack 命令来应用即可。 

###### PackStack 应答文件参数

*    参数 	                                    描述
*    CONFIG_DEFAULT_PASSWORD 	                设置所有 Red Hat OpenStack Platform 服务和用户的密码
*    CONFIG_CINDER_ VOLUMES_CREATE 	            设置将 cinder-volumes 卷组创建为 Cinder 的后端
*    CONFIG_PROVISION_ DEMO 	                设置演示环境的部署（如网络、映像和密钥对）
*    CONFIG_SERVICE_ INSTALL 	                启用/禁用 SERVICE 在 Red Hat OpenStack Platform 环境中的部署（如 CONFIG_KEYSTONE_INSTALL 和 CONFIG_CINDER_INSTALL 等）
*    CONFIG_NEUTRON_ML2_TYPE_DRIVERS 	        配置供 Neutron 使用的网络驱动程序（如 vxlan、vlan 和 gre）
*    CONFIG_NEUTRON_ML2_TENANT_NETWORK_TYPES 	配置 Red Hat OpenStack Platform 环境中可用的租户网络类型（如 vxlan、vlan 和 gre）
*    CONFIG_CONTROLLER_ HOST 	                设置控制器节点的 IP 地址
*    CONFIG_COMPUTE_HOSTS 	                    设置计算节点的 IP 地址

###### 使用 PackStack 安装 OpenStack。

1. 在 servera 上，使用 systemctl 停止并禁用 NetworkManager 服务。

    [root@servera ~]# systemctl stop NetworkManager.service
    [root@servera ~]# systemctl disable NetworkManager.service

2. 生成名为 /root/demoanswers.txt 的默认应答文件，再查看生成的文件。

    [root@servera ~]# packstack --gen-answer-file /root/demoanswers.txt
                
3. 修改应答文件 /root/demoanswers.txt，以便在 servera 上部署控制器与计算节点，并在 serverb 上部署计算节点。尽可能使用默认值，但更改以下几项：

    CONFIG_DEFAULT_PASSWORD=redhat
    CONFIG_SWIFT_INSTALL=n
    CONFIG_CEILOMETER_INSTALL=n
    CONFIG_NTP_SERVERS=172.25.254.254
    CONFIG_COMPUTE_HOSTS=172.25.250.10,172.25.250.11
    CONFIG_KEYSTONE_ADMIN_PW=redhat
    CONFIG_CINDER_VOLUMES_CREATE=n
    CONFIG_NEUTRON_ML2_VXLAN_GROUP=239.1.1.2
    CONFIG_NEUTRON_OVS_BRIDGE_MAPPINGS=physnet1:br-ex
    CONFIG_NEUTRON_OVS_TUNNEL_IF=eth1
    CONFIG_PROVISION_DEMO=n

4. 使用修改后的应答文件 /root/demoanswers.txt 运行 packstack，以部署新的 OpenStack 节点。

    [root@servera ~]# packstack --answer-file /root/demoanswers.txt
    ...
     **** Installation completed successfully ******
    ...

5. 在 servera 上，检查 /etc/sysconfig/network-scripts/ifcfg-br-ex（外部 Open vSwitch 网桥）是否为如下所示：

    [root@servera ~]# cat /etc/sysconfig/network-scripts/ifcfg-br-ex
    DEVICE=br-ex
    BOOTPROTO=static
    ONBOOT=yes
    TYPE=OVSBridge
    DEVICETYPE=ovs
    USERCTL=yes
    PEERDNS=yes
    IPV6INIT=no
    IPADDR=172.25.250.10
    NETMASK=255.255.255.0
    GATEWAY=172.25.250.254
    DNS1=172.25.250.254
    DOMAIN="lab.example.com example.com"

6. 在 servera 上，检查 /etc/sysconfig/network-scripts/ifcfg-eth0（外部 Open vSwitch 网桥 br-ex 的端口）是否为如下所示：

    [root@servera ~]# cat /etc/sysconfig/network-scripts/ifcfg-eth0
    DEVICE=eth0
    ONBOOT=yes
    TYPE=OVSPort
    DEVICETYPE=ovs
    OVS_BRIDGE=br-ex

     
### 验证 OpenStack 功能

###### Red Hat OpenStack Platform 服务

PackStack 使用可用的硬件资源部署 Red Hat OpenStack Platform 服务。这些服务由多个组件组成，可以部署在不同的计算机上。Nova 或 Neutron 等服务必须符合特定的架构，因此一些组件必须部署在一起。除了 Red Hat OpenStack Platform 服务外，PackStack 也将部署一些额外的辅助服务，如 MariaDB 和 RabbitMQ，它们为 Red Hat OpenStack Platform 永久配置以及组件间通信服务提供支持。

###### Red Hat OpenStack Platform 节点

PackStack 将 Red Hat OpenStack Platform 服务部署在一台或多台计算机内。在这种部署中，利用两台计算机安装 Red Hat OpenStack Platform 环境：一台部署为控制器节点，另一台则作为计算节点。它们在 Red Hat OpenStack Platform 中担当不同的角色。所有 Red Hat OpenStack Platform 核心服务（如 Glance 和 Cinder）部署在控制器节点中。计算节点仅配置其上运行实例所需的服务。一个部署中通常包含一个控制器节点，以及一个或多个计算节点。计算节点可以动态添加到 Red Hat OpenStack Platform 环境中。 

###### 部署实例

若要检查所有 Red Hat OpenStack Platform 组件是否都已安装正确，一个好办法是部署实例。部署实例不仅涉及到管理实例部署的 Red Hat OpenStack Platform 服务 (Nova)，也涉及到其他服务，如用于身份验证和授权的 Keystone、用于映像管理的 Glance，以及用于联网管理的 Neutron。下表中显示可在统一 CLI（用于管理 Red Hat OpenStack Platform 服务的新 CLI）中使用的部分命令。它们可用于检查是否满足实例、映像、类别和可用网络的最低要求。

使用 openstack 命令部署实例

*    命令 	说明
*    openstack image list 	列出可用的映像
*    openstack network list 	列出可用的网络
*    openstack flavor list 	列出可用的类别
*    openstack server create 	创建新实例
*    openstack server list 	列出可用的服务器

###### Red Hat OpenStack Platform 状态概述

Red Hat OpenStack Platform 提供一组用于管理 Red Hat OpenStack Platform 环境的命令。这些命令通过 openstack-* 前缀区分。它们为获取 Red Hat OpenStack Platform 环境的状态概述提供了良好的工具。openstack-status 命令提供 Red Hat OpenStack Platform 环境的全面概述，包括不同的服务和组件状态，以及配置的虚拟资源（如映像）。openstack-service 提供与给定服务关联的所有组件的状态。 

###### 演示：验证 OpenStack 功能

1. 在 servera 上，检查 Red Hat OpenStack Platform 组件状态。

    [root@servera ~]# openstack-status
    == Nova services ==
    openstack-nova-api:                     active
    openstack-nova-cert:                    active
    openstack-nova-compute:                 active
    openstack-nova-network:                 inactive  (disabled on boot)
    openstack-nova-scheduler:               active
    openstack-nova-conductor:               active
    ...

2. 检查与 servera 上 Nova 服务关联的组件是否正确运行。

    [root@servera ~]# openstack-service status nova
    MainPID=1215 Id=openstack-nova-api.service ActiveState=active
    MainPID=1194 Id=openstack-nova-cert.service ActiveState=active
    MainPID=1937 Id=openstack-nova-compute.service ActiveState=active
    MainPID=1193 Id=openstack-nova-conductor.service ActiveState=active
    MainPID=1195 Id=openstack-nova-consoleauth.service ActiveState=active
    MainPID=1196 Id=openstack-nova-novncproxy.service ActiveState=active
    MainPID=1211 Id=openstack-nova-scheduler.service ActiveState=active

3. 在 workstation 上，打开 Firefox，再导航到 servera.lab.example.com。使用 demouser 作为用户名，redhat 作为密码，登录 Horizon。

4. 转到计算，然后单击实例下的启动实例。使用 demovm 作为实例名称，m1.small 作为类别，选择从映像引导作为实例引导来源，demoimage 作为映像名称。在访问权限与安全性选项卡中，选择 demokeypair 作为密钥对。最后，在网络选项卡中，单击 demonet 网络上的加号按钮。完成后，单击启动。等待其状态为活动。

5. 完成后，从 Horizon 注销。


### 总结

在本章中，您可以学到：

*    PackStack 是用于 Red Hat OpenStack Platform 的应答文件型部署工具。它适合通过多种支持的架构部署概念验证 Red Hat OpenStack Platform 环境。
*    应答文件是可轻松自定义的配置文件，为自定义 Red Hat OpenStack Platform 提供的大部分服务和功能以进行测试提供了灵活性。
*    OSP-d 部署工具支持部署生产就绪的 Red Hat OpenStack Platform 环境。
*    openstack-status 和 openstack-service 命令支持 Red Hat OpenStack Platform 服务的状态报告。


