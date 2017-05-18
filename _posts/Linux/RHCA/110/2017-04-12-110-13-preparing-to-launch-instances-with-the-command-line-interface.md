---
layout: post
title:  "使用命令行界面为启动实例做准备(110-13)"
categories: Linux
tags: RHCA 110
---

### 使用命令行界面管理映像

###### 映像管理

每个实例使用模板来创建其虚拟磁盘。此模板称为映像，通常包含最小的软件要求，如可引导的 OS 等。此模板由 Glance 服务存储和提供，该服务管理映像目录，根据其配置提供给部分用户或所有用户，从而在 Red Hat OpenStack Platform 环境中创建和部署新实例。

在后端，负责管理实例部署的服务查询 Glance 服务，以便其提供映像给在实例将要部署到的计算节点上运行的 Nova 组件。包含映像的文件随后将从运行 Glance 的计算机复制到计算节点上，并存储在那里的缓存中。如果有任何其他实例使用该计算节点上的映像启动，将直接从缓存中提供该映像，不必再进行额外的传输。在实例启动过程中，新的虚拟磁盘将扩大到启动该实例时所指定的磁盘大小。

> 在创建映像时，务必要尽可能保持最小的大小，以后再通过利用 cloud-init 等应用或配置管理工具进行自定义。

部署 Glance 服务的计算机上通常也托管其他 Red Hat OpenStack Platform 服务，并与实际计算节点分开。这意味着需要网络带宽来将映像从托管 Glance 服务的计算机传输到计算节点上。根据映像目录策略和映像大小，这在长远上可能会成为问题。例如，如果用户被允许创建自定义映像并上传到目录中，映像流量可能会影响到 Red Hat OpenStack Platform 环境性能。此问题可通过不同的解决方案解决。例如，可使用红帽 Ceph 存储（一种可扩展的分布式存储系统）作为 Glance 的存储后端。

> 如果将红帽 Ceph 存储用作 Glance 服务的存储后端，则创建映像时无法使用 copy-from 选项。先下载映像，然后使用 file 选项。

openstack 命令通过 image 子命令提供映像管理功能。下表提供了其中一些可用的选项。

*    使用 openstack 命令管理映像
*    子命令 	描述
*    image create 	创建新映像
*    image delete 	删除映像
*    image list 	列出映像
*    image show 	显示映像详细信息
*    image set 	设置映像属性（如公共、最小磁盘/RAM）
*    image save 	保存映像

> Red Hat OpenStack Platform 支持将多种格式的映像，包括 QCOW2 和 RAW 格式。

映像关联有若干个属性。它可以是公共或私有映像，具体取决于它是否能被所有人访问。映像也将与某一最小磁盘和 RAM 关联，达到此要求后才能使用该映像创建实例。

###### 使用命令行界面管理映像

1. 在 servera 上，提供 /root/keystonerc_admin 文件，以获取管理员的权限。

    [root@servera ~]$ source /root/keystonerc_admin
    [root@servera ~(keystone_admin)]$ 

2. 列出当前可用的映像。应该会列出 demoimage 映像。

    [root@servera ~(keystone_admin)]$ openstack image list
    +--------------------------------------+-----------+
    | ID                                   | Name      |
    +--------------------------------------+-----------+
    | 3b99a245-9b4d-4e7f-a3eb-b99fec180e2e | demoimage |
    +--------------------------------------+-----------+

3. 显示 demoimage 的详细信息。确认该映像是公共映像，即 visibility 设置是 private。

    [root@servera ~(keystone_admin)]$ openstack image show demoimage
    +------------------+------------------------------------------------------+
    | Field            | Value                                                |
    +------------------+------------------------------------------------------+
    | checksum         | f9d8a0f77fcad5c280a400a4097018c4                     |
    | container_format | bare                                                 |
    | created_at       | 2016-01-17T16:11:46Z                                 |
    | disk_format      | qcow2                                                |
    | file             | /v2/images/da2c73fe-62ed-4aae-9703-c9c9621e6298/file |
    | id               | da2c73fe-62ed-4aae-9703-c9c9621e6298                 |
    | min_disk         | 0                                                    |
    | min_ram          | 0                                                    |
    | name             | demoimage                                            |
    | owner            | 58a9e2ca484a472fa268880245dac358                     |
    | protected        | False                                                |
    | schema           | /v2/schemas/image                                    |
    | size             | 554980352                                                 |
    | status           | active                                               |
    | updated_at       | 2016-01-17T16:11:47Z                                 |
    | virtual_size     | None                                                 |
    | visibility       | private                                               |
    +------------------+------------------------------------------------------+

4. 更改 demoimage 映像的 visibility 设置，使得该映像成为公共映像。确认 visibility 设置现在为 public。

    [root@servera ~(keystone_admin)]$ openstack image set --public demoimage
    +------------------+------------------------------------------------------+
    | Field            | Value                                                |
    +------------------+------------------------------------------------------+
    | checksum         | f9d8a0f77fcad5c280a400a4097018c4                     |
    | container_format | bare                                                 |
    | created_at       | 2016-01-17T16:11:46Z                                 |
    | disk_format      | qcow2                                                |
    | file             | /v2/images/da2c73fe-62ed-4aae-9703-c9c9621e6298/file |
    | id               | da2c73fe-62ed-4aae-9703-c9c9621e6298                 |
    | min_disk         | 0                                                    |
    | min_ram          | 0                                                    |
    | name             | demoimage                                            |
    | owner            | 58a9e2ca484a472fa268880245dac358                     |
    | protected        | False                                                |
    | schema           | /v2/schemas/image                                    |
    | size             | 554980352                                                 |
    | status           | active                                               |
    | tags             | []                                                   |
    | updated_at       | 2016-01-17T16:11:47Z                                 |
    | virtual_size     | None                                                 |
    | visibility       | public                                               |
    +------------------+------------------------------------------------------+

5. 提供 /root/keystonerc_demouser 文件，以获取 demouser 的权限。

    [root@servera ~(keystone_admin)]$ source /root/keystonerc_demouser
    [root@servera ~(keystone_demouser)]$ 

6. 确认 demoimage 映像已列为可供 demouser 使用，因为它是公共映像。

    [root@servera ~(keystone_demouser)]$ openstack image list
    +--------------------------------------+-----------+
    | ID                                   | Name      |
    +--------------------------------------+-----------+
    | 3b99a245-9b4d-4e7f-a3eb-b99fec180e2e | demoimage |
    +--------------------------------------+-----------+

7. 提供 /root/keystonerc_admin 文件，以获取管理员权限。

    [root@servera ~(keystone_demouser)]$ source /root/keystonerc_admin
    [root@servera ~(keystone_admin)]$ 

8. 将 demoimage 映像保存到 /tmp/demoimage.img 文件。检查映像是否已正确保存。

    [root@servera ~(keystone_admin)]$ openstack image save --file /tmp/demoimage.img demoimage
    [root@servera ~(keystone_admin)]$ ll /tmp/demoimage.img 
    -rw-r--r--. 1 root root 554980352 Nov  2 09:41 /tmp/demoimage.img

9. 删除 demoimage 映像。

    [root@servera ~(keystone_admin)]$ openstack image delete demoimage


### 使用命令行界面管理密钥对

默认情况下，Red Hat OpenStack Platform 部署的实例使用密钥对来允许用户访问这些实例。用户可以创建密钥对，在实例创建时与之关联。实例引导流程将把公钥注入到用户 authorized_keys 中，从而让用户能够使用关联的私钥通过 SSH 登录。

用户可以创建多个密钥对，在实例创建时与不同实例关联。密钥对与实例关联之后，如果私钥丢失，则实例必须重新创建后才能注入不同的密钥对。

可以使用预配置用户密码的方式来创建映像，但不建议这种配置，因为它不支持 Red Hat OpenStack Platform 所基于的自助服务模型。

> 将要注入公钥的用户取决于映像。红帽企业 Linux 映像将公钥注入到 cloud-user 用户。

*    使用 openstack 命令管理密钥对
*    子命令 	            描述
*    keypair create 	创建新密钥对
*    keypair delete 	删除密钥对
*    keypair list 	    列出密钥对
*    keypair show 	    显示密钥对详细信息

通过 openstack keypair 命令创建密钥对时，该命令的输出是私钥，用户需要使用该私钥来通过 SSH 访问实例。为存储私钥供以后使用，可以按照如下所示进行重定向。

    [root@servera ~(keystonerc_demouser)]$ openstack keypair create demokeypair > /tmp/demoprivatekey

###### 使用命令行界面管理密钥对

1. 在 servera 上，提供 /root/keystonerc_admin 文件，以获取管理员的权限。

    [root@servera ~]$ source /root/keystonerc_admin
    [root@servera ~(keystone_admin)]$ 

2. 使用 openstack keypair create 命令创建一个新密钥对，命名为 demokeypair。将关联的私钥存储在 /tmp/demoprivatekey.pem 文件中。

    [root@servera ~(keystone_admin)]$ openstack keypair create demokeypair > /tmp/demoprivatekey.pem
          
3. 使用 openstack keypair list 命令，列出可用的密钥对。其中应当列有 demokeypair。

    [root@servera ~(keystone_admin)]$ openstack keypair list
    +-------------+-------------------------------------------------+
    | Name        | Fingerprint                                     |
    +-------------+-------------------------------------------------+
    | demokeypair | cf:9a:13:83:f1:8c:d9:19:39:65:47:ec:55:60:15:ed |
    +-------------+-------------------------------------------------+

4. 使用 openstack keypair show 命令，显示 demokeypair 密钥对的详细信息。

    [root@servera ~(keystone_admin)]$ openstack keypair show demokeypair
    +-------------+-------------------------------------------------+
    | Field       | Value                                           |
    +-------------+-------------------------------------------------+
    | created_at  | 2015-11-04T06:10:48.000000                      |
    | deleted     | False                                           |
    | deleted_at  | None                                            |
    | fingerprint | cf:9a:13:83:f1:8c:d9:19:39:65:47:ec:55:60:15:ed |
    | id          | 1                                               |
    | name        | demokeypair                                     |
    | updated_at  | None                                            |
    | user_id     | 232ad8758b6c4081b29229490f86a538                |
    +-------------+-------------------------------------------------+

5. 使用 openstack keypair delete 命令，删除 demokeypair 密钥对。

    [root@servera ~(keystone_admin)]$ openstack keypair delete demokeypair
          
6. 删除关联的私钥。

    [root@servera ~(keystone_admin)]$ rm -f /tmp/demoprivatekey.pem

### 总结

在本章中，您可以学到：

*    在 Red Hat OpenStack Platform 中，实例的虚拟磁盘基于称为映像的模板。
*    这些映像由 Glance 服务管理，可以在实例部署时放大，将自身调节到所用类别中指定的大小。
*    借助密钥对，可以实现通过配置了所注入密钥的用户对实例进行免密码访问。配置的用户取决于所使用的映像。在红帽企业 Linux 映像上，该用户是 root 用户。
