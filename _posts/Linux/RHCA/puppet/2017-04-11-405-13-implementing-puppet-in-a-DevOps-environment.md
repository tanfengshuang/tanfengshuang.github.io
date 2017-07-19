---
layout: post
title:  "Implementing Puppet in a DevOps Environment(405-13)"
categories: Linux
tags: RHCA 405
---

### 调配 Vagrant 计算机

###### DevOps 在企业中

开发和运营部门的员工都承担着利用技术来实现业务目标的职责，不论是通过创建应用来简化内部业务流程和工作流，还是为客户提供新颖的产品和解决方案。尽管他们的职责似乎是一致的，但这两个部门通常在目标和方法学上有着天壤之别。

开发人员喜欢快速工作、不受拘束，因为变化是创新的前提。他们希望有敏捷的工作环境，没有妨碍开发周期快速迭代的障碍，如此才能紧跟不断变化的业务要求或者持续增添的新产品功能，从而能维持市场竞争力。因此，开发人员通常需要 root 帐户访问权限来避免遭遇权限问题，也需要完整的操作系统安装，从而不会遇到缺少软件依赖项的情形。

在另一方面，运营部门注重于提供稳定而可靠的基础架构，以运行任务关键型应用。他们还必须确保遵循安全策略和监管合规。由于变化是稳定性的死敌，因此他们会尽力打造一贯而静态的环境。为了维系基础架构的完整性，他们在环境中允许出现的变化的频率和幅度上特别保守。

当开发人员和系统管理员在部署新版软件到生产环境而相遇时，这两个群体之间理念和实践上的不一致会非常严重。由于不受限制开发环境和严格受控生产环境之间的差别，生产部署后经常出现高压补救电话会议和冗长事后剖析会议也就不足为奇了。

近年来，随着企业努力解决开发和运营团队之间的冲突，DevOps 方法在企业中的采用率日渐上升。DevOps 名称来源于这样的一个核心原则，即软件开发和运营性能可以通过改善软件开发人员和 IT 运营专业人士之间的沟通、整合和协作来得到改进和加速。

###### 基础架构即代码

DevOps 非常注重借助自动化、程序化的步骤构建和维护基本组件。一个关键的 DevOps 概念是基础架构即代码。对于目前通过手动执行管理命令和编辑配置文件的许多系统管理员而言，这一概念是一种重要的突破性范式转变。通过将基础架构作为代码进行设计、实施和管理，配置能够在整个环境中可预测且一致地部署和重复。

Puppet 软件在最近几年已发展为管理基础架构代码的流行工具。Puppet 使得软件安装和服务器配置等任务可以实现自动化。这种自动化可以除却因为手动管理而带来的人为错误，因而能大幅提升企业的运营效率，同时还能加强成功成果的可预测性。

Puppet 的宿主/客户端架构提供了集中配置更改。此设计可以有效实施基础架构更改单一入口点，从而加强对基础架构代码的掌控。

Puppet 的声明性质也使它具备自我记录能力，帮助管理员轻松获得基础架构配置的清晰面貌。除了实施配置端状态外，Puppet 也使得管理员能够审核系统配置，并且探测它们何时偏离预期状态。

虽然 Puppet 等配置管理工具能够简化和改善基础架构管理，它们也会带来应当如何管理这种新的基础架构代码的困境。为克服这一难题，管理员最好遵循其对应的开发指南。根据开发的最佳实践，管理员应当使用 Git 等版本控制系统来管理其基础架构代码。

版本控制使得管理员能够为其基础架构代码的不同阶段（如开发、质保和生产）实施生命周期。通过版本控制工具管理基础架构代码，管理员可以在非关键的开发与质保环境中测试其基础架构代码更改，最大程度减少在生产环境中实施部署时的意外和干扰。

###### Vagrant

在为生产部署测试代码时，只有在与生产环境完全一致的开发环境中执行测试，结果才具有相关性和有效性。这不仅适用于软件开发，也适用于基础架构代码更改。借助虚拟化技术，可以轻松而经济地设立一台计算机，用于在生产部署之前测试代码。不过，真正的挑战在于如何在虚拟机上构建一个完全是生产环境翻版的开发环境。

Vagrant 是客服这一难题的实用工具。作为代码管理基础架构的做法可以确保开发和生产环境之间存在一致的系统与软件配置，Vagrant 则进一步简化创建和管理开发所需的一致虚拟化环境的流程。

通过遵循基础架构即代码方法，Vagrant 允许在一个纯文本配置文件中指定整个流程，自动化创建虚拟机、其硬件配置、软件安装、系统配置和开发源代码检索。借助 Vagrant，部署开发环境可以像从版本控制中签出项目并在命令行中执行 vagrant up 那般简单。

Vagrant 将虚拟化环境作为代码进行管理还有一个好处，它非常简单易用，这不仅体现在创建虚拟化开发环境，也体现于不同团队成员共享完全相同的环境。Vagrant 使得最终用户从设置和共享相同虚拟化开发环境的复杂工作中脱身，因此不仅对需要测试基础架构代码更改的管理员，而且对需要测试软件发布的开发人员，它都是一款理想的工具。

######### Vagrant 组件

Vagrant 软件由下列主要组件组成：

Box：box 是包含虚拟机镜像的 tar 存档。box 文件充当 Vagrant 虚拟化环境的基础，用于创建虚拟机实例。若要获得更大的灵活性，镜像中应当仅包含基础操作系统安装。这样，该镜像可在创建各种不同虚拟机时用作起点，无论其应用的具体要求为何。在虚拟机创建之后，可以通过自动化配置来达到这些应用特定的要求。

提供程序：提供程序使得 Vagrant 能够与部署有 Vagrant box 镜像的底层平台交互。Vagrant 随附了适用于 Oracle 的 VirtualBox 的提供程序。目前，也可获得面向 VMware、Hyper-V, 和 KVM 等其他虚拟化平台的备选提供程序。

Vagrantfile：Vagrantfile 是一种纯文本文件，内含用于创建 Vagrant 虚拟化环境的指令。这些指令使用 Ruby 语法编写。此文件的内容可用于规定虚拟机将要构建和配置的方式。

> 注意: 管理员可以自行创建 Vagrant box 镜像，或者利用 HashiCorp 的 Atlas box 目录 (http://www.vagrantcloud.com) 上公开提供的现有镜像。本课程将使用预制的 Red Hat Enterprise Linux box 镜像，因为创建 Vagrant box 镜像不在本课程范畴内。

> 注意: Box 文件及其包含的镜像特定于各提供程序。例如，为 VirtualBox 提供程序创建的 box 文件不兼容 VMware 提供程序。使用多种提供程序的企业将需要具有特定于各个提供程序的单独 box 文件。 


###### 配置 Vagrant 环境

Vagrant 环境自然要相互独立运行。若要创建新的 Vagrant 环境，首先要为新环境创建项目目录。在此项目目录内，创建含有用于部署新 Vagrant 计算机的指令的 Vagrantfile。下例演示了开发人员如何在其工作站上启动 Vagrant 项目。

```
[root@host ~]# mkdir -p /root/vagrant/project
[root@host ~]# cd /root/vagrant/project
[root@host project]# vi Vagrantfile
```

######### 创建基本的 Vagrantfile

以下几行来自 Vagrantfile 文件，向 Vagrant 提供用于创建基本虚拟机的指令。config.vm.box 方法调用指定该虚拟机克隆自名为 rhel7.1 的 box 镜像，该镜像可从 config.vm.box_url 方法调用所指定的位置获取。config.vm.hostname 方法调用指示 Vagrant 在创建虚拟机时将它命名为 sandbox.example.com。

```
Vagrant.configure(2) do |config|
  config.vm.box = "rhel7.1"
  config.vm.box_url = "http://content.example.com/puppet3.6/x86_64/dvd/vagrant/rhel-server-libvirt-7.1-1.x86_64.box"
  config.vm.hostname = "sandbox.example.com"
end
```

######### 管理 Vagrant 计算机

创建了 Vagrantfile 文件后，可以通过从项目的根目录执行 vagrant up 命令来实例化 Vagrant 计算机。

```
[root@host project]# vagrant up
Bringing machine 'default' up with 'libvirt' provider...
==> default: Box 'rhel7.1' could not be found. Attempting to find and install...
    default: Box Provider: libvirt
... Output omitted ...
==> default: Waiting for SSH to become available...
    default:
    default: Vagrant insecure key detected. Vagrant will automatically replace
    default: this with a newly generated keypair for better security.
    default:
    default: Inserting generated public key within guest...
    default: Removing insecure key from the guest if it's present...
    default: Key inserted! Disconnecting and reconnecting using new SSH key...
... Output omitted ...
==> default: Setting hostname...
==> default: Configuring and enabling network interfaces...
==> default: Rsyncing folder: /root/vagrant/project/ => /home/vagrant/sync
```

vagrant up 的输出显示 Vagrant 自动化的所有配置和管理任务，用户无须执行这些任务。它们包含：检索 box 镜像、创建虚拟机、设置主机名称、配置网络接口，以及从主机复制文件到 Vagrant 计算机。如果 Vagrantfile 没有定义静态 IP 寻址，则虚拟化提供程序会为虚拟机动态分配 IP 地址。

Vagrant 计算机启动后，可以使用 vagrant ssh 命令进行访问。在 Vagrant 计算机部署期间，主机上生成一个 SSH 密钥，而后安装到 Vagrant 计算机上 vagrant 用户的 ~/.ssh 目录中。vagrant ssh 命令使用此密钥，对以 vagrant 用户身份向 Vagrant 计算机发起的 SSH 会话进行身份验证。

```
[root@host project]# vagrant ssh
Last login: Wed Oct 21 14:02:44 2015 from 192.168.121.1

[vagrant@sandbox ~]$ id
uid=1000(vagrant) gid=1000(vagrant) groups=1000(vagrant),1001(docker) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

> 注意: Vagrant 软件要求 Vagrant 计算机上有 vagrant 用户帐户可用于 SSH 访问。因此，标准的做法是在创建基础虚拟机镜像期间，创建 vagrant 用户帐户并为其配置 sudo 特权。虽然 Vagrant 使用基于 SSH 密钥的身份验证，但仍然要按照标准做法将 vagrant 用户帐户的密码设置为“vagrant”。

Vagrant 还提供一项实用功能，称为同步文件夹。默认情况下，Vagrant 使用同步文件夹功能，将项目目录的内容复制到 Vagrant 计算机上可由 vagrant 用户访问的目录 (~/sync/)。

```
[root@host project]# ls -l
total 4
-rw-r--r--. 1 root root 227 Oct 21 13:50 Vagrantfile

[vagrant@sandbox project]$ ls -l /home/vagrant/sync
total 4
-rw-r--r--. 1 vagrant vagrant 227 Oct 21 13:50 Vagrantfile
```

为了能对 Vagrant 计算机执行特权命令，构建 box 镜像时应将 sudo 特权授予 vagrant 用户。在配置更加高级的 Vagrant 计算机部署时，此 sudo 特权特别有用，下文中将对此进行探讨。

```
[vagrant@sandbox project]$ sudo -l
Matching Defaults entries for vagrant on this host:
    !visiblepw, always_set_home, env_reset, env_keep="COLORS DISPLAY HOSTNAME
    HISTSIZE INPUTRC KDEDIR LS_COLORS", env_keep+="MAIL PS1 PS2 QTDIR USERNAME
    LANG LC_ADDRESS LC_CTYPE", env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT
    LC_MESSAGES", env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE",
    env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY",
    secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User vagrant may run the following commands on this host:
    (ALL) NOPASSWD: ALL
```

不再需要某一 Vagrant 计算机时，可执行 vagrant halt 命令来关闭该 Vagrant 计算机。另一选择是执行 vagrant destroy 命令，它将停止运行中的计算机，同时清理掉部署该计算机时创建的所有资源。

```
[vagrant@sandbox project]$ exit
logout
Connection to 192.168.121.85 closed.

[root@host project]# vagrant destroy
==> default: Removing domain...
```

######### Vagrant 调配器

上例演示了使用 Vagrant 快速部署简单的虚拟机。尽管这非常便捷，但 Vagrant 在为 DevOps 环境创建开发系统方面的真正实力在于其调配功能。

如前文所述，box 镜像中应当仅包含基础操作系统，以便该镜像可以用作自定义不同虚拟机的基础。Vagrant 调配功能可以自动化在基础操作系统上叠加自定义所需的软件安装与配置更改。

调配是使用 Vagrant 提供的一个或多个调配器来执行的。调配器在 Vagrantfile 文件中通过使用 config.vm.provision 方法调用来启用。下例演示如何将 Vagrant 的 shell 调配器用于自动化 Vagrant 计算机在使用基础操作系统部署后进行的 yum 存储库配置和 Puppet 软件安装。

```
Vagrant.configure(2) do |config|

... Configuration omitted ...

  config.vm.provision "shell", inline: <<-SHELL
    sudo cp /home/vagrant/sync/etc/yum.repos.d/* /etc/yum.repos.d
    sudo yum install -y puppet
  SHELL
end
```


### 练习：调配 Vagrant 计算机

1. 您将使用 Git 来管理 Vagrant 配置更改。在 workstation 上，以 student 身份创建一个中央存储库。

``
[student@workstation ~]$ git init --bare --shared=true /var/git/vagrant.git
Initialized empty shared Git repository in /var/git/vagrant.git/
```

2. 以 root 用户身份登录 serverb，再安装和配置 Git。下载 config-git

```
[root@serverb ~]# curl -O http://materials.example.com/config-git
... Output omitted ...
[root@serverb ~]# chmod 755 config-git
[root@serverb ~]# ./config-git
Install the Git software ......................
... Output omitted ...
Define Git configurations .....................
Deploying SSH keys for authentication .........
The authenticity of host 'workstation (172.25.250.254)' can't be established.
ECDSA key fingerprint is c3:8d:dd:ce:4f:df:f6:fc:40:9b:ee:de:14:ae:7f:30.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
student@workstation's password: student

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'student@workstation'"
and check to make sure that only the key(s) you wanted were added.
```

config-git 将 SSH 密钥身份验证配置到 workstation 上的 student 帐户。使用 ssh 确认密钥已配置正确。

```
[root@serverb ~]# ssh student@workstation
Last login: Wed Nov  4 22:36:03 2015 from 172.25.252.254
```

3. 创建供 Vagrant 工作使用的工作目录 /root/vagrant。从中央 vagrant Git 存储库克隆它。另外，在 Vagrant 工作目录中创建子目录 webapp。

```
[root@serverb ~]# git clone ssh://student@workstation/var/git/vagrant.git
Cloning into 'vagrant'...
warning: You appear to have cloned an empty repository.
[root@serverb ~]# mkdir vagrant/webapp
```

4. 在 /root/vagrant/webapp 目录中，通过从 http://materials.example.com/vagrant/Vagrantfile 下载来创建 Vagrant 配置文件 Vagrantfile。此文件具有以下作用：利用上表中提供的 box 镜像创建 Vagrant 计算机，将计算机 box 镜像命名为 rhel7.1，并将计算机配置为使用主机名称 dev.lab.example.com。

```
[root@serverb ~]# cd vagrant/webapp
[root@serverb webapp]# curl -O http://materials.example.com/vagrant/Vagrantfile
```

5. 将 webapp 子目录和 Vagrantfile 置于 Git 控制之下。将它们签入到本地存储库。

```
[root@serverb webapp]# git add .
[root@serverb webapp]# git status
# On branch master
#
# Initial commit
#
# Changes to be committed:
#   (use "git rm --cached <file>..." to unstage)
#
#       new file:   Vagrantfile
#
[root@serverb webapp]# git commit -m 'Initial version.'
... Output omitted ...
```

6. 利用新配置测试虚拟机部署。

```
[root@serverb webapp]# vagrant up
Bringing machine 'default' up with 'libvirt' provider...
==> default: Creating image (snapshot of base box volume).
==> default: Creating domain with the following settings...
... Output omitted ...
```

7. 验证 puppet 软件包尚未安装到默认的 Red Hat Enterprise Linux 7.1 box 中。另外，也请检查 box 中提供了哪些 Yum 存储库。

```
[root@serverb webapp]# vagrant ssh
[vagrant@dev ~]$ rpm -q puppet
package puppet is not installed
[vagrant@dev ~]$ yum repolist
Loaded plugins: product-id, subscription-manager
repolist: 0
```

8. 由于 puppet 软件包尚未安装，而且没有可用的 Yum 存储库，因此要配置 Vagrant，以便在部署 Vagrant 计算机时创建必要的 Yum 存储库配置文件。

    a. 退出 Vagrant 计算机，并将它关闭。请注意，您的 IP 地址可能与示例中所示的不同。

```
    [vagrant@dev ~]$ exit
    logout
    Connection to 192.168.121.238 closed.

    [root@serverb webapp]# vagrant destroy
    ==> default: Removing domain...
```

    b. 在 Vagrant 工作目录中创建用于保存 Yum 存储库配置文件的目录，然后更改到该目录。

```
    [root@serverb webapp]# mkdir -p /root/vagrant/webapp/etc/yum.repos.d
    [root@serverb webapp]# cd /root/vagrant/webapp/etc/yum.repos.d
```

    c. 通过从 /etc/yum.repos.d/rhel_dvd.repo 复制，创建名为 rhel_dvd.repo 的 Yum 存储库配置文件，使其包含以下内容：

```
    [rhel_dvd]
    gpgcheck = 0
    enabled = 1
    baseurl = http://content.example.com/rhel7.1/x86_64/dvd
    name = Remote classroom copy of dvd
```

    d. 通过从 /etc/yum.repos.d/rhsat_dvd.repo 复制，创建名为 rhsat_dvd.repo 的 Yum 存储库配置文件，使其包含以下内容：

```
    [rhsat_dvd]
    gpgcheck = 0
    enabled = 1
    baseurl = http://content.example.com/rhsat6.1/x86_64/dvd
    name = Remote classroom copy of RH Satellite dvd
```

    e. 改回到 webapp 目录。

```
    [root@serverb webapp]# cd /root/vagrant/webapp
```

    f. 修改 Vagrant 配置文件，从而在调配期间安装 Puppet。主机上的 Yum 存储库配置文件必须复制到 Vagrant 计算机上的 /etc/yum.repos.d 中。配置了 Yum 后，便可安装 puppet 软件包。

    在 Vagrantfile 文件的 Vagrant.configure 代码块内添加下面这几行。它们应当直接插到 “# Define shell provisioner” 注释的下方。

```
    config.vm.provision "shell", inline: <<-SHELL
      sudo cp /home/vagrant/sync/etc/yum.repos.d/* /etc/yum.repos.d
      sudo yum -y install puppet
    SHELL
```

10. 利用新修改的配置调配 Vagrant 计算机。

```
[root@serverb webapp]# vagrant up
Bringing machine 'default' up with 'libvirt' provider...
==> default: Creating image (snapshot of base box volume).
==> default: Creating domain with the following settings...
... Output omitted ...
==> default:   rubygems.noarch 0:2.0.14-24.el7
==> default: 
==> default: Complete!
```

11. 验证 Yum 存储库现已可用，并且 Vagrant 计算机上已安装了 puppet 软件包。

```
[root@serverb webapp]# vagrant ssh

[vagrant@dev ~]$ yum repolist
Loaded plugins: product-id, subscription-manager
repo id      repo name                                   status
rhel_dvd     Remote classroom copy of dvd                4,371
rhsat_dvd    Remote classroom copy of RH Satellite dvd     408
repolist: 4,779
[vagrant@dev ~]$ rpm -q puppet
puppet-3.6.2-4.el7sat.noarch
```

12. 退出 Vagrant 计算机，并将它关闭。

```
[vagrant@dev ~]$ exit
[root@serverb webapp]# vagrant destroy
```

13. 在中央 Git 存储库中保存您的更改及描述性日志消息。

    a. 列出 webapp 目录中的所有文件。

```
    [root@serverb webapp]# ls -a
    .  ..  etc  .vagrant  Vagrantfile
```

    b. .vagrant 子目录中存储 Vagrant 的元数据。通过创建一个包含目录名称的 .gitignore 文件，防止此目录受到 Git 的控制。

```
    [root@serverb webapp]# echo .vagrant > .gitignore
```

    c. 将当前目录中的文件添加到暂存区域，然后将它们与有意义的日志消息一起提交到本地存储库。

```
    [root@serverb webapp]# git add .
    [root@serverb webapp]# git commit -m 'Configured Yum and installed Puppet.'
    ... Output omitted ...
```

    d. 将所有更改推送到上游 Git 存储库。

```
    [root@serverb webapp]# git push
    ... Output omitted ...
```


### 在 DevOps 环境中部署 Vagrant

###### 将 Vagrant 与 Puppet 集成

上一节中演示了部署配有基础操作系统的 Vagrant 计算机。演示了 Vagrant 的调配功能和 shell 调配器，用于在从基础操作系统镜像创建了 Vagrant 计算机后安装 Puppet 等软件。在 Vagrant 计算机上安装 Puppet 后，可以将该计算机集成到组织的 Puppet 基础架构中。

Vagrant 提供 puppet_server 调配器，可以利用组织的 Puppet 基础架构协助 Vagrant 计算机安装软件和配置系统。此调配器可以自动执行 Puppet 代理的配置、与组织 Puppet 宿主的连接，以及对适用 Puppet 模块和清单的检索。

在 DevOps 环境中使用 Vagrant 的 puppet_server 调配器时，完全有可能创建配置与生产系统一致的 Vagrant 计算机。在这两种情形中，系统都是使用基础操作系统安装程序作种子，然后在从 Puppet 宿主应用模块和清单时进行配置。只要将应用到生产系统的相同模块和清单应用到 Vagrant 计算机，这两个系统就能配置成几乎完全一样。而后，Vagrant 计算机将能用作实用的有效开发环境，在生产部署前测试软件发行或基础架构代码更改。 

######### 使用同步文件夹

Vagrant 提供多种其他功能来协助调配过程。如前文所述，一种实用功能是同步文件夹。默认情况下，Vagrant 在 Vagrant 计算机实例化期间使用该功能将主机上项目目录的内容复制到该计算机的某一目录。

也可从主机复制其他内容到 Vagrant 计算机，只需在 Vagrantfile 配置文件中配置额外同步文件夹便可。Vagrant 提供各种保持这些文件夹同步的机制。下例演示了如何向 Vagrantfile 添加使用 rsync 进行内容同步的同步文件夹。Vagrant 的默认同步文件夹类型是 VirtualBox。此文件夹类型提供主机和 Vagrant 计算机文件夹内容的双向同步，主机或 Vagrant 计算机上文件夹的内容出现变更时，内容将持续同步。

不过，VirtualBox 同步文件夹仅可在使用 VirtualBox 提供程序时使用。使用其他提供程序时，将需要使用不同的同步文件夹类型。请务必参考相关文档来了解各种同步文件夹类型的工作方式，因为它们可能在行为上相异。

本课程中不使用 VirtualBox 提供程序，所以将改为使用 rsync 同步文件夹类型。与 VirtualBox 同步文件夹不同，rsync 同步文件夹是单向的，仅从主机复制文件到 Vagrant 计算机，而不会反向复制。另外，rsync 机制也不会持续复制文件夹内容更改。在执行 vagrant up 时，它将仅从主机复制文件夹内容到 Vagrant 计算机。如果在这一初始同步后需要重新复制文件夹内容，可按照下例中所示执行 vagrant rsync 命令。

```
[root@host project]# vagrant rsync
==> default: Rsyncing folder: /root/vagrant/project/ => /home/vagrant/sync
```

下例演示了如何修改 Vagrantfile 配置文件，以添加利用 rsync 机制复制文件夹内容的同步文件夹。如 owner 和 group 选项所指定，目标文件的所有者和组所有权都会设为 puppet。此外，由于 rsync 操作是使用 vagrant 帐户在 Vagrant 计算机上执行的，权限不足会导致文件复制失败。选项 rsync__rsync_path 用于指定在 Vagrant 计算机上应当使用 sudo rsync 命令，而不使用默认的 rsync 命令。

```
Vagrant.configure(2) do |config|

... Configuration omitted ...

  config.vm.synced_folder "puppet/ssl", "/var/lib/puppet/ssl", type: "rsync", owner: "puppet", group: "puppet", rsync__rsync_path: "sudo rsync"
end
```

######### 配置 Vagrant 以进行 Puppet 调配

下列规程说明了如何配置 Vagrant，以使用 puppet_server 调配器自动把软件和配置部署到 Vagrant 计算机。下例演示了如何在 sandbox.example.com 上部署 Vagrant 计算机，并将它配置为与 puppetmaster.example.com 上运行的 Puppet 宿主通信。 

1. 修改 Vagrant 配置文件，使得 Vagrant 计算机在启动时与 puppetmaster.example.com 上运行的 Puppet 宿主通信。

    a. 在 Vagrantfile 文件的 Vagrant.configure 代码块内添加下面这几行。puppet.options 参数指定在首次连接 Puppet 宿主时所进行的证书签名需要的 Puppet 客户端选项。它还为系统指定从 Puppet 宿主检索和应用其配置。

```
    Vagrant.configure(2) do |config|
     
      ... Configuration omitted ...
     
        config.vm.provision "puppet_server" do |puppet|
          puppet.puppet_server = 'puppetmaster.example.com'
          puppet.options = '--onetime --verbose --waitforcert=10 --no-usecacheonfailure --no-daemonize'
        end
      end
```

    b. 启动 Vagrant 计算机。它应当成功注册到 Puppet 宿主，并检索其配置。应用的配置将启用并启动 sandbox.example.com 上的 httpd 服务。

```
    [root@sandbox project]# vagrant up
    Bringing machine 'default' up with 'libvirt' provider...
    ==> default: Creating image (snapshot of base box volume).
    ==> default: Creating domain with the following settings..
    ... Output omitted ...
    ==> default: Running provisioner: puppet_server...
    ==> default: Running Puppet agent...
    ==> default: Info: Creating a new SSL key for sandbox.example.com
    ==> default: Info: Caching certificate for ca
    ==> default: Info: csr_attributes file loading from /etc/puppet/csr_attributes.yaml
    ==> default: Info: Creating a new SSL certificate request for sandbox.example.com
    ==> default: Info: Certificate Request fingerprint (SHA256): 8B:67:38:82:F7:97:6B:25
    ==> default: Info: Caching certificate for sandbox.example.com
    ==> default: Info: Caching certificate_revocation_list for ca
    ==> default: Info: Caching certificate for ca
    ==> default: Info: Retrieving plugin
    ==> default: Info: Caching catalog for sandbox.example.com
    ==> default: Info: Applying configuration version '1444926117'
    ==> default: Notice: /Stage[main]/Main/Node[sandbox.example.com]/Package[httpd]/ensure:
     created
    ==> default: Notice: /Stage[main]/Main/Node[sandbox.example.com]/Service[httpd]/ensure:
     ensure changed 'stopped' to 'running'
    ==> default: Info: /Stage[main]/Main/Node[sandbox.example.com]/Service[httpd]:
     Unscheduling refresh on Service[httpd]
    ==> default: Info: Creating state file /var/lib/puppet/state/state.yaml
    ==> default: Notice: Finished catalog run in 28.26 seconds
```

2. 在第一次运行 Puppet 客户端期间，它会将新的密钥和证书缓存在 Vagrant 计算机的 /var/lib/puppet/ssl 目录下。这些文件需要存档，并供未来的 Vagrant 计算机实例使用。如果缺少这些文件，未来为此项目生成的 Vagrant 计算机实例将尝试生成新的密钥和 Puppet 客户端证书，这将导致与 Puppet 宿主通信失败。

将证书存档到项目目录中的 puppet/ssl 子目录，再关闭 Vagrant 计算机。

    a. 在主机上创建一个目录，以存档 Puppet 证书。

```
    [root@sandbox project]# mkdir -p puppet/ssl
```

    b. 显示 Vagrant 计算机的网络和 SSH 信息。这将提供该计算机的 IP 地址。

```
    [root@sandbox project]# vagrant ssh-config
    Host default
      HostName 192.168.121.88
      User vagrant
      Port 22
      UserKnownHostsFile /dev/null
      StrictHostKeyChecking no
      PasswordAuthentication no
      IdentityFile /root/vagrant/project/.vagrant/machines/default/libvirt/private_key
      IdentitiesOnly yes
      LogLevel FATAL
```

    c. 将 Vagrant 计算机上 /var/puppet/ssl 目录的内容复制到主机上的存档目录。务必要将命令中的 Vagrant 计算机 IP 地址替换为 vagrant ssh-config 命令所显示的地址。rsync 命令的 --rsync-path 选项将允许使用 sudo 特权在 Vagrant 计算机上执行文件复制。rsync 命令的 -e 选项指定通过利用基于密钥的身份验证所建立的 SSH 连接来执行该命令。

```
    [root@sandbox project]# rsync -avz --rsync-path='sudo rsync' -e 'ssh -i /root/vagrant/project/.vagrant/machines/default/libvirt/private_key' vagrant@192.168.121.88:/var/lib/puppet/ssl/ puppet/ssl/
    The authenticity of host '192.168.121.88 (192.168.121.88)' can't be established.
    ECDSA key fingerprint is 51:f7:72:0d:2d:e2:13:1b:84:19:61:41:6a:e5:d4:ac.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added '192.168.121.88' (ECDSA) to the list of known hosts.
    receiving incremental file list
    ./
    crl.pem
    certificate_requests/
    certificate_requests/sandbox.example.com.pem
    certs/
    certs/ca.pem
    certs/sandbox.example.com.pem
    private/
    private_keys/
    private_keys/sandbox.example.com.pem
    public_keys/
    public_keys/sandbox.example.com.pem

    sent 148 bytes  received 8723 bytes  2534.57 bytes/sec
    total size is 10624  speedup is 1.20
```

    d. 关闭 Vagrant 计算机。

```
    [root@sandbox project]# vagrant destroy
```

3. 修改 Vagrant 配置文件，使 /root/vagrant/project/puppet/ssl 目录的内容在 Vagrant 计算机启动时同步到该计算机的 /var/lib/puppet/ssl。验证 Puppet 代理能够与 Puppet 宿主通信。

    a. 在 Vagrantfile 文件的 Vagrant.configure 代码块内添加下面这一行。这会利用 Vagrant 的同步文件夹功能从主机复制文件到 Vagrant 计算机。

```
    Vagrant.configure(2) do |config|
       ... Configuration omitted ...
     
     config.vm.synced_folder "puppet/ssl", "/var/lib/puppet/ssl", type: "rsync",
     owner: "puppet", group: "puppet", rsync__rsync_path: "sudo rsync"
```

    b. 启动新的 Vagrant 计算机，以验证新配置。新计算机应当利用从初次运行 Puppet 客户端时存档的密钥和证书与 Puppet 宿主成功通信。

```
    [root@sandbox project]# vagrant up
    Bringing machine 'default' up with 'libvirt' provider...
    ==> default: Creating image (snapshot of base box volume).
    ==> default: Creating domain with the following settings...
    ... Output omitted ...
    ==> default: Rsyncing folder: /root/vagrant/webapp/puppet/ssl/ => /var/lib/puppet/ssl
    ... Output omitted ...
    ==> default: Running provisioner: puppet_server...
    ==> default: Running Puppet agent...
    ==> default: Info: Retrieving plugin
    ==> default: Info: Caching catalog for sandbox.example.com
    ==> default: Info: Applying configuration version '1444926117'
    ==> default: Notice: /Stage[main]/Main/Node[sandbox.example.com]/Package[httpd]/ensure: created
    ==> default: Notice: /Stage[main]/Main/Node[sandbox.example.com]/Service[httpd]/ensure: ensure changed 'stopped' to 'running'
    ==> default: Info: /Stage[main]/Main/Node[sandbox.example.com]/Service[httpd]: Unscheduling refresh on Service[httpd]
    ==> default: Info: Creating state file /var/lib/puppet/state/state.yaml
    ==> default: Notice: Finished catalog run in 28.20 seconds
```

###### 创建 Vagrant 开发环境

通过组织的 Puppet 基础架构集成和配置了 Vagrant 计算机后，该计算机上应当装有其预期用途需要的所有软件。例如，我们可能希望 Puppet 在 Vagrant 计算机上安装 Apache 并启动 Web 服务，以便用于测试 Web 应用。

除了配置管理外，也可通过将应用源代码融入到 Vagrant 项目中，来进一步自动化开发环境的创建。使应用源代码成为 Vagrant 项目目录的一部分，即可将它融入进来。然后，可以在启动时使用 Vagrant 的同步文件夹功能，从主机系统复制源代码到 Vagrant 计算机。

下例演示了如何将 Web 应用的 DocumentRoot 文件夹的内容融入到 Vagrant 项目目录下的 www 子目录。在启动时，此文件的内容将复制到 Vagrant 计算机的 /var/www/ 目录。

```
Vagrant.configure(2) do |config|

... Configuration omitted ...

  config.vm.synced_folder "www", "/var/www/html", type: "rsync",  owner: "apache", group: "apache", rsync__rsync_path: "sudo rsync"
end
```

######### 使用转发端口

如果开发中的应用具有网络服务，Vagrant 会提供一项网络配置功能，称为转发端口；该功能可将主机系统上的网络端口映射到 Vagrant 计算机上的端口。下例演示了如何修改 Vagrantfile 文件，使 Vagrant 将指向主机系统上端口 8000 的流量转发到 Vagrant 计算机上的端口 80。

```
Vagrant.configure(2) do |config|

... Configuration omitted ...

  config.vm.network :forwarded_port, guest: 80, host: 8000
end
```

> 注意: 对 Vagrantfile 所做的配置更改不会对运行中的现有 Vagrant 计算机有任何影响。若要使更改生效，可通过 vagrant halt 或 vagrant destroy 停止计算机，然后通过 vagrant up 重新启动。另一种方法是执行 vagrant reload 命令。此命令执行与 vagrant halt 加 vagrant up 相同的操作。在发出 vagrant reload 时，不会重新执行使用 Vagrantfile 文件定义的调配器。

######### 创建可重复使用的 Vagrant 开发环境

一旦为集成到组织的 Puppet 基础架构配置了 Vagrant 项目，并且应用源代码已融入到项目目录中，它就能被构建成可以轻松部署并可重复使用的开发环境。开发人员广泛使用版本控制系统来管理应用源代码。如前文所述，Vagrant 设计的其中一个优势是它遵循基础架构即代码的方法。由于 Vagrant 计算机的配置是作为代码进行维护的，因此它也可通过版本控制系统来进行管理。通过将整个 Vagrant 项目目录放入版本控制系统中，如 Git，管理员能够高效捆绑需要的所有组件，将即时可用的 Vagrant 开发环境重新创建到单个 Git 项目中。

假设一个新开发团队被委派了处理应用源代码的任务。这些开发人员仅仅需要在其工作站上安装 Vagrant 软件，然后依次运行 git clone 和 vagrant up。git clone 命令检索所有 Vagrant 项目组件，如 Vagrant 配置和应用源代码等；而 vagrant up 则利用检索到的组件重新创建 Vagrant 开发环境。

由于开发环境的重新创建是通过代码化的指令控制，每一开发人员最终获得的开发环境不仅与他们团队成员的工作站上的各个环境一致，而且与生产服务器上的环境一致。这便为开发人员提供了保障，在其工作站上 Vagrant 开发环境中验证过的应用源代码更改在部署到生产之后将拥有完全一样的行为。此设计最精妙之处或许在于，不需要运营员工方面投入任何工作，这些开发人员便能让一切就绪。

在运营团队需要更改时，此设计的智能性更加凸显。当运营员工对生产环境进行更改时（例如部署新软件版本来修复错误或安全漏洞），需要对相关的开发环境进行同样的更改，从而使它们保持一致。若要实现这一目标，只需将 Puppet 配置提供给 Vagrant 开发计算机。Vagrant 开发计算机上的 Puppet 客户端在幕后应用新的配置。开发人员可以欣然忽视这一点，他们的工作不会受到这些操作变化的影响。

以这种方式利用 Vagrant 来实施开发环境，可以使开发和运营两个团队携手并行，而不是彼此冲突。开发人员能够自己轻松部署作为生产环境的副本的开发环境，而不会给运营团队带来负担。由于运营团队掌控着开发环境对生产环境的复刻，对生产环境的代码部署应该会顺利。 

############ 使用 vagrant push 部署代码

在 Vagrant 开发环境中成功验证了代码更改后，可以通过多种方式将新代码传播到品控和生产等其他下游环境。Vagrant 提供了将应用代码从 Vagrant 项目目录部署到指定目标的功能。此功能通过在项目的 Vagrantfile 中定义部署例程来启用。

下例演示了如何修改 Vagrantfile 文件来定义部署例程，以使用 rsync 从 host 上的 Vagrant 项目目录复制 Web 应用代码到 web 上的 /var/www/html 目录。此外，部署例程也会在远程目标目录上运行 restorecon，以确保设置了适当的 SELinux 上下文。

```
Vagrant.configure(2) do |config|

... Configuration omitted ...

  config.push.define "local-exec" do |push|
    push.inline = <<-SCRIPT
      rsync -avz www/ web:/var/www/html/
      ssh web 'restorecon -rv /var/www/html'
    SCRIPT
  end
end
```

若要调用部署例程，可以发出 vagrant push 命令。 


### 练习：在 DevOps 环境中部署 Vagrant

1. 在 serverb 上，修改 Vagrant 配置文件，使得 Vagrant 计算机在启动后注册到 serverc.lab.example.com 上运行的 Puppet 宿主。启动 Vagrant 计算机，再验证已应用了 Puppet 配置。

    a. 将目录更改到 /root/vagrant/webapp Vagrant 项目目录。

```
    [root@serverb ~]# cd /root/vagrant/webapp
```

    b. 在 /root/vagrant/webapp/Vagrantfile 的 Vagrant.configure 代码块内添加下面这几行。此条目指定 Puppet 宿主，以及用于 Puppet 代理的选项。这几行应当直接插到 “# Define puppet_server provisioner” 注释的下方。

```
    config.vm.provision "puppet_server" do |puppet|
      puppet.puppet_server = 'serverc.lab.example.com'
      puppet.options = '--onetime --verbose --waitforcert=10 --no-usecacheonfailure --no-daemonize'
    end
```

    c. 启动 Vagrant 计算机，并验证它已成功注册到 Puppet 宿主。确认其应用的配置已启用并启动 httpd 服务。

```
    [root@serverb webapp]# vagrant up
    Bringing machine 'default' up with 'libvirt' provider...
    ==> default: Creating image (snapshot of base box volume).
    ==> default: Creating domain with the following settings..
    ... Output omitted ...
    ==> default: Running provisioner: puppet_server...
    ==> default: Running Puppet agent...
    ... Output omitted ...
    ==> default: Info: Caching catalog for dev.lab.example.com
    ==> default: Info: Applying configuration version '1444926117'
    ==> default: Notice: /Stage[main]/Main/Node[dev.lab.example.com]/Package[httpd]/
     ensure: created
    ==> default: Notice: /Stage[main]/Main/Node[dev.lab.example.com]/Service[httpd]/
     ensure: ensure changed 'stopped' to 'running'
    ==> default: Info: /Stage[main]/Main/Node[dev.lab.example.com]/Service[httpd]:
     Unscheduling refresh on Service[httpd]
    ==> default: Info: Creating state file /var/lib/puppet/state/state.yaml
    ==> default: Notice: Finished catalog run in 28.26 seconds
```

    d. 验证 Puppet 已安装了 httpd 软件包。另外，也验证 Puppet 已启用并启动了 httpd 服务。

```
    [root@serverb webapp]# vagrant ssh
    [vagrant@dev ~]$ rpm -q httpd
    httpd-2.4.6-31.el7.x86_64
    [vagrant@dev ~]$ systemctl is-active httpd
    active
    [vagrant@dev ~]$ systemctl is-enabled httpd
    enabled
    [vagrant@dev ~]$ exit
    logout
    Connection to 192.168.121.88 closed.
```

    e. 使用 git diff 以显示您对 Vagrantfile 进行的更改。将更改与描述性日志消息一起签入到本地存储库。

```
    [root@serverb webapp]# git diff
    diff --git a/webapp/Vagrantfile b/webapp/Vagrantfile
    index cb66841..5b2ff35 100644
    --- a/webapp/Vagrantfile
    +++ b/webapp/Vagrantfile
    @@ -10,4 +10,9 @@ Vagrant.configure(2) do |config|
         sudo yum install -y puppet
       SHELL
     
    +  config.vm.provision "puppet_server" do |puppet|
    +    puppet.puppet_server = 'serverc.lab.example.com'
    +    puppet.options = '--onetime --verbose --waitforcert=10 --no-usecacheonfailu
    +  end
    +
     end
    [root@serverb webapp]# git commit -a -m 'Configured puppet agent.'
    ... Output omitted ...
```

2. 存档新获取的 Puppet 证书，并使它们在重新创建 Vagrant 计算机时能被重复利用。

    a. 在主机上创建一个目录，以存档 Puppet 证书。

```
    [root@serverb webapp]# mkdir -p puppet/ssl
```

    b. 显示 Vagrant 计算机的网络和 SSH 信息，以获取它的 IP 地址。

```
    [root@serverb webapp]# vagrant ssh-config
     Host default
      HostName 192.168.121.88
      User vagrant
      Port 22
      UserKnownHostsFile /dev/null
      StrictHostKeyChecking no
      PasswordAuthentication no
      IdentityFile /root/vagrant/webapp/.vagrant/machines/default/libvirt/private_key
      IdentitiesOnly yes
      LogLevel FATAL
```

    c. 将 Vagrant 计算机上 /var/puppet/ssl 目录的内容复制到主机上的存档目录。务必要将命令中的 Vagrant 计算机 IP 地址替换为 vagrant ssh-config 命令所显示的地址。rsync 命令的 --rsync-path 选项将允许使用 sudo 特权在 Vagrant 计算机上执行文件复制。rsync 命令的 -e 选项指定通过利用基于密钥的身份验证所建立的 SSH 连接来执行该命令。

```
    [root@serverb webapp]# rsync -avz --rsync-path='sudo rsync' \
    -e 'ssh -i /root/vagrant/webapp/.vagrant/machines/default/libvirt/private_key' \
    vagrant@192.168.121.88:/var/lib/puppet/ssl/ puppet/ssl/
    The authenticity of host '192.168.121.88 (192.168.121.88)' can't be established.
    ECDSA key fingerprint is 51:f7:72:0d:2d:e2:13:1b:84:19:61:41:6a:e5:d4:ac.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added '192.168.121.88' (ECDSA) to the list of known hosts.
    receiving incremental file list
    ./
    crl.pem
    certificate_requests/
    certificate_requests/dev.lab.example.com.pem
    certs/
    certs/ca.pem
    certs/dev.lab.example.com.pem
    private/
    private_keys/
    private_keys/dev.lab.example.com.pem
    public_keys/
    public_keys/dev.lab.example.com.pem

    sent 148 bytes  received 8723 bytes  2534.57 bytes/sec
    total size is 10624  speedup is 1.20
```

    d. 关闭 Vagrant 计算机。

```
    [root@serverb webapp]# vagrant destroy
```

3. 修改 Vagrant 配置文件，使 /root/vagrant/webapp/puppet/ssl 目录的内容在 Vagrant 计算机启动时同步到该计算机的 /var/lib/puppet/ssl。启动 Vagrant 计算机，再验证 rsync 复制了该文件夹，并且计算机可以应用 Puppet Web 服务器模块。

    a. 在 Vagrantfile 的 Vagrant.configure 代码块中添加下面这一行。这一行应当直接插到 “# Define sync folder(s)” 注释的下方。

```
    config.vm.synced_folder "puppet/ssl", "/var/lib/puppet/ssl", type: "rsync", owner: "puppet", group: "puppet", rsync__rsync_path: "sudo rsync"
```

    b. 启动新的 Vagrant 计算机，以验证新配置。其输出应当显示 rsync 正在复制 /root/vagrant/webapp/puppet/ssl/ 到 /var/lib/puppet/ssl 并且应当应用 Puppet 模块。

```
    [root@serverb webapp]# vagrant up
    Bringing machine 'default' up with 'libvirt' provider...
    ==> default: Creating image (snapshot of base box volume).
    ==> default: Creating domain with the following settings...
    ... Output omitted ...
    ==> default: Rsyncing folder: /root/vagrant/webapp/puppet/ssl/ => /var/lib/puppet/ssl
    ... Output omitted ...
    ==> default: Running provisioner: puppet_server...
    ==> default: Running Puppet agent...
    ==> default: Info: Retrieving plugin
    ==> default: Info: Caching catalog for dev.lab.example.com
    ==> default: Info: Applying configuration version '1444926117'
    ==> default: Notice: /Stage[main]/Main/Node[dev.lab.example.com]/Package[httpd]/ensure: created
    ==> default: Notice: /Stage[main]/Main/Node[dev.lab.example.com]/Service[httpd]/ensure: ensure changed 'stopped' to 'running'
    ==> default: Info: /Stage[main]/Main/Node[dev.lab.example.com]/Service[httpd]: Unscheduling refresh on Service[httpd]
    ==> default: Info: Creating state file /var/lib/puppet/state/state.yaml
    ==> default: Notice: Finished catalog run in 28.20 seconds
```

    c. 将更改与描述性日志消息一起签入到本地 Git 存储库。

```
    [root@serverb webapp]# git add Vagrantfile puppet
    [root@serverb webapp]# git commit -m 'Preserved Puppet SSL keys.'
    ... Output omitted ...
```

4. 配置用于托管 Web 应用的 Vagrant 计算机，再验证该应用。

    a. 在主机上为 Web 应用源代码创建一个目录。为它填充一个简单的 HTML 文件 index.html，其内容为“Welcome to Web App 1.0”。

```
    [root@serverb webapp]# mkdir www
    [root@serverb webapp]# echo 'Welcome to Web App 1.0' > www/index.html
```

    b. 修改 Vagrant 配置文件，使得 Web 应用在 Vagrant 计算机初始化时同步到该计算机上的 /var/www/html。将类型设置为 rsync，所有权设置为用户和组 apache，将确保它将 sudo 用于 rsync。这一行应当插到 Vagrantfile 的 “# Define sync folder(s)” 部分中。

```
    # Define sync folder(s)
    config.vm.synced_folder "puppet/ssl", "/var/lib/puppet/ssl", type: "rsync", owner: "puppet", group: "puppet", rsync__rsync_path: "sudo rsync"
    config.vm.synced_folder "www", "/var/www/html", type: "rsync",  owner: "apache", group: "apache", rsync__rsync_path: "sudo rsync"
```

    c. 在从主机复制过来后，/var/www/html 目录中的文件可能没有正确的 SELinux 上下文。扩展现有 Vagrant 配置文件 shell 调配器，以执行 restorecon 来进行更正。

```
    # Define shell provisioner
    config.vm.provision "shell", inline: <<-SHELL
      sudo cp /home/vagrant/sync/etc/yum.repos.d/* /etc/yum.repos.d
      sudo yum install -y puppet
      sudo restorecon -rv /var/www/html
    SHELL
```

    d. 修改 Vagrant 配置文件主机设置，使得 Vagrant 计算机的 80 端口能够通过 localhost 上的 8000 端口进行访问。在 Vagrant.configure 代码块中添加下面这一行。

```
    # Define host settings
    config.vm.hostname = "dev.lab.example.com"
    config.vm.network :forwarded_port, guest: 80, host: 8000
```

    e. 重新加载 Vagrant 配置，让新引入的更改生效：

```
    [root@serverb webapp]# vagrant reload
    ==> default: Halting domain...
    ==> default: Starting domain.
    ==> default: Waiting for domain to get an IP address...
    ==> default: Waiting for SSH to become available...
    ==> default: Creating shared folders metadata...
    ==> default: Forwarding ports...
    ==> default: 80 => 8000 (adapter eth0)
    ==> default: Rsyncing folder: /root/vagrant/webapp/ => /home/vagrant/sync
    ==> default: Rsyncing folder: /root/vagrant/webapp/puppet/ssl/ =>
     /var/lib/puppet/ssl
    ==> default: Rsyncing folder: /root/vagrant/webapp/www/ => /var/www/html
    ==> default: Machine already provisioned. Run `vagrant provision` or use
     the `--provision`
    ==> default: flag to force provisioning. Provisioners marked to run always
     will still run.
```

    f. 验证 Vagrant 计算机上的 Web 应用使用 localhost 上的 8000 端口正常工作。

```
    [root@serverb webapp]# curl http://localhost:8000
    Welcome to Web App 1.0
```

    g. 将更改与描述性日志消息一起签入到本地 Git 存储库。

```
    [root@serverb webapp]# git add Vagrantfile www
    [root@serverb webapp]# git commit -m 'Synced web content from host.'
    ... Output omitted ...
``` 

5. 添加一个推送组件到 Vagrant 配置。它应当将 Web 应用的源代码部署到在 servera 上运行的生产 Web 服务器的 /var/www/html。另外，也请运行 restorecon 来恢复 SELinux 上下文。在 Vagrantfile 中定义，紧接在 Puppet 调配器定义的后面。

```
config.push.define "local-exec" do |push|
  push.inline = <<-SCRIPT
    rsync -avz www/ servera:/var/www/html/
    ssh servera 'restorecon -rv /var/www/html'
  SCRIPT
end
```
 
6. 将更改与描述性日志消息一起签入到本地 Git 存储库，然后将所有更改推送到中央 Git 存储库。

```
[root@serverb webapp]# git commit -a -m 'Added a push component.'
[master 2174692] Added a push component.
 1 file changed, 7 insertions(+)
[root@serverb webapp]# git push
Counting objects: 33, done.
Compressing objects: 100% (23/23), done.
Writing objects: 100% (30/30), 10.01 KiB | 0 bytes/s, done.
Total 30 (delta 6), reused 0 (delta 0)
To ssh://student@workstation/var/git/vagrant.git
   a413d3c..2174692  master -> master
```


### 实验：在 DevOps 环境中实施 Puppet

1. 以 root 身份登录开发服务器 serverb。安装 git 软件包。使用 “Student User” 作为用户名和 “root@serverb.lab.example.com” 作为电子邮件地址，为 root 用户配置 Git。将 Git 的默认 push 行为设置为 “simple”。

    a. 在 serverb 上安装 git 软件包。

```
    [root@serverb ~]# yum -y install git
```
    
    2. 为 root 配置 Git 用户身份。

```
    [root@serverb ~]# git config --global user.name 'Student User'
    [root@serverb ~]# git config --global user.email 'root@serverb.lab.example.com'
```

    3. 设置 Git 的默认 push 行为。

```
    [root@serverb ~]# git config --global push.default simple
```

2. 在 /root/vagrant 创建 Vagrant 工作目录，然后使用 Vagrantfile 中配置的同步文件夹功能将位于 serverc 上 /var/git/webapp.git 的 Web 应用 Git 项目克隆到其中。

    a. 创建 Vagrant 工作目录 /root/vagrant/。

```
    [root@serverb ~]# mkdir /root/vagrant
```

    b. 将 webapp.git 项目克隆到 Vagrant 工作目录。

```
    [root@serverb ~]# cd /root/vagrant
    [root@serverb vagrant]# git clone serverc:/var/git/webapp.git
    Cloning into 'webapp'...
    The authenticity of host 'serverc (172.25.250.12)' can't be established.
    ECDSA key fingerprint is 85:57:bb:68:73:68:35:76:57:eb:bb:2b:7b:2c:2a:ad.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added 'serverc,172.25.250.12) to the list of known hosts.
    root@serverc's password: redhat
    remote: Counting objects: 22, done.
    remote: Compressing objects: 100% (15/15), done.
    remote: Total 22 (delta 2), reused 22 (delta 2)
    Receiving objects: 100% (22/22), 9.20 KiB | 0 bytes/s, done.
    Resolving deltas: 100% (2/2), done.
```

3. 启动 webapp Git 项目中配置的 Vagrant 计算机，以创建 Web 应用开发环境。这会将 serverb 上的 TCP/8000 配置为转发到 Vagrant 计算机上的 TCP/80。验证 Web 应用正常运行。

    a. 更改到 webapp Git 项目目录。

```
    [root@serverb vagrant]# cd webapp
```

    b. 启动 Vagrant 计算机。

```
    [root@serverb webapp]# vagrant up
```

    c. 验证开发 Web 应用正常运行。

```
    [root@serverb webapp]# curl http://localhost:8000
    Welcome to Web App 1.0
```

4. 修改 Web 应用源文件 /root/vagrant/webapp/www/index.html，使其包含内容“Welcome to Web App 2.0”。将修改后的文件同步到 Vagrant 计算机，验证更改，然后将更改提交到 serverc 上的 Git 存储库。

    a. 修改 index.html 文件。

```
    [root@serverb webapp]# echo 'Welcome to Web App 2.0' > www/index.html
```

    b. 将修改后的文件同步到 Vagrant 计算机。

```
    [root@serverb webapp]# vagrant rsync
    ==> default: Rsyncing folder: /root/vagrant/webapp/ => /home/vagrant/sync
    ==> default: Rsyncing folder: /root/vagrant/webapp/puppet/ssl/ => /var/lib/puppet/ssl
    ==> default: Rsyncing folder: /root/vagrant/webapp/www/ => /var/www/html
```

    c. 验证对开发 Web 应用的更改。

```
    [root@serverb webapp]# curl http://localhost:8000
    Welcome to Web App 2.0
```

    d. 将更改提交到远程 Git 存储库。

```
    [root@serverb webapp]# git add www/index.html
    [root@serverb webapp]# git commit -m 'Update web app to version 2.0'
    [master 365254c] Update web app to version 2.0
     1 file changed, 1 insertion(+), 1 deletion(-)
    [root@serverb webapp]# git push
    root@serverc's password: redhat
    Counting objects: 7, done.
    Compressing objects: 100% (2/2), done.
    Writing objects: 100% (4/4), 347 bytes | 0 bytes/s, done.
    Total 4 (delta 1), reused 0 (delta 0)
    To serverc:/var/git/webapp.git
       58e97d5..365254c  master -> master
```

5. 准备生产服务器 servera，以使用 Puppet 发布的标准配置。将它注册到 serverc 上的 Puppet 宿主，然后应用其配置。

    a. 在 servera 上安装 puppet 软件包。

```
    [root@servera ~]# yum -y install puppet
```

    b. 将 serverc 配置为 Puppet 宿主。

```
    [root@servera ~]# echo 'server = serverc.lab.example.com' >> /etc/puppet/puppet.conf
```

    c. 通过向 Puppet 宿主提交为证书请求签名的请求，来初始化 Puppet 客户端。

```
    [root@servera ~]# systemctl start puppet.service
    [root@servera ~]# systemctl enable puppet.service
    ln -s '/usr/lib/systemd/system/puppet.service' '/etc/systemd/system/multi-user.target.wants/puppet.service'
```

    d. 应用 servera 的 Puppet 配置。

```
    [root@servera ~]# puppet agent --test
```

6. 将来自开发服务器 serverb 的 Web 应用源代码部署到生产服务器 servera。

    a. 从 serverb，连接 servera 上的 Web 服务器，以便在从开发环境部署源代码之前查看该网站。

```
    [root@serverb webapp]# curl http://servera
    ... Output omitted ...
```
    最初会显示 Apache 测试页面，因为该应用尚未部署。

    b. 利用 Vagrantfile 中配置的 push 功能，将 Web 应用源代码从 serverb 部署到 servera。

```
    [root@serverb webapp]# vagrant push
    The authenticity of host 'servera (172.25.250.10)' can't be established.
    ECDSA key fingerprint is 85:57:bb:68:73:68:35:76:57:eb:bb:2b:7b:2c:2a:ad.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added 'servera,172.25.250.10' (ECDSA) to the list of known hosts.
    root@servera's password: redhat
    sending incremental file list
    ./
    index.html

    sent 111 bytes  received 34 bytes  19.33 bytes/sec
    total size is 23  speedup is 0.16
    root@servera's password: redhat
```

    c. 验证 servera 上的网站现在运行新的源代码。其输出应当与之前在 serverb 上本地测试时检索到的输出相同。

```
    [root@serverb webapp]# curl http://servera
    Welcome to Web App 2.0
```


### 总结

在本章中，您可以学到：

*    Vagrant 软件需要 box、提供程序和 Vagrantfile 来创建 Vagrant 计算机。
*    Vagrantfile 用于配置 Vagrant 计算机。
*    从 box 镜像构建 Vagrant 计算机后，Vagrant 部署器可自动化软件安装和系统配置。
*    Vagrant 的 puppet_server 调配器可以自动化集成 Vagrant 计算机到组织的 Puppet 基础架构中。
*    代码部署例程可以在 Vagrantfile 中定义，并可通过 vagrant push 命令调用。
