---
layout: post
title:  "实施 Ansible Vault(407-9)"
categories: Linux
tags: RHCA 407
---


### 配置 Ansible Vault

Ansible 可能需要访问密码或 API 密钥等敏感数据，以便能配置远程服务器。通常，此信息可能以纯文本形式存储在清单变量或其他 Ansible 文件中。但若如此，任何有权访问 Ansible 文件的用户或存储这些 Ansible 文件的版本控制系统都能够访问此敏感数据。这显然存在安全风险。

更加安全地存储此数据主要有两种方式：
1. 使用 Ansible Vault，它随 Ansible 提供，可以加密和解密任何由 Ansible 使用的结构化数据文件。
2. 使用第三方密钥管理服务将数据存储在云端，如 HashiCorp 的 Vault、Amazon 的 AWS Key Management Service 或 Microsoft Azure Key Vault。

> Ansible Vault 并不实施自有的加密函数，而是使用外部 Python 工具集。文件通过利用 AES256 的对称加密（将密码用作机密密钥）加以保护。请注意，这种方式尚未得到第三方正式审核。

###### 创建加密的文件

若要创建新的加密文件，可使用命令 ansible-vault create filename。该命令将提示输入新的 vault 密码，同时利用默认的编辑器打开文件。这是 vim，但在 Ansible 2.1 中可能已改为 vi。可以通过设置和导出 $EDITOR 变量来使用其他编辑器。例如，若要将默认编辑器设为 nano，可设为 export EDITOR=nano。

```
$ ansible-vault create secret.yml
New Vault password: redhat
Confirm New Vault password: redhat
```

除了通过标准输入途径输入 vault 密码外，可以使用 vault 密码文件来存储 vault 密码。此文件需要通过文件权限和其他手段加以严密保护。

```
$ ansible-vault create --vault-password-file=vault-pass secret.yml
```

> 在最近版本的 Ansible 中，用于保护文件的密文是 AES256，但利用较旧版本加密的文件可能依然使用 128 位 AES。

###### 编辑现有的加密文件。

要编辑现有的加密文件，Ansible Vault 提供了 ansible-vault edit filename 命令。此命令将文件解密为一个临时文件，并允许您编辑该文件。保存时，它将复制其内容并删除临时文件。

```
$ ansible-vault edit secret.yml
Vault password: redhat
```

> edit 子命令始终重写文件，因此仅可在进行更改时使用它。这在文件保管在版本控制下时有影响。若要查看文件的内容而不进行更改，始终应使用 view 子命令。

###### 更改加密文件的密码。

可以使用命令 ansible-vault rekey filename 更改 vault 密码。此命令可一次性更新多个数据文件的密钥。它将要求提供原始密码和新密码。

```
$ ansible-vault rekey secret.yml 
Vault password: redhat
New Vault password: RedHat
Confirm New Vault password: RedHat
Rekey successful
```

在使用 vault 密码文件时，请使用 --new-vault-password-file 选项：

```
$ ansible-vault rekey --new-vault-password-file=NEW_VAULT_PASSWORD_FILE secret.yml
```

###### 加密现有的文件

要加密已存在的文件，请使用 ansible-vault encrypt filename 命令。此命令可取多个欲加密文件的名称作为参数。使用 --output=OUTPUT_FILE 选项，可将加密文件保存为新的名称。最多只能将一个输入文件用于 --output 选项。

```
$ ansible-vault encrypt secret1.yml secret2.yml 
New Vault password: redhat
Confirm New Vault password: redhat
Encryption successful
```

###### 查看加密的文件

Ansible Vault 允许您使用 ansible-vault view filename 命令查看加密的文件，而不必打开它进行编辑。

```
$ ansible-vault view secret1.yml 
Vault password: secret
less 458 (POSIX regular expressions)
Copyright (C) 1984-2012 Mark Nudelman

less comes with NO WARRANTY, to the extent permitted by law.
For information about the terms of redistribution,
see the file named README in the less distribution.
Homepage: http://www.greenwoodsoftware.com/less
my_secret: "yJJvPqhsiusmmPPZdnjndkdnYNDjdj782meUZcw"
```

###### 解密现有的文件

已存在的加密文件可以通过 ansible-vault decrypt filename 命令永久解密。在解密单个文件时，可使用 --output 选项以其他名称保存解密的文件。

```
$ ansible-vault decrypt secret1.yml --output=secret1-decrypted.yml
Vault password: redhat
Decryption successful
```


### 利用 Ansible Vault 执行

为了运行通过 Ansible Vault 加密的 playbook，需要向 ansible-playbook 命令提供其加密密码。如果运行命令时没有这么做，它将返回错误：

```
$ ansible-playbook site.yml
ERROR: A vault password must be specified to decrypt vars/api_key.yml
```

若要以交互方式提供 vault 密码，可使用 --ask-vault-pass 选项。

```
$ ansible-playbook --ask-vault-pass site.yml
Vault password: redhat
```

也可利用 --vault-password-file 选项进行指定，来使用以纯文本存储加密密码的文件。密码应当在该文件中存储为一行字符串。由于该文件包含敏感的纯文本密码，因此务必要通过文件权限和其他安全措施对其加以保护。密码文件的默认位置可使用 $ANSIBLE_VAULT_PASSWORD_FILE 环境变量来指定。

```
$ ansible-playbook --vault-password-file=vault-pw-file site.yml
```

> 一个 playbook 使用的、设有 Ansible Vault 保护的所有文件必须使用同一密码进行加密。 

###### 变量文件管理的推荐做法

若要简化管理，务必要设置您的 Ansible 项目，使敏感变量和所有其他变量保存在相互独立的文件中。然后，包含敏感变量的一个或多个文件可通过 ansible-vault 命令进行保护。

请记住，管理组变量和主机变量的首选方式是在 playbook 级别上创建目录。group_vars 目录通常包含名称与它们所应用的主机组匹配的变量文件。host_vars 目录通常包含名称与它们所应用的受管主机名称匹配的变量文件。

不过，除了使用 group_vars 或 host_vars 中的文件外，您也可对每一主机组或受管主机使用多个目录。这些目录而后可包含多个变量文件，它们都由该主机组或受管主机使用。例如，在 playbook.yml 的以下项目目录中，webservers 主机组的成员将使用 group_vars/webservers/vars 文件中的变量，而 demo.example.com 将使用 host_vars/demo.example.com/vars 和 host_vars/demo.example.com/vault 中的变量：

```
.
├── ansible.cfg
├── group_vars
│   └── webservers
│       └── vars
├── host_vars
│   └── demo.example.com
│       ├── vars
│       └── vault
├── inventory
└── playbook.yml
```

在这种情景中，其好处在于用于 demo.example.com 的大部分变量可以放在 vars 中，敏感变量则可放在 vault 中。然后，管理员可以使用 ansible-vault 加密 vault，而将 vars 保留为纯文本。

在本例中，host_vars/demo.example.com 目录内使用的文件名没有什么特别之处。该目录可以包含更多文件，一些由 Ansible Vault 加密，另一些则不加密。

Playbook 变量（与清单变量相对）也可通过 Ansible Vault 保护。敏感的 playbook 变量可以放在单独的文件中，此文件通过 Ansible Vault 加密，并通过 vars_files 指令包含在该 playbook 中。这很有用处，因为 playbook 变量的优先级高于清单变量。 

###### 加快 Vault 运算速度

默认情况下，Ansible 使用 python-crypto 软件包中的函数来加密和解密 vault 文件。如果有许多加密文件，则在启动时解密它们可能会导致明显的延迟。若要加快速度，可安装 python-cryptography 软件包


### Summary

*    Ansible Vault是保护使用 Ansible playbook 进行部署时所用的密码散列和私钥等敏感数据的一种方式。
*    Ansible Vault 可以使用对称加密（通常是 AES-256）来加密和解密由 Ansible 使用的任何结构化数据文件
*    Ansible Vault 可用于创建并加密文本文件（若尚未存在），或者加密和解密现有的文件。
*    由一个 playbook 使用的所有 Vault 文件需要使用相同的密码
*    建议用户将大部分变量保存在一个普通文件中，并将敏感的变量保存在由 Ansible Vault 保护的第二文件中
*    通过安装 python-cryptography 软件包，可以加快红帽企业 Linux 或 CentOS 上 Ansible Vault 操作的速度。
