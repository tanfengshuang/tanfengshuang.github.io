---
layout: post
title:  "优化 Ansible(407-8)"
categories: Linux
tags: RHCA 407
---

### 配置连接类型

默认情况下，Ansible 使用 SSH 协议来远程连接目标节点。假设没有启用 pipelining，Ansible 将使用此 SSH 连接来传输模块和模板文件，运行远程命令，并在受管主机上运行 playbook 中的 play。因此，拥有快速、稳定且安全的 SSH 连接是 Ansible 的头等大事。除了 ssh 外，Ansible 也提供其他连接类型用于连接受管主机，包括 paramiko 和 local。连接类型可以在 Ansible 配置文件 (ansible.cfg)、基于主机或组的清单文件、playbook 或命令行界面中指定。
注意

> pipelining 功能通过执行许多 Ansible 模块而不实际传输文件，减少在远程服务器上执行模块时所需的 SSH 操作数量。若要支持 pipelining，必须在受管主机上的 sudoers 文件中禁用 requiretty 功能。若要启用 pipelining，可在 Ansible 配置文件的 [ssh_connection] 部分中将 pipelining 设为 true。 

连接类型：

*    smart：此连接类型检查本地安装的 SSH 客户端是否支持 ControlPersist 功能。如果支持 ControlPersist，Ansible 将使用本地 SSH 客户端。如果 SSH 客户端不支持 ControlPersist，smart 连接类型将退回到使用 paramiko。smart 连接类型是默认设置。

    ControlPersist 功能在 OpenSSH 5.6 或更新版本中可用。它允许 SSH 连接持久存在，这样在频繁通过 SSH 运行命令时，它们不必一再经由初始的握手。如果达到了 ControlPersist 超时时间，SSH 连接将再次经由 TCP 握手。此设置可以在 OpenSSH 配置中设置，但最好是在 Ansible 设置中完成。在 /etc/ansible/ansible.cfg，其默认设置列在了注释中： #ssh_args = -o ControlMaster=auto -o ControlPersist=60s

    60 秒默认值可以修改，即使初始连接已经退出，其连接也会在较长时间内处于空闲状态。此值应当基于运行 playbook 所需的时间长短以及运行 playbook 的频率。设为 30 分钟或许更加适当： ssh_args = -o ControlMaster=auto -o ControlPersist=30m

*    paramiko：Paramiko 是 SSHv2 协议的 Python 实施。Ansible 支持它作为连接类型，以向后兼容 RHEL6 和更早版本，它们对应的 OpenSSH 版本中没有对 ControlPersist 设置的支持。
*    local：local 连接在本地运行命令，而不通过 SSH 运行。
*    ssh：ssh 连接使用基于 OpenSSH 的连接，它支持 ControlPersist 技术。
*    docker：Ansible 提供了一种新的 docker 连接类型。它通过 docker exec 命令来连接容器。

Ansible 附带了其他连接类型插件，如 chroot、libvirt_lxc、适用于 FreeBSD jail 环境的 jail，以及最新加入的 winrm，后者利用 Python pywinrm 模块与没有 SSH 但支持 PowerShell 远程连接的 Windows 远程主机通信。Ansible 连接是可插拔和可扩展的，未来发行中也计划加入更多连接类型。 


###### Ansible 配置

1. 连接类型可以在 ansible.cfg 文件的 [defaults] 部分中通过 transport键加以指定。

```
[defaults] 
# some basic default values...
... Output omitted ...
#transport      = smart
```

2. 在通过 ansible 运行临时命令或通过 ansible-playbook 运行 playbook 时，可以利用 -c TRANSPORTNAME 选项覆盖 transport 全局值。

在 /etc/ansible/ansible.cfg 中，与各种连接类型相关的默认设置以注释形式显示在其对应连接标题的下方。可以通过在控制 ansible.cfg 文件中以不同的值指定该指令来覆盖它们。ssh 连接类型的设置在 [ssh_connection] 部分下指定。类似地，paramiko 的设置可以在 [paramiko_connection] 部分下找到。

```
[ssh_connection]
... Output omitted ...
# control_path = %(directory)s/%%h-%%r
#control_path = %(directory)s/ansible-ssh-%%h-%%p-%%r
#pipelining = False
#scp_if_ssh = True
#sftp_batch_mode = False
```

3. 清单文件, 也可以通过 ansible_connection 变量，以主机为基础在清单文件中指定连接类型。下例演示了一个清单文件，它将 local 连接类型用于 localhost，并将 ssh 连接类型用于 demo。

```
[targets]
localhost  ansible_connection=local
demo.lab.example.com  ansible_connection=ssh
```

4. Playbook, 也可以在运行完整的 playbook 时指定连接类型。如下例所示，将 --connection=CONNECTIONTYPE 选项设为 ansible-playbook 命令。

```
$ ansible-playbook playbook.yml --connection=local
```
    
5. Playbook, 也可在 playbook 中通过 connection: CONNECTIONTYPE 参数来设置连接类型，如下例中所示。

```
---
- name: Connection type in playbook
  hosts: 127.0.0.1
  connection: local
```

###### 在 playbook 中设置环境变量

Ansible 可以通过在 playbook 中使用 environment: 参数来进行这些类型的环境设置。

get_url、yum 和 apt 等一些 Ansible 模块使用环境变量来设置其代理服务器。下例演示了如何使用 environment: 参数为软件包安装设置代理服务器，该参数将 $http_proxy 环境变量设置为 http://demo.lab.example.com。

```
---
- hosts: devservers
  tasks:
     - name: download a file using demo.lab.example.com as proxy
       get_url:
         url: http://materials.example.copm/file.tar.gz
         dest: ~/Downloads
       environment:
          http_proxy: http://demo.lab.example.com:8080
```

在 playbook 中，可以使用 vars 关键字在变量中定义环境散列，或者可使用 group_vars 关键字。而后可以在 playbook 内通过 environment 参数引用这些变量。

```
---
- name: Demonstrate environment variable in block
  hosts: localhost
  connection: local
  gather_facts: false
  vars:
    myname: ansible

  tasks:
  - block:
    - name: Print variable
      shell: "echo $name"
      register: result
    - debug: var=result.stdout
    environment:
      name: "{{ myname }}"
```

在使用 Ansible 角色时，可以在 playbook 级别上传递环境变量。下例演示了如何在使用 Ansible 角色时传递 $http_proxy 环境变量。

```
---
- hosts: demohost
  roles:
    - php
    - nginx
  environment:
    http_proxy: http://demo.lab.example.com:8080
```

### 配置委派

为完成某些配置任务，可能需要对与当前所配置服务器不同的其他服务器执行操作。例如，这可能包括需要等待其他服务器重新启动、添加服务器到负载平衡器或监控服务器，或者更改当前配置的服务器所需要的 DHCP 或 DNS 数据库等操作。

委派可以帮助在不同于清单中作为 play 目标受管主机的主机上为任务执行必要的操作。委派可以处理的一些情景包括：

*    委派任务到本地计算机
*    委派任务到 play 之外的主机
*    委派任务到存在于清单中的主机
*    委派任务到清单中不存在的主机

###### 委派任务到本地计算机

```
---
- name: delegate_to:localhost example
  hosts: dev
  tasks:
    - name: remote running process
      command: ps
      register: remote_process

    - debug: msg="{{ remote_process.stdout }}"

    - name: Running Local Process
      command: ps
      delegate_to: localhost
      register: local_process

    - debug: 
        msg: "{{ local_process.stdout }}"
```

local_action 关键字是替代 delegate_to: localhost 的简写语法，可以任务为基础使用。

```
---
- name: local_action example
  hosts: dev
  tasks:
    - name: remote running process
      command: ps
      register: remote_process

    - debug: msg="{{ remote_process.stdout }} "

    - name: Running Local Process
      local_action: command 'ps'
      register: local_process

    - debug: 
        msg: "{{ local_process.stdout }}"
```

隐式 localhost功能, 此功能的效用体现于，即使清单中没有定义 localhost 条目，也可以解析 localhost 主机模式。

```
$ cat inventory
$ ansible localhost --list-hosts
  hosts (1):
    localhost
```

> 务必要注意 hosts: all将不会与隐式 localhost 条目匹配。

> 针对隐式 localhost 定义的连接类型是 local_connection。这与默认的 ssh 连接类型不同，后者在清单文件中显式定义了 localhost 时使用。 

> 隐式 localhost连接类型差别有一个副作用，使用隐式 localhost 进行的连接将忽略 remote_user 设置，因为不会涉及到登录过程。如果定义的 remote_user 设置与执行 Ansible 命令的用户不同，则管理员可能会在使用隐式和显式 localhost 定义时获得不同的结果。例如，如果使用了利用 sudo 的特权升级，则隐式 localhost 会从执行 Ansible 命令的用户帐户升级特权，而显式 localhost 则使用 remote_user 帐户升级特权。如果这两个帐户配置了不同的 sudo 特权，则尝试升级特权时可能会获得不同的结果。 


###### 委派任务到 play 之外的主机

Ansible 可以配置为使用 delegate_to 在不属于 play 的一部分的主机上执行任务。该委派的模块依然是每一台计算机运行一次，但不是在目标计算机上运行，而是在由 delegate_to 指定的主机上运行。可用的事实将是适用于原有主机的事实，而非该任务所委派到的主机。任务具有原始目标主机的上下文，但在它所委派到的主机上执行。

下例显示了将任务委派到外部计算机（本例中为 loadbalancer-host）的 Ansible 代码。本例在本地平衡器主机上运行一个命令，将受管主机从负载平衡器中移除，然后部署最新版本的 Web 堆栈。完成该任务后，通过运行一个脚本把受管主机重新添加到负载平衡器池中。

```
- hosts: webservers
  tasks:
    - name: Remove server from load balancer
      command: remove-from-lb {{ inventory_hostname }}
      delegate_to: loadbalancer-host

    - name: deploy the latest version of web stack
      git:repo=git://foosball.example.org/path/to/repo.git dest=/srv/checkout

    - name: Add server to load balancer pool
      command: add-to-lb {{ inventory_hostname }}
      delegate_to: loadbalancer-host
```

###### 委派任务到存在于清单中的主机

如果委派任务到列在清单中的主机，则创建与委派目标的连接时将使用清单数据。这包括 ansible_connection、ansible_host、ansible_port 和ansible_user 等变量的设置。仅使用与连接相关的变量；其余则从原始目标受管主机上读取。

###### 委派任务到清单中不存在的主机

在委派任务到未列在清单中的主机时，Ansible 将使用与用于受管主机相同的连接类型和详情来连接委派的主机。若要调整连接详情，可使用 add_host 模块并利用定义的连接数据在清单中创建临时主机。 

```
- name: test play
  hosts: localhost
  tasks:

    - name: add delegation host
      add_host: name=demo ansible_host=172.25.250.10 ansible_user=devops

    - name: echo Hello
      command: echo "Hello from {{ inventory_hostname }}"
      delegate_to: demo
      register: output

    - debug:
        msg: "{{ output.stdout }}"
```

###### 通过委派同步执行任务

委派的任务针对各个目标受管主机运行。但 Ansible 任务可以在多台受管主机上并行运行。这可能会造成委派的主机上出现争用情况的问题。在任务中使用条件时，或者多个并发任务并行运行时，尤其会有这种可能。这也有可能造成“惊群”(thundering herd) 问题，在委派的主机上一次性打开过多连接。SSH 服务器具有一个 MaxStartups 配置选项，可以限制允许的并发连接数。

###### 委派的事实

通过委派的任务收集的任何事实默认为分配到 delegate_to 主机，而不是实际产生这些事实的主机。可以将 delegate_facts 指令设置为 True，从而将任务收集的事实分配到委派的主机，而非当前主机。 

```
- hosts: app_servers
  tasks:
    - name: gather facts from app servers
      setup:
      delegate_to: "{{item}}"
      with_items: "{{groups['lb_servers']}}"

    - debug: var=ansible_eth0['ipv4']['address']

#inventory file
[app_servers]
demo.lab.example.com
[lb_servers]
workstation.lab.example.com
```


### 配置并行

Ansible 通过在所有主机上并行运行任务，提供对 playbook 执行的更大掌控力度。默认情况下，Ansible 最多仅允许分叉五次(同时在五台不同的计算机上运行特定的一项任务)。这个值在 Ansible 配置文件 ansible.cfg 中设置。有大量（超过五台）受管主机时，可以将 forks 参数更改为更符合环境需要的设置。可以通过在配置文件中为 forks 键指定一个新值来覆盖其默认值，或者使用 ansible-playbook 或 ansible 命令的 --forks 选项来更改值。

```
$ grep forks /etc/ansible/ansible.cfg
#forks = 5
```

###### 并行运行任务

对于任何具体的 play，您都可以在 playbook 中使用 serial 关键字，从 Ansible 配置文件中指定的分叉数暂时减少并行运行的计算机数量。serial 关键字主要用于控制滚动更新。

滚动更新: 如果某一网站部署到 100 台 Web 服务器上，应当同时仅更新其中 10 台。可以在 playbook 中将 serial 键设为 10，来减少同时部署的数量（假设 fork 键之前被设成了较高的值）。也可以百分比的形式指定 serial 关键字，应用到 play 中的主机总数。如果主机数无法均匀分到数道工序中，最后一个工序将包含模数。无论百分比为何，每一工序的主机数始终为 1 或以上。

```
---
- name: Limit the number of hosts this play runs on at the same time
  hosts: appservers
  serial: 2
```

不论设置的分叉数为多少，Ansible 始终仅根据 play 中的当前主机数来加快任务。 

###### 异步任务

有些系统操作可能需要一些时间才能完成。比如，下载大文件或重新引导服务器，完成此类任务需要比较长的时间。通过使用并行和分叉，Ansible 在受管主机上快速启动命令，然后轮询主机的状态，直至全部完成。

1. async 关键字触发 Ansible 在后台运行作业并可稍后检查，其值将是 Ansible 等候命令完成的最长等待时间。对于运行用时特别长的任务，您可以将 Ansible 配置为一直等候到其作业完成。为此，请将 async 的值设为 0。
2. poll 的值指示 Ansible 以何种频率轮询，以检查命令是否已经完成。默认的 poll 值是 10 秒。

```
在示例中，get_url 模块需要很长时间来下载文件，async: 3600 指示 Ansible 等待 3600 秒来等候任务完成，而 poll: 10 则是以秒为单位的轮询时间（检查下载是否已完成）。
---
- name: Long running task
  hosts: demoservers
  remote_user: devops
  tasks:
    - name: Download big file
      get_url: url=http://demo.example.com/bigfile.tar.gz
      async: 3600
      poll: 10
```

3. 推迟异步任务: 运行时间很长的操作或维护脚本可以和其他任务一道执行，其中可通过使用 wait_for 模块来推迟对完成状态的检查。若要将 Ansible 配置为不等候作业完成，可将 poll 的值设为 0；这样，Ansible 启动命令后不轮询其完成状态，而是直接转到后台任务。 对于运行用时特别长的任务，您可以将 Ansible 配置为一直等候到其作业完成。为此，请将 async 的值设为 0。

```
---
- name: Restart and wait until the server is rebooted
  hosts: demoservers
  remote_user: devops
  tasks:
    - name: restart machine
      shell: sleep 2 && shutdown -r now "Ansible updates triggered"
      async: 1
      poll: 0
      become: true
      ignore_errors: true

    - name: waiting for server to come back
      local_action:
        wait_for: 
          host: "{{ inventory_hostname }}"
          state: started
          delay: 30
          timeout: 300
      become: false
```

4. 异步任务状态: 在运行异步任务期间，也可通过使用 Ansible async_status 模块来检查其完成状态。该模块需要将作业或任务标识符用作其参数。

```
---
# Async status - fire-forget.yml
- name: Async status with fire and forget task
  hosts: demoservers
  remote_user: devops
  become: true
  tasks:

    - name: Download big file
      get_url: url=http://demo.example.com/bigfile.tar.gz
      async: 3600
      poll: 0
      register: download_sleeper

    - name: Wait for download to finish
      async_status: "jid={{ download_sleeper.ansible_job_id }}"
      register: job_result
      until: job_result.finished
      retries: 30

$ ansible-playbook fire-forget.yml

PLAY [Async status with fire and forget task] **********************************

TASK [setup] *******************************************************************
ok: [demo.example.com]

TASK [Download big file] *******************************************************
ok: [demo.example.com]

TASK [Wait for download to finish] *********************************************
FAILED - RETRYING: TASK: Wait for download to fins (29 retries left). Result
 was: {u'ansible_job_id': u'963772827414.1563', u'started': 1, u'changed': False,
 u'finished': 0, u'results_file': u'/root/.ansible_async/963772827414.1563',
 'invocation': {'module_name': u'async_status', u'module_args': {u'jid':
 u'963772827414.1563', u'mode': u'status'}}}
... Output omitted ...
changed: [demo.example.com]

PLAY RECAP *********************************************************************
demo.example.com    : ok=3    changed=1    unreachable=0    failed=0
```

### Summary

*    连接类型决定 Ansible 通过何种方式与受管主机连接，从而与它们通信
*    默认情况下，Ansible 使用 smart 连接类型（将使用 SSH 协议来连接远程主机），但也提供了其他连接类型
*    任务可以使用 environment 指令来设置在受管主机上执行任务时的 shell 环境变量
*    Ansible 可以 委派任务到不同于目标受管主机的其他主机上运行，从而在其他服务器上采取所需操作来配置受管主机
*    Ansible 通常尝试同时在多台主机上并行运行任务同步连接的数量可以在配置文件和 playbook 中进行控制
*    async 关键字触发 Ansible 在后台运行作业，poll 关键字则控制 Ansible 检查任务是否已完成的频率
