---
layout: post
title:  "管理 Keystone 身份服务(210-1)"
categories: Linux
tags: RHCA 210
---


### 验证 OpenStack 部署

###### 验证 overcloud 节点

在部署了 overcloud 节点后，为验证部署以供使用，建议连接 overcloud 节点并执行各种任务。

######### 连接 overcloud 节点

Red Hat OpenStack Platform Director 生成一些文件并将它们放置于用户主目录中，这些文件用于启动堆栈。名为 overcloudrc 的文件中包含通过 admin 用户访问时所需的环境变量；OpenStack CLI 工具要求控制器上具有面向外部的 keystone 端点，以便和 overcloud 交互。 

```
[stack@servera ~]$ cat ~/overcloudrc
export OS_NO_CACHE=True
export COMPUTE_API_VERSION=1.1
export OS_USERNAME=admin
export no_proxy=,172.25.250.100
export OS_TENANT_NAME=admin
export OS_CLOUDNAME=overcloud
export OS_AUTH_URL=http://172.25.250.10:5000/v2.0/
export NOVA_VERSION=1.1
export OS_PASSWORD=4RdFMaZGhhqGFagR2Kn99W8Fg
```

作为 overcloud 部署的一部分，每个 overcloud 节点中将注入一个公钥。可以通过 heat-admin 用户远程登录 overcloud 节点。文件 /home/stack/stackrc 包含用于访问 undercloud 的 admin 凭据，它指向 Director 节点上运行的 keystone。

```
[stack@servera ~]$ source ~/stackrc
[stack@servera ~]$ openstack server list
+--------------------------------------+-------------------------+
| ID                                   | Name                    |
+--------------------------------------+-------------------------+
| bac770c8-70c9-49c8-b571-052eee6aaba8 | overcloud-novacompute-0 |
| 22f6ca15-850a-4f8f-b54d-1c10a6bf35b4 | overcloud-controller-0  |
| 9611d326-d331-4032-9ac5-a53b9079a575 | overcloud-cephstorage-0 |
+--------------------------------------+-------------------------+
---------+-------------------------+
 Status  | Networks                |
---------+-------------------------+
 SHUTOFF | ctlplane=172.24.250.114 |
 SHUTOFF | ctlplane=172.24.250.115 |
 SHUTOFF | ctlplane=172.24.250.116 |
---------+-------------------------+
[stack@servera ~]$ ssh heat-admin@172.24.250.115
```

###### 验证 Ceph 存储节点

Red Hat OpenStack Platform Director 提供了相关的功能，可以在部署 overcloud 节点期间调配 Ceph 存储节点。Ceph 存储可以充当 Glance、Cinder 和 Nova 临时存储的后端。默认情况下，Ceph 存储节点不与其他 overcloud 节点一起配置。需要创建节点配置文件并在数据内省期间或通过手动标记进行分配，以识别可以成为 OSD 节点的主机。

Ceph 监控程序自动部署在控制器节点上，因此具有必要的 Ceph 配置文件，以及使用 cephx 进行身份验证所需的密钥环文件。

*    ceph -s 等命令可用于检查 Ceph 群集健康状态
*    ceph osd tree 命令可用于验证 OSD 是否作为群集的一部分运行
*    ceph osd lspools 命令则可用于列出 storage-environment.yaml 环境文件创建的池。此文件使用 Ceph 作为 Glance 映像，并且使用 Cinder 块服务、Nova 临时存储，以及虚拟磁盘。

######### 验证联网并创建额外的租户网络

用于部署 overcloud 节点默认 Heat 堆栈模板强制将 overcloud 节点的 eth0 或 nic1 桥接到名为 br-ex 的 Open vSwitch 网桥。这使得 overcloud 节点对外路由到 PXE/调配网络。利用 openstack overcloud deploy 命令在 overcloud 节点部署期间指定 --neutron-public-interface nic2 选项将有助于指向正确的外部接口。它可以通过以下命令验证：

```
[heat-admin@overcloud-controller-0 ~]$ sudo osv-vsctl show | grep -A10 br-ex
Bridge br-ex
        Port br-ex
            Interface br-ex
                type: internal
        Port "qg-c3515610-50"
            Interface "qg-c3515610-50"
                type: internal
        Port phy-br-ex
            Interface phy-br-ex
                type: patch
                options: {peer=int-br-ex}
        Port "eth1"
            Interface "eth1"
    Bridge br-int
        fail_mode: secure
        Port "tapd83daacc-41"
            tag: 1
            Interface "tapd83daacc-41"
                type: internal
        Port int-br-ex
            Interface int-br-ex
                type: patch
                options: {peer=phy-br-ex}
        Port "qr-9ded508d-46"
            tag: 1
            Interface "qr-9ded508d-46"
                type: internal
        Port br-int
            Interface br-int
                type: internal
        Port patch-tun
            Interface patch-tun
                type: patch
```

创建要用于 overcloud Nova 实例的外部网络，以便能够通过外部网络上的浮动 IP 连接这些实例。外部网络网桥 br-ex 通过 Heat 模板映射，因此不需要将它重新映射到提供商网络扩展。

```
[heat-admin@overcloud-controller-0 ~]$ sudo openstack-config --get /etc/neutron/l3_agent.ini DEFAULT external_network_bridge
br-ex
```

创建外部和内部网络，并为它们连接子网。IP 范围应当为外部网络面向外部的 CIDR 块。要使用浮动 IP，需要创建路由器，路由器的网关需要设置到外部网络，而且路由器接口应当连接有内部子网。

###### 通过启动实例验证 OpenStack overcloud

在部署 overcloud 后 undercloud 将依然存在。undercloud 变身为生命周期管理工具，以及用于修改当前 overcloud 配置的安装程序。

*    验证部署需要检查 Ironic node 状态。
*    可以使用典型的系统管理步骤来启动和连接，从而进行验证。
*    可以使用 Tempest 对实时 OpenStack 部署运行一组集成测试。

######### 部署后的 Ironic 节点状态

要在部署后检查 overcloud Ironic 节点状态，可运行ironic node-list命令。overcloud 实例现在与这些节点关联，openstack server list可显示这些节点以及可用于访问它们的相关 IP 地址。

```
[stack@director ~]$ ironic node-list
+--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
| UUID                                 | Name       | Instance UUID                        | Power State | Provisioning State | Maintenance |
+--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
| b3f20a88-d2ef-4410-ad8f-1ba8af5612d7 | controller | 72496eba-134e-4e47-9160-26d6e6a64a1f | power on    | active             | False       |
| 181a1107-1329-466d-9cf7-fd44c7b531d9 | compute1   | d86b204f-5bd9-4e43-be2c-39171b01c72f | power on    | active             | False       |
+--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+

[stack@director ~]$ openstack server list
+--------------------------------------+-------------------------+--------+-------------------------+
| ID                                   | Name                    | Status | Networks                |
+--------------------------------------+-------------------------+--------+-------------------------+
| d86b204f-5bd9-4e43-be2c-39171b01c72f | overcloud-novacompute-0 | ACTIVE | ctlplane=172.24.250.120 |
| 72496eba-134e-4e47-9160-26d6e6a64a1f | overcloud-controller-0  | ACTIVE | ctlplane=172.24.250.119 |
+--------------------------------------+-------------------------+--------+-------------------------+
```

overcloud 部署在 stack 用户的主目录下创建文件。这些文件包括：

*    tripleo-overcloud-passwords：含有用于访问这些 overcloud 节点上运行的 OpenStack 服务的所有密码。
*    overcloudrc：含有使用 admin 用户管理该 overcloud 时需要的环境变量。

######### 启动和连接实例来验证部署

要启动和连接实例，需要 overcloud 环境文件overcloudrc。这可以复制到 overcloud 环境之外的客户端，以执行典型的系统管理任务。这些任务可通过 Horizon Web UI 执行，或利用 OpenStack 统一 CLI 脚本化执行。

```
[student@demo ~]$ scp stack@director:~/overcloudrc /home/student/overcloudrc
```
    
需要在客户端上提供 overcloudrc 文件，以访问 overcloud 节点上运行的 OpenStack 服务，它为 admin 用户设置环境变量以用于访问 overcloud 节点。以下屏幕列出了 overcloud 节点上部署的类别。

```
[student@demo ~]$ source ~/overcloudrc
[student@demo ~]$ openstack flavor list
+--------------------------------------+-----------+-------+------+-----------+-------+-----------+
| ID                                   | Name      |   RAM | Disk | Ephemeral | VCPUs | Is Public |
+--------------------------------------+-----------+-------+------+-----------+-------+-----------+
| 1                                    | m1.tiny   |   512 |    1 |         0 |     1 | True      |
| 1e2bea88-b83d-4c9b-8f84-04454c9bf0fa | m1.demo   |   512 |   10 |         0 |     1 | True      |
| 2                                    | m1.small  |  2048 |   20 |         0 |     1 | True      |
| 3                                    | m1.medium |  4096 |   40 |         0 |     2 | True      |
| 4                                    | m1.large  |  8192 |   80 |         0 |     4 | True      |
| 5                                    | m1.xlarge | 16384 |  160 |         0 |     8 | True      |
+--------------------------------------+-----------+-------+------+-----------+-------+-----------+
```

######### Tempest 进行集成测试

Tempest 是用于验证实时 OpenStack 部署的一款工具，可通过运行一组集成测试来确保该部署达到 OpenStack API 的功能性和兼容性。Tempest 软件包中预装了多项测试，但系统管理员也可自行创建测试。这些测试可以是常规的烟雾或完整集成测试，也可以基于情景。后一类测试包含了一系列测试任务，利用依赖关系来测试涉及多项 Red Hat OpenStack Platform 服务的配置的工作流。Tempest 所需的软件包是 openstack-tempest-liberty，需要安装到 undercloud 上。

```
[stack@director ~]$ sudo yum install openstack-tempest-liberty
```
    

从 undercloud 节点运行 /usr/share/openstack-tempest-liberty/tools/configure-tempest-directory，以创建 tempest 目录。

```
[stack@director ~]$ mkdir test_dir
[stack@director ~]$ cd test_dir
[stack@director test_dir]$ /usr/share/openstack-tempest-liberty/tools/configure-tempest-directory
```

要使用当前的 overcloud 环境设置配置 Tempest，可运行 config_tempest.pyPython 脚本来创建 etc/tempest.conf。$OS_AUTH_URL 和 $OS_PASSWORD 都在 overcloudrc 文件中定义。

```
[stack@director test_dir]$ source ~/overcloudrc
[stack@director test_dir]$ ./tools/config_tempest.py --create --debug identity.uri $OS_AUTH_URL identity.admin_password $OS_PASSWORD
2016-02-02 12:14:19.421 14385 INFO tempest [-] Using tempest config file /etc/tempest/tempest.conf
```

可以对此文件进行编辑来满足相关的要求。本例中，应编辑 image_ref 和 image_ssh_user，以使用 small Glance 映像上传。将此值替换为 image_ssh_user 的 cloud-user，以及 image_ref 的 small 映像 UUID。要运行烟雾测试或情景测试，可使用 ~/test_dir/tools/run.test.sh 脚本。

```
[stack@director test_dir]$ ./tools/run-tests.sh '.*scenario'
[stack@director test_dir]$ ./tools/run-tests.sh '.*smoke'
...Output omitted...
{1} ...FlavorsV2TestJSON.test_get_flavor [0.196771s] ... ok
{1} ...FlavorsV2TestJSON.test_list_flavors [0.062779s] ... ok
{0} ...test_security_group_rules_create [0.541896s] ... ok
{0} ...test_security_group_rules_list [0.450489s] ... ok
{0} ...test_security_groups_create_list_delete [2.491846s] ... ok
...Output omitted...
```


### 练习：验证 OpenStack 部署

1. 从 workstation，以 stack 用户身份登录 director

```
[student@workstation ~]$ ssh stack@director
Last login:Wed Nov  9 18:13:11 2016 
from workstation.lab.example.com
[stack@director ~]$ 
```

2. 以 stack 用户身份在 director 上提供 ~/stackrc 环境文件，以允许管理员访问 overcloud。

```
[stack@director ~]$ source ~/stackrc
```

3. 检查 director 上运行的 OpenStack 服务的状态。使用命令 openstack-status 显示完整状态报告。

```
[stack@director ~]$ openstack-status
== Nova services ==
openstack-nova-api:                     active
openstack-nova-cert:                    active
openstack-nova-compute:                 active
openstack-nova-network:                 inactive  (disabled on boot)
openstack-nova-scheduler:               active
openstack-nova-conductor:               active
== Glance services ==
openstack-glance-api:                   active
openstack-glance-registry:              active

... Output omitted ...

== Nova instances ==
+--------------------------------------+-------------------------+--------+------------+-------------+------------------------+
| ID                                   | Name                    | Status | Task State | Power State | Networks               |
+--------------------------------------+-------------------------+--------+------------+-------------+------------------------+
| 9987851c-34d0-4714-bb1d-4fd620ad47e7 | overcloud-cephstorage-0 | ACTIVE | -          | Running     | ctlplane=172.25.250.22 |
| ae0ed4d7-2961-4f06-a7c8-9f2eb8c9c985 | overcloud-controller-0  | ACTIVE | -          | Running     | ctlplane=172.25.250.24 |
| 331b864f-ae8f-4d61-8ccf-170a2c734a0d | overcloud-novacompute-0 | ACTIVE | -          | Running     | ctlplane=172.25.250.23 |
+--------------------------------------+-------------------------+--------+------------+-------------+------------------------+
[stack@director ~]$ 
```

4. 或者，使用命令 openstack-service 显示 OpenStack 服务的单行状态报告。

例如，运行命令 openstack-service status | column -t，列出 director 上运行的所有 OpenStack 服务的状态。

```
[stack@director ~]$ openstack-service status | column -t
MainPID=8905   Id=neutron-dhcp-agent.service                    ActiveState=active
MainPID=8909   Id=neutron-openvswitch-agent.service             ActiveState=active
... Output omitted ...
MainPID=7747   Id=openstack-swift-object.service                ActiveState=active
MainPID=7754   Id=openstack-swift-proxy.service                 ActiveState=active
```

> 注意: 通过到 column -t 命令的管道运行 openstack-service 命令将显示表格式的输出。 

5. 此外，运行命令 openstack-service status service-name 将仅列出与 service-name 服务关联的服务。

例如，运行以下命令将仅显示与 glance 服务关联的一些服务的状态。

```
[stack@director ~]$ openstack-service status glance | column -t
MainPID=7700  Id=openstack-glance-api.service       ActiveState=active
MainPID=7708  Id=openstack-glance-registry.service  ActiveState=active
```

6. 有时可能需要重新启动所有服务、单个服务或一组相关服务。完成该任务有很多种方法，但是，一个简单的方法是使用命令 openstack-service restart。

    a. 以 root 用户身份运行 openstack-service action service 命令将可以重新启动所有任务或一组任务。

    例如，首先运行以下命令，列出 glance 服务的当前 PID。

```
    [stack@director ~]$ openstack-service status glance | column -t
    MainPID=7700  Id=openstack-glance-api.service       ActiveState=active
    MainPID=7708  Id=openstack-glance-registry.service  ActiveState=active
```

    b. 更改用户环境到 root 用户。

```
    [stack@director ~]$ sudo -i
    [root@director ~]# 
```

    c. 运行命令 openstack-service restart glance 以重新启动与 glance 关联的所有服务。请注意，这些命令将静默返回提示。命令完成后，退出 root 环境。

```
    [root@director ~]# openstack-service restart glance
    [root@director ~]# exit
```

    d. 要验证服务已重新启动，则再次运行命令 openstack-service status glance | column -t，并且验证 PID 已更改。

```
    [stack@director ~]$ openstack-service status glance | column -t
    MainPID=4917  Id=openstack-glance-api.service       ActiveState=active
    MainPID=4913  Id=openstack-glance-registry.service  ActiveState=active
```

7. 当管理员发现需要重新引导 director 时，可能会出现其他情况。在这种情况下，必须了解 director 可能需要几分钟才能重新初始化。在验证有效 overcloud 节点之前，可能需要等待足够长的时间，如 3 至 5 分钟。

    a. 首先提供 ~/stackrc 文件，然后列出可用的 overcloud 节点。

```
    [stack@director ~]$ source ~/stackrc
    [stack@director ~]$ openstack server list
    +--------------------------------------+-------------------------+--------+------------------------+
    | ID                                   | Name                    | Status | Networks               |
    +--------------------------------------+-------------------------+--------+------------------------+
    | ae0ed4d7-2961-4f06-a7c8-9f2eb8c9c985 | overcloud-controller-0  | ACTIVE | ctlplane=172.25.250.24 |
    | 331b864f-ae8f-4d61-8ccf-170a2c734a0d | overcloud-novacompute-0 | ACTIVE | ctlplane=172.25.250.23 |
    | 9987851c-34d0-4714-bb1d-4fd620ad47e7 | overcloud-cephstorage-0 | ACTIVE | ctlplane=172.25.250.22 |
    +--------------------------------------+-------------------------+--------+------------------------+
```

    b. 重新引导 director，提供 ~/stackrc 环境文件，然后验证 overcloud 节点可用。

```
    [stack@director ~]$ sudo systemctl reboot
    [stack@director ~]$ source ~/stackrc
    [stack@director ~]$ openstack server list
    +--------------------------------------+-------------------------+--------+------------------------+
    | ID                                   | Name                    | Status | Networks               |
    +--------------------------------------+-------------------------+--------+------------------------+
    | ae0ed4d7-2961-4f06-a7c8-9f2eb8c9c985 | overcloud-controller-0  | ACTIVE | ctlplane=172.25.250.24 |
    | 331b864f-ae8f-4d61-8ccf-170a2c734a0d | overcloud-novacompute-0 | ACTIVE | ctlplane=172.25.250.23 |
    | 9987851c-34d0-4714-bb1d-4fd620ad47e7 | overcloud-cephstorage-0 | ACTIVE | ctlplane=172.25.250.22 |
    +--------------------------------------+-------------------------+--------+------------------------+
```

8. 所有 overcloud 节点可用后，可以通过身份验证到各个节点并执行各种报告服务状态的命令，来单独验证各个节点。

要验证单个 overcloud 节点，请确保已经提供 ~/stackrc 环境文件，然后运行命令 openstack server list 以显示各个 overcloud 节点的IP 地址。

```
[stack@director ~]$ source ~/stackrc
[stack@director ~]$ openstack server list
+--------------------------------------+-------------------------+--------+------------------------+
| ID                                   | Name                    | Status | Networks               |
+--------------------------------------+-------------------------+--------+------------------------+
| ae0ed4d7-2961-4f06-a7c8-9f2eb8c9c985 | overcloud-controller-0  | ACTIVE | ctlplane=172.25.250.24 |
| 331b864f-ae8f-4d61-8ccf-170a2c734a0d | overcloud-novacompute-0 | ACTIVE | ctlplane=172.25.250.23 |
| 9987851c-34d0-4714-bb1d-4fd620ad47e7 | overcloud-cephstorage-0 | ACTIVE | ctlplane=172.25.250.22 |
+--------------------------------------+-------------------------+--------+------------------------+
```

    a. 使用 heat-admin 用户，身份验证到 overcloud-controller-0 并查看 Openstack 服务的状态。

```
    [stack@director ~]$ ssh heat-admin@172.25.250.24
    Last login: Wed Nov  9 22:02:11 2016 
    from director.lab.example.com
    [heat-admin@overcloud-controller-0 ~]$ sudo openstack-service status | column -t
    MainPID=1433  Id=openstack-swift-account-auditor.service ActiveState=active
    MainPID=1440  Id=openstack-swift-proxy.service           ActiveState=active
    ... Output omitted ...
    [heat-admin@overcloud-controller-0 ~]$ exit
    logout
    Connection to 172.25.250.24 closed.
    [stack@director ~]$ 
```

    b. 使用 heat-admin 用户，身份验证到 overcloud-novacompute-0 并查看 Openstack 服务的状态。

```
    [stack@director ~]$ ssh heat-admin@172.25.250.23
    Last login: Wed Nov  9 22:02:49 2016 
    from director.lab.example.com
    [heat-admin@overcloud-novacompute-0 ~]$ openstack-service status | column -t
    MainPID=1915  Id=neutron-openvswitch-agent.service     ActiveState=active
    ... Output omitted ...
    MainPID=1215  Id=openstack-ceilometer-compute.service  ActiveState=active
    MainPID=1917  Id=openstack-nova-compute.service        ActiveState=active
    [heat-admin@overcloud-novacompute-0 ~]$ exit
    logout
    Connection to 172.25.250.23 closed.
    [stack@director ~]$
```

    c. 使用 heat-admin 用户，身份验证到 overcloud-cephstorage-0 并查看 Openstack 服务的状态。

```
    [stack@director ~]$ ssh heat-admin@172.25.250.22
    Last login: Thu Nov 10 22:59:41 2016 
    from director.lab.example.com
    [heat-admin@overcloud-cephstorage-0 ~]$ sudo ceph -s
        cluster 2b6ad156-5280-11e6-82a2-52540001fa0a
         health HEALTH_OK
         monmap e1: 1 mons at {overcloud-controller-0=172.25.253.11:6789/0}
                election epoch 1, quorum 0 overcloud-controller-0
         osdmap e37: 3 osds: 3 up, 3 in
          pgmap v245: 160 pgs, 4 pools, 0 bytes data, 1 objects
                105 MB used, 58229 MB / 58334 MB avail
                     160 active+clean
    [heat-admin@overcloud-cephstorage-0 ~]$ exit
    logout
    Connection to 172.25.250.22 closed.
    [stack@director ~]$ 
```

9. 运行以下命令以验证 overcloud-controller-0 节点的 eth0 接口桥接到 openvswitch 外部访问桥 br-ex。

```
[stack@director ~]$ ssh heat-admin@172.25.250.24
[heat-admin@overcloud-controller-0 ~]$ sudo ovs-vsctl show
fe2d3ca3-428f-484e-9682-20a6d57ea4e1
... Output omitted ...

    Bridge br-ex
        Port "eth0"
            Interface "eth0"

... Output omitted ...
    ovs_version: "2.4.0"
[heat-admin@overcloud-controller-0 ~]$ exit
logout
Connection to 172.25.250.24 closed.
[stack@director ~]$ 
```

10. 运行命令 ironic node-list 也可以获得 overcloud 节点上的更多信息，如实例 ID、电源状态和维护级别。

```
[stack@director ~]$ source ~/stackrc
[stack@director ~]$ ironic node-list
+--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
| UUID                                 | Name       |               Instance UUID          | Power State | Provisioning State | Maintenance |
+--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
| 840c868a-ebc3-4477-a427-f3b4ba6da393 | controller | ae0ed4d7-2961-4f06-a7c8-9f2eb8c9c985 | power on    | active             | False       |
| fe281ce3-adc5-4907-9472-6e7520be3688 | compute1   | 331b864f-ae8f-4d61-8ccf-170a2c734a0d | power on    | active             | False       |
| e6a908e6-23ad-4510-9b61-3acd72aabec5 | ceph       | 9987851c-34d0-4714-bb1d-4fd620ad47e7 | power on    | active             | True        |
+--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
```

11. 运行命令 heat stack-list 将显示用于创建 overcloud 的堆栈的有用信息。

```
[stack@director ~]$ source ~/stackrc
[stack@director ~]$ heat stack-list
+--------------------------------------+------------+-----------------+---------------------+--------------+
| id                                   | stack_name | stack_status    | creation_time       | updated_time |
+--------------------------------------+------------+-----------------+---------------------+--------------+
| 499fb3c8-4037-43e6-9f48-c287bc41949c | overcloud  | CREATE_COMPLETE | 2016-07-25T15:55:15 | None         |
+--------------------------------------+------------+-----------------+---------------------+--------------+
```

    a. heat stack-show overcloud 命令可用于显示堆栈的深层详细信息。建议将命令输出到格式化选项，以更方便查看。
    
    注意: 通过 -S 选项使用 less 命令时，箭头键可用于上下左右滚动。完成后，使用“q”选项退出 less。

```
    [stack@director ~]$ source ~/stackrc
    [stack@director ~]$ heat stack-show overcloud | less -S
    +-----------------------+-------------------------------------------------------
    | Property              | Value
    +-----------------------+-------------------------------------------------------
    | capabilities          | []
    | creation_time         | 2016-07-25T15:55:15
    | description           | Deploy an OpenStack environment, consisting of several
    |                       | node types (roles), Controller, Compute, BlockStorage,
    |                       | SwiftStorage and CephStorage. The Storage roles enable
    |                       | independent scaling of the storage components, but the
    |                       | minimal deployment is one Controller and one Compute
    |                       | node.
    | disable_rollback      | True
    | id                    | 499fb3c8-4037-43e6-9f48-c287bc41949c

    ... Output omitted ...

    | parent                | None
    | stack_name            | overcloud
    | stack_owner           | admin
    | stack_status          | CREATE_COMPLETE
    | stack_status_reason   | Stack CREATE completed successfully
    | stack_user_project_id | 53e2044502be42ee953e8ed6463596df
    | tags                  | null
    | template_description  | Deploy an OpenStack environment, consisting of several
    |                       | node types (roles), Controller, Compute, BlockStorage,
    |                       | SwiftStorage and CephStorage. The Storage roles enable
    |                       | independent scaling of the storage components, but the
    |                       | minimal deployment is one Controller and one Compute
    |                       | node.
    | timeout_mins          | 240
    | updated_time          | None
    +-----------------------+-------------------------------------------------------
```

### 运行 OpenStack 统一命令行界面

Horizon 为云用户和管理员提供了一个环境，但一些任务需要更大的灵活性（如脚本功能）来支持操作。Red Hat OpenStack Platform 提供了一套 CLI 工具，实施所有通过 OpenStack 服务 API 提供的功能。

在 Red Hat OpenStack Platform 的较早版本中，CLI 工具集由一套命令组成，每一命令对应一项 OpenStack 服务（例如， keystone 用于 Keystone 身份服务）。OpenStack CLI 采用了一种新方法，即基于 openstack 命令的统一 CLI，其宗旨是简化 Red Hat OpenStack Platform 环境中的操作。

###### Keystone 凭据 

与 Horizon 要求用户名和密码一样，OpenStack CLI 工具集也要求用户正确验证身份后才能被授权使用 Red Hat OpenStack Platform 环境中的各种服务。若要成功通过身份验证，用户需要至少指定下表中所述的参数。它们使用环境变量或作为 CLI 上 openstack 命令的标志来指定。

Keystone 身份验证参数

*    参数 	详情 	环境变量 	openstack 命令标志
*    用户 	用户的用户名 	OS_USERNAME 	--os-username
*    密码 	用户的密码 	OS_PASSWORD 	--os-password
*    项目 	用户所属的项目 	OS_PROJECT_NAME 	--os-project-name
*    Keystone 端点 	用于访问 Keystone 的 URL 	OS_AUTH_URL 	--os-auth-url

OS_AUTH_URL 环境变量中包含的端点对应于 Keystone 服务的公共端点，其默认绑定到 5000 端口。另外还有两种端点，即内部端点（由 OpenStack 服务用于执行向 Keystone 的请求）和管理端点（允许修改用户和租户，而公共和内部 API 则不）。

如果通过环境变量指定这些身份验证参数，最佳的做法是为各组用户身份验证参数创建一个 keystonerc_username 文件。例如，如果名为 testuser 的用户属于 mytestproject租户，密码为 redhat，并且其 Keystone 端点位于 demo 计算机，则该用户将关联有下列 keystonerc 文件。

```
export OS_USERNAME=testuser
export OS_PASSWORD=redhat
export OS_PROJECT_NAME=mytestprojectservera
export OS_AUTH_URL=demo:5000/v2.0
```

身份验证参数也可以在执行 openstack 命令时指定，这时要借助前面表中详述的相关标志。对于上一示例，如果要列出 OpenStack 映像服务 Glance 中可用的映像，可运行以下命令：

```
[student@demo ~]$ openstack --os-username testuser --os-password redhat --os-project-name mytestproject --os-auth-url demo:5000/v2.0 image list
```

> 注意: 默认情况下，undercloud 在 /root/ 目录下创建 keystonerc_admin 文件，该文件中包含所部署 Red Hat OpenStack Platform 环境的管理员用户的身份验证参数。

###### 使用 Keystone 验证身份和授权

以上参数供 Keystone 用于针对不同的 OpenStack 服务验证用户身份，并授权这些资源的访问权限。在后端，Keystone 使用基于 PKI 的基础架构，它同时受 Keystone 和 OpenStack 服务支持，而这是基于令牌的。

令牌是一个文本字符串，用于识别用户的身份。令牌通常在有限的时间内有效，之后令牌将被吊销并且不再有效。Keystone 维护一个包含所有已吊销令牌的吊销列表，也供 OpenStack 服务使用。

每当用户使用用户名、密码和 Keystone 服务的端点 URL发出请求，Keystone 验证该用户对不同服务的访问权限。然后，返回一个由 Keystone 签名的令牌，以便 OpenStack 服务可以使用与 Keystone 所用相同的 PKI 型基础架构来验证该令牌。这将尽可能减少必要的 Keystone 查询数量。Keystone 实际上是一种证书颁发机构 (CA)，使用密钥和证书为令牌签名，然后将令牌返回给用户。

以下示意图描述了身份验证过程。在向 OpenStack 发出请求时，用户将附上用户名、密码和项目，或者通过参数直接包含在命令中，或者使用 OS_* 环境变量。Keystone 检查用户的授权信息，并检索对相关服务的访问权限，然后生成令牌、为其签名，再发回给用户。 

令牌被用户接收后，它将与请求一道发送到 OpenStack 服务。服务检查该令牌是否有效，以及是否使用密钥、证书和吊销列表（由 OpenStack 服务通过 Keystone API 检索而得）。令牌将被验证签名和有效日期，还会被检查是否尚未吊销。整个过程由服务自行执行，无需每一验证都必须直接请求 Keystone。如果令牌有效，服务将返回请求的结果；否则返回错误。 

###### 使用 admin 令牌恢复 Keystone

虽然 Director 提供了包含 admin 用户在内的少许用户配置，供您管理 Red Hat OpenStack Platform 环境，但有时会出现无法使用管理凭据访问此环境的情形。为了能从这种状况中恢复，Keystone 配置提供了 admin 令牌，它提供了对环境的管理访问权限。

此令牌是一个文本字符串，作为 admin_token 参数在 /etc/keystone/keystone.conf Keystone 配置文件中定义。它可通过设置 OS_TOKEN 环境变量来使用。这还要求使用 OS_URL 变量，以便能指定要使用的 Keystone 端点。admin 令牌也可通过 openstack 命令的 --os-token 和 --os-url 参数来使用。

###### 统一 CLI

统一 CLI 旨在提供一种更加便捷的方式，让用户通过使用唯一命令 openstack 与各种 Red Hat OpenStack Platform 服务交互。此命令提供一系列用于与 Red Hat OpenStack Platform 服务交互的子命令，也提供一些自定义其输出的标志。下表显示了子命令的子集及其说明。

```
[student@demo ~]$ openstack flags subcommand options
```

统一 CLI 子命令

*    子命令 	    详情
*    user 	    用户管理
*    project 	项目管理
*    server 	实例管理
*    image 	    映像管理
*    volume 	块存储管理
*    network 	网络管理

如需有关通过 openstack 命令提供的子命令及其选项的帮助，您可以按照以下格式使用 help 子命令来获取。

```
[student@demo ~]$ openstack help subcommand option
```


### 练习：运行 OpenStack 统一 CLI

1. 从 workstation 检查 /home/student/overcloudrc 文件中的管理员凭据。管理员用户的用户名是 admin，其密码是一个随机字符串。用于执行所有管理任务的租户是 admin，Keystone 端点 IP 地址则对应于在其上运行 Keystone 服务的控制器节点。

```
[student@workstation ~]$ cat overcloudrc
...Output omitted...
export OS_USERNAME=admin
export OS_PASSWORD=wWG6MGCZGrzuQZ8u382vvMgTz
export OS_AUTH_URL=http://192.168.0.11:5000/v2.0
...Output omitted...
export OS_TENANT_NAME=admin
...Output omitted...
```

2. 提供overcloudrc文件以获取 admin 权限。

```
[student@workstation ~]$ source overcloudrc
```

3. 检查环境中已正确设置了 OS_* 环境变量。OS_USERNAME 将包含 admin 用户名，其对应于 OpenStack 管理超级用户；OS_PASSWORD 将包含随机字符串，因为这是 undercloud 为 admin 超级用户配置的密码；OS_TENANT_NAME 将包含 admin，因为这是支持管理任务的租户/项目；而 OS_AUTH_URL 将是控制器节点的公共 IP 地址，并将 5000 端口用作公共端点，因为 Keystone 服务正在该计算机上运行。

```
[student@workstation ~]$ env | grep OS_
...Output omitted...
OS_PASSWORD=wWG6MGCZGrzuQZ8u382vvMgTz
OS_AUTH_URL=http://192.168.0.11:5000/v2.0
OS_USERNAME=admin
OS_TENANT_NAME=admin
```

4. 将overcloudrc文件用作模板，在/home/student目录中为实验脚本创建的 myuser 用户创建新的overcloudrc凭据文件，并将它命名为 overcloudrc_myuser。更改/home/student/overcloudrc_myuser文件中的以下变量。

统一 CLI 子命令

*    环境变量	新值
*    OS_USERNAME	myuser
*    OS_PASSWORD	红帽
*    OS_TENANT_NAME	myproject

```
[student@workstation ~]$ cat overcloudrc_myuser
...Output omitted...
export OS_USERNAME=myuser
export OS_PASSWORD=redhat
export OS_AUTH_URL=http://192.168.0.11:5000/v2.0
...Output omitted...
export OS_TENANT_NAME=myproject
...Output omitted...
```

5. 提供 overcloudrc_myuser 文件，以获取 Red Hat OpenStack Platform 环境中的 myuser 用户权限。这将启用之前配置并与 myuser 用户相关的所有 OS_* 环境变量值。

```
[student@workstation ~]$ source overcloudrc_myuser
```

6. 运行 openstack help image 命令，获取与管理映像相关的信息。

```
[student@workstation ~]$ openstack help image
Command "image" matches:
  image add project
  image create
  image delete
  image list
  image remove project
  image save
  image set
  image show
```

7. 使用image list子命令，检查 myimg 和 mywebimg 映像是否可用。

```
[student@workstation ~]$ openstack image list
+--------------------------------------+---------+
| ID                                   | Name    |
+--------------------------------------+---------+
| 666920e1-5274-4ca4-a831-247688d4d3e6 | mywebimg|
| 4d8950eb-5e76-4ca4-a8d6-9c7f8dddd2e5 | myimg   |
+--------------------------------------+---------+
```

8. 将overcloudrc文件用作模板，在/home/student目录中为实验脚本创建的 myseconduser 用户创建新的overcloudrc凭据文件，并将它命名为 overcloudrc_myseconduser。更改/home/student/overcloudrc_myseconduser文件中的以下变量。

统一 CLI 子命令

*    环境变量	新值
*    OS_USERNAME	myseconduser
*    OS_PASSWORD	红帽
*    OS_TENANT_NAME	mysecondproject

```
[student@workstation ~]$ cat overcloudrc_myseconduser
...Output omitted...
export OS_USERNAME=myseconduser
export OS_PASSWORD=redhat
export OS_AUTH_URL=http://192.168.0.11:5000/v2.0
...Output omitted...
export OS_TENANT_NAME=mysecondproject
...Output omitted...
```

9. 提供 overcloudrc_myseconduser 文件，以获取 Red Hat OpenStack Platform 环境中的 myseconduser 用户权限。

```
[student@workstation ~]$ source overcloudrc_myseconduser
```

10. 使用 子命令，再次检查 myimg 和 mywebimg映像是否可用。image list myproject 下将不列出任何映像。

```
[student@workstation ~]$ openstack image list
```

11. 提供 overcloudrc_myuser 文件，以获取 Red Hat OpenStack Platform 环境中的 myuser 用户权限。

```
[student@workstation ~]$ source overcloudrc_myuser
```

12. 使用image delete子命令，删除 mywebimg 映像。

```
[student@workstation ~]$ openstack image delete mywebimg
```

13. 运行 openstack image list 命令，确保 mywebimg 已正确移除。

```
[student@workstation ~]$ openstack image list
+--------------------------------------+---------+
| ID                                   | Name    |
+--------------------------------------+---------+
| 4d8950eb-5e76-4ca4-a8d6-9c7f8dddd2e5 | myimg   |
+--------------------------------------+---------+
```

14. 从 workstation，以 student 用户身份打开一个新终端，再以 stack 用户身份使用 SSH 连接 director。提供 stackrc 文件，再运行 neutron port-list 来检索 overcloud controller 节点的公共虚拟 IP 地址。

```
[student@workstation ~]$ ssh stack@director
[stack@director ~]$ source stackrc
[stack@director ~]$ neutron port-list -c name -c fixed_ips
+-------------------------------+-------------------------------------------------------+
| name                          | fixed_ips                                             |
+-------------------------------+-------------------------------------------------------+
...Output omitted...
| public_virtual_ip             | {"subnet_id": "06a132dc-cb79-423f-b139-0e36601d15d7",
                                   "ip_address": "192.168.0.11"}                        |
+-------------------------------+-------------------------------------------------------+

[stack@director ~]$ exit
[student@workstation ~]$ 
```

15. 从 workstation，使用用户名 heat-admin 通过 SSH 连接 overcloud-controller-0 节点。将 IP 替换为上一命令返回的 IP。

检索位于 /etc/keystone/keystone.conf 配置文件中的 admin_token 的值，完成后从控制器退出。

```
[student@workstation ~]$ ssh -i overcloud.pem heat-admin@192.168.0.11
[heat-admin@overcloud-controller-0 ~]$ sudo grep admin_token /etc/keystone/keystone.conf
...Output omitted...
admin_token = F9sQ8YNzVfJjR9DNCBKq3pAex
...Output omitted...
[heat-admin@controller ~]$ logout
```

16. 从 workstation 提供 overcloudrc 凭据文件，再运行 openstack endpoint show keystone 命令来检索 Keystone 服务的 adminurl。

```
[student@workstation ~]$ source overcloudrc
[student@workstation ~]$ openstack endpoint show keystone
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| adminurl     | http://172.25.250.21:35357/v2.0  |
| enabled      | True                             |
| id           | 197693ac69bd4b8eac1f32778f8d0be2 |
| internalurl  | http://172.25.251.11:5000/v2.0   |
| publicurl    | http://192.168.0.11:5000/v2.0    |
| region       | regionOne                        |
| service_id   | f02af8440d6c4ec28d223955008dc224 |
| service_name | keystone                         |
| service_type | identity                         |
+--------------+----------------------------------+
```

17. 在 workstation 上，使用 admin 令牌和 Keystone 的 admin 端点作为参数，列出 Keystone 中配置的当前用户。

```
[student@workstation ~]$ openstack --os-token F9sQ8YNzVfJjR9DNCBKq3pAex  --os-url http://172.25.250.21:35357/v2.0/ user list
+----------------------------------+--------------+
| ID                               | Name         |
+----------------------------------+--------------+
| 329f2371a6924716b8a4e6f4a8e7310a | admin        |
| 5532142d6b59415da5adc2dac91b65f3 | ceilometer   |
| 61d799d689ac4e559801ed7326320d03 | swift        |
| 6cd243fc15c44f92baa7f6cd40dda0fe | myseconduser |
| 861d93de768a4800b17e2c4a4ce13e17 | nova         |
| c664517708a94583975a77dfa304c477 | neutron      |
| c68c8ef26291477ba76edf3c70f7d829 | cinder       |
| cecd7dcdac3844a7815a00efe0331df7 | myuser       |
| d0cac861ecb5406c99ee9629ef35518e | glance       |
| d4a57c4eafd74b5188fd068b8d9a0c5c | heat         |
| e1f1c6a9297946839666408744d14000 | cinderv2     |
+----------------------------------+--------------+
```


### 管理 OpenStack 项目

在 Red Hat OpenStack Platform 环境中，用户将分组到项目中，所以多个用户可以是同一项目的成员，一个用户也可以成为多个项目的成员。这些项目可以映射到用户所属的组织。项目通常分配有一些资源，如存储、计算和网络资源等，它们供该项目所关联的用户使用。借助 project 子命令，openstack命令提供项目管理功能，具体如下表中所述。

使用openstack命令管理项目

*    子命令 	            详情
*    project create 	创建新项目
*    project delete 	删除项目
*    project list 	    列出项目
*    project show 	    显示项目详细信息
*    project set 	    设置项目属性（如，启用/禁用）

> 注意: 在以前的 Red Hat OpenStack Platform 版本中，项目曾被称为租户，所以命令和文档中应该也会提到租户。 

默认情况下，项目具有关联的资源，它们通过配额加以限制。这些配额由不同的 Red Hat OpenStack Platform 服务在它们管理的资源中实施。例如，计算资源将由 Nova 服务管理，并且具有多种可配置的配额与之关联，其中包括实例数量（实例数 配额）、核心数（核心数 配额）和内存（ram 配额）。

所有 Red Hat OpenStack Platform 服务在部署时都对这些配额使用默认值，但可以在全局范围或根据项目修改这些默认值，让管理员能够控制项目及其用户可以使用的资源数量。openstack 命令目前不提供配额管理功能，但它们依然可以通过旧版 OpenStack 服务 CLI 进行管理，例如：nova 命令可以管理计算配额，cinder 命令可管理块存储配额，而 neutron 命令则可管理网络相关的配额。举例而言，nova 命令包含了用于管理计算相关配额的多个子命令，具体如下表所述。

使用 nova 命令管理项目配额

*    子命令 	详情
*    quota-show 	列出项目的配额
*    quota-update 	更新项目的配额
*    quota-delete 	删除项目的配额
*    quota-defaults 	列出项目的默认配额

> 注意: 关于nova quota-*命令的其他信息可通过运行nova help命令来了解。 

###### 演示：管理 OpenStack 项目

1. 从 workstation 以 student 用户身份打开一个新终端，再提供 overcloudrc 文件，以获取 admin 的权限。这将启用所有 OS_* 环境变量，其中至少包含管理员凭据和 Keystone 端点。

```
[student@workstation ~]$ source overcloudrc
[student@workstation ~]$ 
```

2. 使用 project create 子命令，创建一个名为 demoproject 的新项目。

```
[student@workstation ~]$ openstack project create demoproject
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | None                             |
| enabled     | True                             |
| id          | 8399d7c7ef664158abf697b6591cf258 |
| name        | demoproject                      |
+-------------+----------------------------------+
```

2. 使用project show子命令，检查 demoproject 项目的详细信息。

```
[student@workstation ~]$ openstack project show demoproject
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | None                             |
| enabled     | True                             |
| id          | 8399d7c7ef664158abf697b6591cf258 |
| name        | demoproject                      |
+-------------+----------------------------------+
```

3. 列出 Red Hat OpenStack Platform 环境中当前可用的项目。新项目 demoproject 应当会列在其中。

```
[student@workstation ~]$ openstack project list
+----------------------------------+-------------+
| ID                               | Name        |
+----------------------------------+-------------+
...Output omitted...
| 8399d7c7ef664158abf697b6591cf258 | demoproject |
+----------------------------------+-------------+
```

4. 使用 help 选项，检查 nova 命令可用于配额管理的选项。此处使用旧版 Nova CLI，因为 openstack 命令中缺少配额管理支持。

```
[student@workstation ~]$ nova help | grep quota
quota-class-show            List the quotas for a quota class.
quota-class-update          Update the quotas for a quota class.
quota-defaults              List the default quotas for a tenant.
quota-delete                Delete quota for a tenant/user so their quota
quota-show                  List the quotas for a tenant/user.
quota-update                Update the quotas for a tenant/user.
```

5. 使用quota-show选项，获取关于如何使用nova命令显示 demoproject 项目当前配额的信息。

```
[student@workstation ~]$ nova help quota-show
usage: nova quota-show [--tenant <tenant-id>] [--user <user-id>]

List the quotas for a tenant/user.

Optional arguments:
  --tenant <tenant-id>  ID of tenant to list the quotas for.
  --user <user-id>      ID of user to list the quotas for.
```

6. 使用 openstack project list 命令，检索 demoproject 项目的 UUID。

```
[student@workstation ~]$ openstack project list
+----------------------------------+------------+
| ID                               | Name       |
+----------------------------------+------------+
...Output omitted...
| 8399d7c7ef664158abf697b6591cf258 | demoproject |
...Output omitted...
+----------------------------------+------------+
```

7. 显示 demoproject 项目当前的配额。将 UUID 替换为上一命令返回的 UUID。

```
[student@workstation ~]$ nova quota-show --tenant 8399d7c7ef664158abf697b6591cf258
+-----------------------------+-------+
| Quota                       | Limit |
+-----------------------------+-------+
| instances                   | 10    |
| cores                       | 20    |
| ram                         | 51200 |
| floating_ips                | 10    |
| fixed_ips                   | -1    |
| metadata_items              | 128   |
| injected_files              | 5     |
| injected_file_content_bytes | 10240 |
| injected_file_path_bytes    | 255   |
| key_pairs                   | 100   |
| security_groups             | 10    |
| security_group_rules        | 20    |
| server_groups               | 10    |
| server_group_members        | 10    |
+-----------------------------+-------+
```

8. 使用nova命令及quota-update选项，将 demoproject 项目的核心数配额更新为 30。

```
[student@workstation ~]$ nova quota-update --cores 30 8399d7c7ef664158abf697b6591cf258
```

9. 再次显示 demoproject 项目当前的配额。核心数配额现在应当显示为 30。

```
[student@workstation ~]$ nova quota-show --tenant 8399d7c7ef664158abf697b6591cf258
+-----------------------------+-------+
| Quota                       | Limit |
+-----------------------------+-------+
| instances                   | 10    |
| cores                       | 30    |
| ram                         | 51200 |
| floating_ips                | 10    |
| fixed_ips                   | -1    |
| metadata_items              | 128   |
| injected_files              | 5     |
| injected_file_content_bytes | 10240 |
| injected_file_path_bytes    | 255   |
| key_pairs                   | 100   |
| security_groups             | 10    |
| security_group_rules        | 20    |
| server_groups               | 10    |
| server_group_members        | 10    |
+-----------------------------+-------+
```

10. 使用project delete子命令，移除 oldproject 项目。

```
[student@workstation ~]$ openstack project delete oldproject
````

11. 运行 openstack project list 命令，确保 oldproject 已正确移除。

```
[student@workstation ~]$ openstack project list
+----------------------------------+-------------+
| ID                               | Name        |
+----------------------------------+-------------+
| 1186b7a1f5b74edaa625ce90da129002 | service     |
| 4aa0b02f1f914a23b9c4dc671e6cc181 | openstack   |
| 578a28273c3743aab17b4b43bf0d4d18 | demoproject |
| aa21148ad44447338e00d4d5f3df4aac | services    |
| be6177a7063d4f9daa3571db7e6c1bc3 | admin       |
+----------------------------------+-------------+
```


### 练习：管理 OpenStack 项目

1. 从 workstation 提供 overcloudrc 文件，以获取 admin 权限。这将启用所有 OS_* 环境变量，其中至少包含管理员凭据和 Keystone 端点。

```
[student@workstation ~]$ source overcloudrc
```

2. 要显示关于使用 openstack 命令管理项目的信息，可将 help 与 project 选项搭配使用。

```
[student@workstation ~]$ openstack help project
Command "project" matches:
  project create
  project delete
  project list
  project set
  project show
  project usage list
```

3. 要显示关于使用openstack命令创建新项目的信息，可将help与project create选项搭配使用。

```
[student@workstation ~]$ openstack help project create
usage: openstack project create [-h]
                                [-f {html,json,json,shell,table,value,yaml,yaml}]
                                [-c COLUMN] [--max-width <integer>]
                                [--noindent] [--prefix PREFIX]
                                [--description <description>]
                                [--enable | --disable]
                                [--property <key=value>] [--or-show]
                                <project-name>

Create new project

positional arguments:
  <project-name>        New project name
...Output omitted...
```

4. 参考前面的帮助输出，新建一个项目 testproject。

```
[student@workstation ~]$ openstack project create testproject
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | None                             |
| enabled     | True                             |
| id          | 499a1ac01c9344f283aea6405d631686 |
| name        | testproject                      |
+-------------+----------------------------------+
```

5. 要获取关于使用openstack命令显示项目详情的信息，可将help与project show选项搭配使用。

```
[student@workstation ~]$ openstack help project show
usage: openstack project show [-h]
                              [-f {html,json,json,shell,table,value,yaml,yaml}]
                              [-c COLUMN] [--max-width <integer>]
                              [--noindent] [--prefix PREFIX]
                              [--description <description>]
                              [--enable | --disable]
                              [--property <key=value>] [--or-show]
                              <project-name>

Display project details

positional arguments:
  <project>             Project to display (name or ID)
...Output omitted...
```

6. 使用前面的帮助输出，显示新项目 testproject 的详细信息。

```
[student@workstation ~]$ openstack project show testproject
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | None                             |
| enabled     | True                             |
| id          | 499a1ac01c9344f283aea6405d631686 |
| name        | testproject                      |
+-------------+----------------------------------+
```

7. 创建 secondproject 项目。

```
[student@workstation ~]$ openstack project create secondproject
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | None                             |
| enabled     | True                             |
| id          | 3abbd17b8f4445639eb1fdb198bd49b1 |
| name        | secondproject                    |
+-------------+----------------------------------+
```

8. 要获取关于使用nova命令显示项目的当前配额的信息，可将help与quota-show选项搭配使用。

```
[student@workstation ~]$ nova help quota-show
usage: nova quota-show [--tenant <tenant-id>] [--user <user-id>]

List the quotas for a tenant/user.

Optional arguments:
  --tenant <tenant-id>  ID of tenant to list the quotas for.
  --user <user-id>      ID of user to list the quotas for.
```

9. 在能够查看 secondproject 项目的配额之前，需要先使用 openstack project show secondproject 命令检索其项目 ID。

```
[student@workstation ~]$ openstack project show secondproject
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
...Output omitted...
| id          | 499a1ac01c9344f283aea6405d631686 |
| name        | secondproject                    |
+-------------+----------------------------------+
```

10. 使用 nova quota-show --tenant 499a1ac01c9344f283aea6405d631686 命令，检索为 secondproject 设置的配额。将项目 ID 替换为上一命令返回的 ID。

```
[student@workstation ~]$ nova quota-show --tenant 499a1ac01c9344f283aea6405d631686
+-----------------------------+-------+
| Quota                       | Limit |
+-----------------------------+-------+
| instances                   | 10    |
| cores                       | 20    |
| ram                         | 51200 |
| floating_ips                | 10    |
| fixed_ips                   | -1    |
| metadata_items              | 128   |
| injected_files              | 5     |
| injected_file_content_bytes | 10240 |
| injected_file_path_bytes    | 255   |
| key_pairs                   | 100   |
| security_groups             | 10    |
| security_group_rules        | 20    |
| server_groups               | 10    |
| server_group_members        | 10    |
+-----------------------------+-------+
```

11. 使用 nova quota-update --cores 10 499a1ac01c9344f283aea6405d631686 命令，将 secondproject 的核心数配额从默认值 20 更新为 10。该命令不会产生任何输出。

```
[student@workstation ~]$ nova quota-update --cores 10 499a1ac01c9344f283aea6405d631686
```

12. 使用nova quota-show 499a1ac01c9344f283aea6405d631686命令验证 secondproject 项目的配额；确保核心数配额已设置为 10。

```
[student@workstation ~]$ nova quota-show --tenant 499a1ac01c9344f283aea6405d631686
+-----------------------------+-------+
| Quota                       | Limit |
+-----------------------------+-------+
| instances                   | 10    |
| cores                       | 10    |
...Output omitted...
```

13. 使用project delete子命令，移除 oldproject 项目。

```
[student@workstation ~]$ openstack project delete oldproject
```

### 管理 OpenStack 用户和角色

Keystone 要求 Red Hat OpenStack Platform 环境用户通过它进行身份验证，然后才能与其余的服务交互。需要新的 Red Hat OpenStack Platform 用户帐户时，管理员必须创建此帐户并提供适当的凭据。它们包括用户名、密码，以及该用户的资源要部署到其中的项目。这些凭据可作为环境变量 (OS_*) 应用，或作为参数传递到 openstack 命令。借助 openstack 子命令，user 命令提供用户管理功能，具体如下表中所述。

使用 openstack 命令管理用户

*    子命令 	            详情
*    user create 	    创建用户
*    user delete 	    删除用户
*    user list 	        列出用户
*    user show 	        显示用户详细信息
*    user set 	        设置用户属性（如密码）
*    user role list 	列出用户角色分配

###### 用户角色

Red Hat OpenStack Platform 使用基于角色的安全模型。每一角色关联有对一个或多个项目的特定访问权限。默认情况下，undercloud 部署两个角色，它们目前受到 Red Hat OpenStack Platform 服务的支持：_member_，它为关联了此角色的用户提供非管理权限；以及 admin，它为该用户启用管理权限。角色在项目中分配到用户，所以一个角色在不同的项目中可以具有不同的角色。

> 注意: Red Hat OpenStack Platform 超级用户 admin 关联了 admin 项目中的 admin 角色。

借助 role 子命令，openstack 命令提供角色管理功能，具体如下表中所述。

*    子命令 	详情
*    role create 	创建角色
*    role delete 	删除角色
*    role list 	列出角色
*    role show 	显示角色详细信息
*    role add 	添加用户在项目中的角色
*    role remove 	移除用户在项目中的角色

###### 创建自定义角色

Red Hat OpenStack Platform 中提供的两个默认角色支持项目所需的基本访问权限配置；其中，一个用户管理项目本身，其他用户则使用与项目关联的资源。可能会有企业需要超越这种基本方案的访问要求。 Red Hat OpenStack Platform 目前没有相关的 API 支持，用于修改与项目内角色关联的访问权限。

此访问权限由各项 Red Hat OpenStack Platform 服务通过位于该服务配置目录中的 policy.json 文件进行管理（例如，Nova 使用策略文件 /etc/nova/policy.json）。向服务的 API 发出请求时，将检查相关的 policy.json 文件来决定该请求是否有效。

> 注意: policy.json 文件的更新不要求您重新启动相关的服务。

服务的policy.json文件包含一条或多条策略规则，它们与可执行的操作以及可访问的资源关联。每条策略规则包含操作或资源名称，称为目标，它决定提供哪一对象或操作的访问权限，并映射到该服务的其中一个 API 调用。规则决定哪些用户被授予访问权限。

> 注意: policy.json 文件中定义的默认策略允许 admin 角色访问其他项目资源。

下表包含了一些可用于 Nova 服务策略的规则示例。compute:get_all 目标用于限制用户列出项目中的实例。

*    策略 	功能
*    "compute:get_all" : "" 	所有用户可以列出所有项目中的实例。
*    "compute:get_all" : "!" 	包括 admin 角色在内的任何用户可以列出所有项目中的实例。
*    "compute:get_all" : "role:oper" 	只有 oper 角色的用户可以列出所有项目中的实例。
*    "compute:get_all" : "not role:oper" 	除 oper 角色外的所有用户可以列出所有项目中的实例。


### 练习：管理 OpenStack 用户和角色

1. 从 workstation 提供 overcloudrc 文件，以获取 Red Hat OpenStack Platform 环境中的管理员权限。这将启用所有 OS_* 环境变量，其中至少包含管理员凭据和 Keystone 端点。

```
[student@workstation ~]$ source overcloudrc
```

2. 使用 user 选项运行 help 命令，显示关于使用 openstack 命令管理用户的信息。

```
[student@workstation ~]$ openstack help user
Command "user" matches:
  user create
  user delete
  user list
  user role list
  user set
  user show
```

3. 使用user create选项运行help命令，显示关于使用openstack命令创建新用户的信息。

```
[student@workstation ~]$ openstack help user create
usage: openstack user create [-h]
                             [-f {html,json,json,shell,table,value,yaml,yaml}]
                             [-c COLUMN] [--max-width <integer>]
                             [--prefix PREFIX] [--project <project>]
                             [--password <password>] [--password-prompt]
                             [--email <email-address>] [--enable | --disable]
                             [--or-show]
                             <name>

Create new user

positional arguments:
  <name>                New user name
...Output omitted...
```

4. 使用 user create 子命令，在 project1 中创建 user1 用户，密码设为 redhat。

```
[student@workstation ~]$ openstack user create --project project1 --password redhat user1
+------------+----------------------------------+
| Field      | Value                            |
+------------+----------------------------------+
| email      | None                             |
| enabled    | True                             |
| id         | 90d249b9c7c74064ac61ff72439b4c44 |
| name       | user1                            |
| project_id | 499a1ac01c9344f283aea6405d631686 |
| username   | user1                            |
+------------+----------------------------------+
```

5. 使用 user create子命令，在 project1 中创建 adminuser 用户，密码设为 redhat。

```
[student@workstation ~]$ openstack user create --project project1 --password redhat adminuser
+------------+----------------------------------+
| Field      | Value                            |
+------------+----------------------------------+
| email      | None                             |
| enabled    | True                             |
| id         | 90d249b9c7c74064ac61ff72439b4c44 |
| name       | adminuser                        |
| project_id | 283aea6405d631686499a1ac01c9344f |
| username   | adminuser                        |
+------------+----------------------------------+
```

6. 使用 role 选项运行 help 命令，显示关于使用 openstack 命令管理角色的信息。

```
[student@workstation ~]$ openstack help role
Command "role" matches:
  role add
  role create
  role delete
  role list
  role remove
  role show
```

7. 使用 role add 选项运行 help，显示关于使用 openstack 命令将角色添加给项目内的用户的信息。

```
[student@workstation ~]$ openstack help role add
usage: openstack role add [-h]
                          [-f {html,json,json,shell,table,value,yaml,yaml}]
                          [-c COLUMN] [--max-width <integer>] [--noindent]
                          [--prefix PREFIX] --project <project> --user <user>
                          <role>

Add role to project:user

positional arguments:
  <role>                Role to add to <project>:<user> (name or ID)
...Output omitted...
```

8. 使用openstack role list命令，列出当前可用的角色。

```
[student@workstation ~]$ openstack role list
+----------------------------------+-----------------+
| ID                               | Name            |
+----------------------------------+-----------------+
| 01532ab716ad40adb741a7bb5a8bd33c | heat_stack_user |
| 02a10ed09ad04cfbbd75677a7cbe5876 | laboperator     |
| 3ae54183d98e4f8686d8484fb13bdef7 | swiftoperator   |
| 7468a64363644b988b3f2a8111128b88 | ResellerAdmin   |
| 8b8bd9f2d5f9435fac1df5fd189763fc | admin           |
| 9fe2ff9ee4384b1894a90878d3e92bab | _member_        |
+----------------------------------+-----------------+
```

9. 使用 role add 子命令，将角色 admin 添加给 project1 中的 adminuser。

```
[student@workstation ~]$ openstack role add --project project1 --user adminuser admin
+-------+----------------------------------+
| Field | Value                            |
+-------+----------------------------------+
| id    | a7b6afc1a27741b6939383d18d9c1ebf |
| name  | admin                            |
+-------+----------------------------------+
```

10. 使用 role add 子命令，将实验脚本创建的角色 laboperator 添加给 project1 中的 user1。

```
[student@workstation ~]$ openstack role add --project project1 --user user1 laboperator
+-------+----------------------------------+
| Field | Value                            |
+-------+----------------------------------+
| id    | 7b6afc1a27741ab6d9c1e939383d18bf |
| name  | laboperator                      |
+-------+----------------------------------+
```

11. 使用 openstack user list 命令列出用户。user3 用户由实验脚本创建。

```
[student@workstation ~]$ openstack user list
+----------------------------------+-------+
| ID                               | Name  |
+----------------------------------+-------+
...Output omitted...
| 93260f67eb93464d92ee7dd07995e462 | user3 |
...Output omitted...
+----------------------------------+-------+
```

12. 使用 openstack user list 命令，验证实验脚本已创建了 user3 用户。

```
[student@workstation ~]$ openstack user list -f csv
"ID","Name"
...Output omitted...
"53b4a8c568b544a4a7fdc35d5979a76c","user3"
...Output omitted...
"
```

13. 使用 user delete 子命令，删除 user3。

```
[student@workstation ~]$ openstack user delete user3
```

14. 运行 openstack user list 命令，验证 user3 用户已被删除。

```
[student@workstation ~]$  openstack user list -f csv
"ID","Name"
...Output omitted...
```

15. 通过 user set 选项运行 help 命令，显示关于使用 openstack 命令修改用户的信息。

```
[student@workstation ~]$ openstack help user set
usage: openstack user set [-h] [--name <name>] [--project <project>]
                          [--password <user-password>] [--password-prompt]
                          [--email <email-address>] [--enable | --disable]
                          <user>

Set user properties

positional arguments:
  <user>                User to change (name or ID)

optional arguments:
  -h, --help            show this help message and exit
  --name <name>         Set user name
  --project <project>   Set default project (name or ID)
  --password <user-password>
                        Set user password
  --password-prompt     Prompt interactively for password
  --email <email-address>
                        Set user email address
  --enable              Enable user (default)
  --disable             Disable user
```

16. 将 user4 的密码更改为 custompassword。

```
[student@workstation ~]$ openstack user set --password custompassword user4
```

17. 通过将包含管理员凭据的现有 /home/student/overcloudrc 文件复制到 /home/student/overcloudrc_user1，为 user1 创建 overcloudrc 文件。

```
[student@workstation ~]$ cp overcloudrc overcloudrc_user1
```

18. /home/student/overcloudrc_user1 文件包含以下内容：

```
export OS_USERNAME=user1
export OS_PASSWORD=redhat
export OS_AUTH_URL=http://192.168.0.11:5000/v2.0
export OS_TENANT_NAME=project1
...Output omitted...
```

19. 通过将包含管理员凭据的现有 /home/student/overcloudrc 文件复制到 /home/student/overcloudrc_adminuser，为 adminuser 创建 overcloudrc 文件。

```
[student@workstation ~]$ cp overcloudrc overcloudrc_adminuser
```

20. /home/student/overcloudrc_adminuser包含以下内容：

```
export OS_USERNAME=adminuser
export OS_PASSWORD=redhat
export OS_AUTH_URL=http://192.168.0.11:5000/v2.0
export OS_TENANT_NAME=project1
...Output omitted...
```

21. 提供/home/student/overcloudrc_user1文件以获取 user1 权限。

```
[student@workstation ~]$ source overcloudrc_user1
[student@workstation ~]$ 
```

22. 尝试列出 project1 中的用户。它应当发出错误消息，因为 user1 没有关联 project1 中的 admin 角色。

```
[student@workstation ~]$ openstack user list --project project1
Could not find resource project1
```

23. 提供/home/student/overcloudrc_adminuser文件以获取 adminuser 权限。

```
[student@workstation ~]$ source overcloudrc_adminuser
```

24. 列出 project1 中的用户。它应当正常工作，因为 adminuser 已关联了 project1 中的 admin 角色。

```
[student@workstation ~]$ openstack user list --project project1
+----------------------------------+------------+
| ID                               | Name       |
+----------------------------------+------------+
...Output omitted...
| b947a781c1ef6abc025c5b53fd4ac7c1 | adminuser  |
| f1b49ba2c1ca5658f17efa145f6a8a40 | user1      |
+----------------------------------+------------+
```

25. 以 heat-admin 用户身份并使用其关联的 IP 地址 192.168.0.11 和私钥文件 /home/student/overcloud.pem 登录控制器节点，再修改 Keystone 服务的 policy.json 文件（位于 /etc/keystone/policy.json）中的 admin_required 规则，使其包含关联了 laboperator 角色的用户。检查允许列出用户的 identity:list_users 目标是否使用了 admin_required 规则。完成时注销。

```
[student@workstation ~]$ ssh -i overcloud.pem heat-admin@192.168.0.11
[heat-admin@overcloud-controller-0 ~]$ sudo vi /etc/keystone/policy.json
{
    "admin_required": "role:admin or is_admin:1 or role:laboperator",
...Output omitted...
    "identity:list_users": "rule:admin_required",
...Output omitted...
}
```

26. 恢复 policy.json 文件的 SELinux 上下文，然后重新启动 openstack-keystone 服务。服务重启之后，从服务器退出。

```
[heat-admin@overcloud-controller-0 ~]$ sudo restorecon /etc/keystone/policy.json
[heat-admin@overcloud-controller-0 ~]$ sudo systemctl restart openstack-keystone.service
[heat-admin@overcloud-controller-0 ~]$ logout
```

27. 尝试列出 project1 中的用户。您现在应当能够列出用户，因为 laboperator 角色已添加到被允许列出用户的角色列表中。

```
[student@workstation ~]$ source overcloudrc_user1
[student@workstation ~]$ openstack user list --project project1
+----------------------------------+-----------+
| ID                               | Name      |
+----------------------------------+-----------+
...Output omitted...
| 68f09e1bb57e4965a098457d4291dc96 | user1     |
| 5f01204892a043f3b92afde36e048c2b | adminuser |
+----------------------------------+-----------+
```


###  验证 Keystone 用户、角色和项目

###### 验证 Keystone

在使用 OpenStack 统一 CLI 验证 Keystone 用户、角色和项目之前，务必要先了解 Keystone 执行的内部验证流程。默认情况下，到达 OpenStack 服务 API 的所有请求都必须经过 Keystone 执行的身份验证与授权流程。此流程验证用户是否是他们声称的身份，并且他们的权限是否足以执行请求的操作。用户执行的任一请求均包含用户名/密码凭据、项目和身份验证 URL。此身份验证 URL 指向 Keystone API 的公共端点，其提供一组有限的 Keystone API 功能。该请求通过提供的凭据进行验证，确认用户具有操作所需服务/项目的权限。Keystone 生成令牌，供用户客户端用于将请求直接发送到管理该请求的服务（例如，启动实例将由 Nova 进行管理）。这会在 OpenStack 服务之间启动一系列请求，它们都使用用户令牌与 Keystone 进行身份验证。

###### 验证 Keystone 用户

可以通过openstack命令验证 Keystone 用户。初步验证使用 openstack 命令来确保已创建了用户、角色和项目。 

用户的存在性通过发出 openstack user list 命令来验证。

```
[student@demo ~]$ openstack user list
+----------------------------------+------------------+
|                id                |       name       |
+----------------------------------+------------------+
...Output omitted...
| fad9876543210fad9876543210fad987 |      demouser    |
+----------------------------------+------------------+
```

项目的存在性通过发出 openstack project list 命令来验证。

```
[student@demo ~]$ openstack project list
+----------------------------------+-------------+
|                id                |   name      |
+----------------------------------+-------------+
...Output omitted...
| 4567890abcdef1234567890abcdef123 | demoproject |
+----------------------------------+-------------+
```

角色的存在性通过发出命令openstack role list来验证

```
[student@demo ~]$ openstack role list
+----------------------------------+------------------+
|                id                |       name       |
+----------------------------------+------------------+
...Output omitted...
| fad9876543210fad9876543210fad987 |      admin       |
+----------------------------------+------------------+
...Output omitted...
| 34567890abcdef1234567890abcdef12 |      _member_    |
```

list 选项可以和openstack命令的其他选项组合使用，验证用户与项目以及用户与角色之间的映射。例如，命令openstack role list --project project user将显示映射到项目的用户的角色。

```
[student@demo ~]$ openstack user role list --project demoproject demouser
+----------------------------------+------------+--------------+----------------+
|                id                |    Name    | Project      |     User       |
+----------------------------------+------------+--------------+----------------+
...Output omitted...
| 1e567890abcdef1234567890abcdef01 | _member_   | demoproject  |   demouser     |
+----------------------------------+------------+--------------+----------------+
```

以指定的用户身份从 Keystone 检索身份验证令牌，从而验证用户配置是否正确。必须先提供用户的 overcloudrc 文件，然后再使用 openstack token issue 命令检索令牌。

```
[student@demo ~]$ source overcloudrc_demouser
[student@demo ~]$ openstack token issue
+------------+----------------------------------+
|  Property  |              Value               |
+------------+----------------------------------+
| expires    |        2016-01-23T18:36:24Z      |
| id         | 7822db1f1733403a95eff99918f5a281 |
| project_id | 01567890abcdef1234567890abcdef01 |
| user_id    | a4567890abcdef1234567890abcdef10 |
+------------+----------------------------------+
```


### 练习：验证 Keystone 用户、角色和项目

1. 从 workstation 打开一个新终端，再提供位于 /home/student/ 的 overcloudrc 凭据文件。这将启用所有 OS_* 环境变量，其中至少包含管理员凭据和 Keystone 端点。

```
[student@workstation ~]$ source /home/student/overcloudrc
```

2. 验证用户 user1 和 adminuser 均已创建：

```
[student@workstation ~]$ openstack user list
+----------------------------------+-----------+
| ID                               | Name      |
+----------------------------------+-----------+
...Output omitted...
| 01567890abcdef1234567890abcdef01 | user1     |
| 34567890abcdef1234567890abcdef12 | admin     |
...Output omitted...
| 02567890abcdef1234567890abadmf02 | adminuser |
+----------------------------------+-----------+
```

3. 验证 admin 和 _member_ 角色均已创建。

```
[student@workstation ~]$ openstack role list
+----------------------------------+----------+
| ID                               | Name     |
+----------------------------------+----------+
...Output omitted...
| fad9876543210fad9876543210fad987 | admin    |
| 34567890abcdef1234567890abcdef12 | _member_ |
...Output omitted...
+----------------------------------+----------+
```

4. 验证当前存在项目。

```
[student@workstation ~]$ openstack project list
+----------------------------------+-------------+
| ID                               | Name        |
+----------------------------------+-------------+
| 1456780abcdef1234567890abcdef124 | service     |
...Output omitted...
| a4567890abcdef1234567890abcdef10 | project1    |
| b4567890abcdef1234567890abcdef20 | testproject |
...Output omitted...
| 4567890abcdef1234567890abcdef123 | admin       |
+----------------------------------+-------------+
...Output omitted...
```

5. 验证 user1 和 adminuser 已映射到 project1，且未映射到 testproject。

```
[student@workstation ~]$ openstack user list --project project1
+----------------------------------+-----------+
| ID                               | Name      |
+----------------------------------+-----------+
| 01567890abcdef1234567890abcdef01 | user1     |
...Output omitted...
| 02567890abcdef1234567890abadmf02 | adminuser |
+----------------------------------+-----------+

[student@workstation ~]$ openstack user list --project testproject
```

6. 验证 user1 和 adminuser 已分配有正确的角色。

```
[student@workstation ~]$ openstack user role list --project project1 user1
+----------------------------------+----------+----------+-------+
| ID                               | Name     | Project  | User  |
+----------------------------------+----------+----------+-------+
| 1e567890abcdef1234567890abcdef01 | _member_ | project1 | user1 |
+----------------------------------+----------+----------+-------+

[student@workstation ~]$ openstack user role list --project project1 adminuser
+----------------------------------+-------+----------+-----------+
| ID                               | Name  | Project  | User      |
+----------------------------------+-------+----------+-----------+
...Output omitted...
| 2e567890abcdef1234567890abadmf02 | admin | project1 | adminuser |
+----------------------------------+-------+----------+-----------+
```

7. 通过尝试获取 Keystone 令牌来确保正确配置了用户。

由于非管理员用户无法列出用户，因此请使用token issue以测试用户是否存在以及是否在overcloudrc_user1脚本中正确设置了环境变量。

```
[student@workstation ~]$ source overcloudrc_user1
[student@workstation ~]$ openstack token issue
+------------+----------------------------------+
| Property   | Value                            |
+------------+----------------------------------+
| expires    | 2016-01-23T18:36:24Z             |
| id         | 7822db1f1733403a95eff99918f5a281 |
| project_id | a4567890abcdef1234567890abcdef10 |
| user_id    | 01567890abcdef1234567890abcdef01 |
+------------+----------------------------------+

[student@workstation ~]$ source overcloudrc_user2
[student@workstation ~]$ openstack token issue
+------------+----------------------------------+
| Property   | Value                            |
+------------+----------------------------------+
| expires    | 2016-01-23T18:36:24Z             |
| id         | 7933db1f1211403a95eff99918f5a482 |
| project_id | a4567890abcdef1234567890abcdef10 |
| user_id    | fd08b22fa13544019d3327b28d02eee5 |
+------------+----------------------------------+
```

8. 通过尝试使用openstack user list命令列出用户，验证 user1 和 user2 没有任何管理权限。

```
[student@workstation ~]$ source overcloudrc_user1
[student@workstation ~]$ openstack user list
You are not authorized to perform the requested action: admin_required...


[student@workstation ~]$ source overcloudrc_user2
[student@workstation ~]$ openstack user list
You are not authorized to perform the requested action: admin_required...
```


### 实验：管理 Keystone 身份服务

1. 在 workstation 上，提供位于 /home/student/overcloudrc 的 OpenStack 凭据文件。新建一个名为 project1 的项目。

    a. 从 workstation 打开一个新终端，再提供 overcloudrc 文件，以获取 admin 权限。这将启用所有 OS_* 环境变量，其中至少包含管理员凭据和 Keystone 端点。

```
    [student@workstation ~]$ source /home/student/overcloudrc
    [student@workstation ~]$ 
```

    b. 新建一个名为 project1 的项目。

```
    [student@workstation ~]$ openstack project create project1
    +-------------+----------------------------------+
    | Field       | Value                            |
    +-------------+----------------------------------+
    | description | None                             |
    | enabled     | True                             |
    | id          | bf046a97e0b24774b211a6dc64bf8b46 |
    | name        | project1                         |
    +-------------+----------------------------------+
```

2. 创建两个新用户 user1 和 user2，两者都使用 redhat 作为密码。

    a. 创建名为 user1 的新用户，不关联任何项目，使用 redhat 作为密码。

```
    [student@workstation ~]$ openstack user create --password redhat user1
    +----------+----------------------------------+
    | Field    | Value                            |
    +----------+----------------------------------+
    | email    | None                             |
    | enabled  | True                             |
    | id       | 040554231c79497aac352730e2b0560b |
    | name     | user1                            |
    | username | user1                            |
    +----------+----------------------------------+
```

    b. 创建名为 user2 的新用户，不关联任何项目，使用 redhat 作为密码。

```
    [student@workstation ~]$ openstack user create --password redhat user2
    +----------+----------------------------------+
    | Field    | Value                            |
    +----------+----------------------------------+
    | email    | None                             |
    | enabled  | True                             |
    | id       | 351fe0db7c9344d48ac67c04154bb94d |
    | name     | user2                            |
    | username | user2                            |
    +----------+----------------------------------+
```

3. 将 user1 配置为 project1 项目的管理员，再将 user2 配置为同一项目的成员，但没有管理权限。

    a. 将 project1 项目内的 admin 角色关联到 user1 用户。

```
    [student@workstation ~]$ openstack role add --project project1 --user user1 admin
    +-------+----------------------------------+
    | Field | Value                            |
    +-------+----------------------------------+
    | id    | a7b6afc1a27741b6939383d18d9c1ebf |
    | name  | admin                            |
    +-------+----------------------------------+
```

    b. 将 project1 项目内的 _member_ 角色关联到 user2 用户。

```
    [student@workstation ~]$ openstack role add --project project1 --user user2 _member_
    +-------+----------------------------------+
    | Field | Value                            |
    +-------+----------------------------------+
    | id    | 9fe2ff9ee4384b1894a90878d3e92bab |
    | name  | _member_                         |
    +-------+----------------------------------+
```

4. 在/home/student目录中为 user1 和 user2 新建overcloudrc文件，该文件包含这两个用户对应的凭据。将文件命名为overcloudrc_user1和overcloudrc_user2。

    a. 通过将包含管理员凭据的现有/home/student/overcloudrc文件复制到/home/student/overcloudrc_user1中，为 user1 凭据创建一个文件。

```
    [student@workstation ~]$ cp /home/student/overcloudrc /home/student/overcloudrc_user1
```

    b. 该文件应当包含以下内容：

```
    export OS_USERNAME=user1
    export OS_PASSWORD=redhat
    export OS_AUTH_URL=http://192.168.0.11:5000/v2.0
    export OS_TENANT_NAME=project1
    ...Output omitted...
```

    c. 通过将包含管理员凭据的现有/home/student/overcloudrc文件复制到/home/student/overcloudrc_user2中，为 user2 创建一个凭据文件。

```
    [student@workstation ~]$ cp /home/student/overcloudrc /home/student/overcloudrc_user2
```

    d. 该文件应当包含以下内容：

```
    export OS_USERNAME=user2
    export OS_PASSWORD=redhat
    export OS_AUTH_URL=http://192.168.0.11:5000/v2.0
    export OS_TENANT_NAME=project1
    ...Output omitted...
```

5. 修改 project1 项目的密钥对配额，将值设为 2。

    a. 提供/home/student/overcloudrc_user1，以使用 user1 用户获取管理权限。

```
    [student@workstation ~]$ source /home/student/overcloudrc_user1
    [student@workstation ~]$ 
```

    b. 检索 project1 项目的 ID。

```
    [student@workstation ~]$ openstack project show project1
    +-------------+----------------------------------+
    | Field       | Value                            |
    +-------------+----------------------------------+
    ...Output omitted...
    | id          | bf046a97e0b24774b211a6dc64bf8b46 |
    | name        | project1                         |
    +-------------+----------------------------------+
```

    c. 使用 nova 命令及项目 ID 显示 project1 项目的当前配额。注意 key_pairs 配额的当前值。

```
    [student@workstation ~]$ nova quota-show --tenant bf046a97e0b24774b211a6dc64bf8b46
    +-----------------------------+-------+
    | Quota                       | Limit |
    +-----------------------------+-------+
    | instances                   | 10    |
    | cores                       | 20    |
    | ram                         | 51200 |
    | floating_ips                | 50    |
    | fixed_ips                   | -1    |
    | metadata_items              | 128   |
    | injected_files              | 5     |
    | injected_file_content_bytes | 10240 |
    | injected_file_path_bytes    | 255   |
    | key_pairs                   | 100   |
    | security_groups             | 10    |
    | security_group_rules        | 20    |
    | server_groups               | 10    |
    | server_group_members        | 10    |
    +-----------------------------+-------+
```

    d. 将 project1 项目的 key_pairs 配额的当前值更新为 2。该命令不会产生任何输出。

```
    [student@workstation ~]$ nova quota-update --key-pairs 2 bf046a97e0b24774b211a6dc64bf8b46
```

    e. 再次检查 project1 项目的配额。key_pairs 配额现在应当设置为 2。

```
    [student@workstation ~]$ nova quota-show --tenant bf046a97e0b24774b211a6dc64bf8b46
    +-----------------------------+-------+
    | Quota                       | Limit |
    +-----------------------------+-------+
    | instances                   | 10    |
    | cores                       | 20    |
    | ram                         | 51200 |
    | floating_ips                | 50    |
    | fixed_ips                   | -1    |
    | metadata_items              | 128   |
    | injected_files              | 5     |
    | injected_file_content_bytes | 10240 |
    | injected_file_path_bytes    | 255   |
    | key_pairs                   | 2     |
    | security_groups             | 10    |
    | security_group_rules        | 20    |
    | server_groups               | 10    |
    | server_group_members        | 10    |
    +-----------------------------+-------+
```

6. 验证 user1 能够列出 project1 项目内的用户，而 user2 则不行。

    a. 列出 project1 项目内的用户。

```
    [student@workstation ~]$ openstack user list --project project1
    +----------------------------------+------------+
    | ID                               | Name       |
    +----------------------------------+------------+
    | 53fd4ac7c1b947a781c1ef6abc025c5b | user2      |
    | 7efa145f6a8a40f1b49ba2c1ca5658f1 | user1      |
    +----------------------------------+------------+
```

    b. 提供/home/student/overcloudrc_user2以获取 user2 权限。

```
    [student@workstation ~]$ source /home/student/overcloudrc_user2
    [student@workstation ~]$ 
```

    c. 列出 project1 项目内的用户。

```
    [student@workstation ~]$ openstack user list --project project1
    Could not find resource project1
```

7. 删除 project2 项目和 user3 用户。

    a. 提供/home/student/overcloudrc文件，以获取管理权限。

```
    [student@workstation ~]$ source /home/student/overcloudrc
    [student@workstation ~]$ 
```

    b. 删除 project2 项目。

```
    [student@workstation ~]$ openstack project delete project2
```

    c. 删除 user3 用户。

```
    [student@workstation ~]$ openstack user delete user3
```


### 总结

在本章中，您可以学到：

*    统一 CLI 提供了更加简便的方式，用户可以在单一工具中管理所有 Red Hat OpenStack Platform 服务，免除了必须针对每项 Red Hat OpenStack Platform 服务操作一款 CLI 工具的复杂性。
*    Keystone 服务支持通过三个元素（即用户、项目和角色）进行组织行为映射，以满足身份验证的需要。| 一个用户可以是一个或多个项目的成员，多个用户可以是一个项目的成员，一个用户也可在不同项目中与不同的角色关联。
*    配额为限制分配的资源提供了方式。
