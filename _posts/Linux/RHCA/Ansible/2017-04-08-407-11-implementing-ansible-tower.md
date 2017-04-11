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

Ansible 有三种类型的用户：

*    普通用户：普通用户对分配给他们的清单和项目具有读取和写入访问权限。
*    企业管理员：企业管理员具有普通用户的所有权限，以及对整个企业及其所有清单和项目的读取与写入访问权限。企业管理员无权访问属于其他企业的内容。企业管理员可以创建和管理自己企业内的用户。
*    超级用户：超级用户具有对整个 Tower 安装的读取和写入权限。超级用户通常为系统管理员，负责管理 Ansible Tower，并且委派职责给企业管理员。

> 在安装时，创建的首个 Ansible Tower 用户将继承 superuser 特权。

在创建后，管理员可以对用户进行进一步管理。此类管理包括编辑用户：

*    用户凭据: 凭据由 Ansible Tower 用于进行各种身份验证任务，如对受管主机启动作业、与清单同步，以及从版本控制系统导入项目内容等
*    用户权限: 见下面说明
*    企业经理（特权）: 可以将用户配置为企业经理，给予他们管理企业清单和项目的能力
*    企业: Ansible Tower 附带有 default 企业，但管理员可以自行定义企业来进行用户的逻辑分组
*    用户团队: 团队是企业的一个分支，关联有用户、项目、凭据和权限。团队为实施基于角色的访问权限控制方案提供了途径，而且也可委派跨企业职责。借助团队，经理可以将相同权限轻松分配给一组用户，而不必为每一用户单独手动设置权限

###### 用户权限

权限定义分配给用户或用户团队的特权集合。它们决定用户读取、修改以及管理项目、清单和作业模板的能力。

可以分配给用户和团队的权限类型有两种，分别具有特定的一组权限：

1. Inventory：此权限类型授予用户管理清单、组和主机的权限；它实施下列级别：

*    read： 允许查看指定清单内的组和主机。
*    write：允许在指定清单内创建、修改以及删除组和主机。此特权级别并不授予修改清单设置的权限，但授予了对这些设置的读取权限。
*    admin：允许修改指定清单的设置。此权限级别同时授予读取和写入权限。
*    execute commands：允许用户对清单执行命令。

2. Job Template：此权限类型授予从指定的项目对指定的清单启动作业的权限；它实施下列级别：

*    create：允许用户或团队创建作业模板。此角色同时要求 run 和 check 权限。
*    run：允许用户或团队运行作业。给予用户或团队这一权限级别也会授予其 check 权限。
*    check：允许用户或团队以空运行模式启动作业。过程中将报告要修改的项目，但不会真正进行更改。

### 使用 Ansible Tower 管理主机

###### Tower 清单

管理员可以直接在 Ansible Tower 的 Web 界面中管理 Ansible 清单。记住清单是主机的集合，Ansible（或 Ansible Tower）可以在这些主机上运行作业。

*    字段 	        描述
*    Name 	        要创建的清单的名称。这是必填字段。
*    Description 	有关清单的更多描述信息。这是可选字段。
*    Organization 	该清单所属的企业。它控制哪些用户有权访问此清单。
*    Variables 	    特定于主机的变量，使用 JSON 或 YAML 格式，它们可以在 Tower 项目中指定。 

###### 使用 Ansible Tower 管理主机组

清单创建好后，即可在其中创建主机和主机组。组可以逻辑方式组织主机，也可用于从云提供商或其他信息来源进行动态主机发现。下表显示了清单组的可用选项：

*    选项 	        描述
*    Name 	        要创建的清单的名称。这是必填字段。
*    Description 	清单的详细说明，这是可选字段。
*    Variables 	    特定于主机的变量，使用 JSON 或 YAML 格式，它们可以在 Tower 项目中指定。
*    Source 	        从中管理主机的来源。来源可以是云或基础架构提供商。 

1. 组可以编辑或复制。组也可移动到企业内的现有组中，成为其子组。Ansible Tower 中的主机组遵循与清单文件内创建的普通主机组一样的原则。 
2. 管理员可以在清单内选择用于主机调配的各种来源。在从提供商处自动发现主机前，必须为团队或用户定义一组凭据。凭据将用于连接提供商端点。
3. 当用户试图删除含有主机的主机组时，Ansible Tower 将警告用户，并且询问其要删除主机还是将它们移到其他组中。

###### 使用 Ansible Tower 管理主机

在创建清单时，可以手动添加主机，或者从平台或云提供商自动发现主机。此类提供商的示例包括 Amazon EC2、Microsoft Azure、OpenStack 和 Rackspace Public Cloud 等

*    字段 	        描述
*    Host Name 	    要管理的主机的完全限定域名或 IP 地址。这是必填字段。
*    Description 	主机的描述。这是可选字段。
*    Enabled 	    此复选框决定在 play 中包含还是排除该主机。
*    Variables 	    特定于主机的变量，使用 JSON 或 YAML 格式，它们可以在 Tower 项目中指定。

主机创建之后，它可以被删除、复制或移动到其他清单中。每一主机旁边的圆形指出针对该主机运行的作业的状态。空白圆形表示没有对该主机运行作业。绿色圆形表示对该主机运行的上一作业已经成功。红色圆形表示对该主机运行的上一作业已经失败。

### 在 Ansible Tower 中管理作业

###### 在 Ansible Tower 中管理作业

Ansible Tower 允许使用作业来执行临时命令或运行 playbook。playbook 的逻辑集合组织到项目中。Tower 允许创建作业模板，它们可预定义如何作为作业从项目运行 playbook，它们也可被重复使用并与其他用户共享。

1. 项目: 在 Ansible Tower 中，项目是 Ansible playbook 的逻辑集合。这些 playbook 或者驻留于 Ansible Tower 实例上，或者存放于 Tower 支持的源代码版本控制系统，如 Git、Subversion 或 Mercurial。Ansible Tower 从中寻找 playbook 的默认项目目录是 /var/lib/awx/projects，但可使用环境变量 $PROJECT_BASE 进行修改。项目目录和文件应归 awx 用户所有，该用户将运行 Tower 服务。

添加新项目: 要创建新项目，可单击 Projects 页面中的 +。如果您要使用 Tower 实例上的本地目录，在 /var/lib/awx/projects 下手动创建了归 awx 用户所有的项目目录后，则 SCM Type 使用 Manual。 

2. 作业模板: 作业模板包含 playbook 及所需的相关参数，供 Tower 执行作业。创建了作业模板后，它可以重复用于未来的作业，并在团队之间共享。

新建作业模板: 要新建作业模板，可单击 Job Templates 页面中的 +。Create Job Templates 页面中包含下列字段：

*    Name: 与作业模板关联的名称。
*    Job Type: 可以是 Run（启动时执行 playbook）、Check（仅检查语法和环境设置，而不执行 playbook）或 Scan（收集系统信息）。
*    Inventory: 执行此作业模板时要使用的清单。
*    Project: 与此作业模板一起使用的项目。
*    Playbook: 与此作业模板一起启动的 playbook。
*    Credential: 与此作业模板一起使用的用户凭据，以进行受管主机身份验证。
*    Forks: 执行该 playbook 期间要使用的并行或同步进程数量。
*    Limit: 用于进一步限制由该 playbook 管理或受其影响的主机列表的主机模式。
*    Job Tags: 由 playbook 标记组成的逗号分隔列表，用于限制要执行 playbook 中的哪些部分。
*    Extra Variables: 向 playbook 传递额外的命令行变量。这是 ansible-playbook 命令的 -e 或 --extra-vars 参数
*    Prompt for Extra Variables: 如果选中此项，则执行作业时提示用户输入额外变量。
*    Enable Survey: 在作业模板用于启动作业时，通过易用的向导设置额外的变量。调查也会先验证用户输入后再使用。调查仅供具有企业版许可证的客户使用。
*    Allow Callbacks: 允许主机使用 Tower API 通过 调配回调启动此作业模板中的作业。这供受管主机用于启动对自身运行的 playbook，从而不必等待用户从 Tower 控制台启动作业。
3. 启动作业: 在为作业创建了包含通常传递给 ansible-playbook 的所有参数的作业额模板后，可用它启动该作业的 一键式部署。

启动作业模板: 要启动作业模板，请单击 Actions 列下与要启动作业模板对应的火箭按钮。作业可能需要额外的信息才能运行。启动时可能会请求下列数据：

*    凭据
*    用于连接远程管理主机的密码或密语，如果设置成了询问。
*    调查问题，如果对作业进行了此配置。
*    额外的变量，如果作业模板要求提供。

4. 计划作业: Ansible Tower 可以将作业计划在特定的时间，或者重复运行。例如，可能需要在星期二早上八点的维护窗口期内应用任何可用的更新。可以使用 Add Schedule 页面设置运行作业模板的计划。

计划作业: 要计划作业，请单击 Actions 列下与需要计划作业模板对应的日历图标。使用 + 图标按钮来添加新计划。Add Schedule 页面中包含下列字段：

*    Name：计划的名称
*    Start Date：计划的开始日期。
*    State Time：计划的开始时间。
*    Local Time Zone：要使用的时区。
*    Repeat frequency：执行所计划的作业的频率。

检查作业状态: 适用于 playbook 运行的作业的 Jobs 页面中显示该次 playbook 运行的所有任务和事件的详细信息。Jobs 页面由多个区域组成，即 Status、Plays、Tasks、Host Events、Events Summary 和 Hosts Summary。

*    Status：Status 区域中显示作业的状态，状态可以是 Running、Pending、Successful 或 Failed。
*    Plays：Plays 区域显示作为该 playbook 一部分运行的 play。对于每个 play，Tower 显示其开始时间、已过时间和名称，以及该 play 是成功还是失败。单击某一个 play，可过滤 Tasks 和 Host Events 区域以仅显示与该 play 相关的任务和主机。
*    Tasks： Tasks 区域显示作为该 playbook 中 play 的一部分而运行的任务。对于每一任务，除了显示其详细信息外，还显示该任务的主机状态的摘要。主机状态可以是 Success、Changed、Failure、Unreachable 或 Skipped。
*    Host Events：Host Events 区域显示受选定 play 和任务影响的主机。
*    Event Summary：Events Summary 区域显示受此 playbook 影响的所有主机的事件摘要。对于每一主机，Events Summary 区域显示其主机名称，以及针对该主机完成的任务数量，信息按照状态排列。
*    Host Summary：Host Summary 区域显示一个图表，总结受该次 playbook 运行影响的所有主机的状态。


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

