---
layout: post
title:  "实施角色(407-7)"
categories: Linux
tags: RHCA 407
---

### 描述角色结构

角色为 Ansible 提供了从外部文件加载任务、处理程序和变量的途径。角色也可关联和引用静态的文件和模板。定义角色的文件具有特定的名称，组织到固定的目录结构中，后文中将予以阐述。角色可以编写成满足普通用途需求，并且能被重复利用。

使用 Ansible 角色具有下列优点：

*    角色可以分组内容，从而与他人轻松共享代码
*    可以编写角色来定义系统类型的基本要素：Web 服务器、数据库服务器、git 存储库，或满足其他用途
*    角色使得较大型项目更容易管理
*    角色可以由不同的管理员并行开发 

###### 检查 Ansible 角色结构

Ansible 角色的功能由目录结构来定义。顶级目录定义角色本身的名称。其中一些子目录包含名为 main.yml 的 YAML 文件。files 和 templates 子目录可以包含由 YAML 文件引用的对象。

以下 tree 命令显示了 user.example 角色的目录结构。

```
$ tree user.example
user.example/
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── README.md
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars
    └── main.yml
```

*    defaults 	此目录中的 main.yml 文件包含角色变量的默认值，使用角色时可以覆盖这些默认值。
*    files 	    此目录包含由角色任务引用的静态文件。
*    handlers 	此目录中的 main.yml 文件包含角色的处理程序定义。
*    meta 	    此目录中的 main.yml 文件包含与角色相关的信息，如作者、许可证、平台和可选的角色依赖项。
*    tasks 	    此目录中的 main.yml 文件包含角色的任务定义。
*    templates 	此目录包含由角色任务引用的 Jinja2 模板。
*    tests 	    此目录可以包含清单和 test.yml playbook，可用于测试角色。
*    vars 	    此目录中的 main.yml 文件定义角色的变量值。 

###### 定义变量和默认值

*    角色变量通过在角色目录层次结构中创建含有 key: value 对的 vars/main.yml 文件来定义。与其他变量一样，这些角色变量在角色 YAML 文件中引用：{{ VAR_NAME }}。这些变量具有较高的优先级，无法被清单变量覆盖。
*    默认变量允许为包含或依赖的角色设置变量的默认值。它们通过在角色目录层次结构中创建含有 key: value 对的 defaults/main.yml 文件来定义。默认变量具有任何可用变量中最低的优先级。它们很容易被包括清单变量在内的任何其他变量覆盖。

在 vars/main.yml 或 defaults/main.yml 中定义具体的变量，但不要在两者中都定义。有意要覆盖变量的值时，应使用默认变量。 


###### 在 playbook 中使用 Ansible 角色

```
---
- hosts: remote.example.com
  roles:
    - role1
    - role2
```

```
---
- hosts: remote.example.com
  roles:
    - role: role1
    - role: role2
      var1: val1
      var2: val2
```

###### 定义角色依赖项

角色依赖项使得角色可以将其他角色作为依赖项包含在 playbook 中。例如，一个定义文档服务器的角色可能依赖于另一个安装和配置 Web 服务器的角色。依赖关系在角色目录层次结构中的 meta/main.yml 文件内定义。

```
---
dependencies:
  - { role: apache, port: 8080 }
  - { role: postgres, dbname: serverlist, admin_user: felix }
```

默认情况下，角色仅作为依赖项添加到 playbook 中一次。若有其他角色也将它作为依赖项列出，它不会再次运行。此行为可以被覆盖，将 meta/main.yml 文件中的 allow_duplicates 变量设置为 yes 即可。 


###### 控制执行顺序

通常而言，角色任务在使用角色的 playbook 的任务之前执行。Ansible 提供了覆盖此默认行为的方式：pre_tasks 和 post_tasks 任务。pre_tasks 任务在应用任何角色之前执行。post_tasks 任务在所有角色完成后执行。

```
---
- hosts: remote.example.com
  pre_tasks:
    - debug:
        msg: 'hello'
  roles:
    - role1
    - role2
  tasks:
    - debug:
        msg: 'still busy'
  post_tasks:
    - debug: 
        msg: 'goodbye'
```


### 创建角色

在 Ansible 中创建角色不需要特别的开发工具。创建和使用角色包含三个步骤：
1. 创建角色目录结构。
2. 定义角色内容。
3. 在 playbook 中使用角色。 

###### 创建角色目录结构

Ansible 在项目目录的 roles 子目录中查找角色。角色也可以保存在由 Ansible 配置文件中 roles_path 变量引用的目录中(此变量包含要搜索的目录的冒号分隔列表)。每个角色具有自己的目录，其包含专门命名的子目录。下列目录结构包含了定义 motd 角色的文件。

```
$ tree roles/
roles/
└── motd
    ├── defaults
    │   └── main.yml
    ├── files
    ├── handlers
    ├── tasks
    │   └── main.yml
    └── templates
        └── motd.j2
```

files 子目录包含固定内容的文件，而 templates 子目录则包含使用时可由角色部署的模板。其他子目录中可以包含 main.yml 文件，它们定义默认的变量值、处理程序、任务、角色元数据或变量，具体取决于所处的子目录。如果某一子目录存在但为空，如上例中的 handlers，它将被忽略。如果角色没有利用功能，其子目录可以一起省略，如上例中的 meta 和 vars 子目录。

###### 定义角色内容

创建了目录结构后，必须定义 Ansible 角色的内容。ROLENAME/tasks/main.yml 文件可以作为一个不错的起点。此文件定义了要在该角色应用的受管主机上调用哪些模块。

下列 tasks/main.yml 文件管理受管主机上的 /etc/motd 文件。它使用 template 模块将名为 motd.j2 的模板复制到受管主机上。此模板从角色的 templates 子目录检索。 

```
$ cat roles/motd/tasks/main.yml
---
# tasks file for motd

- name: deliver motd file
  template:
    src: templates/motd.j2
    dest: /etc/motd
    owner: root
    group: root
    mode: 0444

$ cat roles/motd/templates/motd.j2
This is the system {{ ansible_hostname }}.

Today's date is: {{ ansible_date_time.date }}.

Only use this system with permission.
You can ask {{ system_owner }} for access.

$ cat roles/motd/defaults/main.yml
---
system_owner: user@host.example.com
```

###### 在 playbook 中使用角色

要访问角色，可在 playbook 的 roles: 部分引用它。下列 playbook 引用了 motd 角色。由于没有指定变量，因此该角色将使用其默认变量值应用。

```
$ cat use-motd-role.yml
---
- name: use motd role playbook
  hosts: remote.example.com
  user: devops
  become: true

  roles:
    - motd
```

###### 通过变量更改角色的行为

与参数一样，角色也可搭配变量使用，来覆盖之前定义的默认值。在被引用时，也必须指定 variable: value 对。下例演示了如何将 motd 角色与 system_owner 角色变量的不同值搭配使用。角色应用到受管主机时，指定的值 someone@host.example.com 将取代变量引用。

```
$ cat use-motd-role.yml
---
- name: use motd role playbook
  hosts: remote.example.com
  user: devops
  become: true

  roles:
    - role: motd
      system_owner: someone@host.example.com
```


### 使用 Ansible Galaxy 部署角色

###### Ansible Galaxy

Ansible Galaxy是一个 Ansible 角色公共资源库，这些角色由许许多多 Ansible 管理员和用户编写。它是一个包含数千 Ansible 角色的档案库，具有可搜索的数据库，可帮助 Ansible 用户确定或许有助于他们完成管理任务的角色。Ansible Galaxy 含有面向新的 Ansible 用户和角色开发人员的文档和视频链接。 

###### ansible-galaxy 命令行工具

1. 确定和安装角色

*    ansible-galaxy search 子命令在 Ansible Galaxy 上搜索作为参数指定的字符串。--author、--platforms 和 --galaxy-tags 选项可用于缩小搜索结果范围。下例中显示了其描述中包含 “install” 和 “git” 并可用于企业 Linux (el) 平台的角色的名称。
*    ansible-galaxy info 子命令显示与角色相关的更多详细信息。以下命令显示了 Ansible Galaxy 提供的 davidkarban.git 角色的相关信息。由于该信息需要多个屏幕来显示，因此 ansible-galaxy 使用 less 显示角色的信息。 
*    ansible-galaxy install 子命令从 Ansible Galaxy 下载角色，并将它安装到控制节点本地。角色的默认安装位置是 /etc/ansible/roles。此位置可以通过 role_path 配置变量的值或命令行的 -p DIRECTORY 选项来覆盖。 
*    单个 ansible-galaxy install 命令可以安装多个角色，只需通过 -r 选项指定 YAML 文件，指定要下载和安装的角色。可以使用 name 值来确定角色在本地的名称。

```
$ cat roles2install.yml
# From Galaxy
- src: author.rolename

# From a webserver, where the role is packaged in a gzipped tar archive
- src: https://webserver.example.com/files/sample.tgz
  name: ftpserver-role
$ ansible-galaxy init -r roles2install.yml
```

从 Ansible Galaxy 下载并安装的角色可以和任何其他角色一样在 playbook 中使用。在 roles: 部分中利用其完整的 AUTHOR.NAME 角色名称来加以引用。以下 use-git-role.yml playbook 引用了 davidkarban.git 角色。

```
$ cat use-git-role.yml
---
- name: use davidkarban.git role playbook
  hosts: remote.example.com
  user: devops
  become: true

  roles:
    - davidkarban.git
```

2. 管理下载的模块

*    ansible-galaxy 命令可以管理本地角色。角色可以在当前项目的 roles 目录中找到，也可在 roles_path 变量中列出的其中一个目录中找到。ansible-galaxy list 子命令列出本地找到的角色。 
*    可以使用 ansible-galaxy remove 子命令本地删除角色。

3. 使用 ansible-galaxy 创建角色

ansible-galaxy init 命令为将要开发的新角色创建目录结构。角色的作者和名称作为命令的参数来指定，它将在当前目录中创建目录结构。ansible-galaxy 在执行大部分操作时，与 Ansible Galaxy 网站 API 交互。--offline 选项允许 init 命令在未接通互联网时使用。

```
$ ansible-galaxy init --offline student.example
- student.example was created successfully
$ ls student.example/
defaults  files  handlers  meta  README.md  tasks  templates  tests  vars
```

### Summary

*    角色以一定的方式组织 Ansible 任务，从而实现重复使用和共享。
*    角色变量应当在 defaults/main.yml 中定义，这时它们将用作参数，否则应当在 vars/main.yml 中定义。
*    角色依赖项可以在角色的 meta/main.yml 文件的 dependencies 部分中定义。
*    要在角色之前和之后应用的任务可以通过 pre_tasks 和 post_tasks 任务包含在 playbook 中。
*    Ansible 角色在 playbook 的 roles 部分中加以引用。
*    在 playbook 中使用角色时可以覆盖默认角色变量。
*    Ansible Galaxy是一个 Ansible 角色公共资源库，这些角色由 Ansible 用户编写。
*    ansible-galaxy 命令可以搜索角色，显示角色相关的信息，以及安装、列举、删除或初始化角色。
*    ansible-galaxy init --offline 命令可以创建 Ansible 角色具备的定义良好的目录结构。 
