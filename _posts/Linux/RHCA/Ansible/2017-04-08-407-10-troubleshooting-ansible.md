---
layout: post
title:  "对 Ansible 进行故障排除(407-10)"
categories: Linux
tags: RHCA 407
---


### 对 playbook 进行故障排除

###### Ansible 中的日志文件

默认情况下，Ansible 配置为不将其输出记录到任何日志文件。它提供了一个内置日志基础架构，可以通过 ansible.cfg 配置文件的 default 部分中的 log_path 参数进行配置，或者通过 $ANSIBLE_LOG_PATH 环境变量来配置。如果进行了其中任一/全部配置，Ansible 会把来自 ansible 和 ansible-playbook 命令的输出存储到通过 ansible.cfg 配置文件或 $ANSIBLE_LOG_PATH 环境变量配置的日志文件中。

如果要将 Ansible 日志文件保存在默认的日志文件目录 /var/log 下，则必须以 root 用户身份运行 playbook，或者必须已开启 /var/log 上的权限。更为常见的是，日志文件存储在本地 playbook 目录中。

> 红帽建议配置 logrotate 来管理 Ansible 的日志文件。 

###### 调试模块

在 Ansible 提供的模块中，debug 模块可以为控制节点上发生的情况提供更好的洞察。此模块可以提供特定变量在 playbook 执行时的值。在对使用变量互相通信的任务（例如，将一项任务的输出用作后续任务的输入）进行管理时，此功能可以发挥关键作用。下方的示例在 debug 语句内使用 msg 和 var 语句来显示 ansible_memfree_mb 事实和 output 变量在执行时的值。

```
- debug: msg="The free memory for this system is {{ ansible_memfree_mb }}"

- debug: var=output verbosity=2
```

###### 管理错误

*    --syntax-check 选项检查 playbook 的 YAML 语法。
*    --step 选项以交互方式执行任务，询问是否应执行该任务。
*    --start-at-task 选项从给定任务起开始执行，避免重复执行前面的所有任务。如果 playbook 含有大量任务，使用 ansible-playbook 命令中的 --step 或 --start-at-task 选项可能会有帮助。

```
# ansible-playbook play.yml --step
      
# ansible-playbook play.yml --start-at-task="start httpd service"
      
# ansible-playbook play.yml --syntax-check
```

###### 使用 ansible-playbook 调试

当每一受管主机执行任务时，该主机显示在对应的 TASK 标头下，同时显示有该受管主机上的任务状态，状态可设为 ok、fatal 或 changed。在 playbook 的底部，PLAY RECAP 部分显示了其输出状态中对每一受管主机执行的任务数。

ansible-playbook 命令提供的默认输出并不包含足够详细的信息，以用对受管主机上可能出现的问题进行故障排除。ansible-playbook -v 命令提供了额外的调试信息，总共有四个级别。 

*    选项 	描述
*    -v 	显示输出数据。
*    -vv 	显示输出和输入数据。
*    -vvv 	包含关于和受管主机连接的信息。
*    -vvvv 	增加了连接插件相关的额外详细程度选项，包括受管主机上用于执行脚本的用户，以及所执行的脚本。

###### playbook 管理的推荐做法

尽管上文讨论的工具或可帮助识别和改正 playbook 中的问题，在开发这些 playbook 时，务必要牢记一些推荐做法，它们有助于简化对问题进行故障排除的流程。下面列出了一些 playbook 开发推荐做法。

*    始终命名任务，在 name 中提供对任务用途的描述。在执行 playbook 时，此 name 会显示出来。
*    包含注释，以添加与任务相关的其他内嵌文档。
*    有效利用垂直空白。YAML 语法大体基于空格，因此要避免使用制表符，以免出现错误。
*    尽可能使 playbook 简单。仅使用您需要的功能。


### 对受管主机进行故障排除

###### 检查模式用作测试工具

您可以使用 ansible-playbook --check 命令对 playbook 运行冒烟测试。此选项执行 playbook，但不对受管主机的配置进行任何更改。如果 playbook 中使用的模块支持检查模式，则将显示要在受管主机上进行的更改。如果模块不支持该模式，则不会显示这些更改。

```
# ansible-playbook --check playbook.yml
```
  

当 playbook 中包含的所有任务都必须以检查模式执行时，可使用 ansible-playbook --check 命令；但是，如果仅其中一部分任务需要以检查模式执行，则 always_run 选项是更佳的方案。每一任务都可以关联 always_run 子句。如果任务必须以检查模式执行，此子句的值为 true；否则，其值为 false。下列任务以检查模式执行。

```
  tasks:
    - name: task in check mode
      shell: uname -a
      always_run: yes
```

下列任务不以检查模式执行。

```
  tasks:
    - name: task in check mode
      shell: uname -a
      always_run: false
```

> 如果任务利用了条件，则 ansible-playbook --check 命令可能无法正常运作。

Ansible 也提供 --diff 选项。此选项报告对受管主机上模板文件的更改。与 --check 选项结合使用时，将显示这些更改，但不实际执行它们。

```
# ansible-playbook --check --diff playbook.yml
```

###### 使用模块进行测试

一些模块可以提供关于受管主机状态为何的额外信息。下表中列出了一些 Ansible 模块，可用于测试和调试受管主机上的问题。

1. uri 模块提供了一种方式，可以检查 RESTful API 是否返回需要的内容。

```
  tasks:
    - action: uri url=http://api.myapp.com return_content=yes
      register: apiresponse

    - fail: msg='version was not provided'
      when: "'version' not in apiresponse.content"
```

2. script 模块支持在受管主机上执行脚本，如果该脚本的返回代码不是零，则报告失败。脚本必须位于控制节点上，并将传输到受管主机（并在其上执行）。

```
  tasks:
    - script: check_free_memory
```

3. stat 模块可以检查不直接由 Ansible 管理的文件和目录是否存在。在下例中查看 assert 模块的用法，该示例检查受管主机上是否存在某一文件。

```
  tasks:
    - stat: path=/var/run/app.lock
      register: lock

    - assert:
        that:
          - lock.stat.exists
```

######使用临时命令进行测试

下面几个示例演示了一些可通过使用临时命令在受管主机上执行的检查

```
检查 httpd 软件包目前是否已安装到了 demohost 受管主机上
# ansible demohost -u devops -b -m yum -a 'name=httpd state=present'

检查 demohost 受管主机中配置的磁盘上当前的可用空间。
# ansible demohost -a 'lsblk'

检查 demohost 受管主机上当前的可用内存。
# ansible demohost -a 'free -m'
```

###### 正确的测试等级

Ansible 确保正确完成对 playbook 中包含的并由其模块执行的配置。它监控所有模块是否报告有故障，一旦遇到任何故障将立即停止 playbook。这可确保出现故障之前执行的任何任务都没有错误。正因为此，无需检查 Ansible 所管理的任务的结果是否已正确应用到受管主机上。当需要更直接的故障排除时，可以在 playbook 中添加一些健康检查，或者以临时命令形式直接运行它们。 


### Summary

*    Ansible 提供了内置日志记录功能。默认情况下，不启用此功能。
*    ansible.cfg 配置文件 default 部分中的 log_path 参数指定所有 Ansible 输出重定向到的日志文件位置。
*    debug 模块提供运行 playbook 时的额外调试信息（如变量的当前值）。
*    ansible-playbook 命令的 -v 选项提供多种级别的输出详细程度。这可在运行 playbook 时用于调试 Ansible 任务。
*    --check 选项使得支持检查模式的 Ansible 模块能够显示要执行的更改，而不将这些更改应用到受管主机上。
*    可以使用临时命令对受管主机执行其他检查。
*    只要 playbook 成功完成，就无需重复检查 Ansible 执行的配置。 
