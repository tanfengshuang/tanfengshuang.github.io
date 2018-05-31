---
layout: post
title:  "添加额外的计算节点(210-9)"
categories: Linux
tags: RHCA 210
---

### 添加额外的计算节点

###### 扩展 OpenStack

云计算的一大特点是能够快速扩大或缩小基础架构。管理员可以在其基础架构中调配不同的节点，它们可以履行多种角色（如计算、存储或控制器）并可预装有基础操作系统。然后，管理员可以根据需要将这些节点集成到自己的环境中。云计算提供的服务可以自动考虑负载用量的增减，并在环境需要缩放时适时警告管理员。在传统的计算模型中，通常需要手动在现有环境中安装、配置和集成新服务器，因此需要额外的时间和精力来调配节点。自动缩放是云计算模型提供的一大益处，例如，它允许对负载激增进行快速响应。
注意

虽然云计算模型提供了扩展功能，这并不表明“传统”计算模型无此能力。相反，云计算模型的开发是为了促进自动缩放功能；例如，通过尽可能做到硬件不可知，或者通过允许管理员批量管理其服务器。虽然使用了自动缩放这一术语，但管理员仍然需要手动执行一些任务，具体取决于他们用于调配服务器的解决方案。

Director 通过 Heat 实施缩放功能。在需要时，管理员可以重新运行用于部署 overcloud 的命令，来增加或减少他们想要的角色。以下代码段演示如何将 OpenStack 环境扩展至最多 10 个计算节点。Director 将接着自动审查当前的配置，并重新配置适当的服务来调配具有 10 个计算节点的 OpenStack 环境。

```
[stack@demo ~]$ openstack overcloud deploy --compute-scale 10
```

> 注意: 尽管并不完整，此命令演示了如何能够扩展 OpenStack 环境。

虽然扩展通常用于向环境添加更多容量，管理员也可缩减环境；例如，不再需要激增期间使用的节点时。缩减可用于释放资源。

> 警告: 扩展环境是一种非破坏性流程（即，不会删除此环境中运行的资源），而缩减环境则会删除在要被移除的节点上运行的资源。管理员应当先删除节点上运行的所有资源，然后再缩减环境。例如，他们可以将服务迁移到另一节点，或者对它们进行备份。

以下命令演示了如何从 overcloud 删除节点：

```
[stack@demo ~]$ openstack overcloud node delete --stack overcloud NODE
```

###### 添加节点

管理员可以启动更多选定角色（如计算节点或存储节点）的服务器或在应当要减小容量时删除一些服务器，无缝增大或减小运行中的 overcloud 的资源容量。要向 OpenStack 环境添加一个或多个节点，应当使用以下步骤：

*    服务器应当调配有受支持的操作系统，如 Red Hat Enterprise Linux 7。
*    然后，管理员应当向红帽网络渠道订阅系统 rhel-7-server-openstack-8.0-rpms。
*    应创建用于描述服务器规格的文件。可使用 openstack baremetal import --json SERVER.json 命令添加节点。
*    然后启动节点的内省，这将使得 Ironic 能够发现节点的功能并使它可用。
*    如果使用手动标记，管理员接着应当标记该节点，以便将现有的类别映射到此节点。
*    最后，管理员可以通过增大他们想要扩展的资源（即，控制节点、存储节点或计算节点）扩展他们的环境。

Director 自动重新配置 Keystone 或 Nova 等 OpenStack 服务，使节点得到集成并就绪可用。 


### 练习：添加额外的计算节点

1. 从 workstation，以 stack 用户身份通过 SSH 连接到 director.lab.example.com，再提供文件 stackrc。

```
[student@workstation ~]$ ssh stack@director.lab.example.com
[stack@director ~]$ source stackrc
```

2. 使用 openstack 命令，列出正在运行的服务器。

```
[stack@director ~]$ openstack server list
+--------------------------------------+-------------------------+--------+-------------------------+
| ID                                   | Name                    | Status | Networks                |
+--------------------------------------+-------------------------+--------+-------------------------+
| 245e5136-514e-44e2-9fec-2307fd3eb529 | overcloud-controller-0  | ACTIVE | ctlplane=172.25.250.24  |
| 4c3ae9cb-cfa0-4a2c-b0e9-d0d38d11cfc6 | overcloud-novacompute-0 | ACTIVE | ctlplane=172.25.250.23  |
| 52937b28-e541-4636-9b69-c839027afba5 | overcloud-cephstorage-0 | ACTIVE | ctlplane=172.25.250.22  |
+--------------------------------------+-------------------------+--------+-------------------------+
```

如上所示，当前环境中已配置有控制器、计算节点和存储节点。

3. 使用 ironic 命令，列出可用的节点。您应当看到已注册了这三个节点。

```
[stack@director ~]$ ironic node-list
+--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
| UUID                                 | Name       | Instance UUID                        | Power State | Provisioning State | Maintenance |
+--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
| adfe4c64-c02f-4913-9649-5fd4ef0de53e | controller | 245e5136-514e-44e2-9fec-2307fd3eb529 | power on    | active             | False       |
| 89b62f9a-2a90-417c-9fb5-67d856e14e86 | compute1   | 4c3ae9cb-cfa0-4a2c-b0e9-d0d38d11cfc6 | power on    | active             | False       |
| 384cb4b6-4dd7-498f-9cf7-db6d0bd03406 | ceph       | 52937b28-e541-4636-9b69-c839027afba5 | power on    | active             | False       |
+--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
```

4. 从materials.example.com，检索包含第二计算节点 compute2 的定义的 JSON 文件。

```
[stack@director ~]$ wget http://materials.example.com/instackenv-onenode.json
```

5. 检查该文件。它以 JSON 格式定义要使用 pxe_ipmitool 驱动程序进行注册的节点。

```
{
  "nodes": [
    {
      "pm_user": "admin",
      "arch": "x86_64",
      "name": "compute2",
      "pm_addr": "192.168.1.113",
      "pm_password": "password",
      "pm_type": "pxe_ipmitool",
      "mac": [
        "52:54:00:00:fa:0d"
      ],
      "cpu": "2",
      "memory": "6144",
      "disk": "40"
    }
  ]
}
```

6. 使用 openstack baremetal import 命令将 JSON 文件导入到 Ironic，并确保节点已正确导入。

```
[stack@director ~]$ openstack baremetal import --json /home/stack/instackenv-onenode.json
[stack@director ~]$ ironic node-list
+--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
| UUID                                 | Name       | Instance UUID                        | Power State | Provisioning State | Maintenance |
+--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
...Output omitted...                                                                        ...Output omitted...
| fe5d3bd6-5dcd-4c52-984c-5d7a21502504 | compute2   | None                                 | power off   | available          | False       |
+--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
```

7. 在扩展环境之前，需要先配置计算节点。

在可以内省节点之前，必须给它分配引导映像。为此，可运行openstack baremetal configure boot命令。该命令将查找 Glance 中的引导映像（bm-deploy-ramdisk 和 bm-deploy-ramdisk），并将它们连接到计算节点。在下列输出中，deploy_kernel 和 deploy_ramdisk 已分配到部署期间要使用的适当内核和 ramdisk。

使用 ironic node show 确认引导映像已分配到 compute2。

```
[stack@director ~]$ openstack baremetal configure boot
[stack@director ~]$ ironic node-show compute2 | grep -1 deploy*
| driver_info            | {u'ipmi_password': u'******', u'ipmi_address': u'192.168.1.113',         |
|                        | u'ipmi_username': u'admin', u'deploy_kernel': u'f5d0dc0b-                |
|                        | 435c-46c9-8abc-d45b32aa3192', u'deploy_ramdisk': u'58e7b36b-6934-4611-   |
|                        | 9e53-e4232ba13947'}
```

8. 将 compute2 节点移入维护模式，然后再内省。使用 ironic node-set-maintenance 命令将 maintenance 设为 true。

```
[stack@director ~]$ ironic node-set-maintenance compute2 true
[stack@director ~]$ ironic node-list
+--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
| UUID                                 | Name       | Instance UUID                        | Power State | Provisioning State | Maintenance |
+--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
...Output omitted...                                                                        ...Output omitted...
| fe5d3bd6-5dcd-4c52-984c-5d7a21502504 | compute2   | None                                 | power off   | available          | True        |
+--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
```

9. compute2 节点已就绪，可使用 openstack baremetal introspection start UUID 命令内省。将节点的 UUID 替换为您环境中的 UUID。

```
[stack@director ~]$ openstack baremetal introspection start fe5d3bd6-5dcd-4c52-984c-5d7a21502504
```

10. 使用openstack baremetal introspection status UUID命令监控内省的状态。等待 finished 过渡到 True，然后使用 Ctrl+C 退出 watch 命令。

```
[stack@director ~]$ watch -n 1 openstack baremetal introspection status fe5d3bd6-5dcd-4c52-984c-5d7a21502504
+----------+-------+
| Field    | Value |
+----------+-------+
| error    | None  |
| finished | True  |
+----------+-------+
```

11. 启用节点。

```
[stack@director ~]$ ironic node-set-maintenance compute2 false
```

12. 确保节点标记为 available。

```
[stack@director ~]$ ironic node-list
+--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
| UUID                                 | Name       | Instance UUID                        | Power State | Provisioning State | Maintenance |
+--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
...Output omitted...                                ...Output omitted...
| fe5d3bd6-5dcd-4c52-984c-5d7a21502504 | compute2   | None                                 | power off   | available          | False       |
+--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
```

13. 最后，使用 ironic node-update 命令更新节点配置文件。由于它是计算节点，因此需要 compute 配置文件。

```
[stack@director ~]$ ironic node-update compute2 add properties/capabilities='profile:compute,boot_option:local'
+-----------------------------------------------------------------------+
| Value                                                                 |
+-----------------------------------------------------------------------+

| None                                                                  |
| power off                                                             |
| pxe_ipmitool                                                          |
| None                                                                  |
| {u'memory_mb': u'6144',
   u'cpu_arch': u'x86_64',
   u'local_gb': u'39',                                                  |
|  u'cpus': u'2',
   u'capabilities': u'profile:compute,boot_option:local'}               |
| None                                                                  |
| compute2                                                              |

+-----------------------------------------------------------------------+
```

14. 如果 /home/stack 中没有 templates 目录，请以 stack 用户身份创建该目录。复制位于 /usr/share/openstack-tripleo-heat-templates/ 的默认模板。

```
[stack@director ~]$ mkdir templates
[stack@director ~]$ cp -rf /usr/share/openstack-tripleo-heat-templates/* templates
```

15. 检索位于http://materials.example.com/overcloud-heat-templates/templates.tar.bz2的templates.tar.bz2存档，并提取文件。

```
[stack@director ~]$ curl -s -O http://materials.example.com/overcloud-heat-templates/templates.tar.bz2
[stack@director ~]$ tar -xvjpf templates.tar.bz2
```

16. 准备好节点时，将 2 用作计算节点数目来横向扩展环境。当前的 overcloud 已使用位于/home/stack/templates的模板进行了部署。为避免对 overcloud 进行不必要的更改，可在扩展堆栈时重新利用下列选项：

*    参数	                值
*    模板	                /home/stack/templates
*    control-flavor	        control
*    compute-flavor	        compute
*    ceph-storage-flavor	ceph-storage
*    control-scale	        1 日关闭。
*    compute-scale	        2
*    ceph-storage-scale	    1 日关闭。
*    neutron-tunnel-types	vxlan
*    neutron-network-type	vxlan

环境文件	

*    ~/templates/compute-extraconfig.yaml
*    ~/templates/environments/network-isolation.yaml
*    ~/templates/network-environment.yaml
*    ~/templates/environments/storage-environment.yaml
*    ~/templates/pre-config-fix.yaml

再次利用创建 overcloud 时使用的参数，运行 openstack overcloud 命令。若要添加第二计算节点，可将 2 用于 --compute-scale。

```
[stack@director ~]$ openstack overcloud deploy --templates ~/templates \
> --control-scale 1 --ceph-storage-scale 1 --compute-scale 2 \
> --control-flavor control --compute-flavor compute --ceph-storage-flavor ceph-storage \
> --neutron-tunnel-types vxlan --neutron-network-type vxlan \
> -e ~/templates/compute-extraconfig.yaml \
> -e ~/templates/environments/network-isolation.yaml \
> -e ~/templates/network-environment.yaml
> -e ~/templates/environments/storage-environment.yaml \
> -e ~/templates/pre-config-fix.yaml
Deploying templates in the directory /home/stack/templates/
```

> 重要: 完成更新需要 60 分钟。

17. 该 overcloud 是使用虚拟机部署的，而非物理硬件。观察到了争用情形，这可导致部署的元素不一致地挂起。在本课程发布之时，尚无此问题的自动化解决方案。不过，您可通过以下步骤进行检测，并根据需要手动纠正部署。

解决方案部分通过/home/stack/templates/pre-config-fix.yaml环境文件进行实施，该文件在上一步中用于openstack overcloud deploy命令。

    a. 在 workstation 上打开一个新终端，再以 stack 用户身份和密码 redhat 通过 SSH 连接到 director。使用 ironic node-list 命令，观察 Ironic 节点的状态过渡：available -> deploying -> wait call-back -> deploying -> active。

    在 deploying 阶段中，上传至 Glance 的 overcloud-full.qcow2 映像被转移到裸机 Ironic 节点。完成后，cloud-init 将重新引导，并调整文件系统的大小。然后，它将进入 active 调配状态。

    [stack@director ~]$ source stackrc
    [stack@director ~]$ ironic node-list
    +--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
    | UUID                                 | Name       | Instance UUID                        | Power State | Provisioning State | Maintenance |
    +--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
    ...Output omitted...                                                                        ...Output omitted...
    | fe5d3bd6-5dcd-4c52-984c-5d7a21502504 | compute2   | b15c5576-ab81-447e-be22-5ee67a5ed63f | power off   | active             | False       |
    +--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
    
    b. 验证裸机 Ironic 节点已启动了关联的 overcloud nova 实例。

    [stack@director ~]$ openstack server list
    +--------------------------------------+-------------------------+--------+-------------------------+
    | ID                                   | Name                    | Status | Networks                |
    +--------------------------------------+-------------------------+--------+-------------------------+
    | b15c5576-ab81-447e-be22-5ee67a5ed63f | overcloud-novacompute-1 | ACTIVE | ctlplane=172.25.250.25  |
    | 245e5136-514e-44e2-9fec-2307fd3eb529 | overcloud-controller-0  | ACTIVE | ctlplane=172.25.250.24  |
    | 4c3ae9cb-cfa0-4a2c-b0e9-d0d38d11cfc6 | overcloud-novacompute-0 | ACTIVE | ctlplane=172.25.250.23  |
    | 52937b28-e541-4636-9b69-c839027afba5 | overcloud-cephstorage-0 | ACTIVE | ctlplane=172.25.250.22  |
    +--------------------------------------+-------------------------+--------+-------------------------+
    
    c. 从 overcloud-novacompute-1 控制台并使用 root 用户和 ROOTPW 密码进行登录。

    验证文件 /etc/sysconfig/network-scripts/ifcfg-eth* 是否为零字节文件，再检查 /var/log/cloud-init.log 中是否有错误。对 overcloud-novacompute-1 执行这些验证步骤。

    [root@overcloud-novacompute-1 ~]# ls -l /etc/sysconfig/network-scripts/ifcfg-eth*
    -rw-r--r--. 1 root root 0 Sep 10 04:06 /etc/sysconfig/network-scripts/ifcfg-eth0
    [root@overcloud-novacompute-1 ~]# tailf /var/log/cloud-init.log
    ---Output omitted---
    cloud-init: 2016-09-10 00:05:50,867 - util.py[WARNING]: Route info failed: Unexpected error while running command.
    cloud-init: Command: ['netstat', '-rn']
    cloud-init: Exit code: 1
    cloud-init: Reason: -
    cloud-init: Stdout: 'Kernel IP routing table\nDestination     Gateway         Genmask         Flags   MSS Window  irtt Iface\n'
    cloud-init: Stderr: ''
    cloud-init: ci-info: +++++++++++++++++++++++Net device info+++++++++++++++++++++++
    cloud-init: ci-info: +--------+------+-----------+-----------+-------------------+
    cloud-init: ci-info: | Device |  Up  |  Address  |    Mask   |     Hw-Address    |
    cloud-init: ci-info: +--------+------+-----------+-----------+-------------------+
    cloud-init: ci-info: |  lo:   | True | 127.0.0.1 | 255.0.0.0 |         .         |
    cloud-init: ci-info: | eth0:  | True |     .     |     .     | 52:54:00:00:fa:0c |
    cloud-init: ci-info: +--------+------+-----------+-----------+-------------------+
    cloud-init: ci-info: !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!Route info failed!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
    ---Output omitted---

    d. cloud-init 可能会导致 overcloud 部署挂起，因为节点的网络无法访问。Heat 引擎将无法通过访问元数据服务器来配置节点。os-refersh-config用于对 overcloud 节点应用配置的元数据 URL 在/etc/os-collect-config.conf中指定。

    [root@overcloud-novacompute-1 ~]# cat /etc/os-collect-config.conf 
    [DEFAULT]
    command = os-refresh-config

    [cfn]
    metadata_url = http://172.25.250.10:8000/v1/
    stack_name = overcloud-Compute-n2onlrw2eva3-0-zsn5w6hvh3qy
    secret_access_key = 03c62d5e36d94f0cb9225f5e03af1fff
    access_key_id = a683f206faa54f8c8d6de1d903c56d4d
    path = NovaCompute.Metadata

    os-collect-config开始轮询元数据的更改。要查看此问题，请使用journalctl -u os-collect-config。

    [root@overcloud-novacompute-1 ~]# journalctl -u os-collect-config --no-full
    -- Logs begin at Sat 2016-09-10 04:04:05 UTC, end at Sat 2016-09-10 14:21:04 UTC. --
    systemd[1]: Started Collect metadata and run hook commands..
    systemd[1]: Starting Collect metadata and run hook commands....
    WARNING os_collect_config.ec2 [-] ('Connection aborted.', error(101, 'Ne...chable'))
    WARNING os-collect-config [-] Source [ec2] Unavailable.
    WARNING os-collect-config [-] Source [request] Unavailable.
    ---Output omitted--

    e. 删除零字节文件 /etc/sysconfig/network-scripts/ifcfg-eth* 以解决问题，再使用 reboot 命令手动重新启动节点。

    > 重要: 此操作仅可在有问题的 overcloud 节点上执行。

    [root@overcloud-novacompute-1 ~]# rm -rf /etc/sysconfig/network-scripts/ifcfg-eth*
    [root@overcloud-novacompute-1 ~]# reboot

18. 在 director 节点终端上，成功更新 overcloud Heat 模板部署后将显示下列消息。

Stack overcloud UPDATE_COMPLETE
Overcloud Endpoint: http://192.168.0.11:5000/v2.0
Overcloud Deployed

19. 使用openstack命令，确保已成功创建计算节点 overcloud-novacompute-1。

[stack@director ~]$ openstack server list
+--------------------------------------+-------------------------+--------+-------------------------+
| ID                                   | Name                    | Status | Networks                |
+--------------------------------------+-------------------------+--------+-------------------------+
| b15c5576-ab81-447e-be22-5ee67a5ed63f | overcloud-novacompute-1 | ACTIVE | ctlplane=172.25.250.25  |
| 245e5136-514e-44e2-9fec-2307fd3eb529 | overcloud-controller-0  | ACTIVE | ctlplane=172.25.250.24  |
| 4c3ae9cb-cfa0-4a2c-b0e9-d0d38d11cfc6 | overcloud-novacompute-0 | ACTIVE | ctlplane=172.25.250.23  |
| 52937b28-e541-4636-9b69-c839027afba5 | overcloud-cephstorage-0 | ACTIVE | ctlplane=172.25.250.22  |
+--------------------------------------+-------------------------+--------+-------------------------+

如上所示，环境现在已包含一个控制器节点、一个存储节点，以及两个计算节点。


### 验证额外的计算节点

###### 验证额外的计算节点

新计算节点添加至环境中后，管理员可以开始使用该节点。他们可以让 Nova 调度程序决定使用节点的时间，或者强制在该节点上部署实例。在扩展环境后可以使用计算节点之前，管理员应当：

> 注意: 在部署时，计算节点配置为使用 Ceph 作为其存储后端。

1. 使用 ironic node-list 命令确保新计算节点已经集成并标记为 active。

    [stack@director ~]$ ironic node-list
    +--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
    | UUID                                 | Name       | Instance UUID                        | Power State | Provisioning State | Maintenance |
    +--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
    ...Output omitted...                                                                        ...Output omitted...
    | afdc84a9-92cc-4ba6-8d6c-d69ceb835fca | compute2   | 4c3ae9cb-cfa0-4a2c-b0e9-d0d38d11cfc6 | power on    | active             | False       |
    +--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
    
2. 使用 openstack server list 确保某一虚拟实例已绑定至物理计算节点。该实例也应标记为 ACTIVE，并且已检索到 IP 地址。

    [stack@director ~]$ openstack server list
    +--------------------------------------+-------------------------+--------+--------------------------+
    | ID                                   | Name                    | Status | Networks                 |
    +--------------------------------------+-------------------------+--------+--------------------------+
    | 4c3ae9cb-cfa0-4a2c-b0e9-d0d38d11cfc6 | overcloud-novacompute-1 | ACTIVE | ctlplane =172.24.250.119 |
    +--------------------------------------+-------------------------+--------+--------------------------+

节点就绪时，管理员可以在该计算节点上生成新实例。


### 练习：验证额外的计算节点

1. 从 workstation，打开一个终端，并以 stack 用户身份通过 SSH 连接到 director 节点。提供 undercloud Keystone 凭据文件 stackrc。

[student@workstation ~]$ ssh stack@director
[stack@director ~]$ source stackrc

2. 运行 ironic node-list 命令，以确保新计算节点 compute2 标记为 active，并且已经为该节点声明了虚拟实例。列出相关实例的字段是 Instance UUID。

[stack@director ~]$ ironic node-list
+--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
| UUID                                 | Name       | Instance UUID                        | Power State | Provisioning State | Maintenance |
+--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
...Output omitted...                                                                        ...Output omitted...
| afdc84a9-92cc-4ba6-8d6c-d69ceb835fca | compute2   | 4c3ae9cb-cfa0-4a2c-b0e9-d0d38d11cfc6 | power on    | active             | False       |
+--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+

3. 使用 openstack server list 命令，确认虚拟实例已标记为 ACTIVE 并且拥有 IP 地址。

[stack@director ~]$ openstack server list
+--------------------------------------+-------------------------+--------+--------------------------+
| ID                                   | Name                    | Status | Networks                 |
+--------------------------------------+-------------------------+--------+--------------------------+
...Output omitted...
| 4c3ae9cb-cfa0-4a2c-b0e9-d0d38d11cfc6 | overcloud-novacompute-1 | ACTIVE | ctlplane=172.25.250.25   |
+--------------------------------------+-------------------------+--------+--------------------------+

4. 该设置脚本创建一个基础 overcloud 环境，该环境包含 small 映像、专用网络 dev_privnet、公共网络 dev_pubnet 以及类别 m2.small。它还添加了允许通过 ICMP 访问实例的安全组规则。

    a. 首先，提供 overcloud Keystone 凭据文件 overcloudrc。

    [stack@director ~]$ source overcloudrc

    b. 检索计算节点的 FQDN。此名称将作为参数传递。

    [stack@director ~]$ openstack hypervisor list
    +----+-------------------------------------+
    | ID | Hypervisor Hostname                 |
    +----+-------------------------------------+
    ...Output omitted...
    |  2 | overcloud-novacompute-1.localdomain |
    +----+-------------------------------------+

    c. 列出当前的可用性区域。应当仅列出 nova 可用性区域，它是默认区域。

    [stack@director ~]$ nova availability-zone-list
    +----------------------------------------+--------------+
    | Name                                   | Status       |
    +----------------------------------------+--------------+
    | internal                               | available    |
    | |- overcloud-controller-0.localdomain  |                                        |
    | | |- nova-conductor                    | enabled :-)  |
    | | |- nova-scheduler                    | enabled :-)  |
    | | |- nova-consoleauth                  | enabled :-)  |
    | nova                                   | available    |
    | |- overcloud-novacompute-0.localdomain |              |
    | | |- nova-compute                      | enabled :-)  |
    | |- overcloud-novacompute-1.localdomain |              |
    | | |- nova-compute                      | enabled :-)  |
    +----------------------------------------+--------------+

    d. 创建实例 dev_test-nodes；将可用区域 nova 与参数 --availability zone 一起传递，从而强行在新计算节点上创建该实例。

    [stack@director ~]$ openstack server create \
    > --image small \
    > --flavor m2.small \
    > --nic net-id=dev_privnet \
    > --availability-zone nova:overcloud-novacompute-1.localdomain \
    > --wait \
    > dev_test-nodes
    +---------------------------------+-------------------------------------+
    | Field                           | Value                               |
    +---------------------------------+-------------------------------------+
    ...Output omitted...
    | OS-EXT-SRV-ATTR:host            | overcloud-novacompute-1.localdomain |
    | OS-EXT-SRV-ATTR:
      hypervisor_hostname             | overcloud-novacompute-1.localdomain |
    | OS-EXT-SRV-ATTR:instance_name   | instance-00000007                   |
    ...Output omitted...
    | addresses                       | dev_privnet=192.168.1.17           |
    ...Output omitted...
    +---------------------------------+-------------------------------------+

    > 注意: nova 是默认的可用性区域。运行 nova availability-zone-list，列出所有可用性区域及它们所分配的角色。

    e. 确保实例已标记为 ACTIVE。

    [stack@director ~]$ openstack server list
    +--------------------------------------+----------------+--------+--------------------------+
    | ID                                   | Name           | Status | Networks                 |
    +--------------------------------------+----------------+--------+--------------------------+
    | b86af924-55a6-4366-b1d1-4764b4c7b001 | dev_test-nodes | ACTIVE | dev_privnet=192.168.1.17 |
    +--------------------------------------+----------------+--------+--------------------------+
    
5. 将实验脚本分配的浮动 IP 关联到实例 dev_test-nodes。

    a. 列出可用的浮动 IP。

    [stack@director ~]$ openstack ip floating list
    +--------------------------------------+------------+---------------+----------+-------------+
    | ID                                   | Pool       | IP            | Fixed IP | Instance ID |
    +--------------------------------------+------------+---------------+----------+-------------+
    | f0e28903-3ed9-4875-9f12-7f73ca25c39c | dev_pubnet | 172.25.250.51 | None     | None        |
    +--------------------------------------+------------+---------------+----------+-------------+

    b. 将之前分配的浮动 IP 关联到实例 dev_test-nodes。

    [stack@director ~]$ openstack ip floating add 172.25.250.51 dev_test-nodes

6. 实验脚本创建了一条允许 ICMP 数据包的安全组规则。使用ping，尝试访问该实例。

[stack@director ~]$ ping -c 3 172.25.250.51
ping -c 3 172.25.250.51
PING 172.25.250.51 (172.25.250.51) 56(84) bytes of data.
64 bytes from 172.25.250.51: icmp_seq=1 ttl=63 time=1.49 ms
64 bytes from 172.25.250.51: icmp_seq=2 ttl=63 time=1.20 ms
64 bytes from 172.25.250.51: icmp_seq=3 ttl=63 time=0.830 ms

--- 172.25.250.51 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 0.830/1.178/1.495/0.272 ms


### 在计算节点之间迁移实例

###### 迁移

通过迁移，云管理员可以将实例从一个计算节点移到另一个计算节点。在计算主机需要维护时，可以迁移其上运行的实例。需要在许多计算主机之间重新分布负载时，也可利用迁移功能。

######### 迁移种类

Red Hat OpenStack Platform 8 中可以进行三种迁移：

*    非实时迁移：实例关机一段时间，以便它从一个计算主机迁移到另一个计算主机。因此，停机期间无法访问其上运行的服务。
*    实时迁移：在这种迁移中，几乎没有实例停机时间，但它要求共享存储，并且能够迁移实例的所有计算主机应当都能访问该存储。在 nova.conf 中的默认配置选项下，实例在迁移之前会被暂停。
*    块实时迁移：这种迁移不要求共享存储，可以使用临时磁盘进行。临时磁盘利用块迁移在网络上复制。迁移具有重度 I/O 工作负载的实例不应使用块实时迁移。

> 注意: 使用 sync 命令同步磁盘上实例的数据，然后再迁移实例。

######### 实时迁移配置选项

libvirt 的部分实时迁移配置选项可以在 /etc/nova/nova.conf 中进行编辑。它们包含以下设置：

实时迁移配置选项

*    参数 	                                    详情
*    live_migration_retry_count = 30 	        实时迁移中需要的重试次数。
*    max_concurrent_live_migrations = 1 	    可以同时运行的实时迁移数量上限。
*    live_migration_bandwidth = 0 	            要使用的最大带宽，以 MiB/s 为单位。若设为 0，则选取合适的值作为默认值。
*    live_migration_completion_timeout = 800 	为成功完成迁移可等待的超时值（秒数），此后将中止操作。
*    live_migration_downtime = 500 	            允许的最长停机时间，以毫秒为单位。
*    live_migration_downtime_delay = 75 	    迁移停机时间递增步进之间等待的时间（秒数）。
*    live_migration_downtime_steps = 10 	    达到最大停机时间值所需的递增步数。
*    live_migration_flag = VIR_MIGRATE_UNDEFINE_SOURCE, VIR_MIGRATE_PEER2PEER, VIR_MIGRATE_LIVE, VIR_MIGRATE_TUNNELLED 	    为实时迁移设置的迁移标志。
*    live_migration_progress_timeout = 150 	        迁移的数据传输向前递进可等待的时间（秒数），此后将中止操作。
*    live_migration_uri = qemu+tcp://%s/system 	    迁移目标 URI。

###### 使用统一 CLI 迁移实例

python-openstackclient 软件包提供可迁移实例的命令行选项。

```
[student@demo ~]$ openstack server migrate --live compute-hostname instance
```

可以用于 openstack server migrate 命令的一部分选项。

openstack server migrate 命令选项。

*    参数 	                详情
*    --live COMPUTE HOST 	其中，COMPUTE HOST 是必须要迁移实例的计算节点。
*    --shared-migration 	执行共享实时迁移（默认）。
*    --block-migration 	    执行块实时迁移
*    --disk-overcommit 	    允许目标主机上过量使用磁盘。
*    --no-disk-overcommit 	不允许目标主机上过量使用磁盘（默认）。
*    ----wait 	            等待调整大小完成。
*    INSTANCE 	            其中，INSTANCE 是要迁移的实例的名称（名称或 ID）。

> 注意: 如果迁移不成功，请检查来源和目标计算节点上的 nova-compute 和 nova-scheduler 日志文件。

###### 在系统管理程序之间迁移实例

若要在同一 OpenStack 环境中的计算主机之间迁移实例，可使用 openstack server migrate 命令。Nova 通过 libvirt 使用 SSH 在主机之间迁移实例。所有节点必须配置有 SSH 密钥身份验证，并关联到 nova 用户，以便 libvirt 能够使用 SSH 在节点之间迁移实例。

> 注意: 在迁移之前，请使用 sync 命令将磁盘上的数据与实例上的内存同步。


### 练习：在计算节点之间迁移实例

1. 从 workstation，提供 overcloud Keystone 凭据文件 overcloudrc。

[student@workstation ~]$ source overcloudrc

2. 首先使用 nova service-list 命令获取运行 Nova 服务的节点列表。

[student@workstation ~]$ nova service-list --binary nova-compute
+----+--------------+-------------------------------------+------+---------+-------+----------------------------+-----------------+
| Id | Binary       | Host                                | Zone | Status  | State | Updated_at                 | Disabled Reason |
+----+--------------+-------------------------------------+------+---------+-------+----------------------------+-----------------+
| 5  | nova-compute | overcloud-novacompute-0.localdomain | nova | enabled | up    | 2016-02-22T13:18:46.000000 | -               |
| 6  | nova-compute | overcloud-novacompute-1.localdomain | nova | enabled | up    | 2016-02-22T13:18:40.000000 | -               |
+----+--------------+-------------------------------------+------+---------+-------+----------------------------+-----------------+

3. 检查托管实例 dev_test-nodes 的计算节点，以及该实例使用的类别。

[student@workstation ~]$ openstack server show dev_test-nodes
+--------------------------------------+-------------------------------------+
| Field                                | Value                               |
+--------------------------------------+-------------------------------------+
| OS-DCF:diskConfig                    | MANUAL                              |
| OS-EXT-AZ:availability_zone          | nova                                |
| OS-EXT-SRV-ATTR:host                 | overcloud-novacompute-1.localdomain |
| OS-EXT-SRV-ATTR:hypervisor_hostname  | overcloud-novacompute-1.localdomain |
| OS-EXT-SRV-ATTR:instance_name        | instance-00000002                   |
...Output omitted...
| flavor                               | m2.small (6d89f295-ca1fd659fa93)    |
+--------------------------------------+-------------------------------------+

4. 检索 m2.small 类别的规格。

[student@workstation ~]$ openstack flavor show m2.small
+----------------------------+--------------------------------------+
| Field                      | Value                                |
+----------------------------+--------------------------------------+
| OS-FLV-DISABLED:disabled   | False                                |
| OS-FLV-EXT-DATA:ephemeral  | 0                                    |
| disk                       | 10                                   |
| id                         | 6d89f295-59fe-4f4a-a84c-ca1fd659fa93 |
| name                       | m2.small                             |
| os-flavor-access:is_public | True                                 |
| properties                 |                                      |
| ram                        | 1024                                 |
| rxtx_factor                | 1.0                                  |
| swap                       | 1                                    |
| vcpus                      | 2                                    |
+----------------------------+--------------------------------------+

如您所见，该类别使用 10 GB 磁盘和 1 GB 内存。

5. 确认第一计算节点 overcloud-novacompute-0 具有充足的资源，可以托管要迁移的实例。使用 -c 标志，将结果筛选为仅显示可用内存和可用磁盘空间。确保可用的磁盘和内存均大于类别 m2.small。

[student@workstation ~]$ openstack hypervisor show overcloud-novacompute-0.localdomain -c free_ram_mb -c free_disk_gb
+--------------+-------+
| Field        | Value |
+--------------+-------+
| free_disk_gb | 30    |
| free_ram_mb  | 6025  |
+--------------+-------+

6. 使用openstack server migrate命令迁移实例。

[student@workstation ~]$ openstack server migrate --shared-migration dev_test-nodes

7. 迁移需要经过验证后才能完成。检查服务器的状态。

[student@workstation ~]$ openstack server list
+--------------------------------------+----------------+---------------+-----------------------------------------+
| ID                                   | Name           | Status        | Networks                                |
+--------------------------------------+----------------+---------------+-----------------------------------------+
| afb6b5e9-8b85-494e-84d4-4c02aad39295 | dev_test-nodes | VERIFY_RESIZE | dev_privnet=192.168.1.17, 172.25.250.51 |
+--------------------------------------+----------------+---------------+-----------------------------------------+

以上输出表明实例正在等待验证。该实例可以访问，并且已迁移到新的计算节点上。

8. 使用 nova resize-confirm 命令激活新实例。它将删除实例在旧计算节点中的虚拟磁盘，并保留实例已迁移到的计算节点中的虚拟磁盘。

[student@workstation ~]$ nova resize-confirm dev_test-nodes

9. 使用 openstack server show 命令检查实例已成功迁移。

[student@workstation ~]$ openstack server show dev_test-nodes
+--------------------------------------+-------------------------------------+
| Field                                | Value                               |
+--------------------------------------+-------------------------------------+
| OS-DCF:diskConfig                    | MANUAL                              |
| OS-EXT-AZ:availability_zone          | nova                                |
| OS-EXT-SRV-ATTR:host                 | overcloud-novacompute-0.localdomain |
| OS-EXT-SRV-ATTR:hypervisor_hostname  | overcloud-novacompute-0.localdomain |
| OS-EXT-SRV-ATTR:instance_name        | instance-00000002                   |
...Output omitted...

> 注意: 如果迁移不成功，请检查来源和目标计算节点上的 nova-compute 和 nova-scheduler 日志文件。

10. 检查来源和目标计算节点，以及 Status 字段，它应显示为 confirmed。

```
[student@workstation ~]$ nova migration-list
+-------------------------------------+-------------------------------------+-------------------------------------+-------------------------------------+
| Source Node                         | Dest Node                           | Source Compute                      | Dest Compute                        |
+-------------------------------------+-------------------------------------+-------------------------------------+-------------------------------------+
...Output omitted...
| overcloud-novacompute-1.localdomain | overcloud-novacompute-0.localdomain | overcloud-novacompute-1.localdomain | overcloud-novacompute-0.localdomain |
+-------------------------------------+-------------------------------------+-------------------------------------+-------------------------------------+


----------------+-----------+--------------------------------------+------------+------------+----------------------------+----------------------------+
 Dest Host      | Status    | Instance UUID                        | Old Flavor | New Flavor | Created At                 | Updated At                 |
----------------+-----------+--------------------------------------+------------+------------+----------------------------+----------------------------+
 172.24.250.115 | confirmed | afb6b5e9-8b85-494e-84d4-4c02aad39295 | 9          | 9          | 2016-02-24T11:06:56.000000 | 2016-02-24T11:07:06.000000 |
----------------+-----------+--------------------------------------+------------+------------+----------------------------+----------------------------+
```

11. 使用ping，确保仍然可通过其公共 IP 172.25.250.51 访问该实例。

```
[student@workstation ~]$ ping -c 3 172.25.250.51
PING 172.25.250.51 (172.25.250.51) 56(84) bytes of data.
64 bytes from 172.25.250.51: icmp_seq=1 ttl=63 time=0.836 ms
64 bytes from 172.25.250.51: icmp_seq=2 ttl=63 time=0.731 ms
64 bytes from 172.25.250.51: icmp_seq=3 ttl=63 time=0.688 ms

--- 172.25.250.51 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 0.688/0.751/0.836/0.069 ms
```


### 实验：添加额外的计算节点

在本实验中，您将通过添加额外的计算节点，横向扩展您的 OpenStack undercloud 环境。您将从一个计算节点生成实例，并将该实例迁移到第二计算节点。最后，您将通过运行 ping 命令确保该实例仍然能被访问。该环境将提供有下列资源：

*    undercloud Keystone 凭据，位于 director.lab.example.com 上的 /home/stack/stackrc 下。
*    overcloud Keystone 凭据，位于 workstation.lab.example.com 上的 /home/student/overcloudrc 下。
*    用于部署 overcloud 的 Heat 模板，它们位于 director 上的 /usr/share/openstack-tripleo-heat-templates/ 下。
*    包含第二计算节点 compute2 的定义的 JSON 文件，位于 http://materials.example.com/instackenv-onenode.json。
*    Glance 映像 small。
*    Nova 类别 m2.small。
*    公共网络 lab_pubnet。
*    专用网络 lab_privnet。
*    专用子网 lab_pubsub，位于 172.25.250.0/24 范围内。该网络用于提供外部连接，也是浮动 IP 的池。
*    专用子网 lab_privsub，位于 192.168.1.0/24 范围内。该网络运行供实例使用的 DHCP 服务器。
*    undercloud 模板文件，位于 director.lab.example.com 上的 /home/stack/templates/ 下。
*    default 安全组具有允许 ICMP 流量的规则。
*    从 lab_pubnet 池分配的一个浮动 IP。

1. 检查当前的环境

从 director.lab.example.com，提供 undercloud Keystone 凭据。使用 ironic 命令，检查配置的节点：controller、compute1 和 ceph。

    a. 从 workstation.lab.example.com，以 stack 身份通过 SSH 连接到 director.lab.example.com，再提供 overcloud 凭据文件 stackrc。

    [student@workstation ~]$ ssh stack@director.lab.example.com
    [stack@director ~]$ source stackrc

    b. 使用 openstack 命令，列出正在运行的服务器。

    [stack@director ~]$ openstack server list
    +--------------------------------------+-------------------------+--------+-------------------------+
    | ID                                   | Name                    | Status | Networks                |
    +--------------------------------------+-------------------------+--------+-------------------------+
    | 4c3ae9cb-cfa0-4a2c-b0e9-d0d38d11cfc6 | overcloud-novacompute-0 | ACTIVE | ctlplane=172.23.250.24  |
    | 245e5136-514e-44e2-9fec-2307fd3eb529 | overcloud-controller-0  | ACTIVE | ctlplane=172.23.250.23  |
    | 52937b28-e541-4636-9b69-c839027afba5 | overcloud-cephstorage-0 | ACTIVE | ctlplane=172.23.250.22  |
    +--------------------------------------+-------------------------+--------+-------------------------+

    如上所示，当前环境中配置有控制器、计算和 Ceph 节点。

    c. 使用 ironic 命令，列出可用的节点。

    [stack@director ~]$ ironic node-list
    +--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
    | UUID                                 | Name       | Instance UUID                        | Power State | Provisioning State | Maintenance |
    +--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
    | adfe4c64-c02f-4913-9649-5fd4ef0de53e | controller | 245e5136-514e-44e2-9fec-2307fd3eb529 | power on    | active             | False       |
    | 89b62f9a-2a90-417c-9fb5-67d856e14e86 | compute1   | 4c3ae9cb-cfa0-4a2c-b0e9-d0d38d11cfc6 | power on    | active             | False       |
    | 384cb4b6-4dd7-498f-9cf7-db6d0bd03406 | ceph       | 52937b28-e541-4636-9b69-c839027afba5 | power on    | active             | False       |
    +--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+

2. 添加计算节点

从 director 服务器，将第二计算节点 compute2 添加到当前环境中。检索包含节点定义的 JSON 文件（位于 http://materials.example.com/instackenv-onenode.json），再将节点导入到 Ironic 中。导入节点后，为它分配内核和 ramdisk 映像。内省节点，再为它分配 compute 配置文件。

    a. 从 materials.example.com，检索包含第二计算节点 compute2 的定义的 JSON 文件。

    [stack@director ~]$ wget http://materials.example.com/instackenv-onenode.json
    ...Output omitted...

    b. 使用 openstack baremetal import 命令将 JSON 文件导入到 Ironic，并确保节点已正确导入。

    [stack@director ~]$ openstack baremetal import --json /home/stack/instackenv-onenode.json
    [stack@director ~]$ ironic node-list
    +--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
    | UUID                                 | Name       | Instance UUID                        | Power State | Provisioning State | Maintenance |
    +--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
    ...Output omitted...                                                                        ...Output omitted...
    | fe5d3bd6-5dcd-4c52-984c-5d7a21502504 | compute2   | None                                 | power off   | available          | False       |
    +--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+

    c. 将内核和 ramdisk 映像分配到 compute2 节点。为此，请运行 openstack baremetal configure boot 命令。该命令将查找 Glance 中的引导映像（bm-deploy-ramdisk 和 bm-deploy-ramdisk），并将它们连接到计算节点。在下列输出中，deploy_kernel 和 deploy_ramdisk 已分配到部署期间要使用的适当内核和 ramdisk。

    使用 ironic node show 确认引导映像已分配到 compute2 节点。

    [stack@director ~]$ openstack baremetal configure boot
    [stack@director ~]$ ironic node-show compute2 | grep -1 deploy*
    | driver_info            | {u'ipmi_password': u'******', u'ipmi_address': u'192.168.1.113',         |
    |                        | u'ipmi_username': u'admin', u'deploy_kernel': u'f5d0dc0b-                |
    |                        | 435c-46c9-8abc-d45b32aa3192', u'deploy_ramdisk': u'58e7b36b-6934-4611-   |
    |                        | 9e53-e4232ba13947'}

    d. 在可以内省节点之前，先将它设为维护状态。

    [stack@director ~]$ ironic node-set-maintenance compute2 true

    e. compute2 节点已就绪，可使用 openstack baremetal introspection start UUID 命令内省。将 UUID 替换为 compute2 节点的 UUID。

    [stack@director ~]$ openstack baremetal introspection start fe5d3bd6-5dcd-4c52-984c-5d7a21502504

    f. 使用 openstack baremetal introspection status UUID 命令监控内省的状态。等待 finished 过渡到 True，然后使用 Ctrl+C 退出 watch 命令。

    [stack@director ~]$ watch -n 10 openstack baremetal introspection status fe5d3bd6-5dcd-4c52-984c-5d7a21502504
    +----------+-------+
    | Field    | Value |
    +----------+-------+
    | error    | None  |
    | finished | True  |
    +----------+-------+

    g. 启用节点。

    [stack@director ~]$ ironic node-set-maintenance compute2 false

    h. 确保节点标记为 available。

    [stack@director ~]$ ironic node-list
    +--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
    | UUID                                 | Name       | Instance UUID                        | Power State | Provisioning State | Maintenance |
    +--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
    ...Output omitted...                                                                       ...Output omitted...
    | fe5d3bd6-5dcd-4c52-984c-5d7a21502504 | compute2   | None                                 | power off   | available          | False       |
    +--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+

    i 最后，使用 ironic node-update 命令更新节点配置文件。由于它是计算节点，因此需要 compute 配置文件。

    [stack@director ~]$ ironic node-update compute2 add properties/capabilities='profile:compute,boot_option:local'
    +-----------------------------------------------------------------------+
    | Value                                                                 |
    +-----------------------------------------------------------------------+

    | None                                                                  |
    | power off                                                             |
    | pxe_ipmitool                                                          |
    | None                                                                  |
    | {u'memory_mb': u'6144',
       u'cpu_arch': u'x86_64',
       u'local_gb': u'39',                                                  |
    |  u'cpus': u'2',
       u'capabilities': u'profile:compute,boot_option:local'}               |
    | None                                                                  |
    | compute2                                                              |

    +-----------------------------------------------------------------------+

3. 创建模板

在 /home/stack 下创建 templates 目录。复制位于 /usr/share/openstack-tripleo-heat-templates/ 的模板文件。检索位于 http://materials.example.com/overcloud-heat-templates/templates.tar.bz2 的模板文件，将它们提取到 templates 目录。

    a. 以 stack 用户身份，在 /home/stack 中创建 templates 目录。

    [stack@director ~]$ mkdir templates

    b. 将位于 /usr/share/openstack-tripleo-heat-templates/ 的默认模板文件复制到 templates 目录。

    [stack@director ~]$ cp -rf /usr/share/openstack-tripleo-heat-templates/* templates

    c. 检索 templates.tar.bz2 存档（位于 http://materials.example.com/overcloud-heat-templates/templates.tar.bz2），并提取文件。

    [stack@director ~]$ curl -s -O http://materials.example.com/overcloud-heat-templates/templates.tar.bz2
    [stack@director ~]$ tar -xvjpf templates.tar.bz2

4. 扩展环境

通过标记节点并定义其引导映像准备好节点后，将 2 用作计算节点数目来扩展环境。当前的 overcloud 已使用位于 /home/stack/templates 的模板进行了部署。为避免对 overcloud 进行不必要的更改，可在扩展堆栈时重新利用下列选项：

*    参数	                值
*    模板	                /home/stack/templates
*    control-flavor	        control
*    compute-flavor	        compute
*    ceph-storage-flavor	ceph-storage
*    control-scale	        1 日关闭。
*    ceph-storage-scale	    1 日关闭。
*    compute-scale	        2
*    neutron-tunnel-types	vxlan
*    neutron-network-type	vxlan

环境文件	

*    ~/templates/compute-extraconfig.yaml
*    ~/templates/environments/network-isolation.yaml
*    ~/templates/network-environment.yaml
*    ~/templates/environments/storage-environment.yaml
*    ~/templates/pre-config-fix.yaml


    a. 再次利用创建 overcloud 时使用的参数，运行 openstack overcloud 命令。若要添加第二计算节点，可将 2 用于 --compute-scale。

    [stack@director ~]$ openstack overcloud deploy --templates ~/templates \
    > --control-scale 1 --ceph-storage-scale 1 --compute-scale 2 \
    > --control-flavor control --compute-flavor compute --ceph-storage-flavor ceph-storage \
    > --neutron-tunnel-types vxlan --neutron-network-type vxlan \
    > -e ~/templates/compute-extraconfig.yaml \
    > -e ~/templates/environments/network-isolation.yaml \
    > -e ~/templates/network-environment.yaml \
    > -e ~/templates/environments/storage-environment.yaml \
    > -e ~/templates/pre-config-fix.yaml
    Deploying templates in the directory /home/stack/templates/

    > 重要: 完成更新需要 60 分钟。

    b. 在命令运行时，从 workstation.lab.example.com 打开一个新终端。以 stack 用户身份 SSH 连接到 director 节点， 再提供 undercloud Keystone 凭据文件 stackrc。使用 ironic node-list 查询节点的状态，再等待 Provisioning State 显示为 wait call-back。

    [student@workstation ~]$ ssh stack@director
    [stack@director ~]$ source stackrc
    [stack@director ~]$ watch -n 10 ironic node-list
    +--------------------------------------+------------+-------------+--------------------+-------------+
    | UUID                                 | Name       | Power State | Provisioning State | Maintenance |
    +--------------------------------------+------------+-------------+--------------------+-------------+
    ...Output omitted...                                ...Output omitted...
    | fe5d3bd6-5dcd-4c52-984c-5d7a21502504 | compute2   | power on    | wait call-back     | False       |
    ...Output omitted...                                ...Output omitted...
    +--------------------------------------+------------+-------------+--------------------+-------------+

    c. 等待约 5 到 10 分钟。在运行 ironic node-list 命令的终端上，确保状态已从 wait call-back 过渡到 active，然后继续操作。

    +--------------------------------------+------------+-------------+--------------------+-------------+
    | UUID                                 | Name       | Power State | Provisioning State | Maintenance |
    +--------------------------------------+------------+-------------+--------------------+-------------+
    ...Output omitted...                                ...Output omitted...
    | fe5d3bd6-5dcd-4c52-984c-5d7a21502504 | compute2   | power on    | active             | False       |
    ...Output omitted...                                ...Output omitted...
    +--------------------------------------+------------+-------------+--------------------+-------------+

    d. 该 overcloud 是使用虚拟机部署的，而非物理硬件。观察到了争用情形，这可导致部署的元素不一致地挂起。在本课程发布之时，尚无此问题的自动化解决方案。不过，您可通过以下步骤进行检测，并根据需要手动纠正部署。

    解决方案部分通过 /home/stack/templates/pre-config-fix.yaml 环境文件进行实施，该文件在上一步中用于 openstack overcloud deploy 命令。

    从 overcloud-novacompute-1 控制台并使用 root 用户和 ROOTPW 密码进行登录。

    验证文件 /etc/sysconfig/network-scripts/ifcfg-eth* 是否为零字节文件，再检查 /var/log/cloud-init.log 中是否有错误。对 overcloud-novacompute-1 执行这些验证步骤。

    [root@overcloud-novacompute-1 ~]# ls -l /etc/sysconfig/network-scripts/ifcfg-eth*
    -rw-r--r--. 1 root root 0 Sep 10 04:06 /etc/sysconfig/network-scripts/ifcfg-eth0
    [root@overcloud-novacompute-1 ~]# tailf /var/log/cloud-init.log
    ---Output omitted---
    cloud-init: 2016-09-10 00:05:50,867 - util.py[WARNING]: Route info failed: Unexpected error while running command.
    cloud-init: Command: ['netstat', '-rn']
    cloud-init: Exit code: 1
    cloud-init: Reason: -
    cloud-init: Stdout: 'Kernel IP routing table\nDestination     Gateway         Genmask         Flags   MSS Window  irtt Iface\n'
    cloud-init: Stderr: ''
    cloud-init: ci-info: +++++++++++++++++++++++Net device info+++++++++++++++++++++++
    cloud-init: ci-info: +--------+------+-----------+-----------+-------------------+
    cloud-init: ci-info: | Device |  Up  |  Address  |    Mask   |     Hw-Address    |
    cloud-init: ci-info: +--------+------+-----------+-----------+-------------------+
    cloud-init: ci-info: |  lo:   | True | 127.0.0.1 | 255.0.0.0 |         .         |
    cloud-init: ci-info: | eth0:  | True |     .     |     .     | 52:54:00:00:fa:0c |
    cloud-init: ci-info: +--------+------+-----------+-----------+-------------------+
    cloud-init: ci-info: !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!Route info failed!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
    ---Output omitted---

    e. cloud-init 可能会导致 overcloud 部署挂起，因为节点的网络无法访问。Heat 引擎将无法通过访问元数据服务器来配置节点。os-refersh-config用于对 overcloud 节点应用配置的元数据 URL 在/etc/os-collect-config.conf中指定。

    [root@overcloud-novacompute-1 ~]# cat /etc/os-collect-config.conf 
    [DEFAULT]
    command = os-refresh-config

    [cfn]
    metadata_url = http://172.25.250.10:8000/v1/
    stack_name = overcloud-Compute-n2onlrw2eva3-0-zsn5w6hvh3qy
    secret_access_key = 03c62d5e36d94f0cb9225f5e03af1fff
    access_key_id = a683f206faa54f8c8d6de1d903c56d4d
    path = NovaCompute.Metadata

    os-collect-config开始轮询元数据的更改。要查看此问题，请使用journalctl -u os-collect-config。

    [root@overcloud-novacompute-1 ~]# journalctl -u os-collect-config --no-full
    -- Logs begin at Sat 2016-09-10 04:04:05 UTC, end at Sat 2016-09-10 14:21:04 UTC. --
    systemd[1]: Started Collect metadata and run hook commands..
    systemd[1]: Starting Collect metadata and run hook commands....
    WARNING os_collect_config.ec2 [-] ('Connection aborted.', error(101, 'Ne...chable'))
    WARNING os-collect-config [-] Source [ec2] Unavailable.
    WARNING os-collect-config [-] Source [request] Unavailable.
    ---Output omitted--

    f. 删除零字节文件/etc/sysconfig/network-scripts/ifcfg-eth*以解决问题，再使用reboot命令手动重新启动节点。

    > 重要: 此操作仅可在有问题的 overcloud 节点上执行。

    [root@overcloud-novacompute-1 ~]# rm -rf /etc/sysconfig/network-scripts/ifcfg-eth*
    [root@overcloud-novacompute-1 ~]# reboot

    g. 使用 heat 命令，确保已成功更新堆栈。

    [stack@director ~]$ heat stack-list
    +--------------------------------------+------------+-----------------+---------------------+---------------------+
    | id                                   | stack_name | stack_status    | creation_time       | updated_time        |
    +--------------------------------------+------------+-----------------+---------------------+---------------------+
    | a958aaee-d033-4e32-9e99-afed31198c9b | overcloud  | UPDATE_COMPLETE | 2016-02-02T06:42:17 | 2016-02-16T02:53:42 |
    +--------------------------------------+------------+-----------------+---------------------+---------------------+

    h. 使用 openstack 命令，确保已成功创建计算节点 overcloud-novacompute-1。

    [stack@director ~]$ openstack server list
    +--------------------------------------+-------------------------+--------+-------------------------+
    | ID                                   | Name                    | Status | Networks                |
    +--------------------------------------+-------------------------+--------+-------------------------+
    | b15c5576-ab81-447e-be22-5ee67a5ed63f | overcloud-novacompute-1 | ACTIVE | ctlplane=172.25.250.25  |
    | 4c3ae9cb-cfa0-4a2c-b0e9-d0d38d11cfc6 | overcloud-novacompute-0 | ACTIVE | ctlplane=172.25.250.24  |
    | 245e5136-514e-44e2-9fec-2307fd3eb529 | overcloud-controller-0  | ACTIVE | ctlplane=172.25.250.23  |
    | 52937b28-e541-4636-9b69-c839027afba5 | overcloud-cephstorage-0 | ACTIVE | ctlplane=172.25.250.22  |
    +--------------------------------------+-------------------------+--------+-------------------------+

    如上所示，环境现在已包含一个控制器节点、一个存储节点，以及两个计算节点。

5. 启动实例

在 workstation.lab.example.com 上以 student 用户身份提供 ~/overcloudrc 文件，然后在第一计算节点 compute1上使用 small 映像和 m2.small 类别生成实例 lab_test-nodes；将它附加到专用网络 lab_privnet。实例处于运行状态后，从浮动 IP 池 lab_pubnet 为它关联一个浮动 IP。

    a. 在 workstation.lab.example.com 上以 student 用户身份打开一个新终端，再提供位于 /home/student/overcloudrc 下的 Keystone 凭据。

    [student@workstation ~]$ source overcloudrc

    b. 创建实例 lab_test-nodes；将可用区域 nova 与参数 --availability zone 一起传递，从而强行在第一计算节点上创建该实例。

    [student@workstation ~]$ openstack server create \
    > --image small \
    > --flavor m2.small \
    > --nic net-id=lab_privnet \
    > --availability-zone nova:overcloud-novacompute-0.localdomain \
    > --wait \
    > lab_test-nodes
    +---------------------------------+-------------------------------------+
    | Field                           | Value                               |
    +---------------------------------+-------------------------------------+
    ...Output omitted...
    | OS-EXT-SRV-ATTR:host            | overcloud-novacompute-0.localdomain |
    | OS-EXT-SRV-ATTR:
      hypervisor_hostname             | overcloud-novacompute-0.localdomain |
    | OS-EXT-SRV-ATTR:instance_name   | instance-00000007                   |
    ...Output omitted...
    | addresses                       | lab_privnet=192.168.1.17            |
    ...Output omitted...
    +---------------------------------+-------------------------------------+

    c. 确保实例已标记为 ACTIVE。

    [student@workstation ~]$ openstack server list
    +--------------------------------------+----------------+--------+---------------------------+
    | ID                                   | Name           | Status | Networks                  |
    +--------------------------------------+----------------+--------+---------------------------+
    | b86af924-55a6-4366-b1d1-4764b4c7b001 | lab_test-nodes | ACTIVE | lab_privnet=192.168.1.17  |
    +--------------------------------------+----------------+--------+---------------------------+
    
6. 将浮动 IP 关联到实例

关联浮动 IP 后，使用 ping 命令确保您可以访问该实例。

    a. 列出可用的浮动 IP。

    [student@workstation ~]$ openstack ip floating list
    +--------------------------------------+-------------+---------------+----------+-------------+
    | ID                                   | Pool        | IP            | Fixed IP | Instance ID |
    +--------------------------------------+-------------+---------------+----------+-------------+
    | f0e28903-3ed9-4875-9f12-7f73ca25c39c | lab_pubnet  | 172.25.250.51 | None     | None        |
    +--------------------------------------+-------------+---------------+----------+-------------+

    b. 将浮动 IP 172.25.250.51 分配给实例 lab_test-nodes。

    [student@workstation ~]$ openstack ip floating add 172.25.250.51 lab_test-nodes

    c. 使用 ping，尝试访问该实例。

    [student@workstation ~]$ ping -c 3 172.25.250.51
    PING 172.25.250.51 (172.25.250.51) 56(84) bytes of data.
    64 bytes from 172.25.250.51: icmp_seq=1 ttl=63 time=1.49 ms
    64 bytes from 172.25.250.51: icmp_seq=2 ttl=63 time=1.20 ms
    64 bytes from 172.25.250.51: icmp_seq=3 ttl=63 time=0.830 ms

    --- 172.25.250.51 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2002ms
    rtt min/avg/max/mdev = 0.830/1.178/1.495/0.272 ms

7. 迁移实例

使用 configure 动词运行实验脚本 lab nodes-lab，对实例做迁移准备。该脚本将在两个计算节点之间交换 nova 用户的 SSH 密钥，从而在这两个节点上配置迁移。

```
[student@workstation ~]$ lab nodes-lab configure
```

脚本完成后，将实例从第一计算节点 compute1 迁移到第二计算节点 compute2。使用 ping，确保实例 lab_test-nodes 仍可被访问。

    a. 从workstation，提供 overcloud Keystone 凭据文件， overcloudrc。

    [student@workstation ~]$ source overcloudrc

    b. 首先使用命令nova service-list获取运行 Nova 服务的节点列表。

    [student@workstation ~]$ nova service-list --binary nova-compute
    +----+--------------+-------------------------------------+------+---------+-------+----------------------------+-----------------+
    | Id | Binary       | Host                                | Zone | Status  | State | Updated_at                 | Disabled Reason |
    +----+--------------+-------------------------------------+------+---------+-------+----------------------------+-----------------+
    | 5  | nova-compute | overcloud-novacompute-0.localdomain | nova | enabled | up    | 2016-02-22T13:18:46.000000 | -               |
    | 6  | nova-compute | overcloud-novacompute-1.localdomain | nova | enabled | up    | 2016-02-22T13:18:40.000000 | -               |
    +----+--------------+-------------------------------------+------+---------+-------+----------------------------+-----------------+

    c. 检索类别 m2.small 的规格，这是由实例 lab_test-nodes 使用的类别。

    [student@workstation ~]$ openstack flavor show m2.small
    +----------------------------+--------------------------------------+
    | Field                      | Value                                |
    +----------------------------+--------------------------------------+
    | OS-FLV-DISABLED:disabled   | False                                |
    | OS-FLV-EXT-DATA:ephemeral  | 0                                    |
    | disk                       | 10                                   |
    | id                         | 6d89f295-59fe-4f4a-a84c-ca1fd659fa93 |
    | name                       | m2.small                             |
    | os-flavor-access:is_public | True                                 |
    | properties                 |                                      |
    | ram                        | 1024                                 |
    | rxtx_factor                | 1.0                                  |
    | swap                       | 1                                    |
    | vcpus                      | 2                                    |
    +----------------------------+--------------------------------------+

    如上所示，该类别使用 10 GB 磁盘和 1 GB 内存。

    d. 确保第二计算节点 overcloud-novacompute-1 具有充足的资源，可以托管要迁移的实例。使用 -c 标志，将结果筛选为仅显示可用内存和可用磁盘空间。确保可用的磁盘和内存均大于类别 m2.small。

    [student@workstation ~]$ openstack hypervisor show overcloud-novacompute-1.localdomain -c free_ram_mb -c free_disk_gb
    +--------------+-------+
    | Field        | Value |
    +--------------+-------+
    | free_disk_gb | 30    |
    | free_ram_mb  | 6124  |
    +--------------+-------+

    e. 使用openstack server migrate命令迁移实例。

    [student@workstation ~]$ openstack server migrate --shared-migration lab_test-nodes

    f. 迁移需要经过验证后才能完成。检查服务器的状态。

    [student@workstation ~]$ openstack server list
    +--------------------------------------+----------------+---------------+-----------------------------------------+
    | ID                                   | Name           | Status        | Networks                                |
    +--------------------------------------+----------------+---------------+-----------------------------------------+
    | afb6b5e9-8b85-494e-84d4-4c02aad39295 | lab_test-nodes | VERIFY_RESIZE | lab_privnet=192.168.1.17, 172.25.250.51 |
    +--------------------------------------+----------------+---------------+-----------------------------------------+

    以上输出表明实例正在等待验证。

    g. 使用 nova resize-confirm 命令验证迁移。

    [student@workstation ~]$ nova resize-confirm lab_test-nodes

    h. 使用 openstack server show 检查实例已成功迁移。

    [student@workstation ~]$ openstack server show lab_test-nodes
    +--------------------------------------+-------------------------------------+
    | Field                                | Value                               |
    +--------------------------------------+-------------------------------------+
    | OS-DCF:diskConfig                    | MANUAL                              |
    | OS-EXT-AZ:availability_zone          | nova                                |
    | OS-EXT-SRV-ATTR:host                 | overcloud-novacompute-1.localdomain |
    | OS-EXT-SRV-ATTR:hypervisor_hostname  | overcloud-novacompute-1.localdomain |
    | OS-EXT-SRV-ATTR:instance_name        | instance-00000002                   |
    ...Output omitted...

    i. 检查来源和目标计算节点，以及迁移状态。

    [student@workstation ~]$ nova migration-list
    +-------------------------------------+-------------------------------------+-------------------------------------+-------------------------------------+
    | Source Node                         | Dest Node                           | Source Compute                      | Dest Compute                        |
    +-------------------------------------+-------------------------------------+-------------------------------------+-------------------------------------+
    ...Output omitted...
    | overcloud-novacompute-0.localdomain | overcloud-novacompute-1.localdomain | overcloud-novacompute-0.localdomain | overcloud-novacompute-1.localdomain |
    +-------------------------------------+-------------------------------------+-------------------------------------+-------------------------------------+

    ----------------+-----------+--------------------------------------+----------------------------+----------------------------+
     Dest Host      | Status    | Instance UUID                        | Created At                 | Updated At                 |
    ----------------+-----------+--------------------------------------+----------------------------+----------------------------+
     172.24.250.115 | confirmed | afb6b5e9-8b85-494e-84d4-4c02aad39295 | 2016-02-24T11:06:56.000000 | 2016-02-24T11:07:06.000000 |
    ----------------+-----------+--------------------------------------+----------------------------+----------------------------+

    j. 使用ping，确保仍然可通过其公共 IP 172.25.250.51 访问该实例。

    [student@workstation ~]$ ping -c 3 172.25.250.51
    PING 172.25.250.51 (172.25.250.51) 56(84) bytes of data.
    64 bytes from 172.25.250.51: icmp_seq=1 ttl=63 time=0.836 ms
    64 bytes from 172.25.250.51: icmp_seq=2 ttl=63 time=0.731 ms
    64 bytes from 172.25.250.51: icmp_seq=3 ttl=63 time=0.688 ms

    --- 172.25.250.51 ping statistics ---
    3 packets transmitted, 3 received, 0% packet loss, time 2002ms
    rtt min/avg/max/mdev = 0.688/0.751/0.836/0.069 ms


### 总结

在本章中，您可以学到：

*    Director 通过 Heat 实施缩放功能。在需要时，管理员可以重新运行用于部署 overcloud 的命令，来增加或减少他们想要的角色。
*    虽然扩展通常用于向环境添加更多容量，管理员也可缩减环境；例如，不再需要激增期间使用的节点时。
*    新节点添加至环境中后，管理员可以使用可用性区域在其上创建实例。
*    nova 是默认的可用性区域。nova availability-zone-list 命令可列出所有可用性区域，以及它们所分配的角色。
*    Red Hat OpenStack Platform 提供三种迁移类型：非实时迁移、实时迁移，以及块实时迁移。
*    所有节点必须设置有 SSH 密钥身份验证，以便 Nova 服务能够通过 libvirt 使用 SSH 在计算节点之间迁移实例。
