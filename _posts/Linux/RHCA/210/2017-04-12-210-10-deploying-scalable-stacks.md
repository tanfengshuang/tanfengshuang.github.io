---
layout: post
title:  "部署可缩放堆栈(210-10)"
categories: Linux
tags: RHCA 210
---

### 描述 Heat 和 Ceilometer 架构及术语

###### 关于 Heat

OpenStack Heat 项目为 OpenStack 提供编配功能。它由各种服务组成，其中包括引擎服务、应用编程接口 (API) 服务器，以及一些面向 Amazon 编配服务 CloudFormation 的传统实施。编配向系统管理员提供一个能够解析基于文本的指令的统一接口，为服务部署助力。管理员编写 模板来指定要在云中生成的资源，如计算实例或卷。

Heat 模板是一种 YAML 文件，用于描述基于云的应用的基础架构。Heat 模板包含管理员可以使用的大部分资源，只需借助命令行工具，如 nova（用于实例）或 cinder（用于卷）。

Heat 也提供了自动缩放功能，让管理员能够根据资源使用情况动态调整其云基础架构。Heat 支持横向扩展和内向收缩资源，这可以回收未用的资源。使用模板的优点有：

*    它们提供了对所有底层服务 API 的访问。
*    它们是模块化的，并以资源为导向。基础架构元素定义为对象。
*    它们可以作为嵌套堆栈反复定义和重新利用。这使得云基础架构能够以模块化方式定义和重新利用。
*    资源实施是可插拔的，因而能定义自定义资源。
*    它们可以定义高可用性功能。

###### Heat 术语

下表列出了相关的术语，管理员应熟悉这些术语以使用 Heat 正确管理云。

*    术语 	            定义
*    Heat API 	        Heat API 服务器提供 REST API，利用远程过程调用 (RPC) 将 Heat 请求转发到 Heat 引擎。
*    Heat 引擎 	        该服务应用模板，并编配云资源的创建和启动。它将事件状态报告回 API 客户。
*    YAML 	            YAML 格式是一种人类可读的数据序列化语言。Heat 模板基于 YAML，为管理员提供管理云基础架构的便捷方式。
*    Heat 编配模板 (HOT) 	我们将向您发放一份 Heat 编配模板 (HOT)是基于 YAML 的配置文件，管理员通过编写该文件并传递到 Heat API 来部署其云基础架构。HOT 模板格式旨在替代传统的 Heat CloudFormation 兼容格式 (CFN)。
*    CloudFormation 模板 (CFN) 	CFN 是由 Amazon AWS 服务使用的传统模板格式。heat-api-cfn服务管理这种传统格式。
*    Heat 参数 	        Heat 参数是传递给 Heat 的设置，它提供了自定义堆栈的途径。它们在 HOT 模板文件中定义，具有可在未传递值时使用的可选默认值。它们在模板的 parameters 部分中定义。
*    Heat 资源 	        Heat 资源是作为堆栈一部分创建和配置的具体对象。OpenStack 含有一组核心资源，它们跨越所有组件。它们在 HOT 模板的 resources 部分中定义。
*    Heat 输出 	        Heat 输出是 HOT 模板文件中定义的值，在堆栈创建后由 Heat 返回。用户可以通过 Heat API 或客户端工具访问这些值。它们在模板的 output 部分中定义。

###### 关于 Ceilometer

Ceilometer 是 OpenStack 遥测服务，为基于 OpenStack 的云提供用户级使用情况数据。Ceilometer 收集的数据可被用于客户记账、系统监控和警报等。Ceilometer 从现有 OpenStack 组件发送的通知收集数据，如 Nova 使用情况事件，或者通过轮询 OpenStack 基础架构资源（如 Libvirt）来收集。Ceilometer 使用插件系统，供管理员添加新的监控器。

Ceilometer 包含一个存储守护进程，负责收集数据并聚合到 MongoDB 数据库中。只有收集器代理和 API 服务器有权访问该数据库。Ceilometer 通过受信报文通信系统与经身份验证的代理通信。API 服务器、中央代理、数据存储服务和收集器代理可以部署在不同的主机上，以扩展遥测基础架构。

###### Ceilometer 包含下列服务组件：

Ceilometer 服务表

*    术语 	定义
*    openstack-ceilometer-alarm-evaluator 	此服务定义警报的状态变换。
*    openstack-ceilometer-alarm-notifier 	从服务在触发警报时执行用户定义的操作。
*    openstack-ceilometer-api 	此 API 服务器在一个或多个中央管理服务器上运行，提供对 Ceilometer 数据库中数据的访问权限。
*    openstack-ceilometer-central 	此服务在中央管理服务器上运行，轮询与独立于实例或计算节点的资源有关的使用情况统计。由于该代理无法水平缩放，管理员一次只能运行此服务的一个实例。
*    openstack-ceilometer-collector 	此服务在一个或多个中央 Ceilometer 管理服务器上运行，用于监控消息队列。每一收集器处理通知消息并转译为 Ceilometer 消息，再将消息发回到具有相关主题的消息总线。遥测消息不经修改写入到数据存储中。管理员可以选择运行这些代理的时间，因为所有通信都基于对 ceilometer-api 服务的 AMQP 或 REST 调用，这与 openstack-ceilometer-alarm-evaluator 服务类似。
*    openstack-ceilometer-compute 	此服务在每个计算节点上运行，用于轮询资源使用情况统计。每一 Nova 计算节点必须部署并运行 ceilometer-compute 代理。
*    openstack-ceilometer-notification 	此服务将指标从各种 OpenStack 服务推送到收集器服务。


###### Ceilometer 术语

下表列出了相关的术语，管理员应熟悉这些术语以使用 Ceilometer 正确监控其云。

*    术语 	        定义
*    总线侦听器代理 	总线侦听器代理获取 OpenStack 通知总线 (奥斯陆) 上生成的事件，并将它们转换为 Ceilometer 样本。
*    轮询代理 	    轮询代理是在 OpenStack 架构内中央管理节点或者计算节点上运行的服务，用于测量使用情况并将结果发送到收集器。
*    数据存储 	    数据存储是用于记录 Ceilometer 所收集的数据的存储系统。
*    量表 	        量表是针对资源而跟踪的测量。例如，一个实例具有多个量表，如实例持续时间、CPU 使用时间、磁盘 I/O 请求数，等等。
*    计量 	        计量是收集与任何可记账事物的内容、对象、时间和数量相关的信息的过程。此过程的结果是可被处理的“票据”或样本的集合。
*    通知 	        通知是通过外部 OpenStack 系统（如 Nova 或 Glance）并利用 Oslo 通知机制发送的消息。这些通知通常由 Ceilometer 通过通知器 RPC 驱动程序发送和接收。
*    资源 	        资源是被计量的 OpenStack 实体，如实例、卷和映像等。
*    样本 	        特定 Ceilometer 量表的数据样本，如实例的 CPU 使用量。

###### 使用 Ceilometer 跟踪资源

管理员可以根据部署的 OpenStack 服务，使用 Ceilometer 跟踪其基础架构中的各种资源。Ceilometer 数据收集包括：

*    租户的 VCPU 数。
*    每一 Nova 资源的 CPU 使用量，以百分比表示。
*    磁盘分配情况。
*    正在运行的实例数量。
*    内存使用情况。
*    传入和传出的网络数据包。
    
    
### 启动堆栈

配置了 Heat 后，管理员可以启动其 Heat 堆栈。Heat 堆栈是指通过同一个界面（通过 Horizon 控制面板或使用命令行界面）部署和管理的多个基础架构资源的集合。堆栈提供统一的人类可读格式，可用于标准化和加速交付。尽管 Heat 项目是作为模拟 AWS CloudFormation 而启动（从而使其能够兼容 CloudFormation (CFN) 使用的模板格式），但它还支持自己的原生模板格式，即 Heat 编配模板 (HOT)。

在使用 Horizon 控制面板时，管理员可以从 Heat 的可视化功能中获益。此类功能包括能够查看基于对象的资源，它们利用链接描述其依赖项。图 10.3: Horizon 中的 Heat 编配服务 显示了 Horizon 控制面板中的 Heat 实施。圆圈里的对象代表 Heat 资源。绿色显示的对象表示已成功创建。红色显示的对象表示创建该资源时出现错误。使用控制面板为 Heat 资源故障排除提供了快捷方式。管理员可以单击任何对象来获取更多信息。

HOT 模板是使用 YAML 语法编写的，包含三个主要部分：

1. 参数: 通过模板部署时提供的输入参数。
2. 资源: 要部署的基础架构元素，如虚拟机或网络端口。
3. 输出: 输出参数由 Heat 动态生成，可通过命令行界面或 Horizon 控制面板访问。例如，输出可以提供从 Heat 模板创建的实例的公共 IP 地址和名称。 

###### 模板标题

每个 HOT 模板都必须包含值为 2013-05-23 的 heat_template_version 密钥（这是 HOT 的当前版本）。尽管描述为可选，但管理员应包含部分文本以描述用户可以通过模板做些什么：

```
heat_template_version: 2013-05-23

description: >
  This is my HOT template.
```

######### 参数

parameters 部分定义可用参数的列表。对于每个参数，管理员需要至少定义数据类型、可选的默认值（未另外指定时使用）、可选的描述以及用于验证数据自身的约束：

```
Parameters:
  <param name>:
    type: <string | number | json | comma_delimited_list>
    description: <description of the parameter>
    default: <default value for parameter>
    hidden: <true | false>
    constraints:
      <parameter constraints>
```

请考虑以下参数，它用于指定要使用的 SSH 密钥名称。string 关键字表示这些参数需要密钥本身的名称。

```
parameters:
  key_name:
    type: string
    description: Name of the key pair to assign to servers
```

######### 资源

模板的资源部分定义了在通过模板部署堆栈时 Heat 将创建的项。这可能包括存储、网络、端口、路由器、安全组、防火墙规则以及任何其他众多可用资源。资源需要某个类型（如 Nova 实例和各种属性），所有这些都依赖于正在使用的类型：

```
resources:
  resource ID
    type: <resource type>
    properties:
      <property name>: <property value>
    # more resource specific metadata
```

请考虑下列 resource，它指定使用 image、flavor 和 key_name 资源创建 Nova 实例，这三个参数分别决定要使用的映像、类别和密钥名称。

```
resources:
  web_server:
    type: OS::Nova::Server
    properties:
      name: Web Server
      image: { get_param: image }
      flavor: { get_param: flavor }
      key_name: { get_param: key_name }
      networks:
        - port: { get_resource: web_server_port }
```

get_param 函数检索参数的值，以在创建资源期间使用。例如，如果管理员希望动态设置数据库的密码，则将使用 get_param 以使用用户指定的值：

```
user_data:
        str_replace:
          mysql -u root -p$db_rootpassword
        params:
        $db_rootpassword: { get_param: db_root_password }
```

get_resource 函数用于引用同一模板中的另一资源。在运行时，资源解析为资源的引用 ID，引用 ID 是特定于资源类型的。例如，引用浮动 IP 地址资源将在运行时返回对应的 IP 地址。

######### 输出

模板的 outputs 部分定义在部署堆栈之后用户可以使用的参数（通过 API 或命令行界面）。这可能包括分配给实例的 IP 地址或者已注入的密码等信息：

```
outputs:
  <parameter name>:
    description: <description>
    value: <parameter value>
```

请考虑以下 output，它用 host 替代实例的浮动 IP：

```
outputs:
  Login_URL:
      description: The web server URL
      value:
        str_replace:
          template: http://host
          params:
            host: { get_attr: [ web_server_floating_ip, floating_ip_address ] }
```

> 注意: 可以在 Heat 编配模板 (HOT) 规范页面中找到资源及其可用属性的完整列表。本节末尾提供的参考列表中可以找到此页面的 URL。

###### 环境文件

可以手动指定密钥和其他参数的值，或者通过创建环境文件来填充或覆盖它们。环境文件必须使用 YAML 格式编写。例如，用户可以在此文件中指定自己的密码，并在部署堆栈时使用此密码。在先前示例中，parameters 包含 key_name 值。在环境文件中指定密钥名称：

```
parameters:
  key_name: my_user_key
```

在部署堆栈时，环境文件通过-e选项以参数形式指定：

```
[student@workstation ~]$ heat stack-create -r -f demo.template -e /home/demo/env.yaml my-stack
```

> 警告: 此文件需要值的具体位置。请注意参数名称前有空格。如果不在前面插入空格，则尝试启动堆栈时 Heat 将显示以下消息：

```
environment has wrong section SECTION NAME
```

###### 全部放在一起

以下模板创建一个实例，将某个 Neutron 端口连接到此实例并为此实例分配一个安全组。最后，它输出关联到实例的浮动 IP 地址。

```
heat_template_version: 2013-05-23

description: >
  My HOT template.
parameters:
  public_net_id:
    type: string
    description: >
      ID of public network for which floating IP addresses will be allocated
  private_net_id:
    type: string
    description: ID of private network into which servers get deployed
  private_subnet_id:
    type: string
    description: ID of private sub network into which servers get deployed

resources:
  my_server:
    type: OS::Nova::Server
    properties:
      name: First Instance
      image: small
      flavor: m1.small
      key_name: default
      networks:
        - port: { get_resource: my_server_port }

  my_server_port:
    type: OS::Neutron::Port
    properties:
      network_id: { get_param: private_net_id }
      fixed_ips:
        - subnet_id: { get_param: private_subnet_id }
      security_groups: [{ get_resource: server_security_group }]

  my_server_floating_ip:
    type: OS::Neutron::FloatingIP
    properties:
      floating_network_id: { get_param: public_net_id }
      port_id: { get_resource: my_server_port }

  server_security_group:
    type: OS::Neutron::SecurityGroup
    properties:
      description: Add security group rules for server
      name: security-group
      rules:
        - remote_ip_prefix: 0.0.0.0/0
          protocol: tcp
          port_range_min: 22
          port_range_max: 22

outputs:
  my_server_public_ip:
    description: Floating IP address of My Server
    value: { get_attr: [ my_server_floating_ip, floating_ip_address ] }
```

    
### 练习：启动堆栈

1. 从 workstation.lab.example.com 打开一个终端，再提供 overcloud 凭据文件 overcloudrc。

```
[student@workstation ~]$ source overcloudrc
```

2. 将目录更改为/home/student/stacks。

```
[student@workstation ~]$ cd ~/stacks
```

3. 从http://materials.example.com/heat/multi-tier.yaml检索 Heat 模板文件。

```
[student@workstation stacks]# wget http://materials.example.com/heat/multi-tier.yaml
...Output omitted...
2016-01-18 18:55:43 (528 MB/s) - ‘multi-tier.yaml’ saved [5146/5146]
```

4. 显示 multi-tier.yaml 文件的内容。检查各个部分，如 parameters、resources 和 outputs。找到定义新实例的 OS::Nova::Server 资源。

```
heat_template_version: 2013-05-23
description: A Multi-tier Architecture

parameters:
  image:
    type: string
    description: Image used for servers
  db_server_name:
    type: string
    default: DatabaseServer
    description: Name for the database server
...Output omitted...
resources:
  web_server:
    type: OS::Nova::Server
    properties:
      name: { get_param: web_server_name }
      image: { get_param: image }
      flavor: { get_param: flavor }
      key_name: { get_param: key_name }
...Output omitted...
outputs:
  web_server_private_ip:
    description: IP address of Web Server in private network
    value: { get_attr: [ web_server, first_address ] }
  web_server_public_ip:
    description: Floating IP address of Web Server in public network
    value: { get_attr: [ web_server_floating_ip, floating_ip_address ] }
  db_server_private_ip:
    description: IP address of DB Server in private network
    value: { get_attr: [ db_server, first_address ] }
  db_server_public_ip:
    description: Floating IP address of DB Server in public network
    value: { get_attr: [ db_server_floating_ip, floating_ip_address ] }
...Output omitted...
```

5. 可以从命令行使用 -P 选项 (-P key=value) 或使用环境文件来覆盖 Heat 参数。

创建一个名为 ~/stacks/environment.yaml 的文件，它定义要使用的 Glance 映像 small、Nova 类别 m2.small、安全密钥 demo_key1，以及网络配置。确保 environment.yaml 文件已写入且格式如下所示：

```
parameters:
  public_net: dev_pubnet
  private_net: dev_privnet
  private_subnet: dev_privsub
  key_name: dev_key1
  flavor: m2.small
  image: small
```

> 警告: 此文件需要值的具体位置。请注意各参数定义前有两个空格。如果参数没有正确缩进，则尝试启动堆栈时 Heat 将显示以下消息：

```
environment has wrong section SECTION NAME
```

6. 在创建堆栈之前，使用--help选项查看可用于heat stack-create命令的选项。确定创建堆栈时必须使用的选项。

```
[student@workstation stacks]# heat stack-create --help
usage: heat stack-create [-f <FILE>] [-e <FILE or URL>]
                         [--pre-create <RESOURCE>] [-u <URL>] [-o <URL>]
                         [-c <TIMEOUT>] [-t <TIMEOUT>] [-r]
                         [-P <KEY1=VALUE1;KEY2=VALUE2...>] [-Pf <KEY=FILE>]
                         [--poll [SECONDS]] [--tags <TAG1,TAG2>]
                         <STACK_NAME>

Create the stack.

Positional arguments:
  <STACK_NAME>          Name of the stack to create.

Optional arguments:
  -f <FILE>, --template-file <FILE>:                                            ------------》 1
                        Path to the template.
  -e <FILE or URL>, --environment-file <FILE or URL>:                           ------------》 2
                        Path to the environment, it can be specified multiple
                        times.
  --pre-create <RESOURCE>
                        Name of a resource to set a pre-create hook to.
                        Resources in nested stacks can be set using slash as a
                        separator: nested_stack/another/my_resource. You can
                        use wildcards to match multiple stacks or resources:
                        nested_stack/an*/*_resource. This can be specified
                        multiple times
  -u <URL>, --template-url <URL>
                        URL of template.
  -o <URL>, --template-object <URL>
                        URL to retrieve template object (e.g. from swift).
  -c <TIMEOUT>, --create-timeout <TIMEOUT>
                        Stack creation timeout in minutes. DEPRECATED use
                        --timeout instead.
  -t <TIMEOUT>, --timeout <TIMEOUT>
                        Stack creation timeout in minutes.
  -r, --enable-rollback:                                                        ------------》 3
                        Enable rollback on create/update failure.
  -P <KEY1=VALUE1;KEY2=VALUE2...>, --parameters <KEY1=VALUE1;KEY2=VALUE2...>
                        Parameter values used to create the stack. This can be
                        specified multiple times, or once with parameters
                        separated by a semicolon.
  -Pf <KEY=FILE>, --parameter-file <KEY=FILE>
                        Parameter values from file used to create the stack.
                        This can be specified multiple times. Parameter value
                        would be the content of the file
  --poll [SECONDS]      Poll and report events until stack completes. Optional
                        poll interval in seconds can be provided as argument,
                        default 5.
  --tags <TAG1,TAG2>    A list of tags to associate with the stack.
```

    1. 此选项允许您在创建堆栈时将文件作为参数传递。该文件必须为基于 YAML 的文件，用于定义要创建的 OpenStack 资源，如 Nova 服务器或 Neutron 浮动 IP 地址。
    2. 此选项允许您在创建堆栈时将环境文件作为参数传递。该文件必须为基于 YAML 的文件，用于覆盖作为参数传递的文件中找到的参数。
    3. 此选项允许 Heat 在出现资源创建失败时回滚堆栈。这是清理由 Heat 创建的所有资源的便捷方式。

7. 当环境文件准备就绪时，使用传递环境文件和模板文件的选项来启动 Heat 堆栈。

-r 选项允许 Heat 在因出错而部署失败时回滚资源。此选项允许 Heat 将系统返回到之前的状态；例如，移除浮动 IP 地址，删除实例等。

[student@workstation stacks]# heat stack-create -r -f multi-tier.yaml -e environment.yaml multi-tier 
+--------------------------------------+------------+--------------------+---------------------+--------------+
| id                                   | stack_name | stack_status       | creation_time       | updated_time |
+--------------------------------------+------------+--------------------+---------------------+--------------+
| 31cfd5a5-2abd-43cd-9a80-20c0fcc42966 | multi-tier | CREATE_IN_PROGRESS | 2016-01-19T06:45:06 | None         |
+--------------------------------------+------------+--------------------+---------------------+--------------+

8. 运行 heat stack-list 命令，追踪堆栈创建的进度。一两分钟后，stack_status 应当过渡到 CREATE_COMPLETE 状态。

[student@workstation stacks]# heat stack-list
+--------------------------------------+------------+-----------------+---------------------+--------------+
| id                                   | stack_name | stack_status    | creation_time       | updated_time |
+--------------------------------------+------------+-----------------+---------------------+--------------+
| 108b5299-8acb-4072-bc56-e0973333d94c | multi-tier | CREATE_COMPLETE | 2016-01-20T01:30:47 | None         |
+--------------------------------------+------------+-----------------+---------------------+--------------+

9. 检查部署的 Heat 资源。通过 resource-list STACK NAME 命令，您可以检查组成堆栈的每一资源的状态。

[student@workstation stacks]# heat resource-list multi-tier
+------------------------+--------------------------------------+----------------------------+-----------------+---------------------+
| resource_name          | physical_resource_id                 | resource_type              | resource_status | updated_time        |
+------------------------+--------------------------------------+----------------------------+-----------------+---------------------+
| db_net_port            | ae22c4b5-5748-43e6-b5e8-1169bd4d2745 | OS::Neutron::Port          | CREATE_COMPLETE | 2016-01-20T01:30:47 |
| db_server              | 6e9aebac-b182-4da6-9d45-35232315c951 | OS::Nova::Server           | CREATE_COMPLETE | 2016-01-20T01:30:47 |
| db_server_floating_ip  | 2aae42c8-b284-4e06-9009-0bc9ea1d1529 | OS::Neutron::FloatingIP    | CREATE_COMPLETE | 2016-01-20T01:30:47 |
| servers_security_group | 64b3d8a3-a2d9-4eba-bd4f-0541a8362693 | OS::Neutron::SecurityGroup | CREATE_COMPLETE | 2016-01-20T01:30:47 |
| web_net_port           | 46885f11-51f9-467e-8388-186deea1b65a | OS::Neutron::Port          | CREATE_COMPLETE | 2016-01-20T01:30:47 |
| web_server             | 52ba33ac-6db5-463e-ba02-ff9690a051c0 | OS::Nova::Server           | CREATE_COMPLETE | 2016-01-20T01:30:47 |
| web_server_floating_ip | 64a3c540-dec2-4e84-8a61-7b9f04a7e08e | OS::Neutron::FloatingIP    | CREATE_COMPLETE | 2016-01-20T01:30:47 |
+------------------------+--------------------------------------+----------------------------+-----------------+---------------------+

> 注意: 此命令可用于故障排除目的。它让您能够快速获取部署失败的资源的名称，从而将故障排除引向可能有问题的 OpenStack 服务。切勿传递 -r 选项，因为这会在资源失败时回滚整个堆栈。

10. 使用 heat stack-show STACK NAME 命令检查为堆栈创建的资源的详细信息。

```
[student@workstation stacks]# heat stack-show multi-tier
+-----------------------+--------------------------------------------------------+
| Property              | Value                                                  |
+-----------------------+--------------------------------------------------------+
| capabilities          | []                                                     |
| creation_time         | 2016-01-20T01:30:47                                    |
| description           | A Multi-tier Architecture                              |
| disable_rollback      | False                                                  |
| id                    | 108b5299-8acb-4072-bc56-e0973333d94c                   |
| notification_topics   | []                                                     |
| outputs               | [                                                      |
...Output omitted...
|                       |   {                                                    |
|                       |     "output_value": "172.25.250.52",                   |
|                       |     "description": "Floating IP address of Web Server in
                               public network",                                  |
|                       |     "output_key": "web_server_public_ip"               |
|                       |   },                                                   |
...Output omitted...
|                       |   {                                                    |
|                       |     "output_value": "172.25.250.51",                   |
|                       |     "description": "Floating IP address of DB Server in
                               public network",                                  |
|                       |     "output_key": "db_server_public_ip"                |
|                       |   },                                                   |
|                       |   {                                                    |
|                       |     "output_value": "http://172.25.250.52/",           |
|                       |     "description": "This URL is the \"external\"
                               URL that can be used to access the web server.",  |
|                       |     "output_key": "website_url"                        |
|                       |   },                                                   |
|                       |   {                                                    |
|                       |     "output_value": "http://172.25.250.52/
                                               scaling_check-db.php"             |
|                       |     "description": "This script allows you to test the
                               connectivity to the database.",                   |
|                       |     "output_key": "page_url"                           |
|                       |   }                                                    |
|                       | ]                                                      |
...Output omitted...
| parent                | None                                                   |
| stack_name            | multi-tier                                             |
| stack_owner           | None                                                   |
| stack_status          | CREATE_COMPLETE                                        |
| stack_status_reason   | Stack CREATE completed successfully                    |
| stack_user_project_id | 271053444e4546f09d3e4c654e85a53a                       |
| tags                  | None                                                   |
| template_description  | A Multi-tier Architecture                              |
| timeout_mins          | None                                                   |
| updated_time          | None                                                   |
+-----------------------+--------------------------------------------------------+
```

> 注意: 按照设计，IP 地址不由调用堆栈的用户控制或指定；它们由 Heat 选取，具体根据作为参数传递的子网名称。可使用openstack server list命令显示分配给堆栈实例的地址。

```
[student@workstation stacks]# openstack server list
+--------------------------------------+----------------+--------+-------------------------------------+
| ID                                   | Name           | Status | Networks                            |
+--------------------------------------+----------------+--------+-------------------------------------+
| 6e9aebac-b182-4da6-9d45-35232315c951 | DatabaseServer | ACTIVE | privnet=192.168.1.16, 172.25.250.51 |
| 52ba33ac-6db5-463e-ba02-ff9690a051c0 | WebServer      | ACTIVE | privnet=192.168.1.17, 172.25.250.52 |
+--------------------------------------+----------------+--------+-------------------------------------+
```

11. 使用ping命令，确保您可以通过其公共 IP 地址访问这两个服务器。

```
[student@workstation stacks]# ping -c 1 172.25.250.52
PING 172.25.250.52 (172.25.250.52) 56(84) bytes of data.
64 bytes from 172.25.250.52: icmp_seq=1 ttl=63 time=1.09 ms

--- 172.25.250.52 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 1.091/1.091/1.091/0.000 ms
[student@workstation stacks]# ping -c 1 172.25.250.51
PING 172.25.250.51 (172.25.250.51) 56(84) bytes of data.
64 bytes from 172.25.250.51: icmp_seq=1 ttl=63 time=1.32 ms

--- 172.25.250.51 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 1.325/1.325/1.325/0.000 ms
```

12. 从 director 节点，使用 SSH 密钥连接数据库服务器，确保数据库服务正在运行。本例中，其 IP 地址为 172.25.250.51。务必将此地址替换为分配给您的实例的浮动 IP 地址。私钥已保存到/home/student/stacks/dev_key1.pem。

```
[student@workstation stacks]# ssh -i dev_key1.pem cloud-user@172.25.250.51
Warning: Permanently added '172.25.250.51' (ECDSA) to the list of known hosts.
[cloud-user@databaseserver ~]$ systemctl status mariadb.service
● mariadb.service - MariaDB database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor preset: disabled)
   Active: active (running) since Tue 2016-01-19 21:30:07 EST; 2min 30s ago
 Main PID: 1152 (mysqld_safe)
   CGroup: /system.slice/mariadb.service
           ├─1152 /bin/sh /usr/bin/mysqld_safe --basedir=/usr
           └─1309 /usr/libexec/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/mysql/plugin --log...

...Output omitted...
[cloud-user@databaseserver ~]$ exit
```

> 注意: 如果连接时显示以下消息，请等待一两分钟，以便 cloud-init 进程安装 cloud-user 的公钥。

```
Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
```

13. 对 Web 服务器重复同样的操作：建立与 Web 服务器的 SSH 连接，然后查询 httpd 服务。务必将 IP 地址替换为由 Heat 分配的地址。

```
[student@workstation stacks]# ssh -i dev_key1.pem cloud-user@172.25.250.52
Warning: Permanently added '172.25.250.52"' (ECDSA) to the list of known hosts.
[cloud-user@webserver ~]$ systemctl status httpd.service
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: active (running) since Tue 2016-01-19 21:30:04 EST; 5min ago
...Output omitted...
[cloud-user@webserver ~]$ exit
```

14. 模板在数据库服务器上安装并配置了 MySQL 数据库，并且检索一个用于从 Web 服务器测试数据库连接的脚本。使用heat stack-show命令检查模板输出。查找测试连通性的页面的 Web 地址；该文件名为scaling_check-db.php。

```
[student@workstation stacks]# heat stack-show multi-tier
...Output omitted...
|   {                                                                 |
|     "output_value": "http://172.25.250.52/scaling_check-db.php",    |
|     "description": "This script allows you to test the connectivity
      to the database.",                                              |
|     "output_key": "page_url"                                        |
|   }
...Output omitted...
```

15. 从workstation，打开一个 Web 浏览器并导航到前面显示的 URL。使用以下信息填写字段。将 openstack server list命令返回的数据库浮动 IP 地址用于 MySQL hostname 字段：

*    字段	            值
*    MySQL 主机名	    192.168.1.16
*    MySQL 用户名	    root 用户
*    MySQL 密码	        红帽
*    MySQL 数据库名称	    mysql

单击提交。验证后，脚本应显示一条消息，表明其能够连接到数据库：

```
Test MySQL step 2
Congratulations!
Successfully connected to the database.
```


### 自动横向扩展和内向收缩

###### 使用 Heat 自动缩放

Heat 编配服务实施自动缩放功能，这使得管理员可以动态横向扩展和内向收缩其堆栈。Heat 与 Ceilometer 警报通信（后者由 Aodh 服务管理），以确定触发扩展的条件。管理员可以监控各种不同资源，包括：

*    CPU 负载
*    CPU 使用率
*    磁盘使用量
*    网络使用量

Ceilometer 监控的所有指标几乎都能用于动态缩放 Heat 堆栈。下列 Heat 资源类型可用于动态缩放堆栈：

常见 Heat 资源类型

OS::Heat::AutoScalingGroup
    1. 资源可缩放任意资源。必要属性包括 max_size、min_size 和 resource，后者是 HOT 模板。
    2. 可选属性包括 cooldown、desired_capacity 和 rolling_updates。

OS::Heat::ScalingPolicy
    1. 此资源管理 OS::Heat::AutoScalingGroup 资源。必要属性包括 adjustment_type、auto_scaling_group_id 和 scaling_adjustment。
    2. 可选属性包括 cooldown 和d min_adjustment_step。
    
OS::Ceilometer::Alarm
    1. 此资源定义 Ceilometer 警报。管理员可以定义在受监视资源满足指定条件时要执行的操作。例如，该资源可以监控内存耗用情况并调用一个操作（利用 alarm_action 属性）。必要属性包括 meter_name 和 threshold。
    2. 可选属性包括 alarm_actions、comparison_operator和 evaluation_periods。

OS::Neutron::HealthMonitor
    1. 此资源允许管理员创建运行状况监视器（由 Neutron LBaaS 支持）。运行状况监视器是一种 Neutron 服务，用于检查实例是否仍然在指定的协议和端口上运行。必要属性包括 delay、max_retries、pool、timeout 和 type。
    2. 可选属性包括 expected_codes、http_method 和 url_path。

OS::Neutron::Pool
    1. 此资源管理 Neutron 池，后者代表一组节点。池定义节点所驻留的子网、平衡算法和节点本身。管理员可以使用此资源创建面向前端的负载平衡器，后者本身将传入请求重新路由到后端服务器。必要属性包括 lb_algorithm、listener和 protocol。
    2. 可选属性包括 session_persistence。

OS::Neutron::LoadBalancer
    1. 此资源可以和 OS::Neutron::Pool 资源搭配，用于管理 Neutron 负载平衡器。它定义将传入请求路由到 Neutron 池中定义的实例的负载平衡器。必要属性包括 pool_id 和 protocol_port。

管理员可以在模板中使用这些资源来执行各种不同任务。例如：

*    创建在网络流量达到特定阈值时自动缩放的 Web 服务器场
*    创建在传入连接数达到特定阈值时自动缩放的数据库群集
*    创建在 CPU 使用率超过特定水平时自动缩放的应用服务器组

Heat 资源可以动态检索 Ceilometer 警报的值，并通过添加或删除资源来执行实时操作。自动缩放可被用于构建复杂的环境，通过动态添加或移除资源来自动调整其容量。这有助于性能、可用性，以及对基础架构成本和使用量的掌控。

> 警告: 当堆栈内向收缩时，写入实例的数据将丢失。管理员必须确保存储这些数据，例如，保存到文件服务器上。

###### Heat 自动缩放模板

在部署可缩放堆栈时，通常需要三个文件：

*    定义可缩放堆栈的模板文件。
*    包含要缩放的资源的模板文件。
*    定义或覆盖模板变量的环境文件。

以下文件演示了如何实施可缩放的堆栈。首先是定义用于部署可缩放堆栈的变量和资源的模板文件。resources 部分包含导入要扩展的资源的 OS::Heat::AutoScalingGroup 资源类型、横向扩展和内向收缩策略，以及一组 Ceilometer 警报：

```
...Output omitted...
parameters:
  server_name:
    type: string
    description: Name of the instance to create
  db_root_password:                                 -----------------------》  1
    type: "OS::Heat::RandomString"
    properties:
      length: 16
      sequence: lettersdigits

resources:
  autoscaling_group:
    type: OS::Heat::AutoScalingGroup                -----------------------》  2
    properties:
      cooldown: 60
      desired_capacity: 1
      min_size: 1                                   -----------------------》  3
      max_size: 5
      resource:
        type: scaling_db_servers.yaml               -----------------------》  4
        properties:
          db_root_password: { get_param: db_root_password }
          server_name: { get_param: server_name }
          pool_id: { get_resource: pool }
          public_net: { get_param: public_net }
          metadata: { "metering.stack": {get_param: "OS::stack_id"} }

  scaleup_policy:
    type: OS::Heat::ScalingPolicy                   -----------------------》  5
    properties:
      adjustment_type: change_in_capacity
      auto_scaling_group_id: { get_resource: autoscaling_group }
      cooldown: 45
      scaling_adjustment: 1
  scaledown_policy:
    type: OS::Heat::ScalingPolicy
    properties:
      adjustment_type: change_in_capacity
      auto_scaling_group_id: { get_resource: autoscaling_group }
      cooldown: 45
      scaling_adjustment: -1
  cpu_alarm_high:
    type: OS::Ceilometer::Alarm                     -----------------------》  6
    properties:
      description: Scale-up if the average CPU > 30% for 30 seconds
      meter_name: cpu_util
      statistic: avg
      period: 30
      evaluation_periods: 1
      threshold: 30
      alarm_actions:
        - { get_attr: [scaleup_policy, alarm_url] }
      comparison_operator: gt
  cpu_alarm_low:
    type: OS::Ceilometer::Alarm
    properties:
      description: Scale-down if the average CPU < 15% for 10 minutes
      meter_name: cpu_util
      statistic: avg
      period: 600
      evaluation_periods: 1
      threshold: 15
      alarm_actions:
        - { get_attr: [scaledown_policy, alarm_url] }
      comparison_operator: lt
...Output omitted...
```

1. 声明由数据库模板使用的变量。
2. 实施 Heat 自动缩放功能。
3. 定义可缩放堆栈的属性。
4. 导入要缩放的资源。
5. 定义自动缩放策略。
6. 定义用于监控堆栈资源的 Ceilometer 警报。

下列模板文件scaling_db_servers.yaml包含数据库服务器的定义。该模板由 OS::Heat::AutoscalingGroup 资源管理：

```
...Output omitted...
parameters:
  server_name:
    type: string
    default: DBServer
    description: Name for the web server
  db_root_password:
    type: "OS::Heat::RandomString"
    properties:
      length: 16
      sequence: lettersdigits
    description: Root Password for the database
  pool_id:
    type: string
    default: 0
    description: Neutron pool to contact
  metadata:
    type: json
    default: { }
...Output omitted...
resources:
  web_server:
    type: OS::Nova::Server                          -----------------------》  1
    properties:
      name: { get_param: server_name }
      image: { get_param: image }
      flavor: { get_param: flavor }
      key_name: { get_param: key_name }
      networks:
        - port: { get_resource: net_port }
      user_data_format: RAW
      user_data:
      str_replace:
      template: |                                   -----------------------》  2
        #!/bin/bash -v
        # Setup MySQL root password and create a user
        yum -y install mariadb-server
        systemctl start mariadb
        mysqladmin -u root password db_rootpassword
        cat << EOF | mysql -u root --password=db_rootpassword
        CREATE DATABASE db_name;
        GRANT ALL PRIVILEGES ON db_name.* TO "db_user"@"localhost"
        IDENTIFIED BY "db_password";
        FLUSH PRIVILEGES;
        EXIT
        EOF
        setsebool -P httpd_can_network_connect_db=1
      params:
        db_rootpassword: { get_param: db_root_password }
  net_port:
    type: OS::Neutron::Port
    properties:
      network: { get_param: private_net }
      fixed_ips:
        - subnet: { get_param: private_subnet }
      security_groups: [{ get_param: security_group }]
...Output omitted...
```

1. 创建 Nova 实例。
2. 初始化数据库服务器。

下列环境文件覆盖模板文件中定义的部分变量：

```
parameters:                             -----------------------》  1
  server_name: production_db_server     -----------------------》  2
  image: mariadb_x86-64                 -----------------------》  3
  flavor: m2.large                      -----------------------》  4
  db_root_password: sRYgp@w)=Q+7        -----------------------》  5
```

1. 覆盖模板文件中定义的参数。
2. 覆盖服务器名称。
3. 指定用于这些服务器的 Glance 映像。
4. 定义用于这些服务器的类别。
5. 覆盖默认的数据库密码。

###### 自动缩放工作流

下例演示了如何根据实例的 CPU 使用率自动缩放一个堆栈。

1. 使用环境文件和模板文件创建堆栈。

    [student@demo ~]$ heat stack-create -r -e scaling_environment.yaml -f scaling_template.yaml db_production
    +--------------------------------------+-----------------+-----------------+---------------------+--------------+
    | id                                   | stack_name      | stack_status    | creation_time       | updated_time |
    +--------------------------------------+-----------------+-----------------+---------------------+--------------+
    | 141d0d73-30e0-41aa-b599-d469e0fafd75 |  db_production  | CREATE_COMPLETE | 2016-07-14T13:34:57 | None         |
    +--------------------------------------+-----------------+-----------------+---------------------+--------------+

2. 创建用于监控 CPU 使用率的两个 Ceilometer 警报。起初，堆栈没有缩放：

    +----------------------------+----------------------------------------------+-------+----------+---------+------------+---------------------------------+------------------+
    | Alarm ID                   | Name                                         | State | Severity | Enabled | Continuous | Alarm condition                 | Time constraints |
    +----------------------------+----------------------------------------------+-------+----------+---------+------------+---------------------------------+------------------+
    | 1e179d5d-107b-3a35ca188fa4 | prod_autoscaling-cpu_alarm_high-q22xr7mvfwmm | ok    | low      | True    | True       | cpu_util > 30.0 during 1 x 30s  | None             |
    | 3968fcac-db01-a8a5690b3ad1 | prod_autoscaling-cpu_alarm_low-sau3x7ht5zvg  | ok    | low      | True    | True       | cpu_util < 2.0 during 1 x 600s  | None             |
    +----------------------------+----------------------------------------------+-------+----------+---------+------------+---------------------------------+------------------+
    
3. 当 CPU 使用率超过 30%，Ceilometer 警报触发。相关警报的状态从 ok 或 alarm：

    +----------------------------+----------------------------------------------+----------+----------+---------+------------+---------------------------------+------------------+
    | Alarm ID                   | Name                                         | State    | Severity | Enabled | Continuous | Alarm condition                 | Time constraints |
    +----------------------------+----------------------------------------------+----------+----------+---------+------------+---------------------------------+------------------+
    | 1e179d5d-107b-3a35ca188fa4 | prod_autoscaling-cpu_alarm_high-q22xr7mvfwmm | alarm    | low      | True    | True       | cpu_util > 30.0 during 1 x 30s  | None             |
    | 3968fcac-db01-a8a5690b3ad1 | prod_autoscaling-cpu_alarm_low-sau3x7ht5zvg  | ok       | low      | True    | True       | cpu_util < 2.0 during 1 x 600s  | None             |
    +----------------------------+----------------------------------------------+----------+----------+---------+------------+---------------------------------+------------------+

4. Ceilometer 调用 Heat 以横向扩展堆栈：


    +-------------------+--------------------------------------+--------------------------------------+--------------------+---------------------+
    | resource_name     | id                                   | resource_status_reason               | resource_status    | event_time          |
    +-------------------+--------------------------------------+--------------------------------------+--------------------+---------------------+
    | scaleup_policy    | ef6eb7a9-860c-45e6-93e6-2ee438adcf3e | state changed                        | CREATE_COMPLETE    | 2016-07-14T13:36:08 |
    | cpu_alarm_low     | 0933aed1-fd33-4a6e-9216-404770ff7da6 | state changed                        | CREATE_IN_PROGRESS | 2016-07-14T13:36:08 |
    | cpu_alarm_high    | 0b30eb80-3675-4bf0-80e4-0d106f377e6c | state changed                        | CREATE_IN_PROGRESS | 2016-07-14T13:36:09 |
    | cpu_alarm_low     | 8fe61dbb-dfb6-4aca-b56b-051ce5ddff2d | state changed                        | CREATE_COMPLETE    | 2016-07-14T13:36:10 |
    | cpu_alarm_high    | e424b8fb-deab-4435-9df8-43d295ab5305 | state changed                        | CREATE_COMPLETE    | 2016-07-14T13:36:10 |
    | db_production     | b3674c58-4eba-4973-9816-4599579d5d7f | Stack CREATE completed successfully  | CREATE_COMPLETE    | 2016-07-14T13:36:10 |
    +-------------------+--------------------------------------+--------------------------------------+--------------------+---------------------+

5. Heat 检查模板并创建必要的资源：

    +--------------------------------------+---------------+--------+------------+-------------+-----------------------------------------+
    | ID                                   | Name          | Status | Task State | Power State | Networks                                |
    +--------------------------------------+---------------+--------+------------+-------------+-----------------------------------------+
    | 366451bb-6f64-4bb1-b91d-385b68f874b7 | db_production | BUILD  | -          | Running     | dev_privnet=192.168.1.16, 172.25.250.51 |
    | 366451bb-6f64-4bb1-b91d-385b68f874b7 | db_production | ACTIVE | -          | Running     | dev_privnet=192.168.1.20, 172.25.250.55 |
    +--------------------------------------+---------------+--------+------------+-------------+-----------------------------------------+

6. 当 CPU 负载下降时，监控 CPU 使用率下限的 Ceilometer 警报触发，使 Heat 内向收缩堆栈：

    +-------------------+--------------------------------------+--------------------------------------+--------------------+---------------------+
    | resource_name     | id                                   | resource_status_reason               | resource_status    | event_time          |
    +-------------------+--------------------------------------+--------------------------------------+--------------------+---------------------+
    | scaledown_policy  | 29b6497f-551d-4c91-b5b0-2b7d6ee5191f | state changed                        | CREATE_COMPLETE    | 2016-07-14T13:45:04 |
    | scaleup_policy    | ef6eb7a9-860c-45e6-93e6-2ee438adcf3e | state changed                        | CREATE_COMPLETE    | 2016-07-14T13:36:08 |
    | cpu_alarm_low     | 0933aed1-fd33-4a6e-9216-404770ff7da6 | state changed                        | CREATE_IN_PROGRESS | 2016-07-14T13:36:08 |
    | cpu_alarm_high    | 0b30eb80-3675-4bf0-80e4-0d106f377e6c | state changed                        | CREATE_IN_PROGRESS | 2016-07-14T13:36:09 |
    | cpu_alarm_low     | 8fe61dbb-dfb6-4aca-b56b-051ce5ddff2d | state changed                        | CREATE_COMPLETE    | 2016-07-14T13:36:10 |
    | cpu_alarm_high    | e424b8fb-deab-4435-9df8-43d295ab5305 | state changed                        | CREATE_COMPLETE    | 2016-07-14T13:36:10 |
    | db_production     | b3674c58-4eba-4973-9816-4599579d5d7f | Stack CREATE completed successfully  | CREATE_COMPLETE    | 2016-07-14T13:36:10 |
    +-------------------+--------------------------------------+--------------------------------------+--------------------+---------------------+
    
7. 响应第一次警报时创建的多余资源被移除：

    +--------------------------------------+---------------+--------+------------+-------------+-----------------------------------------+
    | ID                                   | Name          | Status | Task State | Power State | Networks                                |
    +--------------------------------------+---------------+--------+------------+-------------+-----------------------------------------+
    | 366451bb-6f64-4bb1-b91d-385b68f874b7 | db_production | ACTIVE | deleting   | Running     | dev_privnet=192.168.1.16, 172.25.250.51 |
    | 366451bb-6f64-4bb1-b91d-385b68f874b7 | db_production | ACTIVE | -          | Running     | dev_privnet=192.168.1.20, 172.25.250.55 |
    +--------------------------------------+---------------+--------+------------+-------------+-----------------------------------------+

### 练习：自动横向扩展和内向收缩

1. 从 workstation，以 student 用户身份打开一个新终端。创建 ~/Downloads/scaling 目录并更改到其中：

```
[student@workstation ~]$ mkdir ~/Downloads/scaling
[student@workstation ~]$ cd ~/Downloads/scaling
```

2. 从 http://materials.example.com/heat/ 检索两个 Heat 模板文件，即 scaling_template.yaml 和 scaling_web_server.yaml。

```
[student@workstation scaling]$ wget http://materials.example.com/heat/scaling_template.yaml
...Output omitted...
100%[======================================>] 5,425       --.-K/s   in 0s

2016-07-13 22:27:52 (255 MB/s) - ‘scaling_template.yaml’ saved [5425/5425]
[student@workstation scaling]$ wget http://materials.example.com/heat/scaling_web_server.yaml
...Output omitted...
100%[======================================>] 2,715       --.-K/s   in 0s

2016-07-13 22:28:25 (435 MB/s) - ‘scaling_web_server.yaml’ saved [2715/2715]
```

3. 检查 scaling_template.yaml 文件。查看模板文件需要的参数，以及堆栈使用的资源。检查 OS::Heat::AutoScalingGroup 部分，该部分定义横向扩展和内向收缩堆栈的规则。min_size 和 max_size 属性定义扩展资源的范围。

```
type: scaling_web_server.yaml 资源调用 scaling_web_server.yaml 模板文件，作为要扩展的资源。properties 用于覆盖模板文件需要的参数。

...Output omitted...
  # Auto scaling group
  autoscaling_group:
    type: OS::Heat::AutoScalingGroup
    properties:
      cooldown: 60
      desired_capacity: 1
      min_size: 1
      max_size: 5
      resource:
        type: scaling_web_server.yaml
        properties:
          key_name: { get_param: key_name }
          image: { get_param: image }
          server_name: { get_param: server_name }
          flavor: { get_param: flavor }
          pool_id: { get_resource: pool }
          public_net: { get_param: public_net }
          private_net: { get_param: private_net }
          private_subnet: { get_param: private_subnet }
          metadata: { "metering.stack": {get_param: "OS::stack_id"} }
          security_group: { get_param: security_group }
...Output omitted...
```

4. 检查 scaling_web_server.yaml 模板文件。查看必要的参数，以及模板文件所定义的资源。

```
...Output omitted...
    type: OS::Nova::Server
    properties:
      name: { get_param: server_name }
      image: { get_param: image }
      flavor: { get_param: flavor }
      key_name: { get_param: key_name }
      networks:
        - port: { get_resource: net_port }
      user_data_format: RAW
      user_data: |
        #!/bin/bash
        echo "Hello world"
        echo "Setting up the Web Server"
        yum install -y php php-mysql wget mariadb-server
        systemctl start httpd
        systemctl start mariadb
        mysqladmin password redhat
        wget -P /var/www/html/ http://classroom.example.com/materials/heat/scaling_check-db.php
        wget -P /home/cloud-user/ http://classroom.example.com/materials/heat/scaling_launchme.sh
        chmod +x /home/cloud-user/scaling_launchme.sh
        chown cloud-user:cloud-user /home/cloud-user/scaling_launchme.sh
        setsebool -P httpd_can_network_connect_db=1
...Output omitted...
```

5. type: OS::Nova::Server 部分指出将生成一个实例。该模板使用 user_data 功能配置 Web 服务器和数据库服务器。它也会检索将用于压力测试服务器的脚本。

模板文件要求定义下列资源的环境文件：

*    SSH 密钥对的名称。
*    映像的名称。
*    实例的名称。
*    专用网络的名称。
*    专用子网的名称。
*    公共用网络的名称。
*    MySQL root 用户的密码。

创建名为 scaling_environment.yaml 的环境文件。在其中填充下列必要的参数：

```
parameters:
  key_name: dev_key1
  image: small
  server_name: dev_WebServer
  flavor: m2.small
  private_subnet: dev_privsub
  private_net: dev_privnet
  db_root_password: redhat
  public_net: dev_pubnet
```

> 警告: 在各参数定义前包含两个空格。省略这些空格会导致解析错误。

6. 提供/home/student/overcloudrc文件。

```
[student@workstation scaling]$ source ~/overcloudrc
```

7. 使用 heat stack-create 命令创建 dev_autoscaling 堆栈。包含 -e 选项以指定 scaling_environment.yaml 作为环境文件，并且包含 -f 选项以指定 scaling_template.yaml 作为模板。指定 -r 选项，在 Heat 资源创建失败时自动回滚堆栈。

[student@workstation scaling]$ heat stack-create -r -e scaling_environment.yaml -f scaling_template.yaml dev_autoscaling
+--------------------------------------+-----------------+--------------------+---------------------+--------------+
| id                                   | stack_name      | stack_status       | creation_time       | updated_time |
+--------------------------------------+-----------------+--------------------+---------------------+--------------+
| 3e25833c-a845-43cb-b80b-7d25b5a1ea72 | dev_autoscaling | CREATE_IN_PROGRESS | 2016-07-13T20:46:52 | None         |
+--------------------------------------+-----------------+--------------------+---------------------+--------------+

8. 使用watch命令，监控堆栈的创建。等待 stack_status 的值从 CREATE_IN_PROGRESS 变为 CREATE_COMPLETE。

[student@workstation scaling]$ watch -n 5 heat stack-list
+--------------------------------------+-----------------+-----------------+---------------------+--------------+
| id                                   | stack_name      | stack_status    | creation_time       | updated_time |
+--------------------------------------+-----------------+-----------------+---------------------+--------------+
| 53c968ff-262f-4c06-a25f-8809a793d417 | dev_autoscaling | CREATE_COMPLETE | 2016-07-13T21:05:45 | None         |
+--------------------------------------+-----------------+-----------------+---------------------+--------------+

按 Ctrl+C 组合键退出该命令。

> 注意: 运行 heat stack-list 命令需要一些时间，所以给它多一点时间后再尝试。

9. 使用 heat stack-show 命令查看堆栈的相关信息。输出中包含模板中定义的信息。

注意 output_value 行显示了 scaling_check-db.php 脚本的 URL。此脚本测试从 Web 服务器到数据库的连通性。

```
[student@workstation scaling]$ heat stack-show dev_autoscaling
+-----------------------+-----------------------------------------------------+
| Property              | Value                                               |
+-----------------------+-----------------------------------------------------+
| capabilities          | []                                                  |
| creation_time         | 2016-07-13T20:46:52                                 |
| description           | HOT template that scales web servers. Creates an    |
                          extra database server that is not scaled.           |
| disable_rollback      | False                                               |
| id                    | 3e25833c-a845-43cb-b80b-7d25b5a1ea72                |
| links                 | http://192.168.0.11:8004/v1/
                          0f965ea9b1df436288e2fcb6fec503fb/stacks/            |
                          dev_autoscaling/3e25833c-a845-43cb-b80b-7d25b5a1ea7 |
| notification_topics   | []                                                  |
| outputs               | [                                                   |
|                       |   {                                                 |
|                       |     "output_value": "192.168.1.16",                 |
|                       |     "description": "The IP address of the load      |
                              balancing pool",                                |
|                       |     "output_key": "pool_ip_address"                 |
|                       |   },                                                |
|                       |   {                                                 |
|                       |     "output_value": "http://172.25.251.11:8000/     |
|                       |     "description": "This URL is the webhook to scale|
                                              up the autoscaling group.       |
                                              You can invoke the scale-up     |
                                              operation by doing an HTTP POST |
                                              to this URL; no body nor extra  |
                                              headers are needed.\n",         |
|                       |     "output_key": "scale_up_url"                    |
...Output omitted...
|                       |     "output_value": "http://172.25.250.51/",        |
|                       |     "description": "This URL is the external        |
                              URL that can be used to access the Website.\n", |
|                       |     "output_key": "website_url"                     |
|                       |   },                                                |
|                       |   {                                                 |
|                       |     "output_value": "http://172.25.250.51/          |
                               scaling_check-db.php",                         |
|                       |     "description": "This script allows you to test  |
                               the connectivity to the database.\n",          |
|                       |     "output_key": "page_url"                        |
|                       |   }                                                 |
|                       | ]                                                   |
...Output omitted...
|                       |   "OS::stack_name": "dev_autoscaling",              |
|                       |   "key_name": "dev_key1",                           |
|                       |   "image": "small",                                 |
|                       |   "private_subnet": "dev_privsub",                  |
|                       |   "private_net": "dev_privnet",                     |
|                       |   "db_root_password": "redhat",                     |
|                       |   "security_group": "default",                      |
|                       |   "flavor": "m2.small",                             |
|                       |   "public_net": "dev_pubnet"                        |
|                       | }                                                   |
| parent                | None                                                |
| stack_name            | dev_autoscaling                                     |
| stack_owner           | None                                                |
| stack_status          | CREATE_COMPLETE                                     |
| stack_status_reason   | Stack CREATE completed successfully                 |
| stack_user_project_id | 7f89c4367ec34c1bb22454acef8072b1                    |
| tags                  | null                                                |
| template_description  | HOT template that scales web servers.               |
                          Creates an extra database server that is            |
                          not scaled.                                         |
| timeout_mins          | None                                                |
| updated_time          | None                                                |
+-----------------------+-----------------------------------------------------+
```

10. 从 workstation 打开 Firefox，然后再打开 output_value 显示的 URL。以上屏幕中显示的 URL 是 http://172.25.250.51/scaling_check-db.php，但实际的 IP 地址可能有所不同。将下列值填写到出现的 Web 表单的字段中：

*    字段	            收益总值
*    MySQL Hostname	    本地主机
*    MySQL Username	    root 用户
*    MySQL Password	    红帽

填好字段后，请单击 Submit。应出现以下消息：

```
Test MySQL step 2
Congratulations!
Successfully connected to the database
```

11. 从workstation终端，使用ceilometer alarm-list命令列出创建的警报。

[student@workstation scaling]$ ceilometer alarm-list
+----------------------------------+---------------------------------------------+-------+----------+---------+------------+---------------------------------+------------------+
| Alarm ID                         | Name                                        | State | Severity | Enabled | Continuous | Alarm condition                 | Time constraints |
+----------------------------------+---------------------------------------------+-------+----------+---------+------------+---------------------------------+------------------+
| 4b173374-4798-9625-44d3f4a96e87  | dev_autoscaling-cpu_alarm_low-tt7qwhtl4h7n  | alarm | low      | True    | True       | cpu_util < 15.0 during 1 x 600s | None             |
| 7df0d905-4a73-9ca0-0cdb9b34138f  | dev_autoscaling-cpu_alarm_high-c5xobmyeyr27 | ok    | low      | True    | True       | cpu_util > 30.0 during 1 x 30s  | None             |
+----------------------------------+---------------------------------------------+-------+----------+---------+------------+---------------------------------+------------------+

当 CPU 使用率低于 15% 达到十分钟时，dev_autoscaling-cpu_alarm_low-tt7qwhtl4h7n 警报将触发。当 CPU 使用率高于 30% 达到三十秒时，dev_autoscaling-cpu_alarm_high-c5xobmyeyr27 警报将触发。这两个警报都由 Heat 用于横向扩展和内向收缩堆栈。

12. 使用 openstack 命令显示与 dev_WebServer 实例关联的浮动 IP 地址。示例输出中显示浮动 IP 地址为 172.25.250.54，但实际的 IP 地址可能有所不同。

[student@workstation scaling]$ openstack server list
+--------------------------------------+---------------+--------+-----------------------------------------+
| ID                                   | Name          | Status | Networks                                |
+--------------------------------------+---------------+--------+-----------------------------------------+
| c0daa5c6-3993-4745-bf0e-7afc5a9b22fa | dev_WebServer | ACTIVE | dev_privnet=192.168.1.19, 172.25.250.54 |
+--------------------------------------+---------------+--------+-----------------------------------------+

13. 使用 ssh 以及位于 /home/student/Downloads/dev_key1.pem 的公钥，通过其公共 IP 地址连接实例。在上面的输出中，实例具有浮动 IP 地址 172.25.250.54。

[student@workstation scaling]$ ssh -i /home/student/Downloads/dev_key1.pem cloud-user@172.25.250.54
[cloud-user@dev_WebServer ~]$ 

14. 模板文件安装了一个名为 scaling_launchme.sh 的脚本。该脚本对服务器进行压力测试，并触发监控 CPU 高使用率的 Ceilometer 警报。运行该脚本。

[cloud-user@dev_WebServer ~]$ ./scaling_launchme.sh
Stressing the server...

15. 从workstation打开一个新终端，再提供overcloudrc凭据文件，然后使用watch命令监控 Ceilometer 警报。等待 dev_autoscaling-cpu_alarm_high-c5xobmyeyr27 警报的状态从 ok 过渡到 alarm。

[student@workstation ~]$ source overcloudrc
[student@workstation ~]$ watch -n 5 ceilometer alarm-list
+------------------------------+---------------------------------------------+-------+----------+---------+------------+---------------------------------+------------------+
| Alarm ID                     | Name                                        | State | Severity | Enabled | Continuous | Alarm condition                 | Time constraints |
+------------------------------+---------------------------------------------+-------+----------+---------+------------+---------------------------------+------------------+
...Output omitted...                                                            ...Output omitted...
| 7df0d905-9ca0-0cdb9b34138f   | dev_autoscaling-cpu_alarm_high-c5xobmyeyr27 | alarm | low      | True    | True       | cpu_util > 30.0 during 1 x 30s  | None             |
+------------------------------+---------------------------------------------+-------+----------+---------+------------+---------------------------------+------------------+

16. 等待一两分钟，再使用 watch 和 openstack 命令监控实例。Heat 将通过生成两个新实例来自动横向扩展资源。

[student@workstation ~]$ watch -n 5 openstack server list
Every 5.0s: openstack server list        Wed Jul 13 23:29:47 2016

+--------------------------------------+---------------+--------+------------+-------------+-----------------------------------------+
| ID                                   | Name          | Status | Task State | Power State | Networks                                |
+--------------------------------------+---------------+--------+------------+-------------+-----------------------------------------+
| 366451bb-6f64-4bb1-b91d-385b68f874b7 | dev_WebServer | BUILD  | -          | Running     | dev_privnet=192.168.1.16, 172.25.250.51 |
| cde7ba51-4263-4de8-89bf-ba7121960fbe | dev_WebServer | ACTIVE | -          | Running     | dev_privnet=192.168.1.20, 172.25.250.55 |
| c0daa5c6-3993-4745-bf0e-7afc5a9b22fa | dev_WebServer | ACTIVE | -          | Running     | dev_privnet=192.168.1.19, 172.25.250.54 |
+--------------------------------------+---------------+--------+------------+-------------+-----------------------------------------+

17. 当脚本创建的后台进程停止运行时，Heat 将通过删除之前创建的两个实例来内向收缩资源。

返回到与 dev_WebServer 实例连接的终端会话。运行 top 命令，确保调用 yes 的进程不再运行。

```
[cloud-user@dev_WebServer ~]$ top
top - 17:34:39 up 27 min,  1 user,  load average: 0.97, 0.89, 0.54
...Output omitted...
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 1447 cloud-u+  20   0  107896    620    528 R 100.0  0.1  11:20.31 yes
...Output omitted...
```


18. 当 yes 进程结束时，返回到正在运行 openstack 命令的 workstation 终端。观察输出中之前创建的两个实例被删除掉。两个实例被删除后，按 Ctrl+C 组合键退出 watch 命令。

```
Every 1.0s: openstack server list        Wed Jul 13 23:29:47 2016

+--------------------------------------+---------------+--------+------------+-------------+-----------------------------------------+
| ID                                   | Name          | Status | Task State | Power State | Networks                                |
+--------------------------------------+---------------+--------+------------+-------------+-----------------------------------------+
| 366451bb-6f64-4bb1-b91d-385b68f874b7 | dev_WebServer | ACTIVE | deleting   | Running     | dev_privnet=192.168.1.16, 172.25.250.51 |
| c0daa5c6-3993-4745-bf0e-7afc5a9b22fa | dev_WebServer | ACTIVE | -          | Running     | dev_privnet=192.168.1.20, 172.25.250.55 |
| c0daa5c6-3993-4745-bf0e-7afc5a9b22fa | dev_WebServer | ACTIVE | -          | Running     | dev_privnet=192.168.1.19, 172.25.250.54 |
+--------------------------------------+---------------+--------+------------+-------------+-----------------------------------------+

Every 1.0s: openstack server list        Wed Jul 13 23:37:13 2016

+--------------------------------------+---------------+--------+------------+-------------+-----------------------------------------+
| ID                                   | Name          | Status | Task State | Power State | Networks                                |
+--------------------------------------+---------------+--------+------------+-------------+-----------------------------------------+
| c0daa5c6-3993-4745-bf0e-7afc5a9b22fa | dev_WebServer | ACTIVE | -          | Running     | dev_privnet=192.168.1.19, 172.25.250.54 |
+--------------------------------------+---------------+--------+------------+-------------+-----------------------------------------+
```

19. 检查 Ceilometer 警报的状态。

[student@workstation ~]$ ceilometer alarm-list
+----------------------------------+---------------------------------------------+-------+----------+---------+------------+---------------------------------+------------------+
| Alarm ID                         | Name                                        | State | Severity | Enabled | Continuous | Alarm condition                 | Time constraints |
+----------------------------------+---------------------------------------------+-------+----------+---------+------------+---------------------------------+------------------+
| 4b173374-4798-9625-44d3f4a96e87  | dev_autoscaling-cpu_alarm_low-tt7qwhtl4h7n  | alarm | low      | True    | True       | cpu_util < 15.0 during 1 x 600s | None             |
| 7df0d905-4a73-9ca0-0cdb9b34138f  | dev_autoscaling-cpu_alarm_high-c5xobmyeyr27 | ok    | low      | True    | True       | cpu_util > 30.0 during 1 x 30s  | None             |
+----------------------------------+---------------------------------------------+-------+----------+---------+------------+---------------------------------+------------------+

20. 检查 dev_autoscaling-cpu_alarm_high-c5xobmyeyr27 警报的历史记录。使用 ceilometer alarm-history 命令及 Alarm ID 来检查警报。

[student@workstation ~]$ ceilometer alarm-history 7df0d905-46d9-4a73-9ca0-0cdb9b34138f
+------------------+----------------------------+---------------------------------------------------------------+
| Type             | Timestamp                  | Detail                                                        |
+------------------+----------------------------+---------------------------------------------------------------+
| state transition | 2016-07-13T21:32:56.207000 | state: ok                                                     |
| state transition | 2016-07-13T21:23:56.271000 | state: alarm                                                  |
| state transition | 2016-07-13T21:07:56.206000 | state: ok                                                     |
| creation         | 2016-07-13T21:06:43.932000 | name: dev_autoscaling-cpu_alarm_high-c5xobmyeyr27             |
|                  |                            | description: Scale-up if the average CPU > 30% for 30 seconds |
|                  |                            | type: threshold                                               |
|                  |                            | rule: cpu_util > 30.0 during 1 x 30s                          |
|                  |                            | severity: low                                                 |
|                  |                            | time_constraints: None                                        |
+------------------+----------------------------+---------------------------------------------------------------+


### 实验：部署可缩放堆栈

在本实验中，您将部署两个 Heat 堆栈；第一堆栈在现有网络环境中配置两个 Web 服务器。第一堆栈配置两个 Web 服务器，分别拥有自己的 Web 页面。该堆栈还向 Web 服务器关联浮动 IP 地址。

第二堆栈实施 Ceilometer 警报，并根据资源使用情况自动缩放。

环境中提供有下列资源：

*    Keystone 凭据文件，位于/home/student/overcloudrc。
*    Glance 映像 small。
*    Nova 类别 m2.small。
*    公共网络 lab_pubnet。
*    专用网络 lab_privnet。
*    专用子网 lab_pubsub，位于 172.25.250.0/24 范围内。该网络将提供外部连接。
*    专用子网 lab_privsub，位于 192.168.1.0/24 范围内。该网络运行供实例使用的 DHCP 服务器。
*    路由器 lab_router1 与公共网络和专用网络连接，确保外部连接。
*    密钥对 lab_key1。私钥位于 /home/student/Downloads/lab/lab_key1.pem 中。

实验的工作目录是 /home/student/Downloads/lab/。

使用下列详细信息，启动第一个 Heat 堆栈 lab_web_servers：

*    堆栈名称	    lab_web_servers
*    环境文件	    /home/student/Downloads/lab/environment_web.yaml


使用下列详细信息，启动第二个 Heat 堆栈 lab_web_scaling：

*    堆栈名称	lab_scaling
*    环境文件	/home/student/Downloads/lab/scaling_environment.yaml

1. 准备第一堆栈

以 student 用户身份登录 workstation.lab.example.com。从 http://materials.example.com/heat/web_servers.yaml 检索 HOT 模板文件，并将它保存在 ~/Downloads/lab/ 下，然后检查其内容。

    a. 从 workstation，打开一个终端并导航到 lab 目录。

    [student@workstation ~]$ cd Downloads/lab

    b. 提供 overcloud 凭据文件overcloudrc。

    [student@workstation lab]$ source ~/overcloudrc

    c. 从http://materials.example.com/heat/web_servers.yaml检索 Heat 模板文件。将它保存在/home/student/Downloads/lab下。

    [student@workstation lab]$ wget http://materials.example.com/heat/web_servers.yaml
    ...Output omitted...
    2016-01-20 23:50:21 (546 MB/s) - ‘web_servers.yaml’ saved [4584/4584]

    d. 打开该文件，并检查 Heat 模板文件需要的参数。下列屏幕突出显示了环境文件中需要值的相关资源。
    
    ```
    [student@workstation lab]$ cat web_servers.yaml
    parameters:
      image:
        type: string
        description: Image used for servers
    ...Output omitted...
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
    ```

2. 检索有关 OpenStack 环境的信息

使用openstack命令检索有关环境的信息。使用此信息填充 Heat 环境文件。

    a. 检索密钥对的名称。

    [student@workstation lab]$ openstack keypair list
    +----------+-------------------------------------------------+
    | Name     | Fingerprint                                     |
    +----------+-------------------------------------------------+
    | lab_key1 | c2:b7:37:2d:99:82:23:34:6c:d4:e8:9d:6d:6d:3d:ec |
    +----------+-------------------------------------------------+

    b. 检索网络的名称。

    [student@workstation lab]$ openstack network list
    +--------------------------------------+-------------+--------------------------------------+
    | ID                                   | Name        | Subnets                              |
    +--------------------------------------+-------------+--------------------------------------+
    | 4f037379-da19-44e7-b316-42c782a2c706 | lab_privnet | f0ae9684-e8f8-4a8e-8ecc-18d592028787 |
    | 79a62e32-3f8f-4c18-bf61-2839c96fedad | lab_pubnet  | 3d37be98-4c40-40de-830d-fd0eccc7d6c2 |
    +--------------------------------------+-------------+--------------------------------------+

3. 创建环境文件

在检查资源后，创建所需的环境文件environment_web.yaml。将 small 用作映像、m2.small 用作类别、lab_pubnet 用作公共网络、lab_privnet 用作专用网络、lab_privsub 用作专用子网，以及 lab_key1 用作密钥对。

创建 /home/student/Downloads/lab/environment_web.yaml 文件并添加以下参数：

```
parameters:
 public_net: lab_pubnet
 private_net: lab_privnet
 private_subnet: lab_privsub
 key_name: lab_key1
 flavor: m2.small
 image: small
```

4. 启动 lab_web_servers 堆栈

检索了文件后，使用回滚选项启动堆栈。

    a. Heat 堆栈现已就绪，可以启动。将模板文件和环境文件作为参数传递。

    [student@workstation lab]$ heat stack-create -r -f web_servers.yaml -e environment_web.yaml lab_web_servers
    +--------------------------------------+-----------------+--------------------+---------------------+--------------+
    | id                                   | stack_name      | stack_status       | creation_time       | updated_time |
    +--------------------------------------+-----------------+--------------------+---------------------+--------------+
    | 7737b519-9795-40f4-9206-2c2b0f1435ed | lab_web_servers | CREATE_IN_PROGRESS | 2016-01-21T08:27:46 | None         |
    +--------------------------------------+-----------------+--------------------+---------------------+--------------+

    b. 使用heat stack-list命令，监控堆栈的状态。等它进入 CREATE_COMPLETE 状态，然后继续。

    [student@workstation lab]$ heat stack-list
    +--------------------------------------+-----------------+-----------------+---------------------+--------------+
    | id                                   | stack_name      | stack_status    | creation_time       | updated_time |
    +--------------------------------------+-----------------+-----------------+---------------------+--------------+
    | 7737b519-9795-40f4-9206-2c2b0f1435ed | lab_web_servers | CREATE_COMPLETE | 2016-01-21T08:27:46 | None         |
    +--------------------------------------+-----------------+-----------------+---------------------+--------------+

5. 访问实例

创建了堆栈后，可检索浮动 IP 地址。使用lab_key1.pem私钥连接实例，以测试 SSH 连通性。确认 httpd 服务正在运行。使用 workstation 上的 FireFox 确认 Web 服务器正在运行。

    a. 创建了堆栈后，使用 openstack server list 命令检查创建的两个服务器并检索其公共 IP 地址，它们在 172.25.250.0/24 范围内。

    [student@workstation lab]$ openstack server list
    +--------------------------------------+-------------+--------+------------+-------------+-----------------------------------------+
    | ID                                   | Name        | Status | Task State | Power State | Networks                                |
    +--------------------------------------+-------------+--------+------------+-------------+-----------------------------------------+
    | 5bcdb34b-1918-4e3e-a457-bafd3af442e1 | WebServer_a | ACTIVE | -          | Running     | lab_privnet=192.168.1.16, 172.25.250.51 |
    | c8590b17-f849-4fe3-85bf-2cfe5831c6da | WebServer_b | ACTIVE | -          | Running     | lab_privnet=192.168.1.17, 172.25.250.52 |
    +--------------------------------------+-------------+--------+------------+-------------+-----------------------------------------+

    b. 利用 lab_key1.pem 私钥，创建与第一 Web 服务器的 SSH 连接。使用 systemctl 命令查询 httpd 服务的状态。

    [student@workstation lab]$ ssh -i lab_key1.pem cloud-user@172.25.250.51
    [cloud-user@webserver-a ~]$ systemctl status httpd.service
    ● httpd.service - The Apache HTTP Server
       Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
       Active: active (running) since Thu 2016-01-21 03:30:30 EST; 26min ago
    ...Output omitted...
    [cloud-user@webserver-a ~]$ exit

    c. 对第二 Web 服务器重复同样的操作：发起 SSH 连接，然后查询 httpd 服务的状态。

    [student@workstation lab]$ ssh -i lab_key1.pem cloud-user@172.25.250.52
    [cloud-user@webserver-b ~]$ systemctl status httpd.service
    ● httpd.service - The Apache HTTP Server
       Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
       Active: active (running) since Thu 2016-01-21 03:30:31 EST; 27min ago
    ...Output omitted...
    [cloud-user@webserver-b ~]$ exit

    d. 在 workstation 上打开 Firefox，再导航到 http://172.25.250.51。将 IP 地址替换为您的实例的 IP 地址。Web 页面中应当显示下列消息。

    This is the first Web server

    e. 导航到第二浮动 IP 地址，本例中为 172.25.250.52。应出现以下消息。

    This is the second Web server

6. 准备第二堆栈，即 lab_web_scaling 堆栈

检索第二堆栈 lab_web_scaling 的 Heat 模板。该堆栈需要两个文件，scaling_template.yaml 和 scaling_web_server.yaml，它们位于 http://materials.example.com/heat。将这两个文件保存到 /home/student/Downloads/lab 目录中，再检查它们。

    a. 在 workstation.lab.example.com 上，导航到 /home/student/Downloads/lab，再使用 wget 检索第二堆栈的主要 Heat 模板文件，它位于 http://materials.example.com/heat/scaling_template.yaml。

    ```
    [student@workstation lab]$ wget http://materials.example.com/heat/scaling_template.yaml
    ...Output omitted...
    100%[======================================>] 5,425       --.-K/s   in 0s

    2016-07-13 22:27:52 (255 MB/s) - ‘scaling_template.yaml’ saved [5425/5425]
    ```

    b. 从 http://materials.example.com/heat/scaling_web_server.yaml 检索模板文件 scaling_web_server.yaml。

    ```
    [student@workstation lab]$ wget http://materials.example.com/heat/scaling_web_server.yaml
    ...Output omitted...
    100%[======================================>] 2,715       --.-K/s   in 0s

    2016-07-13 22:28:25 (435 MB/s) - ‘scaling_web_server.yaml’ saved [2715/2715]
    ```

    c. 检查scaling_template.yaml文件。查看模板文件需要的参数，以及堆栈使用的资源。检查 OS::Heat::AutoScalingGroup 部分，该部分定义横向扩展和内向收缩堆栈的规则。min_size 和 max_size 属性定义扩展资源的范围。

    type: scaling_web_server.yaml 调用 scaling_web_server.yaml 模板文件，作为要扩展的资源。properties 用于覆盖模板文件需要的参数。

    ```
    ...Output omitted...
    resources:
      # Auto scaling group
      autoscaling_group:
        type: OS::Heat::AutoScalingGroup
        properties:
          cooldown: 60
          desired_capacity: 1
          min_size: 1
          max_size: 5
          resource:
            type: scaling_web_server.yaml
            properties:
              key_name: { get_param: key_name }
              image: { get_param: image }
              server_name: { get_param: server_name }
              flavor: { get_param: flavor }
              pool_id: { get_resource: pool }
              public_net: { get_param: public_net }
              private_net: { get_param: private_net }
              private_subnet: { get_param: private_subnet }
              metadata: { "metering.stack": {get_param: "OS::stack_id"} }
              security_group: { get_param: security_group }
    ...Output omitted...
    ```

    d. 检查scaling_web_server.yaml模板文件。查看所需的参数，以及模板文件定义的资源。type: OS::Nova::Server 指出将生成一个实例。该模板使用 user_data 功能配置 Web 和数据库服务器，并检索用于压力测试该服务器的远程文件。

    ```
    ...Output omitted...
    resources:
      web_server:
        type: OS::Nova::Server
        properties:
    ...Output omitted...
          user_data: |
            #!/bin/bash
            echo "Hello world"
            echo "Setting up the Web Server"
            yum install -y php php-mysql wget mariadb-server
            systemctl start httpd
            systemctl start mariadb
            mysqladmin password redhat
            wget -P /var/www/html/ http://classroom.example.com/materials/heat/scaling_check-db.php
            wget -P /home/cloud-user/ http://classroom.example.com/materials/heat/scaling_launchme.sh
            chmod +x /home/cloud-user/scaling_launchme.sh
            chown cloud-user:cloud-user /home/cloud-user/scaling_launchme.sh
            setsebool -P httpd_can_network_connect_db=1
    ...Output omitted...
    ```

7. 创建环境文件

为 lab_web_scaling 堆栈创建环境文件，存为 /home/student/Downloads/lab/scaling_environment.yaml。在环境文件中包含下列资源：

*    密钥对的名称，lab_key1
*    映像的名称，small
*    实例的名称，lab_WebServer
*    专用子网的名称，lab_privsub
*    专用网络的名称，lab_privnet
*    公共网络的名称，lab_pubnet
*    MySQL root 用户的密码，redhat

使用文本编辑器创建 scaling_environment.yaml 环境文件。在其中填充必要的变量。环境文件内容应当类似于如下：

```
parameters:
  key_name: lab_key1
  image: small
  server_name: lab_WebServer
  flavor: m2.small
  private_subnet: lab_privsub
  private_net: lab_privnet
  db_root_password: redhat
  public_net: lab_pubnet
```

> 警告: 请务必在各参数定义前插入两个空格，以避免语法错误。

8. 部署第二堆栈

将 scaling_environment.yaml 环境文件用作参数，创建第二堆栈 lab_web_scaling。使用回滚选项。

    a. Heat 堆栈现已就绪，可以启动。指定相关的选项，它们将模板文件和环境文件包含为参数。

    [student@workstation lab]$ heat stack-create -r -f scaling_template.yaml -e scaling_environment.yaml lab_web_scaling
    +--------------------------------------+-----------------+--------------------+---------------------+--------------+
    | id                                   | stack_name      | stack_status       | creation_time       | updated_time |
    +--------------------------------------+-----------------+--------------------+---------------------+--------------+
    ...Output omitted...
    | d463763e-72a9-4e7c-8592-0512e60eab7d | lab_web_scaling | CREATE_IN_PROGRESS | 2016-01-21T08:27:46 | None         |
    +--------------------------------------+-----------------+--------------------+---------------------+--------------+

    b. 使用heat stack-list命令，监控堆栈的状态。等它进入 CREATE_COMPLETE 状态，然后继续。

    [student@workstation lab]$ heat stack-list
    +--------------------------------------+-----------------+-----------------+---------------------+--------------+
    | id                                   | stack_name      | stack_status    | creation_time       | updated_time |
    +--------------------------------------+-----------------+-----------------+---------------------+--------------+
    ...Output omitted...
    | d463763e-72a9-4e7c-8592-0512e60eab7d | lab_web_scaling | CREATE_COMPLETE | 2016-01-21T08:27:46 | None         |
    +--------------------------------------+-----------------+-----------------+---------------------+--------------+

9. 压力测试堆栈

第二堆栈 lab_web_scaling 部署了 Ceilometer 来监控实例的 CPU 使用情况。从 workstation，使用 ssh 命令并以 cloud-user 用户身份连接第一个 lab_WebServer 实例，连接时请使用 /home/student/Downloads/lab/lab_key1.pem 中保存的密钥以及关联至该实例的浮动 IP 地址。登录后，运行 scaling_launchme.sh 脚本来压力测试服务器。

    a. 使用 openstack 命令检索与 lab_WebServer 实例关联的浮动 IP 地址。示例输出中显示浮动 IP 地址为 172.25.250.54，但实际的 IP 地址可能有所不同。

    [student@workstation lab]$ openstack server list
    +--------------------------------------+---------------+--------+-----------------------------------------+
    | ID                                   | Name          | Status | Networks                                |
    +--------------------------------------+---------------+--------+-----------------------------------------+
    ...Output omitted...
    | c0daa5c6-3993-4745-bf0e-7afc5a9b22fa | lab_WebServer | ACTIVE | lab_privnet=192.168.1.19, 172.25.250.54 |
    +--------------------------------------+---------------+--------+-----------------------------------------+

    b. 使用 ssh 通过其公共 IP 地址连接实例。在上面的输出中，实例使用浮动 IP 地址 172.25.250.54。需要连接的公钥位于 /home/student/Downloads/lab/lab_key1.pem。

    [student@workstation lab]$ ssh -i /home/student/Downloads/lab/lab_key1.pem cloud-user@172.25.250.54
    [cloud-user@lab_WebServer ~]$ 

    c. 模板文件下载了一个名为 scaling_launchme.sh 的脚本。该脚本对服务器进行压力测试，并触发指示 CPU 使用量较高的 Ceilometer 警报。运行该脚本。

    [cloud-user@lab_WebServer ~]$ ./scaling_launchme.sh
    Stressing the server...

10. 监控 Ceilometer 警报

该堆栈包含两个 Ceilometer 警报，即 lab_web_scaling-cpu_alarm_high 和 lab_web_scaling-cpu_alarm_low。使用 ceilometer 命令检查警报。确保 lab_web_scaling-cpu_alarm_high 触发，并指示 Heat 横向扩展资源。

    a. 从 workstation 打开一个新终端，再提供 overcloudrc 凭据文件，然后监控 Ceilometer 警报。等待 lab_web_scaling-cpu_alarm_high-c5xobmyeyr27 警报的状态从 ok 过渡到 alarm。

    [student@workstation ~]$ source overcloudrc
    [student@workstation ~]$ watch -n 5 ceilometer alarm-list
    +------------------------------+---------------------------------------------+-------+----------+---------+------------+---------------------------------+------------------+
    | Alarm ID                     | Name                                        | State | Severity | Enabled | Continuous | Alarm condition                 | Time constraints |
    +------------------------------+---------------------------------------------+-------+----------+---------+------------+---------------------------------+------------------+
    ...Output omitted...                                                                    ...Output omitted...
    | 7df0d905-9ca0-0cdb9b34138f   | lab_web_scaling-cpu_alarm_high-c5xobmyeyr27 | alarm | low      | True    | True       | cpu_util > 30.0 during 1 x 30s  | None             |
    +------------------------------+---------------------------------------------+-------+----------+---------+------------+---------------------------------+------------------+

    b. 一两分钟后，使用 watch 命令和 openstack 命令监控实例。Heat 将通过生成新实例来自动横向扩展资源。

    [student@workstation ~]$ watch -n 5 openstack server list
    Every 5.0s: nova list              Wed Jul 13 23:29:47 2016

    +--------------------------------------+---------------+--------+------------+-------------+-----------------------------------------+
    | ID                                   | Name          | Status | Task State | Power State | Networks                                |
    +--------------------------------------+---------------+--------+------------+-------------+-----------------------------------------+
    ...Output omitted...
    | 366451bb-6f64-4bb1-b91d-385b68f874b7 | lab_WebServer | BUILD  | -          | Running     | lab_privnet=192.168.1.16, 172.25.250.51 |
    | cde7ba51-4263-4de8-89bf-ba7121960fbe | lab_WebServer | ACTIVE | -          | Running     | lab_privnet=192.168.1.20, 172.25.250.55 |
    | c0daa5c6-3993-4745-bf0e-7afc5a9b22fa | lab_WebServer | ACTIVE | -          | Running     | lab_privnet=192.168.1.19, 172.25.250.54 |
    +--------------------------------------+---------------+--------+------------+-------------+-----------------------------------------+

11. 内向收缩堆栈

等待一两分钟，等实例中停止运行脚本所创建的进程。在结束时，Heat 将通过删除之前创建的额外实例来内向收缩资源。确保 Heat 堆栈已内向收缩。

检查警报的状态，确保它们已过渡到 ok 状态。检查 lab_web_scaling-cpu_alarm_high 警报的历史记录。

    a. 返回到与 lab_WebServer 实例连接的终端，再运行 top 命令来确保名为 yes 的进程不再运行。

    [cloud-user@lab_WebServer ~]$ top
    top - 17:34:39 up 27 min,  1 user,  load average: 0.97, 0.89, 0.54
    ...Output omitted...
      PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
     1447 cloud-u+  20   0  107896    620    528 R 100.0  0.1  11:20.31 yes
    ...Output omitted...

    b. 当 yes 进程结束时，监控正在运行 watch 命令的终端。观察输出中之前创建的额外实例被删除掉。当额外实例被删除后，按 CTRL+C 组合键退出 watch 命令。

    Every 1.0s: openstack server list        Wed Jul 13 23:29:47 2016

    +--------------------------------------+---------------+--------+------------+-------------+-----------------------------------------+
    | ID                                   | Name          | Status | Task State | Power State | Networks                                |
    +--------------------------------------+---------------+--------+------------+-------------+-----------------------------------------+
    ...Output omitted...
    | 366451bb-6f64-4bb1-b91d-385b68f874b7 | lab_WebServer | ACTIVE | deleting   | Running     | lab_privnet=192.168.1.16, 172.25.250.51 |
    | c0daa5c6-3993-4745-bf0e-7afc5a9b22fa | lab_WebServer | ACTIVE | -          | Running     | lab_privnet=192.168.1.20, 172.25.250.55 |
    | c0daa5c6-3993-4745-bf0e-7afc5a9b22fa | lab_WebServer | ACTIVE | -          | Running     | lab_privnet=192.168.1.19, 172.25.250.54 |
    +--------------------------------------+---------------+--------+------------+-------------+-----------------------------------------+

    Every 1.0s: openstack server list        Wed Jul 13 23:29:47 2016

    +--------------------------------------+---------------+--------+------------+-------------+-----------------------------------------+
    | ID                                   | Name          | Status | Task State | Power State | Networks                                |
    +--------------------------------------+---------------+--------+------------+-------------+-----------------------------------------+
    ...Output omitted...
    | c0daa5c6-3993-4745-bf0e-7afc5a9b22fa | lab_WebServer | ACTIVE | -          | Running     | lab_privnet=192.168.1.19, 172.25.250.54 |
    +--------------------------------------+---------------+--------+------------+-------------+-----------------------------------------+

    c. 检查 Ceilometer 警报的状态。

    [student@workstation ~]$ ceilometer alarm-list
    +-----------------------------+---------------------------------------------+-------+----------+---------+------------+---------------------------------+------------------+
    | Alarm ID                    | Name                                        | State | Severity | Enabled | Continuous | Alarm condition                 | Time constraints |
    +-----------------------------+---------------------------------------------+-------+----------+---------+------------+---------------------------------+------------------+
    | 4b173374-9625-44d3f4a96e87  | lab_web_scaling-cpu_alarm_low-tt7qwhtl4h7n  | alarm | low      | True    | True       | cpu_util < 15.0 during 1 x 600s | None             |
    | 7df0d905-9ca0-0cdb9b34138f  | lab_web_scaling-cpu_alarm_high-c5xobmyeyr27 | ok    | low      | True    | True       | cpu_util > 30.0 during 1 x 30s  | None             |
    +-----------------------------+---------------------------------------------+-------+----------+---------+------------+---------------------------------+------------------+
    
    d. 检查 lab_web_scaling-cpu_alarm_high-c5xobmyeyr27 警报的历史记录。使用 ceilometer alarm-history 命令并指定要检查的警报 Alarm ID。

    [student@workstation ~]$ ceilometer alarm-history 7df0d905-46d9-4a73-9ca0-0cdb9b34138f
    +------------------+----------------------------+---------------------------------------------------------------+
    | Type             | Timestamp                  | Detail                                                        |
    +------------------+----------------------------+---------------------------------------------------------------+
    | state transition | 2016-07-13T21:32:56.207000 | state: ok                                                     |
    | state transition | 2016-07-13T21:23:56.271000 | state: alarm                                                  |
    | state transition | 2016-07-13T21:07:56.206000 | state: ok                                                     |
    | creation         | 2016-07-13T21:06:43.932000 | name: lab_web_scaling-cpu_alarm_high-c5xobmyeyr27             |
    |                  |                            | description: Scale-up if the average CPU > 30% for 30 seconds |
    |                  |                            | type: threshold                                               |
    |                  |                            | rule: cpu_util > 30.0 during 1 x 30s                          |
    |                  |                            | severity: low                                                 |
    |                  |                            | time_constraints: None                                        |
    +------------------+----------------------------+---------------------------------------------------------------+

### 总结

在本章中，您可以学到：

*    OpenStack Heat 项目旨在为 OpenStack 提供编配功能。它由各种服务组成，其中包括引擎服务、API 服务器，以及一些面向 Amazon 编配服务 CloudFormation 的传统实施。
*    Heat 模板用于描述基于云的应用的基础架构。模板为人类可读的纯文本文件，可以在版本控制系统 (VCS) 中维护。
*    Heat 编配模板 (HOT) 是一种 YAML 文件，包含三个主要部分：parameters、resources 和 outputs。
*    parameters 部分定义可分配给模板的值的列表。每一参数具有确定的数据类型和可选的默认值，如果使用模板时未指定值则使用该默认值。
*    Heat 编配服务实施自动缩放功能。这使得 OpenStack 能够根据使用情况横向扩展和内向收缩实例。
*    借助 Ceilometer，系统管理员可以决定触发缩放的条件。


