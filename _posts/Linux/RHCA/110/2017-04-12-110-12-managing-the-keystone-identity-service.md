---
layout: post
title:  "管理 Keystone 身份服务(110-12)"
categories: Linux
tags: RHCA 110
---

### 运行 OpenStack 统一命令行界面

Horizon 为云用户和管理员提供了用户友好的环境，但一些任务需要更大的灵活性（如脚本功能）来支持操作。Red Hat OpenStack Platform 提供了一套 CLI 工具，实施所有通过 OpenStack 服务 API 提供的功能。

在过去的 Red Hat OpenStack Platform 版本中，这一 CLI 工具集由一组客户端工具组成，分别用于各项 OpenStack 服务（例如，keystone 适用于 Keystone 身份服务）；不过，最近 OpenStack CLI 在 Red Hat OpenStack Platform 中采用一种新方法，即统一 CLI，它基于 openstack 命令，旨在简化 Red Hat OpenStack Platform 环境中的操作。

###### Keystone 凭据

与 Horizon 要求用户名和密码一样，OpenStack CLI 工具集也要求用户验证身份后才能被授权使用 Red Hat OpenStack Platform 环境中的各种服务。为通过身份验证，用户需要（至少）指定下表所述的参数。参数的指定可以通过使用环境变量，或者在执行 openstack 命令时加上标志。

Keystone 身份验证参数

*    参数 	        描述 	                    环境变量 	            openstack命令标志
*    用户 	        用户的用户名 	                OS_USERNAME 	    --os-username
*    密码 	        用户的密码 	                OS_PASSWORD 	    --os-password
*    项目 	        用户所属的项目 	            OS_PROJECT_NAME 	--os-project-name
*    Keystone 端点 	每个项目允许的固定 IP 地址数 	OS_AUTH_URL 	    --os-auth-url

> OS_AUTH_URL 环境变量中包含的端点对应于 Keystone 服务的公共端点，其默认绑定到 5000 端口。 

如果通过环境变量指定这些身份验证参数，最佳的做法是为各组用户身份验证参数创建一个 keystone_<username> 文件。例如，如果名为 testuser 的用户属于 mytestproject 租户，密码为 redhat，并且其 Keystone 端点位于 servera 计算机，则该用户将关联有下列 keystonerc 文件。

```
export OS_USERNAME=testuser
export OS_PASSWORD=redhat
export OS_PROJECT_NAME=mytestproject
export OS_AUTH_URL=servera:5000/v2.0
```

身份验证参数也可以在执行 openstack 命令时指定，这时要借助使用前面表中详述的标志。对于上例，若要列出 Glance（OpenStack 映像服务）中可用的映像，应当发出下列命令。

```
[root@servera ~]# openstack --os-username testuser --os-password redhat --os-project-name mytestproject --os-auth-url servera:5000/v2.0 image list
```
    
> 默认情况下，PackStack 在 /root/ 目录下创建 keystonerc_admin 文件，该文件中包含所部署 Red Hat OpenStack Platform 环境的管理员用户的身份验证参数。 

###### 统一 CLI

统一 CLI 旨在提供一种更加便捷的方式，让用户通过使用唯一命令 openstack 与各种 Red Hat OpenStack Platform 服务交互。此命令提供一系列用于和 Red Hat OpenStack Platform 服务交互的子命令，也提供一些自定义其输出的标志。下表列出了其中一些子命令，您可以通过以下格式使用这些命令。

```
[root@servera ~]# openstack <flags> <subcommand> <option>
```

统一 CLI 子命令

*    子命令 	    描述
*    用户 	    用户管理
*    project 	项目管理
*    server 	实例管理
*    映像 	    映像管理
*    volume 	块存储管理
*    network 	网络管理

如需以上各个子命令的用法和选项的其他帮助，以及通过 openstack 命令提供的其他子命令的帮助，您可以按照以下格式使用 help 子命令来获取。

```
[root@servera ~]# openstack help <subcommand> <option>
```

###### 运行 OpenStack 统一命令行界面

1. 登录 servera 计算机，再提供 keystonerc_admin 文件以获取 Red Hat OpenStack Platform 环境中的管理员权限。

    [root@servera ~]# source /root/keystonerc_admin
                
2. 检查 /root/keystonerc_admin 文件的内容。该文件包含 admin 用户的凭据，包括多个不同变量：OS_USERNAME 变量指定其用户名，即 admin；OS_PASSWORD 变量指定其密码，即 redhat；OS_TENANT_NAME 变量指定其项目名称，即 admin；OS_AUTH_URL 变量则指定 Keystone 端点 URI。

    [root@servera ~(keystone_admin)]# cat /root/keystonerc_admin
    ...
    export OS_USERNAME=admin
    export OS_PASSWORD=redhat
    export OS_AUTH_URL=http://172.25.250.10:5000/v2.0
    ...
    export OS_TENANT_NAME=admin
    ...
            
3.  使用 help 和 user 子命令，获取关于 openstack 命令中提供的用户管理选项的信息。

    [root@servera ~(keystone_admin)]# openstack help user
    Command "user" matches:
      user role list
      user show
      user set
      user delete
      user create
      user list

4. 使用 openstack 命令的 user list 子命令，列出 PackStack 在 Keystone 中配置的用户。

    [root@servera ~(keystone_admin)]# openstack user list
    +----------------------------------+---------+
    | ID                               | Name    |
    +----------------------------------+---------+
    | 0d4f0e46f73e47f8bb11f299a6e4e932 | glance  |
    | 232ad8758b6c4081b29229490f86a538 | admin   |
    | 28d66c92711f43e2a3e0f3d32ee22263 | neutron |
    | b02f4c6bd8c543e38a7caf15c640d627 | cinder  |
    | c1372daa6c434371899229a2edf1a1b4 | nova    |
    +----------------------------------+---------+

5. 使用 help 和 project 子命令，获取关于 openstack 命令中提供的项目管理选项的信息。

    [root@servera ~(keystone_admin)]# openstack help project
    Command "project" matches:
      project delete
      project list
      project set
      project show
      project create
      project usage list

6. 使用 openstack 命令的 project list 子命令，列出 PackStack 在 Keystone 中配置的项目。

    [root@servera ~(keystone_admin)]# openstack project list
    +----------------------------------+----------+
    | ID                               | Name     |
    +----------------------------------+----------+
    | 2cb5f3156d4243d19335056a36691666 | admin    |
    | 631eefd2a92d49c9a042014617aeb373 | services |
    +----------------------------------+----------+


### 使用命令行界面管理项目

在 Red Hat OpenStack Platform 环境中，用户可以分组到项目中，所以多个用户可以是同一项目的成员，一个用户也可以成为多个项目的成员。这些项目可以映射到用户所属的组织。项目通常分配有一些资源，如存储、计算和网络资源等，它们供该项目所关联的用户使用。借助 project 子命令，openstack 命令提供项目管理功能，具体如下表中所述。

使用 openstack 命令管理项目

*    子命令 	            描述
*    project create 	创建新项目
*    project delete 	删除项目
*    project list 	    列出项目
*    project show 	    显示项目详细信息
*    project set 	    设置项目属性（如，启用/禁用）

> 在以前的 Red Hat OpenStack Platform 版本中，项目曾被称为租户，所以命令和文档中应该也会提到租户。 

默认情况下，项目具有关联的资源，它们通过配额加以限制。这些配额由不同的 Red Hat OpenStack Platform 服务在它们管理的资源中实施。例如，计算资源由 Nova 服务管理，并且具有多种可配置的配额与之关联，其中包括实例数量（实例配额）、核心数（核心数配额）和内存大小（ram 配额）。

所有 Red Hat OpenStack Platform 服务在部署时都对这些配额使用默认值，但可以在全局范围或根据项目修改这些默认值，让管理员能够控制项目及其用户可以使用的资源数量。openstack 命令目前不提供配额管理功能，但它们依然可以通过旧版 OpenStack 服务 CLI 进行管理，例如：nova 命令可以管理计算配额，cinder 命令可管理块存储配额，而 neutron 命令则可管理网络相关的配额。举例而言，nova 命令包含了用于管理计算相关配额的多个子命令，具体如下表所述。

使用 nova 命令管理项目配额

*    子命令 	描述
*    quota-show 	列出项目的配额
*    quota-update 	更新项目的配额
*    quota-delete 	删除项目的配额
*    quota-defaults 	列出项目的默认配额

> 关于 nova quota-* 命令的其他信息可通过运行 nova help 命令来了解。 

###### 使用命令行界面管理项目

1. 提供 keystonerc_admin，以获取 Red Hat OpenStack Platform 环境中的管理员权限。这将启用所有 OS_* 环境变量，其中至少包含管理员凭据和 Keystone 端点。

    [root@servera ~]# source /root/keystonerc_admin
    [root@servera ~(keystone_admin)]# 

2. 使用 project create 子命令，创建一个名为 demoproject 的新项目。

    [root@servera ~(keystone_admin)]# openstack project create demoproject
    +-------------+----------------------------------+
    | Field       | Value                            |
    +-------------+----------------------------------+
    | description | None                             |
    | enabled     | True                             |
    | id          | 8399d7c7ef664158abf697b6591cf258 |
    | name        | demoproject                      |
    +-------------+----------------------------------+

3. 使用 project show 子命令，检查 demoproject 项目的详细信息。

    [root@servera ~(keystone_admin)]# openstack project show demoproject
    +-------------+----------------------------------+
    | Field       | Value                            |
    +-------------+----------------------------------+
    | description | None                             |
    | enabled     | True                             |
    | id          | 8399d7c7ef664158abf697b6591cf258 |
    | name        | demoproject                      |
    +-------------+----------------------------------+
          
4. 列出 Red Hat OpenStack Platform 环境中当前可用的项目。新项目 demoproject 应当会列在其中。

    [root@servera ~(keystone_admin)]# openstack project list
    +----------------------------------+-------------+
    | ID                               | Name        |
    +----------------------------------+-------------+
    | 2cb5f3156d4243d19335056a36691666 | admin       |
    | 631eefd2a92d49c9a042014617aeb373 | services    |
    | 8399d7c7ef664158abf697b6591cf258 | demoproject |
    +----------------------------------+-------------+
            
5. 使用 help 选项，检查 nova 命令可用于配额管理的选项。此处使用旧版 Nova CLI，因为 openstack 命令中缺少配额管理支持。

    [root@servera ~(keystone_admin)]# nova help | grep quota
    quota-class-show            List the quotas for a quota class.
    quota-class-update          Update the quotas for a quota class.
    quota-defaults              List the default quotas for a tenant.
    quota-delete                Delete quota for a tenant/user so their quota
    quota-show                  List the quotas for a tenant/user.
    quota-update                Update the quotas for a tenant/user.

6. 也使用 help 选项，检查 cinder 命令可用于配额管理的选项。

    [root@servera ~(keystone_admin)]# cinder help | grep quota
    quota-class-show    Lists quotas for a quota class.
    quota-class-update  Updates quotas for a quota class.
    quota-defaults      Lists default quotas for a tenant.
    quota-delete        Delete the quotas for a tenant.
    quota-show          Lists quotas for a tenant.
    quota-update        Updates quotas for a tenant.
    quota-usage         Lists quota usage for a tenant.
            

7. 使用 quota-show 选项，获取关于如何使用 nova 命令显示 demoproject 项目当前配额的信息。

    [root@servera ~(keystone_admin)]# nova help quota-show
    usage: nova quota-show [--tenant <tenant-id>] [--user <user-id>]

    List the quotas for a tenant/user.

    Optional arguments:
      --tenant <tenant-id>  ID of tenant to list the quotas for.
      --user <user-id>      ID of user to list the quotas for.

8. 显示 demoproject 项目当前的配额。

    [root@servera ~(keystone_admin)]# nova quota-show --tenant demoproject
    +-----------------------------+-------+
    | Quota                       | Limit |
    +-----------------------------+-------+
    | instances                   | 10    |
    | cores                       | 20    |
    | ram                         | 51200 |
    | floating_ips                | 10    |
    | fixed_ips                   | -1    |
    | metadata_items              | 128   |
    | injected_files              | 5     |
    | injected_file_content_bytes | 10240 |
    | injected_file_path_bytes    | 255   |
    | key_pairs                   | 100   |
    | security_groups             | 10    |
    | security_group_rules        | 20    |
    | server_groups               | 10    |
    | server_group_members        | 10    |
    +-----------------------------+-------+

9. 使用 nova 命令及 quota-update 选项，将 demoproject 项目的核心数配额更新为 30。

    [root@servera ~(keystone_admin)]# nova quota-update --cores 30 demoproject
              
10. 再次显示 demoproject 项目当前的配额。核心数配额现在应当显示为 30。

    [root@servera ~(keystone_admin)]# nova quota-show --tenant demoproject
    +-----------------------------+-------+
    | Quota                       | Limit |
    +-----------------------------+-------+
    | instances                   | 10    |
    | cores                       | 30    |
    | ram                         | 51200 |
    | floating_ips                | 10    |
    | fixed_ips                   | -1    |
    | metadata_items              | 128   |
    | injected_files              | 5     |
    | injected_file_content_bytes | 10240 |
    | injected_file_path_bytes    | 255   |
    | key_pairs                   | 100   |
    | security_groups             | 10    |
    | security_group_rules        | 20    |
    | server_groups               | 10    |
    | server_group_members        | 10    |
    +-----------------------------+-------+

11. 使用 project delete 子命令，删除 demoproject 项目。

    [root@servera ~(keystone_admin)]# openstack project delete demoproject


### 使用命令行界面管理用户

Keystone 要求 Red Hat OpenStack Platform 环境用户通过它进行身份验证，然后才能与其余的服务交互。一旦有新的用户需要使用 Red Hat OpenStack Platform 环境，管理员必须为其创建帐户，并提供此用户的凭据，包括用户名、密码，以及将部署其资源的项目。该用户必须要使用这些凭据，或者通过 OS_* 环境变量，或者借助 openstack 命令中相关的标志。借助 user 子命令，openstack 命令提供用户管理功能，具体如下表中所述。

*    使用 openstack 命令管理用户
*    子命令 	            描述
*    user create 	    创建新用户
*    user delete 	    删除用户
*    user list 	        列出用户
*    user show 	        显示用户详细信息
*    user set 	        设置用户属性（如密码）
*    user role list 	列出用户角色分配

一个用户可以同时从属于多个项目，多个用户可以属于同一项目。在每个租户中，用户可以和一个角色关联。默认情况下，PackStack 部署两个角色，它们目前受到 Red Hat OpenStack Platform 服务的支持：_member_ 角色，它为关联了此角色的用户提供非管理权限；以及 admin 角色，它为该用户启用管理权限。角色在项目中分配到用户，所以一个用户在不同的项目中可以具有不同的角色。

> Red Hat OpenStack Platform 超级用户 admin 关联了 admin 项目中的 admin 角色。

借助 role 子命令，openstack 命令提供角色管理功能，具体如下表中所述。

*    使用 openstack 命令管理角色
*    子命令 	        描述
*    role create 	创建新角色
*    role delete 	删除角色
*    role list 	    列出角色
*    role show 	    显示角色详细信息
*    role add 	    添加用户在项目中的角色
*    role remove 	删除用户在项目中的角色

###### 使用命令行界面管理用户

1. 提供 keystonerc_admin 文件，以获取 Red Hat OpenStack Platform 环境中的管理员权限。这将启用所有 OS_* 环境变量，其中至少包含管理员凭据和 Keystone 端点。

    [root@servera ~]# source /root/keystonerc_admin
    [root@servera ~(keystone_admin)]# 

2. 获取关于如何使用 openstack 命令的 users 子命令在 Keystone 中管理用户的信息。

    [root@servera ~(keystone_admin)]# openstack help user
    Command "user" matches:
      user role list
      user show
      user set
      user delete
      user create
      user list

3. 使用 user create 子命令新建一个用户，命名为 demouser，并将密码设为 redhat。

    [root@servera ~(keystone_admin)]# openstack user create --password redhat demouser
    +----------+----------------------------------+
    | Field    | Value                            |
    +----------+----------------------------------+
    | email    | None                             |
    | enabled  | True                             |
    | id       | 8f14163a8c2d4e39909aa3222c13e5e9 |
    | name     | demouser                         |
    | username | demouser                         |
    +----------+----------------------------------+

4. 使用 user set 子命令，将 demouser 的电子邮件设置为 demouser@example.com。

    [root@servera ~(keystone_admin)]# openstack user set --email demouser@example.com demouser

5. 使用 user list 子命令，列出 Red Hat OpenStack Platform 环境中配置的用户。

    [root@servera ~(keystone_admin)]# openstack user list
    +----------------------------------+----------+
    | ID                               | Name     |
    +----------------------------------+----------+
    ...
    | bc863a8c87014876817d2aa7356a1ad0 | demouser |
    ...
    +----------------------------------+----------+

6. 使用 help 和 role 选项，获取关于如何使用 openstack 命令管理角色的信息。用户在项目中的角色分配使得该用户能够执行特定的操作，具体根据该角色所提供的访问权限。

    [root@servera ~(keystone_admin)]# openstack help role
    Command "role" matches:
      role remove
      role create
      role delete
      role add
      role list
      role show

7. 使用 help 和 role add 选项，获取关于如何使用 openstack 命令创建角色并添加给项目内的用户的信息。

```
    [root@servera ~(keystone_admin)]# openstack help role add
    usage: openstack role add [-h] [-f {shell,table,value}] [-c COLUMN]
                              [--max-width <integer>] [--prefix PREFIX] --project
                              <project> --user <user>
                              <role>

    Add role to project:user

    positional arguments:
      <role>                Role to add to <project>:<user> (name or ID)
    ...
```

8. 使用 role add 子命令，为 demoproject 项目的 demouser 用户添加 admin 角色。

    [root@servera ~(keystone_admin)]# openstack role add --user demouser --project demoproject admin
    +-------+----------------------------------+
    | Field | Value                            |
    +-------+----------------------------------+
    | id    | a7b6afc1a27741b6939383d18d9c1ebf |
    | name  | admin                            |
    +-------+----------------------------------+

9. 使用 user role list 子命令，检查 demouser 用户在 demoproject 项目中的 admin 角色是否已正确启用。

    [root@servera ~(keystone_admin)]# openstack user role list --project demoproject demouser
    +----------------------------------+-------+-------------+----------+
    | ID                               | Name  | Project     | User     |
    +----------------------------------+-------+-------------+----------+
    | a7b6afc1a27741b6939383d18d9c1ebf | admin | demoproject | demouser |
    +----------------------------------+-------+-------------+----------+
            
10. 使用 role remove 子命令，为 demoproject 项目的 demouser 用户移除 admin 角色。

    [root@servera ~(keystone_admin)]# openstack role remove --user demouser --project demoproject admin
              
11. 使用 user delete 子命令，移除 demouser 用户。

    [root@servera ~(keystone_admin)]# openstack user delete demouser
              
12. 使用 project delete 子命令，删除 demoproject 项目。

    [root@servera ~(keystone_admin)]# openstack project delete demoproject

### 总结

在本章中，您可以学到：

*    统一 CLI 提供了更加简便的方式，用户可以在单一工具中管理所有 Red Hat OpenStack Platform 服务，免除了必须针对每项 Red Hat OpenStack Platform 服务操作一款 CLI 工具的复杂性。
*    Keystone 服务支持通过三个元素（即用户、项目和角色）进行组织行为映射，以满足身份验证的需要。|一个用户可以是一个或多个项目的成员，多个用户可以是一个项目的成员，一个用户也可在不同项目中与不同的角色关联。
*    配额为限制分配的资源提供了方式。
