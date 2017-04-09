---
layout: post
title:  "实施 Ansible Tower(407-11)"
categories: Linux
tags: RHCA 407
---

### 描述 Ansible Tower

###### 为何选择 Ansible Tower？

Ansible Tower 是基于 Web 的 UI，它为 IT 自动化带来了企业级解决方案。它为管理部署和监控资源提供了简单易用的控制面板。Ansible Tower 可以完善 Ansible，增加了自动化、可视化管理和监控功能。

Tower 为管理员提供用户访问控制。它协助将 SSH 凭据用于管理，同时又能防止直接访问或转让这些凭据。Ansible 清单可以图形化管理，或者与各种不同的云资源同步，如红帽 OpenStack 平台等。Tower 的控制面板是用户的入口点，提供由 Tower 管理的主机的高级概述。这包括这些主机上近期的活动，以及它们可能引发的问题。

这些自动化功能不仅能够帮助实施持续交付和配置管理，同时也能将其管理整合到一个工具中。借助 Tower 的系统跟踪功能，可以审核与管理由 Tower 管理的主机的历史记录，并且比较不同主机的状态。Tower 的远程命令执行功能允许对主机运行原子操作，利用 Tower 基于角色的访问权限控制引擎控制谁可执行操作，并且日志记录执行的任何操作。

Ansible Tower 包含 Tower 安装向导，此部署工具可以简化 Ansible Tower 的部署。通过向导选择要部署的 Tower 配置后，系统即使用 playbook 来安装 Tower。 

###### Ansible Tower 架构

Ansible Tower 基于一个 Django Web 应用，要求安装有 Ansible。它也依赖于使用 PostgreSQL 的数据库后端。默认安装会将 Ansible Tower 所需的所有组件安装到单一计算机上。

Ansible Tower 可以安装在三种场景中，具体取决于部署的规模以及受管主机的数量：
*    具有内部数据库的单一计算机安装：在这种部署中，Web 界面、REST API 后端和数据库全都在同一台计算机上运行。它使用操作系统提供的 PostgreSQL，并将 Tower 配置成把它用作其数据库。
*    具有外部数据库的单一计算机安装：在这种部署中，Web 界面和 REST API 在一台计算机上运行，而外部的 PostgreSQL 数据库在另一台计算机上运行。此数据库的复制和故障转移并不由 Ansible Tower 配置，而是另行管理。
*    利用多台计算机的主动/被动冗余：在这种部署模式中，一个时间点上 Web 界面和 REST API 在一个主节点上处于活动状态， 但有一个或多个被动辅助节点处于备用状态，并可在主节点故障时接管。外部 PostgreSQL 或 MongoDB 数据库必须在主、辅节点以外的计算机上运行。

###### Ansible Tower 功能

Tower 提供诸多功能，如基于角色的访问权限控制、一键式部署、集中日志记录以及 RESTful API 等。LDAP 或 Active Directory 也可用于管理 Ansible Tower 的用户。Tower 并不支持 Ansible CLI 提供的全部功能，例如 Ansible CLI 是其最佳选项的 playbook 开发支持。下表中包含 Ansible Tower 提供的一些最为相关的功能：

*    支持自动化 playbook 运行、清单同步和项目来源控制，因此这些都可计划在给定的时间执行，以监控受管主机的状态。
*    一个支持通过 Tower 控制面板提供的所有功能的 RESTful API。Tower 也提供支持此 RESTful API 的 CLI。
*    改善了 playbook 执行。通过对用户友好的方式提供 playbook 变量和选择凭据，在 playbook 执行期间进行监控，并且通过许多不同的视图可视化呈现其结果，包括含有每一受管主机的历史记录的图形化清单。
*    基于角色的访问权限控制，允许不同的团队或用户根据自己的要求来管理 Tower 资源。 


### 部署 Ansible Tower

安装 Ansible Tower 时，控制节点应当已装有最新版本的 Ansible。Tower 是基于浏览器的应用，其安装过程中会安装 PostgreSQL、Django和 Apache HTTPD 等多个依赖项。Ansible Tower 要求安装于独立的服务器上，不可与任何其他应用处于同一位置

###### Ansible Tower 安装

安装 Ansible Tower 的第一步是解压缩安装捆绑包，再运行一个配置脚本，它将设置用于安装 Tower 的 playbook 和清单文件。您可以从远程控制节点或本地运行配置步骤和 playbook。

1. 解压缩 tarball

```
# tar xvf ansible-tower-setup-bundle-1.el7.tar.gz
```
  
2. Tower 计算机配置, 提取后的存档包含一个 configure 脚本，它用于根据脚本执行时所传递的参数来设置 playbook 和清单文件。如果相同的设置需要复制到多台计算机，可以多次使用同一 playbook，无需重复运行 configure 脚本。

```
# ./configure
```

configure 脚本具有下列选项：

*    选项 	                        描述
*    -l， --local 	                将 Ansible Tower 安装到具有内置 PostgreSQL 数据库的本地计算机上。
*    --no-secondary-prompt 	        跳过关于添加辅助 Tower 节点的提示。
*    -A， --no-autogenerate 	        禁用为 PostgreSQL 自动生成密码的功能，改为提示用户输入密码。
*    -o FILE， --option-file=FILE 	使用 FILE 作为配置的答案来源。 

3. 运行 Tower 安装, 在创建了 tower_setup_conf.yml playbook 和清单文件后，将运行解压缩后的安装捆绑包中的 setup.sh 脚本来执行 playbook， 并且安装 Ansible Tower。

```
# ./setup.sh
```

*    选项 	    描述
*    -c FILE 	指定存储 Tower 配置的文件。默认文件是安装捆绑包目录中的 tower_setup_conf.yml。
*    -i FILE 	指定要用作主机清单的文件。默认文件是安装捆绑包目录中的 inventory。
*    -p 	        要求 Ansible 在连接远程计算机时提示输入 SSH 密码。
*    -s 	        要求 Ansible 在安装 Tower 时提示输入远程计算机上的 sudo 密码。
*    -u 	        要求 Ansible 在安装 Tower 时提示输入远程计算机上的 su 密码。
*    -e 	        设置额外的 Ansible 变量，供 Tower 在安装期间使用。
*    -b 	        执行数据库备份，而不安装 Tower。
*    -r          BACKUP_FILE 	从指定的文件恢复数据库，而不安装 Tower。 

###### Ansible Tower 控制面板

使用 Web UI 登录 Ansible Tower 后，管理员可以查看一个图表，其中显示了所有近期的作业活动、受管主机的数量，以及指向具有问题的主机列表的指针。控制面板中还显示关于 playbook 中完成的任务执行的实时数据。假定 http://demo.lab.example.com/ 是用于 Ansible Tower Web UI 的 URL。 

Tower 还配置一个最小的 Munin 实例来监控其自身，从而通过图表显示内存消耗和 CPU 用量，以及已排队和活动中作业的数量等信息。Munin 控制面板可通过 http://demo.lab.example.com/munin 访问，它要求输入 admin 用户名以及安装期间提供的密码。

*    Ansible Tower SSL 证书: Ansible Tower 使用自签名证书进行 HTTPS 通信，/etc/tower/awx.cert 是其证书文件，/etc/tower.awx.key 则是其密钥文件。可以根据需要，将这些文件替换为公司自有的 CA 证书，但文件名必须相同。
Ansible Tower REST API
*    Ansible Tower 通过 REST API 利用 URI 路径提供访问其资源的途径，如项目、作业、清单和凭据等。这些 API 可以通过浏览器使用 http://demo.lab.example.com/api/ 链接进行访问。 

```
$ curl -s
      http://demo.lab.example.com/api/v1/ping/ | json_reformat
{
    "instances": {
        "primary": "demo.lab.example.com", "secondaries": [

        ]
    },
    "ha": false,
    "role": "primary",
    "version": "2.4.5"
}
```

可以使用命令 tower-manage 更改 Ansible Tower 的管理密码。该命令需要以 root 用户身份在 Tower 服务器上运行：

```
# tower-manage changepassword admin
```


### 在 Ansible Tower 中配置用户 


### 使用 Ansible Tower 管理主机


### 在 Ansible Tower 中管理作业




### Summary

*    Ansible Tower 提供诸多功能，如基于角色的访问权限控制、一键式部署、集中日志记录以及 RESTful API 等。
*    Ansible Tower 支持自动化运行 playbook，可以计划在指定的时间执行。
*    Tower 还配置一个最小的 Munin 实例来监控其自身，从而通过图表显示内存消耗和 CPU 用量，以及已排队和活动中作业的数量等信息。
*    凭据由 Ansible Tower 用于进行各种身份验证，如对受管主机启动作业、与清单同步，以及从版本控制系统导入项目内容等。
*    用户权限定义分配给用户和团队的特权集合，其使用的是基于角色的访问权限控制 (RBAC)。这些权限提供读取、修改以及管理项目、清单和作业模板的能力。
*    清单可用于定义主机和主机组。
*    可以手动添加 Ansible Tower 主机，或者从平台或云提供商自动发现主机。
*    在 Ansible Tower 中，项目是 Ansible playbook 的逻辑集合。
*    项目目录和文件应归 awx 用户和组所有，Ansible Tower 服务以此用户和组身份运行。
*    作业模板包含 playbook，以及用于运行 Ansible 作业的相关参数集合。

