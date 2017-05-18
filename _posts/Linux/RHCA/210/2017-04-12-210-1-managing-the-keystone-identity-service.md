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
+--------------------------------------+------------+
| UUID                                 | Name       |
+--------------------------------------+------------+
| b3f20a88-d2ef-4410-ad8f-1ba8af5612d7 | controller |
| 181a1107-1329-466d-9cf7-fd44c7b531d9 | compute1   |
+--------------------------------------+------------+
--------------------------------------+
 Instance UUID                        |
--------------------------------------+
 72496eba-134e-4e47-9160-26d6e6a64a1f |
 d86b204f-5bd9-4e43-be2c-39171b01c72f |
--------------------------------------+
-------------+--------------------+-------------+
 Power State | Provisioning State | Maintenance |
-------------+--------------------+-------------+
 power on    | active             | False       |
 power on    | active             | False       |
-------------+--------------------+-------------+
[stack@director ~]$ openstack server list
+--------------------------------------+-------------------------+
| ID                                   | Name                    |
+--------------------------------------+-------------------------+
| d86b204f-5bd9-4e43-be2c-39171b01c72f | overcloud-novacompute-0 |
| 72496eba-134e-4e47-9160-26d6e6a64a1f | overcloud-controller-0  |
+--------------------------------------+-------------------------+
--------+-------------------------+
 Status | Networks                |
--------+-------------------------+
 ACTIVE | ctlplane=172.24.250.120 |
 ACTIVE | ctlplane=172.24.250.119 |
--------+-------------------------+
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
+--------------------------------------+-----------+-------+
| ID                                   | Name      |   RAM |
+--------------------------------------+-----------+-------+
| 1                                    | m1.tiny   |   512 |
| 1e2bea88-b83d-4c9b-8f84-04454c9bf0fa | m1.demo   |   512 |
| 2                                    | m1.small  |  2048 |
| 3                                    | m1.medium |  4096 |
| 4                                    | m1.large  |  8192 |
| 5                                    | m1.xlarge | 16384 |
+--------------------------------------+-----------+-------+
------+-----------+-------+-----------+
 Disk | Ephemeral | VCPUs | Is Public |
------+-----------+-------+-----------+
    1 |         0 |     1 | True      |
   10 |         0 |     1 | True      |
   20 |         0 |     1 | True      |
   40 |         0 |     2 | True      |
   80 |         0 |     4 | True      |
  160 |         0 |     8 | True      |
------+-----------+-------+-----------+
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


### 



### 



### 



### 总结

在本章中，您可以学到：

*    统一 CLI 提供了更加简便的方式，用户可以在单一工具中管理所有 Red Hat OpenStack Platform 服务，免除了必须针对每项 Red Hat OpenStack Platform 服务操作一款 CLI 工具的复杂性。
*    Keystone 服务支持通过三个元素（即用户、项目和角色）进行组织行为映射，以满足身份验证的需要。| 一个用户可以是一个或多个项目的成员，多个用户可以是一个项目的成员，一个用户也可在不同项目中与不同的角色关联。
*    配额为限制分配的资源提供了方式。
