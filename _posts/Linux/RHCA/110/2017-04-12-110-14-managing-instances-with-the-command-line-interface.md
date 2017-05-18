---
layout: post
title:  "使用命令行界面管理实例(110-14)"
categories: Linux
tags: RHCA 110
---

### 使用命令行界面启动实例

实例由用户部署在项目中。在默认的 PackStack 部署中，实例只能看到部署在同一项目中的实例。如果通过连入外部网络的路由器为该项目启用了外部访问，实例便能够访问该外部网络。

类别是分配到实例的一组资源（如磁盘/内存）。若要部署实例，至少需要配置类别和映像。默认情况下，Red Hat OpenStack Platform 中已配置了一组类别，但目前而言没有配置映像。

> 若要将类别与映像搭配使用，该类别必须满足映像的最低要求（如最小磁盘和 RAM）。

通过 openstack server image create 命令，可以创建用于启动实例的映像。此命令与 openstack image create 的作用相同。通过 openstack server 还可以使用其他命令，如下表中所列。

*    使用 openstack 命令管理实例
*    子命令 	        描述
*    server create 	创建新实例
*    server delete 	删除实例
*    server list 	列出实例
*    server show 	显示实例详细信息
*    server set 	设置实例属性（如 root 密码）

启动实例部署时，Red Hat OpenStack Platform 的计算服务 Nova 将搜索资源足以部署该实例的计算节点。实例要求的资源在使用的类别中指定。选择了计算节点后，映像将传输到此节点上，使它增大到所需的虚拟磁盘大小。完成后，实例便会启动。

如果未做其他指定，实例将部署在默认安全组中。默认情况下，此安全组将允许实例发出的所有传出流量，也允许来自部署在默认安全组中其他实例的传入流量。要启用外部来源（如 SSH）的传入流量，应当向默认安全组添加相应的规则。可以创建新的安全组，按照实例要求对它们自定义，然后使用它们来启动新实例。

###### 使用命令行界面启动实例

1. 在 servera 上，提供 /root/keystonerc_demouser 文件，以获取 demouser 的权限。

    [root@servera ~]$ source /root/keystonerc_demouser
    [root@servera ~(keystone_demouser)]$ 

2. 创建一个名为 demovm 的新实例，该实例使用类别 m1.small、密钥对 demokeypair 和映像 demoimage。

    [root@servera ~(keystone_demouser)]$ openstack server create --flavor m1.small --key-name demokeypair --image demoimage demovm
    +--------------------------------------+--------------------------------------------------+
    | Field                                | Value                                            |
    +--------------------------------------+--------------------------------------------------+
    ...
    | flavor                               | m1.small (2)                                     |
    | hostId                               |                                                  |
    | id                                   | 7b1d7109-4670-4847-bd24-475e34a9eaa5             |
    | image                                | demoimage (d3f8c7cb-8fcd-45b0-92cb-7e614f9ca1f4) |
    | key_name                             | demokeypair                                      |
    | name                                 | demovm                                           |
    | os-extended-volumes:volumes_attached | []                                               |
    | progress                             | 0                                                |
    | project_id                           | 9d5ed8c671a04305b5c1b09414ef683c                 |
    | properties                           |                                                  |
    | security_groups                      | [{u'name': u'default'}]                          |
    ...
    +--------------------------------------+--------------------------------------------------+

3. 列出可用的网络。应当显示两个网络，即 demonet 和 ext。默认情况下，如果未指定网络，并且相关项目中只有一个内部网络（如上例中所示），实例 demovm 将部署到内部网络 demonet 中。

    [root@servera ~(keystone_demouser)]$ openstack network list
    +--------------------------------------+---------+--------------------------------------+
    | ID                                   | Name    | Subnets                              |
    +--------------------------------------+---------+--------------------------------------+
    | e881fed3-6f3c-4dd6-b1bf-aaf97c17b70b | ext     | a747f001-1889-48ce-afc7-dde643e8174b |
    | ab741b1f-4aec-42fb-ae3d-fcf805df517e | demonet | 8137acdb-d8ca-4d0f-9946-0acdc5c62ef7 |
    +--------------------------------------+---------+--------------------------------------+

4. 启动名为 seconddemovm 的另一个实例，该实例使用类别 m1.small、密钥对 demokeypair 和映像 demoimage。使用 nic 参数，也指定该实例应当要部署的网络。当存在多个可用的内部网络时，可以使用此参数。

    [root@servera ~(keystone_demouser)]$ openstack server create --flavor m1.small --key-name demokeypair --image demoimage --nic net-id=demonet seconddemovm
    +--------------------------------------+--------------------------------------------------+
    | Field                                | Value                                            |
    +--------------------------------------+--------------------------------------------------+
    ...
    | flavor                               | m1.small (2)                                     |
    | hostId                               |                                                  |
    | id                                   | 97c30c30-b559-4341-8a1e-f2075b1988e1             |
    | image                                | demoimage (d3f8c7cb-8fcd-45b0-92cb-7e614f9ca1f4) |
    | key_name                             | demokeypair                                      |
    | name                                 | seconddemovm                                     |
    | os-extended-volumes:volumes_attached | []                                               |
    | progress                             | 0                                                |
    | project_id                           | 9d5ed8c671a04305b5c1b09414ef683c                 |
    | properties                           |                                                  |
    | security_groups                      | [{u'name': u'default'}]                          |
    ...
      +--------------------------------------+--------------------------------------------------+

5. 从默认浮动 IP 池 ext 中新建一个浮动 IP。

    [root@servera ~(keystone_demouser)]$ openstack ip floating create ext
    +-------------+--------------------------------------+
    | Field       | Value                                |
    +-------------+--------------------------------------+
    ...                        
    | ip          | 172.25.250.27                        |
    | pool        | ext                                  |
    +-------------+--------------------------------------+

6. 将这个新浮动 IP 关联到 demovm 实例。

    [root@servera ~(keystone_demouser)]$ openstack ip floating add 172.25.250.27 demovm
            
7. 使用 /root/demokeypair.pem 文件中与密钥对 demokeypair 关联的私钥，通过实例关联的 IP 打开连接到实例 demovm 的 SSH 会话。连接之后，从该实例退出。

    [root@servera ~(keystone_demouser)]$ ssh -i /root/demokeypair.pem cloud-user@172.25.250.27
    The authenticity of host '172.25.250.27 (172.25.250.27)' can't be established.
    ECDSA key fingerprint is 55:03:93:b6:bb:29:73:4c:aa:7e:0a:b9:9e:14:18:a5.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added '172.25.250.27' (ECDSA) to the list of known hosts.
    [cloud-user@demovm ~]$ exit 

8. 删除 demovm 和 seconddemovm 实例。

    [root@servera ~(keystone_demouser)]$ openstack server delete demovm
    [root@servera ~(keystone_demouser)]$ openstack server delete seconddemovm


### 使用命令行界面自定义实例

Red Hat OpenStack Platform 是自助维护型服务，提供一系列可满足此环境用户常规要求的映像。理想情况下，这些映像的软件和配置保持在最小程度，使映像大小尽可能小，以免影响 Red Hat OpenStack Platform 环境性能。这些映像通常在创建时需要自定义，从而达到能够让用户进行最少自定义配置的要求。

Red Hat OpenStack Platform 提供了两种工具（文件注入和元数据基础脚本），以便在通过 openstack server create 命令创建实例时实现这样的目标。文件注入功能通过 openstack server create 命令的文件参数提供，允许将文件注入到实例文件系统树中的某一位置上。

Red Hat OpenStack Platform 计算服务 Nova 提供了元数据服务，实例在引导时可以使用此服务来检索特定的配置。此服务可以在任何时间从实例访问，它也关联有一组参数。使用 user-data 参数运行 openstack server create 命令时，可以在实例创建时调配这些实例参数及其关联的值，这样它们便可与实例引导进程（如通过 rc.local）一同使用来自定义实例。这种自定义方法需要映像配置为利用与实例关联的元数据参数。

> 与 user-data 相关的问题可以在位于 /var/log/cloud-init.log 的日志中检查。

###### 使用命令行界面自定义实例

1. 在 servera 上，提供 /root/keystonerc_demouser 文件，以获取 demouser 的权限。

    [root@servera ~]$ source /root/keystonerc_demouser
    [root@servera ~(keystone_demouser)]$ 

2. 创建一个名为 demovm 的新实例，该实例使用类别 m1.small、密钥对 demokeypair 和映像 demoimage。

    [root@servera ~(keystone_demouser)]$ openstack server create --flavor m1.small --key-name demokeypair --image demoimage demovm
    +--------------------------------------+--------------------------------------------------+
    | Field                                | Value                                            |
    +--------------------------------------+--------------------------------------------------+
    ...
    | flavor                               | m1.small (2)                                     |
    | hostId                               |                                                  |
    | id                                   | 7b1d7109-4670-4847-bd24-475e34a9eaa5             |
    | image                                | demoimage (d3f8c7cb-8fcd-45b0-92cb-7e614f9ca1f4) |
    | key_name                             | demokeypair                                      |
    | name                                 | demovm                                           |
    | os-extended-volumes:volumes_attached | []                                               |
    | progress                             | 0                                                |
    | project_id                           | 9d5ed8c671a04305b5c1b09414ef683c                 |
    | properties                           |                                                  |
    | security_groups                      | [{u'name': u'default'}]                          |
    ...
    +--------------------------------------+--------------------------------------------------+

3. 从默认浮动 IP 池 ext 中新建一个浮动 IP。

    [root@servera ~(keystone_demouser)]$ openstack ip floating create ext
    +-------------+--------------------------------------+
    | Field       | Value                                |
    +-------------+--------------------------------------+
    ...
    | ip          | 172.25.250.27                        |
    | pool        | ext                                  |
    +-------------+--------------------------------------+

4. 将这个新浮动 IP 关联到 demovm 实例。

    [root@servera ~(keystone_demouser)]$ openstack ip floating add 172.25.250.27 demovm
          
5. 对 demovm 实例启动 HTTP 查询。由于没有 Web 服务器绑定到端口 80/TCP，应当会返回错误。

    [root@servera ~(keystone_demouser)]$ curl http://172.25.250.27
    curl: (7) Failed connect to 172.25.250.27:80; No route to host

6. 删除 demovm 实例，以释放环境中的资源。这也会同时释放关联的浮动 IP。

    [root@servera ~(keystone_demouser)]$ openstack server delete demovm
          
7. 在 /root 目录中创建名为 install-httpd 的新文件，其包含以下内容：检查此文件开头的 #!/bin/sh 字符串，所有脚本都需要此字符串才能在引导时通过 user-data 参数执行。此脚本将安装和启动 Apache HTTPD 服务。

    #!/bin/sh
    echo "nameserver 172.25.254.254" >> /etc/resolv.conf
    yum -y install httpd
    systemctl start httpd

8. 创建一个名为 demovm 的新实例，该实例使用类别 m1.small、密钥对 demokeypair 和映像 demoimage。将 install-httpd 包含在 user-data 参数中，让它能够在实例引导时执行。

    [root@servera ~(keystone_demouser)]$ openstack server create --flavor m1.small --key-name demokeypair --user-data /root/install-httpd --image demoimage demovm
    +--------------------------------------+--------------------------------------------------+
    | Field                                | Value                                            |
    +--------------------------------------+--------------------------------------------------+
    ...
    | flavor                               | m1.small (2)                                     |
    | hostId                               |                                                  |
    | id                                   | 7b1d7109-4670-4847-bd24-475e34a9eaa5             |
    | image                                | demoimage (d3f8c7cb-8fcd-45b0-92cb-7e614f9ca1f4) |
    | key_name                             | demokeypair                                      |
    | name                                 | demovm                                           |
    | os-extended-volumes:volumes_attached | []                                               |
    | progress                             | 0                                                |
    | project_id                           | 9d5ed8c671a04305b5c1b09414ef683c                 |
    | properties                           |                                                  |
    | security_groups                      | [{u'name': u'default'}]                          |
    ...
    +--------------------------------------+--------------------------------------------------+

9. 将之前使用过的浮动 IP 地址关联到 demovm 实例，因为它现在应当因为实例移除而被取消关联。

    [root@servera ~(keystone_demouser)]$ openstack ip floating add 172.25.250.27 demovm
          
10. 对 demovm 实例启动 HTTP 查询。它现在应当返回 Apache HTTPD 服务器的默认 index.html 网页。

    [root@servera ~(keystone_demouser)]$ curl http://172.25.250.27
    <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

### 总结

在本章中，您可以学到：

*    部署的实例需要映像、密钥对和网络，才能在 Red Hat OpenStack Platform 环境中使用。
*    默认情况下，如果没有指定安全组，实例将被放入默认安全组。
*    借助 user-data 参数使用脚本或通过 file 参数在实例中注入文件，可以在实例引导时对其自定义。
