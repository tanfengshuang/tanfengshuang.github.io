---
layout: post
title:  "部署 Ansible(407-2)"
categories: Linux
tags: RHCA 407
---

### 安装 Ansible


```
$ yum list installed python
$ sudo yum install -y ansible
```

```
$ ansible web.example.com -i myinventory --list-hosts
  hosts (1):
    web.example.com
    
$ ansible 192.168.2.1 -i myinventory --list-hosts
  hosts (1):
    192.168.2.1
    
$ ansible lab -i myinventory --list-hosts
  hosts (2):
    labhost1.example.com
    labhost2.example.com
    
$ ansible labhost1.example.com,test,192.168.2.2 -i myinventory --list-hosts
  hosts (4):
    labhost1.example.com
    test1.example.com
    test2.example.com
    192.168.2.2

$ ansible all -i myinventory --list-hosts
  hosts (6):
    labhost1.example.com
    test1.example.com
    labhost2.example.com
    test2.example.com
    192.168.2.1
    192.168.2.2

$ ansible '*.example.com' -i myinventory --list-hosts
  hosts (4):
    labhost1.example.com
    test1.example.com
    labhost2.example.com
    test2.example.com
    web.example.com
```

###### 高级主机模式

除了通配符外，Ansible 也允许使用不同的排除与包含逻辑来创建复杂的主机模式。包含的实现是通过使用“:”字符分隔主机模式中的组以表示 OR 逻辑。下例演示了使用主机模式引用属于 lab 组或 datacenter1 组的成员的主机。

```
$ ansible lab:datacenter1 -i myinventory --list-hosts
  hosts (3):
    labhost1.example.com
    labhost2.example.com
    test1.example.com
```

与之相对，在和“&”字符结合使用来分隔主机模式中的组时，“:&”字符表示清单中两个组之间的交叉部分。下例演示了使用主机模式引用同时属于 lab 组和 datacenter1 组的成员的主机。

```
$ ansible 'lab:&datacenter1' -i myinventory --list-hosts
  hosts (1):
    labhost1.example.com
```

排除的实现是通过在主机模式中结合使用“:”字符和“!”字符来指明要排除的主机。下例演示了使用主机模式引用 datacenter 组中定义的除 test2.example.com 以外的所有主机。

```
$ ansible 'datacenter:!test2.example.com' -i myinventory --list-hosts
  hosts (3):
    labhost1.example.com
    test1.example.com
    labhost2.example.com
```

下例演示了使用主机模式引用清单文件中定义的所有主机，但 datacenter1 组的受管主机除外。

```
$ ansible 'all:!datacenter1' -i myinventory --list-hosts
  hosts (3):
    labhost2.example.com
    test2.example.com
    web.example.com
    10.1.1.254
    192.168.2.1
    192.168.2.2
```


### 管理 Ansible 配置文件。

###### 配置 Ansible

可以通过修改 Ansible 配置文件中保管的设置，自定义 Ansible 安装的行为。Ansible 将从控制节点上多个可能的位置之一选择其配置文件。

1. 使用 /etc/ansible/ansible.cfg - 安装之后，ansible 软件包将提供一个基本的配置文件，它位于 /etc/ansible/ansible.cfg。如果找不到其他配置文件，则使用此文件。
2. 使用 ~/.ansible.cfg -Ansible 将在用户的主目录中寻找 ~/.ansible.cfg。如果存在此配置并且当前工作目录中也没有 ansible.cfg，则使用此配置取代 /etc/ansible/ansible.cfg。
3. 使用 ./ansible.cfg - 如果执行 ansible 命令的目录中存在 ansible.cfg 文件，则使用它，而不使用全局文件或用户的个人文件。这样，管理员可以创建一种目录结构，将不同的环境或项目保管在单独的目录中，并且每一目录包含为特定的一组设置而定制的配置文件。
4. 使用 $ANSIBLE_CONFIG - 用户可以通过将不同的配置文件放在不同的目录中，然后从适当目录执行 Ansible 命令，以此利用配置文件；但是，随着配置文件数量的增加，这种方法存在局限并且难以管理。有一个更加灵活的选项，即通过 $ANSIBLE_CONFIG 环境变量定义配置文件的位置。定义了此变量时，Ansible 将使用变量所指定的配置文件，而不用上文中提及的任何配置文件。 

> 推荐的做法是在您要运行 Ansible 命令的目录中创建 ansible.cfg 文件。此目录中也将包含任何供您的 Ansible 项目使用的文件，如 playbook。这是用于 Ansible 配置文件的最常用位置。实践中不常使用 ~/.ansible.cfg 或 /etc/ansible/ansible.cfg

###### 配置文件优先级

配置文件的搜索顺序与上述列表相反。只有在未指定任何其他配置文件时，才会使用全局 /etc/ansible/ansible.cfg；通过 $ANSIBLE_CONFIG 环境变量指定的文件将覆盖所有其他配置文件。搜索顺序中排位第一的文件就是 Ansible 要使用其配置设置的文件。Ansible 将仅使用来自此配置文件的设置。即使存在优先级较低的其他文件，其设置也会被忽略，不会与选定配置文件中的设置结合。

因此，如果用户选择自行创建配置文件来取代全局 /etc/ansible/ansible.cfg 配置文件，他们需要将全局文件中所有需要的配置复制到自己的用户级配置文件中。用户级配置文件中未定义的设置将保持未设定状态，即使已在全局配置文件中进行了设置。

由于 Ansible 配置文件可以放入的位置有多种，因此 Ansible 当前使用哪一个配置文件可能会令人困惑，尤其是控制节点上存在多个文件时。若要清楚确定目前使用的配置文件，可使用 --version 选项执行 ansible 命令。除了显示安装的 Ansible 版本外，它还显示当前活动的配置文件。

```
$ ansible --version
ansible 2.0.1.0
  config file = /etc/ansible/ansible.cfg
...output omitted...
```

显示活动的 Ansible 配置文件还有一种方式，那就是在命令行执行 Ansible 命令时使用 -v 选项。

```
$ ansible servers --list-hosts -v
Using /etc/ansible/ansible.cfg as config file
...output omitted...
```

###### Ansible 配置文件

Ansible 配置文件由若干个部分组成，每一部分含有以键/值对形式定义的设置。部分的标题以方括号括起。在默认的 Ansible 配置文件中，各项设置分组到下列六个部分中。

```
$ grep "^\[" /etc/ansible/ansible.cfg
[defaults]
[privilege_escalation]
[paramiko_connection]
[ssh_connection]
[accelerate]
[selinux]
```

通常修改的 Ansible 配置设置:

*    inventory 	        Ansible 清单文件的位置。
*    remote user 	    用于建立与受管主机的连接的用户帐户。
*    become 	        为受管主机上的操作启用或禁用特权升级。
*    become_method 	    定义受管主机上的特权升级方法。
*    become_user 	    在受管主机上升级特权的用户帐户。
*    become_ask_pass 	定义受管主机上的特权升级是否提示输入密码。


### 运行 Ansible 临时命令

> ansible host-pattern -m module [-a 'module arguments'] [-i inventory]

###### command 模块

如果在执行临时命令时省略了 -m 选项，Ansible 将参考 Ansible 配置文件并使用那里定义的模块。如果未定义模块，Ansible 将使用内部预定义的 command 模块。因此，以下临时命令在技术上是等同的。

1. ansible host-pattern -m command -a 'module arguments'
2. ansible host-pattern -a 'module arguments'


```
$ ansible mymanagedhosts -m command -a /usr/bin/hostname
host1.lab.example.com | SUCCESS | rc=0 >>
host1.lab.example.com
host2.lab.example.com | SUCCESS | rc=0 >>
host2.lab.example.com

$ ansible mymanagedhosts -m command -a /usr/bin/hostname -o
host1.lab.example.com | SUCCESS | rc=0 >> (stdout) host1.lab.example.com
host2.lab.example.com | SUCCESS | rc=0 >> (stdout) host2.lab.example.com
```

###### shell 模块

command 模块允许管理员对受管主机快速执行命令。这些命令不是由受管主机上的 shell 加以处理。因此，它们无法访问 shell 环境变量，也不能执行重定向和传送等 shell 操作。

在命令需要 shell 处理的情形中，管理可以使用 shell 模块。与 command 模块类似，只需在临时命令中将要执行的命令作为参数传递给该模块。Ansible 随后对受管主机远程执行该命令。与 command 模块不同的是，这些命令将通过受管主机上的 shell 进行处理。因此，可以访问 shell 环境变量，也可使用重定向和传送等 shell 操作。

下例演示了 command 模块和 shell 模块的区别。如果要尝试使用这两个模块执行 bash 内建指令 set，则只有使用 shell 模块时才会成功。

```
$ ansible localhost -m command -a set
localhost | FAILED | rc=2 >>
[Errno 2] No such file or directory

$ ansible localhost -m shell -a set
localhost | SUCCESS | rc=0 >>
BASH=/bin/sh
BASHOPTS=cmdhist:extquote:force_fignore:hostcomplete:interact
ive_comments:progcomp:promptvars:sourcepath
BASH_ALIASES=()
...output omitted...
```

###### 临时命令配置

默认情况下，remote_user 参数在 /etc/ansible/ansible.cfg 中已注释掉。如果未定义此参数，临时命令将默认为使用与控制节点上执行该临时命令的用户帐户相同的远程用户帐户连接受管主机。

完成了与受管主机的 SSH 连接后，Ansible 继续使用指定的模块执行该临时操作。一旦完成对受管主机的临时命令，Ansible 在控制节点上显示由远程执行的操作产生的任何标准输出。

与所有其他操作一样，将使用发起该操作的用户的权限来执行远程操作。因为操作是通过远程用户发起的，所以它将受限于该用户的权限限制。 

######### 特权升级

在以远程用户身份成功连接受管主机后，Ansible 可以切换为该主机上的其他用户，然后再执行操作。这通过 Ansible 的特权升级功能来实现。例如，借助 sudo 命令，Ansible 临时命令可以在受管主机上利用 root 特权来执行，即使连入该受管主机的 SSH 连接是由非特权远程用户来验证身份。

启用特权升级的配置设置位于 ansible.cfg 配置文件的 [privilege_escalation] 部分下。默认状态下不启用特权升级。要启用特权升级，become 参数必须没有被注释掉，而且须定义为 True。 

```
#become=True
#become_method=sudo
#become_user=root
#become_ask_pass=False
```

Ansible 命令行选项

*    设置 	        命令行选项
*    inventory 	    -i
*    remote_user 	-u
*    become 	        --become， -b
*    become_method 	--become-method
*    become_user 	--become-user
*    become_ask_pass --ask-become-pass， -K

### 管理动态清单

默认情况下，Ansible 提供基于文本的清单格式来定义要管理的主机。在运营大型基础架构时，系统信息通常通过由监控系统管理的外部目录服务提供，或者通过安装服务器提供，如 Zabbix 或 Cobbler。Ansible 支持通过检索信息的脚本从这些外部数据源动态构建清单。

与之类似，云计算和虚拟化基础架构也有关于它们管理的实例和虚拟机的信息，这些信息可能会在短期内创建并删除。Ansible 动态清单也可用于从红帽 OpenStack 平台和 Amazon Web Services EC2 等通用云和虚拟解决方案实时构建主机清单。

Ansible 中的静态清单可以直接在 /etc/ansible/hosts 文件中指定，或者通过 -i 参数指定。对于提供关于动态清单信息的脚本，这也适用。如果清单文件可以执行，它将被视为动态清单程序，Ansible 则将尝试运行它来生成清单。如果文件不可执行，它将被视为静态清单。

###### 编写动态清单程序

如果使用的目录系统或基础架构没有动态清单脚本，可以编写自定义动态清单程序。它可以使用任何编程语言编写，但传递适当的选项时必须以 JSON 格式返回。

为了让 Ansible 使用脚本从外部清单系统获取主机信息，此脚本必须支持 --list 参数，以类似下方 JSON 散列/字典的形式返回主机组和主机信息。在本例中，webservers 显示为主机组，而 web1.lab.example.com 和 web2.lab.example.com 则是该组中的主机。databases 组包含 db1 和 db2 主机。

```
$ ./inventoryscript --list
{
  "webservers"  : [ "web1.lab.example.com", "web2.lab.example.com" ],
  "databases"  : [ "db1.lab.example.com", "db2.lab.example.com" ]
}
```

至少，每个组应当提供其相关主机的主机名称或 IP 地址列表。该脚本还需要支持 --host hostname，返回空的 JSON 散列/字典，或具有与该主机关联的变量的 JSON 散列/字典，例如：

```
$ ./inventoryscript --host demoserver
{
    "ntpserver" : "ntp.lab.example.com",
    "dnsserver" : "dns.lab.example.com"
}
```

> 创建动态清单的脚本必须可以执行，Ansible 才能使用它。 

###### 处理多个清单

Ansible 支持在同一运行中使用多个清单。如果传递给 -i 参数的值或 /etc/ansible/ansible.cfg 配置中 inventory 参数的值是目录，则该目录中包含的文件将被用于从多个清单（无论静态或动态）检索信息。该目录中的可执行文件将用于检索动态清单，其他文件则被用作静态清单。

存在多个清单文件时，它们将按照字母顺序进行检查。因此，如果一个文件的内容依赖于另一文件的内容，则前者的文件名务必要在字母顺序上排于后者的后面。下例演示了 inventorya 文件中具有一个 datacenter 组，该组包含在 inventoryb 文件中定义的 webservers 组。这将导致错误，因为 inventorya 文件将在 inventoryb 文件之前读取，在处理 inventorya 文件时 webserver 组将还没有定义好。 

> Ansible 可以配置为忽略清单目录中的文件，只要它们以特定的后缀结尾

### Summary

*    任何其上装有 Ansible 且能够访问正确的配置文件和 playbook 以管理远程系统（受管主机）的系统称为 控制节点。
*    受管主机在 清单中定义。主机模式用于引用清单中定义的受管主机。
*    清单可以是静态文件，或者由程序从外部来源（如目录服务或云管理系统）动态生成。
*    清单的位置由使用中的 Ansible 配置文件控制，但最常与 playbook 文件保存在一起。
*    Ansible 以一定的优先级顺序在多个位置上寻找其配置文件。它将使用找到的第一个配置文件，并忽略所有其他文件。
*    Ansible 命令用于对受管主机执行一次性 临时命令。
*    临时命令利用 模块及其参数来决定要执行的操作。
*    要求额外权限的临时命令可利用 Ansible 的 特权升级功能。































