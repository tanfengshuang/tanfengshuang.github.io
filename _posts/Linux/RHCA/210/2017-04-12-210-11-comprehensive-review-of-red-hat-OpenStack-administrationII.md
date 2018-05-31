---
layout: post
title:  "(210-11)"
categories: Linux
tags: RHCA 210
---

### 红帽 OpenStack 管理二总复习

总复习划分为三个部分。本章的主要目标是部署和配置 OpenStack 环境，以便能在云中提供服务。以下列表提供了每个实验的概述。

*    利用 OpenStack 统一命令行界面 (CLI)，部署 Web 服务器和部署 SSH 管理服务器，它们将用于登录 Web 实例。
*    利用 Heat 模板部署类似于上一实验的环境。利用 Heat 模板，部署 Web 服务器和部署 SSH 管理服务器，它们将用于登录 Web 实例。
*    部署 overcloud（给定 undercloud 设备）。


### 实验：利用 CLI 部署实例

在总复习的第二部分中，您将利用 CLI 部署 Web 服务器 prodwebvm 和 SSH 管理服务器 prodsshvm，它们将用于登录 prodwebvm 实例。您还要配置与这些实例关联的安全组。为部署这些实例，需要使用一些虚拟资源（如网络）。这些资源已由实验脚本部署，可以在环境中使用。下表详细列出了规格和要求：

prodwebvm 实例的配置

*    映像	    prodimage
*    类别	    m2.small
*    网络	    prodnet
*    密钥对	    prodkeypair
*    私钥（与密钥对关联）	/home/student/lab/prodkeypair.pem
*    安全组	    prodwebsg

prodsshvm 实例的配置

*    映像	    prodimage
*    类别	    m2.small
*    网络	    prodnet
*    密钥对	    prodkeypair
*    私钥（与密钥对关联）	/home/student/lab/prodkeypair.pem
*    安全组	    prodsshsg

prodwebsg 安全组的配置

*    端口	    开放对象
*    TCP/22	    prodsshsg 实例
*    TCP/80	    任何位置

prodsshsg 安全组的配置

*    端口	    开放对象
*    22/TCP	    任何位置

1. 在 workstation 上，以 student 用户身份提供 /home/student/overcloudrc 凭据文件，以获取 admin 凭据。

    a. 在 workstation 上打开一个新终端，再提供 /home/student/overcloudrc 文件，以获取 admin 凭据。

    [student@workstation ~]$ source /home/student/overcloudrc
    [student@workstation ~]$ 

2. 新建一个安全组，命名为 prodsshsg。

    a. 新建一个安全组，命名为 prodsshsg。

    [student@workstation ~]$ openstack security group create prodsshsg
    +-------------+--------------------------------------+
    | Field       | Value                                |
    +-------------+--------------------------------------+
    | description | prodsshsg                            |
    | id          | 988f7c90-ea6d-4e32-8a62-2ae004b4c5a8 |
    | name        | prodsshsg                            |
    | rules       | []                                   |
    | tenant_id   | 4b44f462430a4f9cbd2950f6c31546ab     |
    +-------------+--------------------------------------+

3. 在 prodsshsg 安全组中新建一条规则，以允许从任何位置到 TCP/22 端口的入口流量。

    a. 在 prodsshsg安全组内创建一条规则，它使用 tcp 作为协议、0.0.0.0/0 作为来源 IP 地址，以及端口 22 (SSH) 作为目的地端口。

    [student@workstation ~]$ openstack security group rule create --proto tcp --src-ip 0.0.0.0/0 --dst-port 22 prodsshsg
    +-----------------+--------------------------------------+
    | Field           | Value                                |
    +-----------------+--------------------------------------+
    | group           | {}                                   |
    | id              | 32274213-dc88-4cd2-888a-2179a723ca46 |
    | ip_protocol     | tcp                                  |
    | ip_range        | 0.0.0.0/0                            |
    | parent_group_id | 988f7c90-ea6d-4e32-8a62-2ae004b4c5a8 |
    | port_range      | 22:22                                |
    +-----------------+--------------------------------------+

4. 新建一个安全组，命名为 prodwebsg。

    a. 新建一个安全组，命名为 prodwebsg。

    [student@workstation ~]$ openstack security group create prodwebsg
    +-------------+--------------------------------------+
    | Field       | Value                                |
    +-------------+--------------------------------------+
    | description | prodwebsg                            |
    | id          | 5b9fdc7a-025d-4b6c-b4d0-dfb825f3be7a |
    | name        | prodwebsg                            |
    | rules       | []                                   |
    | tenant_id   | 4b44f462430a4f9cbd2950f6c31546ab     |
    +-------------+--------------------------------------+

5. 在 prodwebsg 安全组中新建两条规则。第一条规则将允许从任何位置到 TCP/80 端口的入口流量，第二条规则将允许从与 prodsshsg 安全组关联的实例到 TCP/22 端口的入口流量。

    a. 在 prodwebsg 安全组内创建一条规则，它使用 tcp 作为协议、0.0.0.0/0 作为来源 IP 地址，以及 80 端口作为目的地端口。

    [student@workstation ~]$ openstack security group rule create --proto tcp --src-ip 0.0.0.0/0 --dst-port 80 prodwebsg
    +-----------------+--------------------------------------+
    | Field           | Value                                |
    +-----------------+--------------------------------------+
    | group           | {}                                   |
    | id              | 36bee3f2-3adb-4a0a-a003-1046d0157553 |
    | ip_protocol     | tcp                                  |
    | ip_range        | 0.0.0.0/0                            |
    | parent_group_id | 5b9fdc7a-025d-4b6c-b4d0-dfb825f3be7a |
    | port_range      | 80:80                                |
    +-----------------+--------------------------------------+

    b. 在 prodwebsg 安全组内创建另一条规则，它使用 tcp 作为协议、prodsshsg 作为来源安全组，以及端口 22 作为目的地端口。

    [student@workstation ~]$ nova secgroup-add-group-rule prodwebsg prodsshsg tcp 22 22
    +-------------+-----------+---------+----------+--------------+
    | IP Protocol | From Port | To Port | IP Range | Source Group |
    +-------------+-----------+---------+----------+--------------+
    | tcp         | 22        | 22      |          | prodsshsg    |
    +-------------+-----------+---------+----------+--------------+

6. 新建一个实例，命名为 prodsshvm。将 prodsshvm 关联到 prodsshsg 安全组。使用 prodimage 映像、m2.small 类别、prodkeypair 密钥对，以及 prodnet 网络。从 ext 池中分配一个新的浮动 IP，并将它关联到 prodsshvm 实例。

    a. 创建一个名为 prodsshvm 实例，该实例使用 prodsshsg 安全组。使用 prodimage 映像、m2.small 类别、prodkeypair 密钥对，以及 prodnet 网络。

    [student@workstation ~]$ openstack server create --flavor m2.small --security-group prodsshsg --image prodimage --key-name prodkeypair --nic net-id=prodnet prodsshvm --wait
    +-------------------+----------------------------+
    | Field             | Value                      |
    +-------------------+----------------------------+
    ...Output omitted...
    | flavor            | m2.small (2)               |
    ...Output omitted...
    | image             | prodimage (...)            |
    | key_name          | prodkeypair                |
    | name              | prodsshvm                  |
    ...Output omitted...
    | security_groups   | [{u'name': u'prodsshsg'}]  |
    ...Output omitted...
    +-------------------+-----------------------------+

    b. 从 ext 浮动 IP 池分配一个浮动 IP。

    [student@workstation ~]$ openstack ip floating create ext
    +-------------+--------------------------------------+
    | Field       | Value                                |
    +-------------+--------------------------------------+
    | fixed_ip    | None                                 |
    | id          | 79ec2726-9769-4db4-adcf-2cf22dd7fab0 |
    | instance_id | None                                 |
    | ip          | 172.25.250.27                        |
    | pool        | ext                                  |
    +-------------+--------------------------------------+

    c. 将前面获取的浮动 IP 关联到 prodsshvm 实例。

    [student@workstation ~]$ openstack ip floating add 172.25.250.27 prodsshvm

7. 再新建一个实例，命名为 prodwebvm。将 prodwebvm 关联到 prodwebsg 安全组。使用 prodimage 映像、m2.small 类别、prodkeypair 密钥对，以及 prodnet 网络。从 ext 池中分配一个新的浮动 IP，并将它关联到 prodwebvm 实例。

    a. 创建一个名为 prodwebvm 的实例，该实例使用 prodwebsg 安全组。使用 prodimage 映像、m2.small 类别、prodkeypair 密钥对，以及 prodnet 网络。

    [student@workstation ~]$ openstack server create --flavor m2.small --security-group prodwebsg --image prodimage --key-name prodkeypair --nic net-id=prodnet prodwebvm --wait
    +--------------------+----------------------------+
    | Field              | Value                      |
    +--------------------+----------------------------+
    ...Output omitted...
    | flavor             | m2.small (2)               |
    ...Output omitted...
    | image              | prodimage (...) |
    | key_name           | prodkeypair                |
    | name               | prodwebvm                  |
    ...Output omitted...
    | security_groups    | [{u'name': u'prodwebsg'}]  |
    ...Output omitted...
    +--------------------+----------------------------+

    b. 从 ext 浮动 IP 池分配另一个浮动 IP。

    [student@workstation ~]$ openstack ip floating create ext
    +-------------+--------------------------------------+
    | Field       | Value                                |
    +-------------+--------------------------------------+
    | fixed_ip    | None                                 |
    | id          | 73848fc3-c9b6-4f33-b113-8ede35e1bd03 |
    | instance_id | None                                 |
    | ip          | 172.25.250.28                        |
    | pool        | ext                                  |
    +-------------+--------------------------------------+

    c. 将前面获取的浮动 IP 关联到 prodwebvm 实例。

    [student@workstation ~]$ openstack ip floating add 172.25.250.28 prodwebvm

8. 为 prodwebvm 实例获取专用 IP 地址。它将在 192.168.1.0/24 子网中。

    a. 列出当前部署的两个实例的详细信息。将显示 prodwebvm 实例的专用 IP 地址。我们将使用此专用 IP 地址测试从 SSH 服务器到 Web 服务器的 SSH 连接。

    [student@workstation ~]$ openstack server list
    +--------------------------------------+-----------+--------+--------------------------------------+
    | ID                                   | Name      | Status | Networks                             |
    +--------------------------------------+-----------+--------+--------------------------------------+
    | 3840437e-f32f-45c5-aaf0-4d6d851f5407 | prodwebvm | ACTIVE | prodnet=192.168.1.28, 172.25.250.28 |
    | 99118fce-529f-4085-8653-511eaa8b86e5 | prodsshvm | ACTIVE | prodnet=192.168.1.27, 172.25.250.27 |
    +--------------------------------------+-----------+--------+--------------------------------------+
    
9. 使用与 prodkey 密钥对关联的私钥 /home/student/lab/prodkeypair.pem 并通过 SSH 连接实例，验证 prodsshvm 实例中 TCP/22 的访问权限已经开启。成功登录后退出。

    a. 使用与密钥对 prodkeypair 关联的私钥 /home/student/lab/prodkeypair.pem，打开与 prodsshvm 实例的 SSH 会话。它应当成功，因为 prodsshsg 允许从任何位置到 prodsshvm 实例的端口 22/TCP (SSH) 的入口流量。完成时注销。

    [student@workstation ~]$ ssh -o ConnectTimeout=5 -i /home/student/lab/prodkeypair.pem cloud-user@172.25.250.27
    [cloud-user@prodsshvm ~]$ logout
    Connection to 172.25.250.27 closed.
    [student@workstation ~]$ 

10. 使用与 prodkey 密钥对关联的私钥 /home/student/lab/prodkeypair.pem，验证 prodwebvm 实例中 TCP/22 端口的访问权限已经关闭。

    a. 使用与密钥对 prodkeypair 关联的私钥 /home/student/lab/prodkeypair.pem，尝试打开与 prodwebvm 实例的 SSH 会话。它将失败。SSH 超时会将您返回到提示符。

    [student@workstation ~]$ ssh -o ConnectTimeout=5 -i /home/student/lab/prodkeypair.pem cloud-user@172.25.250.28
    [student@workstation ~]$ 

11. 使用与 prodkey 密钥对关联的私钥 /home/student/lab/prodkeypair.pem，验证从 prodsshsg 安全组到 prodwebvm 实例中端口 TCP/22 的访问权限已经开启。您需要将此文件从 workstation 复制到 prodsshvm 实例。

    a. 以 cloud-user 用户身份将与密钥对 prodkeypair 关联的私钥 /home/student/lab/prodkeypair.pem 复制到 prodsshvm 实例。

    [student@workstation ~]$ scp -i ~/lab/prodkeypair.pem ~/lab/prodkeypair.pem cloud-user@172.25.250.27:~
    prodkeypair.pem                            100% 1684     1.6KB/s   00:00

    b. 打开与 prodsshvm 实例的 SSH 会话；从该处，使用与密钥对 prodkeypair 关联的私钥 /home/student/lab/prodkeypair.pem 以及前面获取的专用 IP 地址，打开与 prodwebvm 的 SSH 会话。它应当成功，因为 prodwebsg 允许从与 prodsshsg 安全组关联的实例到 prodwebvm 实例的 TCP/22 (SSH) 端口的入口流量。完成时注销。

    [student@workstation ~]$ ssh -i /home/student/lab/prodkeypair.pem cloud-user@172.25.250.27
    [cloud-user@prodsshvm ~]$ ssh -i /home/cloud-user/prodkeypair.pem cloud-user@192.168.1.28
    [cloud-user@prodwebvm ~]$ logout
    Connection to 192.168.1.28 closed.
    [cloud-user@prodsshvm ~]$ logout
    Connection to 172.25.250.27 closed.
    [student@workstation ~]$ 

### 实验：使用 Heat 部署堆栈

在总复习的第三部分中，您将检索一个 Heat 模板文件，该文件可创建 Web 服务器 lab_webvm 和 SSH 管理服务器 lab_sshvm，它们将用于登录 lab_webvm 实例。该环境将提供有下列资源：

*    Keystone 凭据文件，位于/home/student/overcloudrc下。
*    Glance 映像 small。
*    Nova 类别 m2.small。
*    公共网络 lab_pubnet。
*    专用网络 lab_privnet。
*    专用子网 lab_pubsub，位于 172.25.250.0/24 范围内。该网络用于提供外部网络连接。
*    专用子网 lab_privsub，位于 192.168.1.0/24 范围内。该网络运行供实例使用的 DHCP 服务器。
*    路由器 lab_router1 与公共网络和专用网络连接，确保外部连接。
*    密钥对 lab_key1。其私钥将位于/home/student/lab/lab_key1.pem下。
*    lab_sshsg 安全组，与 lab_sshvm 实例相关联。它允许从任何位置访问 TCP/22 端口。
*    lab_websg 安全组，与 lab_webvm 实例相关联。它允许从任何位置访问 TCP/80 端口，并且仅允许从与 lab_sshsg 安全组关联的实例访问 TCP/22 端口。

本实验的工作目录是/home/student/lab/。按照下表启动 Heat 堆栈：

堆栈设置

*    堆栈名称	    comprehensive-s2
*    环境文件	    /home/student/lab/comprehensive-s2-env.yaml

1. 准备堆栈

从 director.lab.example.com，以 stack 用户身份跳转到工作目录 lab，并检索位于 http://materials.example.com/heat/comprehensive-s2.yaml 下的 HOT 模板文件。将文件保存到 /home/student/lab/ 下并进行检查。

    a. 从 workstation 打开一个终端，再跳转到 lab 目录。

    [student@workstation ~]$ cd lab

    b. 提供 overcloud 凭据文件overcloudrc。

    [student@workstation lab]# source ../overcloudrc

    c. 使用 wget，检索位于 http://materials.example.com/heat/comprehensive-s2.yaml 的 Heat 模板文件。将它保存在 /home/student/lab 下。

    [student@workstation lab]# wget http://materials.example.com/heat/comprehensive-s2.yaml

    d. 打开该文件，并检查 Heat 模板文件所依赖的参数。下列屏幕突出显示了环境文件中需要值的相关资源。

    [student@workstation lab]# cat comprehensive-s2.yaml
    parameters:
      image:
        type: string
        description: Image used for servers
    ...Output omitted...
      web_sg:
        type: string
        description: Name of the first security group
      ssh_sg:
        type: string
        description: Name of the second security group
      key_name:
        type: string
        description: SSH key to connect to the servers
      flavor:
        type: string
        description: flavor used by the servers
      public_net:
        type: string
        default: pubnet
        description: Name of public network into which servers get deployed
      private_net:
        type: string
        default: privnet
        description: Name of private network into which servers get deployed
      private_subnet:
        type: string
        default: privsub
        description: Name of private subnet into which servers get deployed
    ...Output omitted...

2. 检索有关 OpenStack 环境的信息

检索与环境相关的各种信息，以填充 Heat 环境文件。

    a. 检索密钥对的名称。

    [student@workstation lab]# openstack keypair list
    +----------+-------------------------------------------------+
    | Name     | Fingerprint                                     |
    +----------+-------------------------------------------------+
    | lab_key1 | c2:b7:37:2d:99:82:23:34:6c:d4:e8:9d:6d:6d:3d:ec |
    +----------+-------------------------------------------------+

    b. 检索网络的名称。

    [student@workstation lab]# openstack network list
    +--------------------------------------+-------------+--------------------------------------+
    | ID                                   | Name        | Subnets                              |
    +--------------------------------------+-------------+--------------------------------------+
    | 4f037379-da19-44e7-b316-42c782a2c706 | lab_privnet | f0ae9684-e8f8-4a8e-8ecc-18d592028787 |
    | 79a62e32-3f8f-4c18-bf61-2839c96fedad | lab_pubnet  | 3d37be98-4c40-40de-830d-fd0eccc7d6c2 |
    +--------------------------------------+-------------+--------------------------------------+

    c. 检索安全组的名称。

    [student@workstation lab]# openstack security group list
    +--------------------------------------+---------+------------------------+
    | ID                                   | Name    | Description            |
    +--------------------------------------+---------+------------------------+
    | d7f6225b-2a8c-486d-ab8e-fdbbfe48c53e | default | Default security group |
    +--------------------------------------+---------+------------------------+

3. 创建环境文件

检查了资源后，创建适当的环境文件。该环境文件应当使用 small 作为映像、m2.small作为类别、lab_pubnet 作为公共网络、lab_privnet 作为专用网络、lab_privsub 作为专用子网、lab_key1 作为密钥对、lab_sshsg 作为 lab_sshvm 实例的安全组，以及 lab_websg 作为 lab_webvm 实例的安全组。

    a. 创建包含必要参数的适当环境文件。将文件命名为 comprehensive-s2-env.yaml，并保存在 /home/student/lab 下。

    ```
    parameters:
     public_net: lab_pubnet
     private_net: lab_privnet
     private_subnet: lab_privsub
     key_name: lab_key1
     flavor: m2.small
     image: small
     web_sg: lab_websg
     ssh_sg: lab_sshsg
     ```

4. 启动堆栈

通过回滚选项创建 comprehensive-s2 Heat。使用环境文件 comprehensive-s2-env.yaml 和模板文件 comprehensive-s2.yaml。确保该堆栈过渡到 CREATE_COMPLETE 状态。

    a. Heat 堆栈现已就绪，可以启动。同时传递模板文件和环境文件。

    [student@workstation lab]# heat stack-create -r -f comprehensive-s2.yaml -e comprehensive-s2-env.yaml comprehensive-s2
    +--------------------------------------+------------------+--------------------+---------------------+--------------+
    | id                                   | stack_name       | stack_status       | creation_time       | updated_time |
    +--------------------------------------+------------------+--------------------+---------------------+--------------+
    | 7737b519-9795-40f4-9206-2c2b0f1435ed | comprehensive-s2 | CREATE_IN_PROGRESS | 2016-01-21T08:27:46 | None         |
    +--------------------------------------+------------------+--------------------+---------------------+--------------+

    b. 使用 heat stack-list 命令，监控堆栈的状态。等待它变为 CREATE_COMPLETE 状态，然后继续。

    [student@workstation lab]# heat stack-list
    +--------------------------------------+-------------------+-----------------+---------------------+--------------+
    | id                                   | stack_name        | stack_status    | creation_time       | updated_time |
    +--------------------------------------+-------------------+-----------------+---------------------+--------------+
    | 7737b519-9795-40f4-9206-2c2b0f1435ed | comprehensive-s2  | CREATE_COMPLETE | 2016-01-21T08:27:46 | None         |
    +--------------------------------------+-------------------+-----------------+---------------------+--------------+

    c. 创建了堆栈后，使用 openstack server list 命令检查创建的两个服务器并检索其关联的浮动 IP 地址，它们在 172.25.250.0/24 范围内。

    [student@workstation lab]# openstack server list
    +--------------------------------------+-----------+--------+-----------------------------------------+
    | ID                                   | Name      | Status | Networks                                |
    +--------------------------------------+-----------+--------+-----------------------------------------+
    | e7bded5a-852c-4420-9df3-d6c2476fde5d | lab_webvm | ACTIVE | lab_privnet=192.168.1.17, 172.25.250.52 |
    | 275bdfcf-501c-4775-8dfd-c0e2f98a1a4e | lab_sshvm | ACTIVE | lab_privnet=192.168.1.16, 172.25.250.51 |
    +--------------------------------------+-----------+--------+-----------------------------------------+
    
    d. 确保已创建了安全组。

    [student@workstation lab]# openstack security group list
    +--------------------------------------+-----------+--------------------------------------+
    | ID                                   | Name      | Description                          |
    +--------------------------------------+-----------+--------------------------------------+
    | 7c5627f3-b5eb-46e6-a5d9-6b14277dbeb3 | default   | Default security group               |
    | b4887e27-94c0-4680-b2ff-1596d230834a | lab_sshsg | Security group for the second server |
    | 09ffc441-bc4e-42ea-90c4-44219e639ba9 | lab_websg | Security group for the first server  |
    +--------------------------------------+-----------+--------------------------------------+

5. 使用与 lab_key1 密钥对关联的私钥 /home/student/lab/lab_key1.pem 并通过 SSH 连接实例，验证 lab_sshvm 实例中 TCP/22 的访问权限已经开启。成功登录后退出。

    a. 使用与密钥对 lab_keypair 关联的私钥 /home/student/lab/lab_key1.pem，打开与 lab_sshvm 实例的 SSH 会话。它应当成功，因为 lab_sshsg 允许从任何位置到 lab_sshvm 实例的 TCP/22 (SSH) 端口的入口流量。完成时注销。

    [student@workstation lab]$ ssh -o ConnectTimeout=5 -i lab_key1.pem cloud-user@172.25.250.51
    [cloud-user@lab_sshvm ~]$ logout
    Connection to 172.25.250.51 closed.

6. 使用与 lab_key1 密钥对关联的私钥 /home/student/lab/lab_key1.pem，验证 lab_sshvm 实例中 TCP/22 端口的访问权限已经关闭。

    a. 使用与密钥对 lab_keypair 关联的私钥 /home/student/lab/lab_key1.pem，尝试打开与 lab_webvm 实例的 SSH 会话。由于应用了安全组规则，该连接将被拒绝；SSH 超时将使您返回到提示符。

    [student@workstation lab]$ ssh -o ConnectTimeout=5 -i /home/student/lab/lab_key1.pem cloud-user@172.25.250.52
    ssh: connect to host 172.25.250.52 port 22: Connection timed out
    [student@workstation lab]$ 

7. 使用与 lab_key1 密钥对关联的私钥 /home/student/lab/lab_key1.pem，验证从 lab_sshsg 安全组到 lab_webvm 实例中端口 TCP/22 的访问权限已经开启。您需要将此文件从 servera 复制到 lab_sshvm 实例。

    a. 将私钥 /home/student/lab/lab_key1.pem 复制到 lab_sshvm 实例。

    [student@workstation lab]$ scp -i lab_key1.pem lab_key1.pem cloud-user@172.25.250.51:
    lab_key1.pem                            100% 1684     1.6KB/s   00:00

    b. 打开与 lab_sshvm 实例的 SSH 会话；从该处，使用私钥 ~/lab_key1.pem 及其专用 IP 地址（之前在 192.168.1.0/24 范围内获取）打开与 lab_webvm 的 SSH 会话。它应当成功，因为 lab_websg 允许从与 lab_sshsg 安全组关联的实例到 lab_webvm 实例的 TCP/22 (SSH) 端口的入口流量。完成时注销。

    [student@workstation lab]$ ssh -i lab_key1.pem cloud-user@172.25.250.51
    [cloud-user@lab_sshvm ~]$ ssh -i lab_key1.pem cloud-user@192.168.1.17
    The authenticity of host '192.168.1.17 (192.168.1.17)' can't be established.
    ECDSA key fingerprint is b5:f0:17:a8:51:3d:d3:62:45:90:71:4f:a2:7c:44:28.
    Are you sure you want to continue connecting (yes/no)? yes
    [cloud-user@lab_webvm ~]$ logout
    Connection to 192.168.1.17 closed.
    [cloud-user@lab_sshvm ~]$ logout
    Connection to 172.25.250.51 closed.
    [student@workstation lab]$ 


### 实验：部署 OpenStack Overcloud

1. 从 workstation，以 stack 用户身份登录 director。提升权限，并安装必要的软件包 rhosp-director-images 和 rhosp-director-images-ipa。将安装的映像上传到 Glance。

    a. 从 workstation，以 stack 用户身份登录 director。

    [student@workstation ~]$ ssh stack@director

    b. 使用 yum 命令和 sudo 命令，安装 rhosp-director-images 和 rhosp-director-images-ipa 软件包。软件包安装好后，/usr/share/rhosp-director-images/ 目录中将有映像。

    [stack@director ~]$ sudo yum -y install rhosp-director-images rhosp-director-images-ipa
    ...Output omitted...

    Installed:
      rhosp-director-images.noarch 0:8.0-20160415.1.el7ost  rhosp-director-images-ipa.noarch 0:8.0-20160415.1.el7ost

    Dependency Installed:
      dwz.x86_64 0:0.11-3.el7 perl-srpm-macros.noarch 0:1-8.el7 redhat-rpm-config.noarch 0:9.1.0-68.el7 rpm-build.x86_64 0:4.11.3-17.el7 rpmdevtools.noarch 0:8.3-5.el7

    Complete!

    c. 在 /home/stack 下创建 images 目录，并复制新安装的映像存档。

    [stack@director ~]$ mkdir images
    [stack@director ~]$ cp /usr/share/rhosp-director-images/overcloud-full-latest-8.0.tar images/
    [stack@director ~]$ cp /usr/share/rhosp-director-images/ironic-python-agent-latest-8.0.tar images/

    d. 跳转到 images 目录，再使用 tar 提取映像。

    [stack@director ~]$ cd images
    [stack@director images]$ for tarfile in *.tar; do tar -xf $tarfile; done
    [stack@director images]$ 

    e. 使用 ~/stackrc 文件提供 undercloud admin 凭据，再将映像导入到 Glance。

    [stack@director images]$ source ../stackrc
    [stack@director images]$ openstack overcloud image upload --image-path ~/images
    Image "overcloud-full-vmlinuz" was uploaded.
    ...Output omitted...

    f. 确认映像上传成功；总共应当列出五个映像，如以下输出中所示：

    [stack@director images]$ openstack image list
    +--------------------------------------+------------------------+
    | ID                                   | Name                   |
    +--------------------------------------+------------------------+
    | cf16ce61-71dd-4846-bfda-f63d803b4410 | bm-deploy-kernel       |
    | 9dc1b4ee-7f2a-4764-99c5-1403bd68fa93 | bm-deploy-ramdisk      |
    | b013d447-4cc5-4f3d-b38e-d9d428253e21 | overcloud-full-vmlinuz |
    | 629da664-bb0c-455f-9027-f32d73c5768a | overcloud-full         |
    | 29c25169-c0c3-4c9a-86f2-1a526a5113ad | overcloud-full-initrd  |
    +--------------------------------------+------------------------+

    g. 转到 stack 用户的主目录。

    [stack@director images]$ cd -

2. 将 PXE/provisioning 子网的 DNS 服务器设为 172.25.250.254，它是 CIDR 为 172.25.250.0/24 的子网。DNS 服务器将在部署 overcloud 期间在节点上设置。要设置名称服务器，可检索子网的 ID，并将它作为参数传递给 neutron subnet-update 命令。

> 注意: 该子网没有任何名称。

```
[stack@director ~]$ neutron subnet-list
+--------------------------------------+---------------------+-----------------+-----------------------------------------------------+
| id                                   | name                | cidr            | allocation_pools                                    |
+--------------------------------------+---------------------+-----------------+-----------------------------------------------------+
...Output omitted...
| 482cd158-4816-4407-b7d3-10b9e9b6f0be |                     | 172.25.250.0/24 | {"start": "172.25.250.20", "end": "172.25.250.30"}  |
+--------------------------------------+---------------------+-----------------+-----------------------------------------------------+

[stack@director ~]$ neutron subnet-update 23b377cd-bbba-4616-aec5-06d93958649a --dns-nameserver 172.25.250.254
Updated subnet: 23b377cd-bbba-4616-aec5-06d93958649a
```

3. 在 stack 用户的主目录上，从 http://materials.example.com/instackenv-twonodes.json 下载 instackenv-twonodes.json 文件。导入并注册 overcloud 节点 controller 和 compute1。

    a. 在 director 上，从 http://materials.example.com/instackenv-twonodes.json 下载 instackenv-twonodes.json 文件。

    ```
    [stack@director ~]$ wget http://materials.example.com/instackenv-twonodes.json
    ...Output omitted...
    100%[===============================>] 626         --.-K/s   in 0s

    2016-06-07 21:26:02 (155 MB/s) - ‘instackenv-twonodes.json’ saved [626/626]
    ```

    b. 检查该 JSON 文件。该 JSON 文件定义两个要使用的节点。controller 和 compute1 节点通过使用 pxe_ipmitool 驱动程序进行定义。

    ```
    {
      "nodes": [
        {
          "pm_user": "admin",
          "arch": "x86_64",
          "name": "controller",
          "pm_addr": "192.168.1.111",
          "pm_password": "password",
          "pm_type": "pxe_ipmitool",
          "mac": [
            "52:54:00:00:fa:0b"
          ],
          "cpu": "2",
          "memory": "6144",
          "disk": "40"
        },
        {
          "pm_user": "admin",
          "arch": "x86_64",
          "name": "compute1",
          "pm_addr": "192.168.1.112",
          "pm_type": "pxe_ipmitool",
          "pm_password": "password",
          "mac": [
            "52:54:00:00:fa:0c"
          ],
          "cpu": "2",
          "memory": "6144",
          "disk": "40"
        }
      ]
    }
    ```
    
    c. 在 Ironic 中导入 JSON 文件，注册 controller 和 compute1 节点。

    ```
    [stack@director ~]$ openstack baremetal import --json instackenv-twonodes.json
    ```
    
    > 重要: openstack baremetal import --json instackenv-twonodes.json 命令为静默运行，除非 instackenv-twonodes.json 中出现了某种错误。

4. 在内省节点之前，分配要用于引导已注册 overcloud 节点的映像。这通过 openstack baremetal configure boot 完成。

    a. 运行 openstack baremetal configure boot 命令，将内核和 ramdisk 映像设置到 compute1 和 controller 节点。此命令不产生任何输出。

    [stack@director ~]$ openstack baremetal configure boot

    b. 使用 ironic node-show nodeid 命令，确保内核和 ramdisk 映像的 ID 已分配给节点。

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
    [stack@director ~]$ for i in $(ironic node-list | grep -v UUID | \
    > awk '{print $2}'); do \
    > ironic node-show $i | grep deploy; echo ; \
    > done
    |   | u'ipmi_username': u'admin', u'deploy_kernel': u'cf16ce61-71dd-4846-bfda- |
    |   | f63d803b4410', u'deploy_ramdisk': u'9dc1b4ee-7f2a-4764-99c5-             |
    |   | u'1', u'deploy_key': u'602G7I1QH0G5ZB4P6GLCAD9CR6VZOUIQ', u'local_gb':   |

    |   | u'ipmi_username': u'admin', u'deploy_kernel': u'cf16ce61-71dd-4846-bfda- |
    |   | f63d803b4410', u'deploy_ramdisk': u'9dc1b4ee-7f2a-4764-99c5-             |
    |   | u'1', u'deploy_key': u'12P7TUGHS9S8UXDJKUGDQ70UYZBSN382', u'local_gb':   |

5. controller 和 compute1 节点已准备由 Ironic 内省。

    a. 使用 ironic node-list 命令，验证 controller 和 compute1 节点已经注册，并已标记为 available。

    [stack@director ~]$ ironic node-list
    +--------------------------------------+------------+-------------+--------------------+-------------+
    | UUID                                 | Name       | Power State | Provisioning State | Maintenance |
    +--------------------------------------+------------+-------------+--------------------+-------------+
    | ec5fa102-0039-4765-8a74-c4e5c6e4061c | compute1   | power off   | available          | False       |
    | b06edb28-ca22-4375-a23f-ae76e2bd4a7d | controller | power off   | available          | False       |
    +--------------------------------------+------------+-------------+--------------------+-------------+
    
    > 重要: 确保两个节点已标记为 available，然后继续。

    b. 启动两个节点的内省过程，并且等待 Introspection completed 消息出现。

    ```
    [stack@director ~]$ openstack baremetal introspection bulk start
    Setting available nodes to manageable...
    Starting introspection of node: ec5fa102-0039-4765-8a74-c4e5c6e4061c
    Starting introspection of node: b06edb28-ca22-4375-a23f-ae76e2bd4a7d
    Waiting for introspection to finish...
    Introspection for UUID ec5fa102-0039-4765-8a74-c4e5c6e4061c finished successfully.
    Introspection for UUID b06edb28-ca22-4375-a23f-ae76e2bd4a7d finished successfully.
    Setting manageable nodes to available...
    Node ec5fa102-0039-4765-8a74-c4e5c6e4061c has been set to available.
    Node b06edb28-ca22-4375-a23f-ae76e2bd4a7d has been set to available.
    Introspection completed.
    ```
    
6. 针对特定的角色创建类别 compute 和 control。此外，也创建名为 baremetal 的基础类别。设置类别的属性，将 cpu_arch 添加到 x86_64，并通过设置 profile 和 boot_option=local 添加与其角色对应的功能。

*    类别名称	    磁盘大小	RAM	    VCPU
*    baremetal	10	    2048	2
*    compute	20	    6144	2
*    controller	30	    6144	2

    a. 创建名为 baremetal 的类别，ram 设为 2048 MB，disk 设为 10 GB，vcpus 则设为 2。

    [stack@director ~]$ openstack flavor create --id auto --ram 2048 --disk 10 \
    > --vcpus 2 baremetal
    +----------------------------+--------------------------------------+
    | Field                      | Value                                |
    +----------------------------+--------------------------------------+
    | OS-FLV-DISABLED:disabled   | False                                |
    | OS-FLV-EXT-DATA:ephemeral  | 0                                    |
    | disk                       | 10                                   |
    | id                         | 88c20cf4-9f42-41e8-b133-3d42697d6238 |
    | name                       | baremetal                            |
    | os-flavor-access:is_public | True                                 |
    | ram                        | 2048                                 |
    | rxtx_factor                | 1.0                                  |
    | swap                       |                                      |
    | vcpus                      | 2                                    |
    +----------------------------+--------------------------------------+

    b. 创建名为 compute 的类别，ram 设为 6144 MB，disk 设为 20 GB，vcpus 则设为 2。

    [stack@director ~]$ openstack flavor create --id auto --ram 6144 --disk 20 \
    > --vcpus 2 compute
    +----------------------------+--------------------------------------+
    | Field                      | Value                                |
    +----------------------------+--------------------------------------+
    | OS-FLV-DISABLED:disabled   | False                                |
    | OS-FLV-EXT-DATA:ephemeral  | 0                                    |
    | disk                       | 20                                   |
    | id                         | 15d7abbe-057b-4ef6-a938-5918f7188468  |
    | name                       | compute                              |
    | os-flavor-access:is_public | True                                 |
    | ram                        | 6144                                 |
    | rxtx_factor                | 1.0                                  |
    | swap                       |                                      |
    | vcpus                      | 2                                    |
    +----------------------------+--------------------------------------+

    c. 创建名为 control 的类别，ram 设为 6144 MB，disk 设为 30 GB，vcpus 则设为 2。

    [stack@director ~]$ openstack flavor create --id auto --ram 6144 --disk 30 \
    > --vcpus 2 control
    +----------------------------+--------------------------------------+
    | Field                      | Value                                |
    +----------------------------+--------------------------------------+
    | OS-FLV-DISABLED:disabled   | False                                |
    | OS-FLV-EXT-DATA:ephemeral  | 0                                    |
    | disk                       | 30                                   |
    | id                         | 4157fac0-8225-4846-9973-d5a93bbd879f |
    | name                       | control                              |
    | os-flavor-access:is_public | True                                 |
    | ram                        | 6144                                 |
    | rxtx_factor                | 1.0                                  |
    | swap                       |                                      |
    | vcpus                      | 2                                    |
    +----------------------------+--------------------------------------+

7. 通过设置 cpu_arch 和 capabilities 更新类别。为 cpu_arch 赋值 x86_64，为 boot_option 赋值 local。

    a. 通过定义 CPU 架构和引导选项，更新 baremetal 类别。

    [stack@director ~]$ openstack flavor set \
    > --property "cpu_arch"="x86_64" \
    > --property "capabilities:boot_option"="local" baremetal
    +---------------+-----------------------------------------------------+
    | Field         | Value                                               |
    +---------------+-----------------------------------------------------+
    | Field         | Value                                               |
    ...Output omitted...
    | disk          | 10                                                  |
    | id            | 88c20cf4-9f42-41e8-b133-3d42697d6238                |
    | name          | baremetal                                           |
    ...Output omitted...
    | properties    | capabilities:boot_option='local', cpu_arch='x86_64' |
    | ram           | 2048                                                |
    | rxtx_factor   | 1.0                                                 |
    | swap          |                                                     |
    | vcpus         | 2                                                   |
    +---------------+-----------------------------------------------------+

    b. 设置 compute 类别的额外属性；定义 CPU 架构、配置文件以及引导选项。

    [stack@director ~]$ openstack flavor set \
    > --property "cpu_arch"="x86_64" \
    > --property "capabilities:profile"="compute" \
    > --property "capabilities:boot_option"="local" compute
    +---------------+------------------------------------------------------------+
    | Field         | Value                                                      |
    +---------------+------------------------------------------------------------+
    ...Output omitted...
    | disk          | 20                                                         |
    | id            | 15d7abbe-057b-4ef6-a938-5918f7188468                       |
    | name          | compute                                                    |
    ...Output omitted...
    | properties    | capabilities:boot_option='local',
                      capabilities:profile='compute', cpu_arch='x86_64'          |
    | ram           | 6144                                                       |
    | rxtx_factor   | 1.0                                                        |
    | swap          |                                                            |
    | vcpus         | 2                                                          |
    +---------------+------------------------------------------------------------+

    c. 为 control 类别重复上一步。务必将配置文件和类别名称替换为使用 (control)。

    [stack@director ~]$ openstack flavor set \
    > --property "cpu_arch"="x86_64" \
    > --property "capabilities:profile"="control" \
    > --property "capabilities:boot_option"="local" control
    +---------------+-----------------------------------------------------------+
    | Field         | Value                                                     |
    +---------------+-----------------------------------------------------------+
    ...Output omitted...
    | disk          | 30                                                        |
    | id            | 4157fac0-8225-4846-9973-d5a93bbd879f                      |
    | name          | control                                                   |
    | os-flavor-acc | True                                                      |
    | properties    | capabilities:boot_option='local',
                      capabilities:profile='control', cpu_arch='x86_64'         |
    | ram           | 6144                                                      |
    | rxtx_factor   | 1.0                                                       |
    | swap          |                                                           |
    | vcpus         | 2                                                         |
    +---------------+-----------------------------------------------------------+

8. 节点需要使用特定的配置文件进行标记，overcloud 部署才能将角色分配到正确的节点。因为环境中只有两个节点，所以可使用手动标记。

要使用特定的配置文件手动标记节点，可针对每一发现的节点添加 profile 选项到 properties/capabilities 参数。使用ironic node-list命令检索节点 ID。

    a. 通过 ironic node-list 命令使用列出的节点 ID。

    [stack@director ~]$ ironic node-list
    +--------------------------------------+------------+-------------+--------------------+-------------+
    | UUID                                 | Name       | Power State | Provisioning State | Maintenance |
    +--------------------------------------+------------+-------------+--------------------+-------------+
    | e7409a60-c28e-40e4-bab8-61f06a5b44c4 | compute1   | power off   | available          | False       |
    | b6eecdcd-63e7-4fd4-a336-8f5eb184329c | controller | power off   | available          | False       |
    +--------------------------------------+------------+-------------+--------------------+-------------+

    b. 使用对应的 properties/capabilities 参数标记 compute1 节点。

    [stack@director ~]$ ironic node-update e7409a60-c28e-40e4-bab8-61f06a5b44c4 \
    > add properties/capabilities='profile:compute,boot_option:local'
    +------------------------+----------------------------------------+
    | Property               | Value                                  |
    +------------------------+----------------------------------------+
    | target_power_state     | None                                   |
    | extra                  | {u'hardware_swift_object':
                              u'extra_hardware-d4046c37-48ed-41aa-    |
                               bad8-90526f74a1b5'}                    |
    | last_error             | None                                   |
    | updated_at             | 2016-06-08T05:29:37+00:00              |
    | maintenance_reason     | None                                   |
    | provision_state        | available                              |
    | clean_step             | {}                                     |
    ...Output omitted...

    c. 使用对应的 properties/capabilities 参数标记 controller 节点。

    [stack@director ~]$ ironic node-update b6eecdcd-63e7-4fd4-a336-8f5eb184329c \
    > add properties/capabilities='profile:control,boot_option:local'
    +------------------------+----------------------------------------+
    | Property               | Value                                  |
    +------------------------+----------------------------------------+
    | target_power_state     | None                                   |
    | extra                  | {u'hardware_swift_object':
                              u'extra_hardware-37d0c3cd-cb84-4014-
                               9615-f6e8c91a1ff5'}                    |
    | last_error             | None                                   |
    | updated_at             | 2016-06-08T05:29:37+00:00              |
    | maintenance_reason     | None                                   |
    | provision_state        | available                              |
    | clean_step             | {}                                     |
    ...Output omitted...

9. 创建 templates 目录，并且复制位于 /usr/share/openstack-tripleo-heat-templates/ 的默认模板。

[stack@director ~]$ mkdir templates
[stack@director ~]$ cp -rf /usr/share/openstack-tripleo-heat-templates/* templates

10. 检索位于http://materials.example.com/overcloud-heat-templates/templates.tar.bz2的templates.tar.bz2存档，并提取文件。该存档会在上一步中创建的现有 templates 目录中添加文件。

[stack@director ~]$ curl -s -O http://materials.example.com/overcloud-heat-templates/templates.tar.bz2
[stack@director ~]$ tar -xvjpf templates.tar.bz2

11. 部署具有一个控制器和一个计算节点的 overcloud。对于本部署，VXLAN 将用作覆盖和隧道网络。

    a. 验证 overcloud 节点的 Provisioning State 已设为 available，然后继续。

    [stack@director ~]$ ironic node-list
    +--------------------------------------+------------+-------------+--------------------+-------------+
    | UUID                                 | Name       | Power State | Provisioning State | Maintenance |
    +--------------------------------------+------------+-------------+--------------------+-------------+
    | e7409a60-c28e-40e4-bab8-61f06a5b44c4 | compute1   | power off   | available          | False       |
    | b6eecdcd-63e7-4fd4-a336-8f5eb184329c | controller | power off   | available          | False       |
    +--------------------------------------+------------+-------------+--------------------+-------------+

    b. 以 stack 用户身份，使用 openstack overcloud deploy 命令从主目录部署 overcloud 环境。
    
    > 重要: 此部署大约需要 30-40 分钟；只有完成这一步，才能进行本实验的其余部分。openstack overcloud deploy 命令必须从 stack 用户的主目录运行。

    [stack@director ~]$ source ~/stackrc
    [stack@director ~]$ openstack overcloud deploy --templates ~/templates \
    > --control-scale 1 --compute-scale 1 \
    > --control-flavor control --compute-flavor compute \
    > --neutron-tunnel-types vxlan --neutron-network-type vxlan \
    > -e ~/templates/compute-extraconfig.yaml \
    > -e ~/templates/environments/network-isolation.yaml \
    > -e ~/templates/network-environment.yaml \
    > -e ~/templates/pre-config-fix.yaml
    Deploying templates in the directory /home/stack/templates/

    c. 
    
    [student@workstation ~]$ ssh stack@director
    Last login:Tue Dec 29 13:30:53 2015 from workstation.lab.example.com
    [stack@director ~]$ source ~/stackrc
    [stack@director ~]$ watch -n 5 ironic node-list
    Every 5.0s: ironic node-list

    +--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
    | UUID                                 | Name       | Instance UUID                        | Power State | Provisioning State | Maintenance |
    +--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
    | ec5fa102-0039-4765-8a74-c4e5c6e4061c | compute1   | d2c393e7-1eb7-417e-9a11-e03a126b62a4 | power on    | wait call-back     | False       |
    | b06edb28-ca22-4375-a23f-ae76e2bd4a7d | controller | b6f40aad-6cb9-4a90-9c26-f4b1daf80cc7 | power on    | wait call-back     | False       |
    +--------------------------------------+------------+--------------------------------------+-------------+--------------------+-------------+
    
    d. 该 overcloud 是使用虚拟机部署的，而非物理硬件。观察到了争用情形，这可导致部署的元素不一致地挂起。在本课程发布之时，尚无此问题的自动化解决方案。不过，您可通过以下步骤进行检测，并根据需要手动纠正部署。

    解决方案部分通过/home/stack/templates/pre-config-fix.yaml环境文件进行实施，该文件在上一步中用于 openstack overcloud deploy命令。

    在 workstation 上打开一个新终端，再以 stack 用户身份和密码 redhat 通过 SSH 连接到 director。使用 ironic node-list 命令，观察 Ironic 节点的状态过渡：available -> deploying -> wait call-back -> deploying -> active。

    在 deploying 阶段中，上传至 Glance 的 overcloud-full.qcow2 映像被转移到裸机 Ironic 节点。完成后，cloud-init 将重新引导，并调整文件系统的大小。文件系统大小调整后，部署将进入 active 调配状态

    [stack@director ~]$ source stackrc
    [stack@director ~]$ ironic node-list
    +--------------------------------------+------------+--------------------------------------+------------+--------------------+-------------+
    | UUID                                 | Name       | Instance UUID                        |Power State | Provisioning State | Maintenance |
    +--------------------------------------+------------+--------------------------------------+------------+--------------------+-------------+
    | b06edb28-ca22-4375-a23f-ae76e2bd4a7d | controller | b6f40aad-6cb9-4a90-9c26-f4b1daf80cc7 |power on    | active           | False         |
    | ec5fa102-0039-4765-8a74-c4e5c6e4061c | compute1   | d2c393e7-1eb7-417e-9a11-e03a126b62a4 |power on    | active           | False         |
    +--------------------------------------+------------+--------------------------------------+------------+--------------------+-------------+
    
    验证 overcloud nova 实例是否已启动，并处于 ACTIVE 状态。

    [stack@director ~]$ openstack server list
    +--------------------------------------+-------------------------+--------+------------------------+
    | ID                                   | Name                    | Status | Networks               |
    +--------------------------------------+-------------------------+--------+------------------------+
    | b6f40aad-6cb9-4a90-9c26-f4b1daf80cc7 | overcloud-controller-0  | ACTIVE | ctlplane=172.25.250.29 |
    | d2c393e7-1eb7-417e-9a11-e03a126b62a4 | overcloud-novacompute-0 | ACTIVE | ctlplane=172.25.250.30 |
    +--------------------------------------+-------------------------+--------+------------------------+

    e. 从 overcloud-controller-0 和 overcloud-novacompute-0 overcloud 节点的控制台登录，登录时使用用户名 root 及密码 ROOTPW。

    验证文件 /etc/sysconfig/network-scripts/ifcfg-eth* 是否为零字节文件，再检查 /var/log/cloud-init.log 中是否有错误。对 overcloud-controller-0 和 overcloud-novacompute-0 执行这些验证步骤。

    [root@overcloud-novacompute-0 ~]# ls -l /etc/sysconfig/network-scripts/ifcfg-eth*
    -rw-r--r--. 1 root root 0 Sep 10 04:06 /etc/sysconfig/network-scripts/ifcfg-eth0
    [root@overcloud-novacompute-0 ~]# tailf /var/log/cloud-init.log
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

    f. cloud-init 可能会导致 overcloud 部署挂起，因为节点的网络无法访问。Heat 引擎将无法通过访问元数据服务器来配置节点。os-refersh-config用于对 overcloud 节点应用配置的元数据 URL 在/etc/os-collect-config.conf中指定。

    [root@overcloud-novacompute-0 ~]# cat /etc/os-collect-config.conf 
    [DEFAULT]
    command = os-refresh-config

    [cfn]
    metadata_url = http://172.25.250.10:8000/v1/
    stack_name = overcloud-Compute-n2onlrw2eva3-0-zsn5w6hvh3qy
    secret_access_key = 03c62d5e36d94f0cb9225f5e03af1fff
    access_key_id = a683f206faa54f8c8d6de1d903c56d4d
    path = NovaCompute.Metadata

    os-collect-config开始轮询元数据的更改。要查看此问题，请使用journalctl -u os-collect-config。

    [root@overcloud-novacompute-0 ~]# journalctl -u os-collect-config --no-full
    -- Logs begin at Sat 2016-09-10 04:04:05 UTC, end at Sat 2016-09-10 14:21:04 UTC. --
    systemd[1]: Started Collect metadata and run hook commands..
    systemd[1]: Starting Collect metadata and run hook commands....
    WARNING os_collect_config.ec2 [-] ('Connection aborted.', error(101, 'Ne...chable'))
    WARNING os-collect-config [-] Source [ec2] Unavailable.
    WARNING os-collect-config [-] Source [request] Unavailable.
    ---Output omitted--

    g. 删除零字节文件/etc/sysconfig/network-scripts/ifcfg-eth*以解决问题，再使用reboot命令手动重新启动节点。

    > 重要: 此操作仅可在有问题的 overcloud 节点上执行。

    [root@overcloud-novacompute-0 ~]# rm -rf /etc/sysconfig/network-scripts/ifcfg-eth*
    [root@overcloud-novacompute-0 ~]# reboot

    h. 执行 heat resource-list -n5 overcloud 命令，观察 overcloud Heat 模板部署的进度。

    [stack@director ~]$ source ~/stackrc
    [stack@director ~]$ heat resource-list -n5 overcloud
    +---------------------------+-----------------------------------------------+-------------------------------------------------------+------------+
    | resource_name             | physical_resource_id                          | resource_type                      resource_status    | stack_name |
    +---------------------------+-----------------------------------------------+-------------------------------------------------------+------------+
    | AllNodesExtraConfig       |                                               | OS::TripleO::AllNodesExtraConfig   INIT_COMPLETE      | overcloud  |
    | AllNodesValidationConfig  |                                               | OS::TripleO::AllNodes::Validation  INIT_COMPLETE      | overcloud  |
    | BlockStorage              | 466578ff-17ab-48e5-bc11-a59f48f9599a          | OS::Heat::ResourceGroup            CREATE_COMPLETE    | overcloud  |
    ...Output omitted...                                                        ...Output omitted...
    

    > 注意: 使用 watch -n 30 'heat resource-list -n5 overcloud | grep -v CREATE_COMPLETE' 查看任务在完成时的情况。

    i. openstack overcloud deploy 成功完成时将显示包含 Overcloud Deployed 的输出。

    ```
    ...Output omitted...
    Overcloud Endpoint: http://192.168.0.11:5000/v2.0/
    Overcloud Deployed
    ```
    
    j. overcloud 部署成功后，使用 openstack server list 命令以及与 overcloud 节点关联的 IP 地址，检查 overcloud 节点状态是否已标记为 ACTIVE。

    [stack@director ~]$ openstack server list
    +--------------------------------------+-------------------------+--------+-------------------------+
    | ID                                   | Name                    | Status | Networks                |
    +--------------------------------------+-------------------------+--------+-------------------------+
    | b6f40aad-6cb9-4a90-9c26-f4b1daf80cc7 | overcloud-controller-0  | ACTIVE | ctlplane=172.25.250.30  |
    | d2c393e7-1eb7-417e-9a11-e03a126b62a4 | overcloud-novacompute-0 | ACTIVE | ctlplane=172.25.250.29  |
    +--------------------------------------+-------------------------+--------+-------------------------+

12. 使用其 IP 地址连接 overcloud 节点 overcloud-controller-0 进行验证，再以 heat-admin 用户身份从 undercloud 登录。

[stack@director ~]$ ssh heat-admin@172.25.250.30
The authenticity of host '172.25.250.30 (172.25.250.30)' can't be established.
ECDSA key fingerprint is d9:20:db:eb:68:b6:16:1f:5c:f3:8b:5e:9c:da:11:2e.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.25.250.30' (ECDSA) to the list of known hosts.
Last login: Tue Jan 05 01:12:24 2016 from 172.25.250.30
[heat-admin@overcloud-controller-0 ~]$ exit
[stack@director ~]$ 

13. 在通过隧道使用两个节点的虚拟化环境中，需要为 overcloud 计算节点上运行的实例配置较小的 MTU。

在 director 上，下载位于 http://materials.example.com/neutron-postdeploy.sh 的脚本 neutron-postdeploy.sh，并执行该脚本。脚本配置 dnsmasq 进程，供 neutron 在控制器节点上播发大小为 1300 字节的 MTU 到实例。更新了 MTU 后，脚本将重启 overcloud-controller-0 节点。

> 注意: 重启控制器节点将需要几分钟时间。

[stack@director ~]$ wget http://materials.example.com/neutron-postdeploy.sh
[stack@director ~]$ chmod a+x neutron-postdeploy.sh
[stack@director ~]$ ./neutron-postdeploy.sh

