---
layout: post
title:  "Configuring Network Encryption(236-15)"
categories: Linux
tags: RHCA 236
---

### 启用管理和 I/O 加密

###### 红帽 Gluster 存储中的网络加密

红帽 Gluster 存储使用 TLS/SSL 支持网络加密。红帽 Gluster 存储用 TLS/SSL 取代一般条件下所用的本地开发身份验证框架，来进行授权和身份验证。务必要清楚的是，TLS/SSL 提供身份验证和加密，但不提供授权。Glusterfs 使用 TLS 验证的身份来授权尝试连接 brick 和卷的客户端。身份验证涉及系统或个人将自己的身份信息提供给第二个系统或个人，而授权则检查该系统或个人是否有权执行操作。在红帽 Gluster 存储范围内，了解 TLS/SSL 的实施与了解 TLS/SSL 概念及其在红帽 Gluster 存储中的应用直接相关。

######### 红帽 Gluster 存储中的 TLS/SSL

Gluster 客户端通过分享含有可证明客户端身份的信息的证书，向服务器节点验证自己的身份。客户端需要与其证书匹配的私钥。私钥保持私密性，不被他人看到或访问。如果私钥遭泄露，则持有该密钥的任何人都能声称拥有其身份，因为证书已经公开。每一证书含有证书认证机构或 CA，它验证证书内容的完整性，以及主体 (principal) 或所有者的身份。如果使用自签名证书，则主体和 CA 相同。如果不使用自签名证书，则 CA 通过追加来自 CA 私钥和证书内容的信息，对证书进行签名。

*    类型 	        描述
*    I/O 加密 	    加密受信存储池中客户端和服务器之间 I/O 连接密。
*    管理加密        加密受信存储池中的管理 glusterd 连接。

> 注意: 使用红帽 Gluster 存储 3.1.2 时，所有服务器和客户端必须包含 glusterfs.pem、glusterfs.key 和 glusterfs.ca 文件。

下表列出了网络加密配置过程中使用的文件：

*    /etc/ssl/glusterfs.pem         系统唯一签名的 TLS 包含在此证书文件中。此文件对所有系统唯一，不得与他人共享。
*    /etc/ssl/glusterfs.key         系统的唯一私钥包含在此文件中。此文件不得与他人共享。
*    /etc/ssl/glusterfs.ca          为证书签名的证书认证机构包含在此文件中。此文件不唯一，并且应当在受信存储池中所有服务器上完全相同。客户端也具有此文件，但客户端上 glusterfs.ca 的内容和服务器上 glusterfs.ca 的内容不同。红帽 Gluster 存储不使用系统的默认证书。所有服务器和客户端的签名 CA 的证书包含在服务器上的 glusterfs.ca 文件中。但是，所有服务器的签名 CA 的证书包含在客户端上的 glusterfs.ca 文件中。如果使用自签名证书，则服务器上的 glusterfs.ca 文件是所有服务器和客户端上证书文件 /etc/ssl/glusterfs.pem 的串联。客户端的 /etc/ssl/glusterfs.ca 文件是所有服务器的 /etc/ssl/glusterfs.pem 文件的串联。
*    /var/lib/glusterd/secure-access    对所有服务器之间的管理 glusterd 连接和客户端之间的连接启用加密。此文件供 glusterd 用于获取卷文件，并且使用更改更新客户端。所有服务器和客户端均需要 glusterd 目录和 secure-access 文件，才能启用管理加密。 

###### 为卷启用 I/O 加密的前提条件

若要为卷配置 I/O 加密，请执行下列步骤： 

1. 生成私钥 glusterfs.key

    [root@demo-a ~]$ openssl genrsa -out /etc/ssl/glusterfs.key 2048

2. 如果有当前有权访问受信存储池的客户端，请将 /etc/ssl/glusterfs.ca 复制到新客户端。
3. 使用生成的私钥创建签名证书。

    [root@demo-a ~]$ openssl req -new -x509 -key /etc/ssl/glusterfs.key -subj "/CN=hostname" -out /etc/ssl/glusterfs.pem

4. 如果有通用 CA，可用它来为证书签名。通过运行以下命令，将生成的 glusterfs.csr 文件提供给 CA，CA 会返回 PEM 文件。将 PEM 文件放入 /etc/ssl。

    [root@demo-a ~]$ openssl req -new -sha256 -key /etc/ssl/glusterfs.key -subj '/CN=hostname' -out glusterfs.csr

###### 自签名证书

也可以使用自签名证书，但需要更多维护并且扩充当前的资料。
*    服务器上的自签名 CA 证书: 从所有服务器和客户端收集 /etc/ssl/glusterfs.pem 证书，并将它们串联为名为 /etc/ssl/glusterfs.ca 的单个文件。将 /etc/ssl/glusterfs.ca 文件传播到受信服务器池中的所有服务器上。
*    客户端上的自签名 CA 证书: 从所有服务器收集 /etc/ssl/glusterfs.pem 证书，并将它们串联为名为 /etc/ssl/glusterfs.ca 的单个文件。将 /etc/ssl/glusterfs.ca 文件传播到所有客户端。             
          
###### 启用管理加密

虽然红帽 Gluster 存储可以单独配置 I/O 加密，我们也建议启用管理加密。如果是红帽 Gluster 存储的新安装，则在所有服务器上执行下列操作：

> 注意: 如果是现有的红帽 Gluster 存储池，则卸载卷，停止该卷，并且停止 glusterd。运行 pkill glusterd 命令，然后执行其他步骤。 

1. 创建 /var/lib/glusterd/secure-access 文件。

[root@demo-a ~]# touch /var/lib/glusterd/secure-access

2. 在所有服务器上启动 glusterd。

[root@demo-a ~]# systemctl start glusterd

如果是新安装，请在所有客户端上执行以下操作： 

1. 创建目录 /var/lib/glusterd。

    [root@demo-a ~]# mkdir -p /var/lib/glusterd

2. 创建 /var/lib/glusterd/secure-access 文件。

    [root@demo-a ~]# touch /var/lib/glusterd/secure-access

3. 将卷挂载到所有客户端上。例如，如果使用原生 Fuse 客户端，可使用以下命令。

    [root@demo-a ~]# mount -t glusterfs servera:/demo-vol  /demo-vol
              
###### 为卷启用 I/O 加密

如果是红帽 Gluster 存储的新安装，请执行下列操作：

> 注意: 如果是现有的红帽 Gluster 存储池，则卸载卷并停止该卷，然后执行其他步骤。 

1. 创建卷，但不要启动它。
2. 设置要访问该卷的所有服务器和客户端的通用名称列表。

[root@demo-a ~]$ gluster volume set VOLNAME  \
> auth.ssl-allow 'server1,server2,client1,client2

3. 启动 server.ssl 和 client.ssl。

[root@demo-a ~]# gluster volume set VOLNAME client.ssl on
[root@demo-a ~]# gluster volume set VOLNAME client.ssl on

4. 启动卷。

    [root@demo-a ~]# gluster volume start VOLNAME

5. 将卷挂载到所有授权的客户端上。例如，如果使用原生 fuse 客户端，可使用以下命令：

    [root@demo-a ~]# mount -t glusterfs servera:/VOLNAME /mountpoint 


### 引导式练习：为卷启用 I/O 加密

为卷 prod-vol 配置 I/O 加密。

1. 验证为 prod-vol 配置的选项。

    a. 检查卷 prod-vol 的属性。

    [root@servera ~]# gluster volume info prod-vol
    Volume Name: prod-vol
    Type: Replicate
    Volume ID: 269f7040-2194-48ef-b7e4-767b8d86e161
    Status: Created
    Number of Bricks: 1 x 2 = 2
    Transport-type: tcp
    Bricks:
    Brick1: servera.podX.example.com:/bricks/brick-a1/brick
    Brick2: serverb.podX.example.com:/bricks/brick-b1/brick
    Options Reconfigured:
    performance.readdir-ahead: on

2. 停止卷 prod-vol。

    [root@servera ~]# gluster volume stop prod-vol
    Stopping volume will make its data inaccessible. Do you want to
    continue? (y/n) y
    volume stop: prod-vol: success

3. 从 ftp://workstation.lab.example.com/pub，将 servera 和 serverb 的服务器密钥、证书和证书认证机构的证书上传到 /etc/ssl。

    a. 从 servera，利用命令 curl 将服务器的密钥、证书以及证书认证机构的证书下载到 /etc/ssl。

    [root@servera ~]# curl ftp://workstation.lab.example.com/pub/servera.pem -o /etc/ssl/glusterfs.pem
    [root@servera ~]# curl ftp://workstation.lab.example.com/pub/servera.key -o /etc/ssl/glusterfs.key
    [root@servera ~]# curl ftp://workstation.lab.example.com/pub/glusterfs.ca -o /etc/ssl/glusterfs.ca

    b. 从 serverb，利用命令 curl 将服务器的密钥、证书以及证书认证机构的证书下载到 /etc/ssl。

    [root@serverb ~]# curl ftp://workstation.lab.example.com/pub/serverb.pem -o /etc/ssl/glusterfs.pem
    [root@serverb ~]# curl ftp://workstation.lab.example.com/pub/serverb.key -o /etc/ssl/glusterfs.key
    [root@serverb ~]# curl ftp://workstation.lab.example.com/pub/glusterfs.ca -o /etc/ssl/glusterfs.ca

4. 配置允许访问 prod-vol 的服务器和客户端。

    a. 允许 servera、serverb 和 workstation 安全访问受信存储池。

    [root@servera ~]# gluster volume set prod-vol auth.ssl-allow \
    > 'servera.lab.example.com,serverb.lab.example.com,workstation.lab.example.com'

5. 为卷 prod-vol 启用 SSL。

    a. 从 servera，为 prod-vol 启用服务器 SSL。

    [root@servera ~]# gluster volume set prod-vol server.ssl on

    b. 从 servera，为 prod-vol 启用客户端 SSL。

    [root@servera ~]# gluster volume set prod-vol client.ssl on

6. 为客户端 workstation 启用管理加密。

    a. 在 workstation上创建目录 /var/lib/glusterd 和文件 /var/lib/glusterd/secure-access。

    [root@workstation ~]# mkdir -p /var/lib/glusterd
    [root@workstation ~]# touch /var/lib/glusterd/secure-access

7. 为服务器 servera 和 serverb 启用管理加密。

    a. 停止 servera 和 serverb 上的 glusterd 服务。

    [root@servera ~]# for I in server{a..b};do ssh ${I} "systemctl stop glusterd";done

    b. 中断 servera 和 serverb 上的 glusterfs 进程。

    [root@servera ~]# for I in server{a..b};do ssh ${I} "pkill glusterfs";done

    c. 在 servera 和 serverb 上创建 /var/lib/glusterd/secure-access 文件。

    [root@servera ~]# for I in server{a..b};do ssh ${I} "touch /var/lib/glusterd/secure-access";done

    d. 在 servera 和 serverb 上启动 glusterd。

    [root@servera ~]# for I in server{a..b};do ssh ${I} "systemctl start glusterd";done

    e. 启动 prod-vol。

    [root@servera ~]# gluster volume start prod-vol

    f. 验证为 prod-vol 配置的选项。

        检查卷 prod-vol 的属性。

        [root@servera ~]# gluster volume info prod-vol
        Volume Name: prod-vol
        Type: Replicate
        Volume ID: 269f7040-2194-48ef-b7e4-767b8d86e161
        Status: Created
        Number of Bricks: 1 x 2 = 2
        Transport-type: tcp
        Bricks:
        Brick1: servera:/bricks/brick-a1/brick
        Brick2: serverb:/bricks/brick-b1/brick
        Options Reconfigured:
        performance.readdir-ahead: on
        auth.ssl-allow: servera.lab.example.com,serverb.lab.example.com,workstation.lab.example.com
        server.ssl: on
        client.ssl: on

    g. 在 workstation 上安装 glusterfs-fuse 软件包。

    [root@workstation ~]# yum -y install glusterfs-fuse

    h. 使用原生 gluster 客户端，将 prod-vol 挂载到 workstation 上的 /mnt 目录。

    [root@workstation ~]# mount -t glusterfs servera:/prod-vol /mnt

### 添加服务器到使用加密的存储池

###### 添加新节点

在初始部署后，新服务器可以添加到使用加密的受信存储池。红帽 Gluster 存储支持通用证书认证机构和自签名证书。这两者的程序稍有差异。

######### 通过通用证书认证机构签名的证书

要将使用由通用 CA 签名的证书的服务器添加到使用加密的受信存储池，请在新服务器建立为对等点后执行下列步骤。

1. 从任何现有服务器复制 /etc/ssl/gluster.ca 到新服务器的 /etc/ssl 目录。gluster.ca 文件需要在所有服务器上均相同，其中应含有签名 CA 的证书。
2. 如果已经为现有服务器配置了管理加密，请创建 /var/lib/glusterd/secure-access 文件。此文件为空，是启用管理加密的必需文件；glusterd 需要它才能获取 volfiles 并将 volfile 更改通知客户端。
3. 在新服务器上启动 glusterd。
4. 将新服务器添加到被允许访问已启用加密的卷的通用名称列表。这会将卷的访问列表设置为包含新服务器。确保这并非仅仅是新服务器，而是要启用 TLS/SSL 授权的所有服务器和客户端。
5. 运行命令 gluster peer probe SERVER，将新服务器添加到受信存储池。 

###### 自签名证书

将使用自签名证书的新服务器添加到受信存储池需要停机，因为 CA 列表无法动态重新加载。若要添加新服务器，请执行以下操作：

1. 通过 openssl 生成私钥和证书。
2. 在现有的服务器上，将新服务器证书的内容附加到 glusterfs.ca，再将它分发到受信存储池中的所有服务器上，包括新服务器。在使用自签名证书时，glusterfs.ca 是每一服务器和每一客户端的所有 /etc/ssl/glusterfs.pem 文件的串联。
3. 在现有的客户端上，将新服务器证书的内容附加到 glusterfs.ca，再将它分发到受信存储池中的所有客户端。在使用自签名证书时，客户端的 glusterfs.ca 文件是来自各服务器的所有 /etc/ssl/gluster.pem 文件的串联。
4. 在受信服务器池中的所有服务器上，停止所有与 glusterfs 相关的进程。

    [root@demo-a ~]# pkill glusterfs

5. 如果已经为现有服务器配置了管理加密，请创建 /var/lib/glusterd/secure-access 文件。此文件为空，是启用管理加密的必需文件；glusterd 需要它才能获取 volfiles 并将 volfile 更改通知客户端。

    [root@demo-a ~]# touch /var/lib/glusterd/secure-access

6. 在新服务器上启动 glusterd。

    [root@demo-a ~]# systemctl start glusterd

7. 在所有启用了加密的卷上，将新服务器添加到 auth.ssl-allow 列表。这会将卷的访问列表设置为包含新服务器。确保这并非仅仅是新服务器，而是要启用 TLS/SSL 授权的所有服务器和客户端。
8. 在所有客户端上卸载并停止所有卷。

    [root@demo-a ~]# unmount
            MOUNT-POINT
    [root@demo-a ~]# gluster volume stop VOLNAME

9. 在所有服务器上重启 glusterd。

    [root@demo-a ~]# systemctl restart glusterd

10. 启动卷。

    [root@demo-a ~]# gluster volume start VOLNAME

11. 将卷挂载到所有客户端上。

    [root@demo-a ~]# mount -t glusterfs servera:/VOLNAME mount-point

12. 运行命令 gluster peer probe SERVER，将新服务器添加到受信存储池。

    [root@demo-a ~]# systemctl restart glusterd


### 引导式练习：添加新节点

添加服务器到使用加密的存储池。

1. 准备 serverc，以添加到受信存储池。

    a. 在 serverc 上停止 glusterd 服务。

    [root@serverc ~]# systemctl stop glusterd

    b. 从 workstation 卸载 /mnt。

    [root@workstation ~]# umount /mnt

2. 上传 serverc 的证书和密钥。

    a. 在 serverc 上，使用命令 curl 上传 serverc.pem、serverc.key 和 glusterfs.ca 文件到 /etc/ssl。

    [root@serverc ~]# curl ftp://workstation.lab.example.com/pub/serverc.pem -o /etc/ssl/glusterfs.pem
    [root@serverc ~]# curl ftp://workstation.lab.example.com/pub/serverc.key -o /etc/ssl/glusterfs.key
    [root@serverc ~]# curl ftp://workstation.lab.example.com/pub/glusterfs.ca -o /etc/ssl/glusterfs.ca

3. 为 serverc 启用管理加密。

    a. 在 /var/lib/glusterd 中创建 secure-access 文件。

    [root@serverc ~]# touch /var/lib/glusterd/secure-access
          
    b. 在 serverc 上启动 glusterd 服务。

    [root@serverc ~]# systemctl start glusterd

4. 为 serverc 启用对 prod-vol 的访问权限。

    a. 从 servera，设置 auth.ssl-allow，以便为 serverc 启用访问权限。将所有服务器和客户端传递为命令的参数。

    [root@servera ~]# gluster volume set prod-vol auth.ssl-allow \
    >servera.lab.example.com,serverb.lab.example.com,serverc.lab.example.com, \
    >workstation.lab.example.com

5. 验证 serverc 的卷访问权限。

    b. 验证 serverc 已添加至被允许访问受信存储池的服务器和客户端列表中。

    [root@servera ~]# gluster volume info prod-vol
    Volume Name: prod-vol
    Type: Replicate
    Volume ID: dcb941ec-90a1-4d1a-8722-9020dfcc2b53
    Status: Started
    Number of Bricks: 1 x 2 = 2
    Transport-type: tcp
    Bricks:
    Brick1: servera.lab.example.com:/bricks/brick-a1/brick
    Brick2: serverb.lab.example.com:/bricks/brick-b1/brick
    Options Reconfigured:
    performance.readdir-ahead: on
    auth.ssl-allow:servera.lab.example.com,serverb.lab.example.com,serverc.lab.example.com,
    workstation.lab.example.com
    server.ssl: on
    client.ssl: on

6. 将 serverc 添加为受信 Gluster 存储池的对等点。

    c. 在 servera.example.com 上，将 serverc 添加为受信 Gluster 存储池的对等点。

    [root@servera ~]# gluster peer probe serverc.lab.example.com

7. 验证 serverc 是对等点。

    d. 验证 serverc 的对等关系状态。

    [root@servera ~]# gluster peer status
    Number of Peers: 2
    Hostname: serverb.lab.example.com
    Uuid: d33677d2-382e-45a0-8337-af9c9a7da8c4
    State: Peer in Cluster (Connected)

    Hostname: serverc.lab.example.com
    Uuid: 2d776332-382e-45a0-8337-af9c9a7da8c4
    State: Peer in Cluster (Connected)


### 授权新客户端

当红帽 Gluster 存储受信存储池配置了网络加密时，添加的所有新客户端都需要进行授权。要授权新客户端通过自签名证书访问使用加密的红帽 Gluster 存储池，请执行以下操作：

1. 生成私钥 glusterfs.key 和证书 glusterfs.pem，并将它们放入/etc/ssl
2. 如果有当前有权访问受信存储池的客户端，请将 /etc/ssl/glusterfs.ca 文件复制到该新客户端。
3. 如果启用了管理加密，请在以下位置创建 secure-access 文件和 glusterd 目录：/var/lib/glusterd/secure-access
4. 如果使用自签名证书，请将来自新客户端的 glusterfs.pem 文件附加到 glusterfs.ca，并将 glusterfs.ca 文件分发到受信存储池中的所有服务器。
5. 如果使用通用 CA，则创建证书请求并让证书认证机构签名。将 glusterfs.pem、glusterfs.key 和 glusterfs.ca 文件放在/etc/ssl 目录中。
6. 为允许访问该卷的所有客户端和服务器设置 SSL 授权列表。
7. 重新启动该卷。
8. 将该卷挂载到新客户端上。 

### 引导式练习：授权新客户端

授权 servere 作为新客户端访问受信存储池。

1. 授权新客户端 servere 访问受信存储池。

    a. 从 servere，利用命令 curl 将服务器的密钥、证书以及证书认证机构的证书上传到 /etc/ssl。

    [root@servere ~]# curl ftp://workstation.lab.example.com/pub/servere.pem -o /etc/ssl/glusterfs.pem
    [root@servere ~]# curl ftp://workstation.lab.example.com/pub/servere.key -o /etc/ssl/glusterfs.key
    [root@servere ~]# curl ftp://workstation.lab.example.com/pub/glusterfs.ca -o /etc/ssl/glusterfs.ca

2. 为 servere 启用管理加密。

    a. 通过创建目录 /var/lib/glusterd 和文件 /var/lib/glusterd/secure-access，为 servere 启用管理加密。

    [root@servere ~]# mkdir -p /var/lib/glusterd
    [root@servere ~]# touch /var/lib/glusterd/secure-access

3. 为 servere 启用 prod-vol 的访问权限。

    a. 将 servere 添加到被允许访问 prod-vol 的服务器和客户端列表中。

    [root@servera ~]# gluster volume set prod-vol auth.ssl-allow \
    > 'servera.lab.example.com,serverb.lab.example.com, \
    > serverc.lab.example.com, \
    > servere.lab.example.com, \
    > workstation.lab.example.com'

4. 验证 servere 的卷访问权限。

    a. 验证 servere 已添加至被允许访问受信存储池的服务器和客户端列表中。

    [root@servera ~]# gluster volume info prod-vol
    Volume Name: prod-vol
    Type: Replicate
    Volume ID: dcb941ec-90a1-4d1a-8722-9020dfcc2b53
    Status: Started
    Number of Bricks: 1 x 2 = 2
    Transport-type: tcp
    Bricks:
    Brick1: servera.lab.example.com:/bricks/brick-a1/brick
    Brick2: serverb.lab.example.com:/bricks/brick-b1/brick
    Options Reconfigured:
    performance.readdir-ahead: on
    auth.ssl-allow: servera.lab.example.com,serverb.lab.example.com,serverc.lab.example.com,
    workstation.lab.example.com,servere.lab.example.com
    server.ssl: on
    client.ssl: on

5. 从 servere 访问 prod-vol。

    a. 在 servere 上安装 glusterfs-fuse 客户端。

    [root@servere ~]# yum -y install glusterfs-fuse

    b. 使用原生 Gluster 客户端，将 prod-vol 挂载到 servere 上的 /mnt 目录。

    [root@servere ~]# mount -t glusterfs servera:/prod-vol /mnt


### 实验：配置网络加密

为卷 dev-vol 配置 I/O 和管理加密，并且授权新客户端 servere 访问加密的受信存储池。

为 dev-vol 卷配置管理加密和 I/O 加密。您的受信存储池包含三台计算机：servera、serverb 和 serverc。所有需要的证书和密钥都已为您创建好，它们位于 ftp://workstation.lab.example.com/pub。servere 计算机应当把 dev-vol 卷挂载到 /mnt。这不需要是永久挂载。 

1. 验证为 dev-vol 配置的选项。

    a. 检查卷 dev-vol 的属性。

    [root@servera ~]# gluster volume info dev-vol
    Volume Name: dev-vol
    Type: Replicate
    Volume ID: 269f7040-2194-48ef-b7e4-767b8d86e161
    Status: Created
    Number of Bricks: 1 x 2 = 2
    Transport-type: tcp
    Bricks:
    Brick1: servera.lab.example.com:/bricks/brick-a2/brick
    Brick2: serverb.lab.example.com:/bricks/brick-b2/brick
    Options Reconfigured:
    performance.readdir-ahead: on
    
2. 停止卷 dev-vol。

    [root@servera ~]# gluster volume stop dev-vol

3. 从 ftp://workstation.lab.example.com/pub，将 servera、serverb 和 serverc 的服务器密钥、证书和证书认证机构的证书上传到 /etc/ssl。

    a. 从 servera，利用 curl 命令将服务器的密钥、证书以及证书认证机构的证书下载到 /etc/ssl。

    [root@servera ~]# curl ftp://workstation.lab.example.com/pub/servera.pem -o /etc/ssl/glusterfs.pem
    [root@servera ~]# curl ftp://workstation.lab.example.com/pub/servera.key -o /etc/ssl/glusterfs.key
    [root@servera ~]# curl ftp://workstation.lab.example.com/pub/glusterfs.ca -o /etc/ssl/glusterfs.ca
                   

    b. 从 serverb，利用 curl 命令将服务器的密钥、证书以及证书认证机构的证书下载到 /etc/ssl。

    [root@serverb ~]# curl ftp://workstation.lab.example.com/pub/serverb.pem -o /etc/ssl/glusterfs.pem
    [root@serverb ~]# curl ftp://workstation.lab.example.com/pub/serverb.key -o /etc/ssl/glusterfs.key
    [root@serverb ~]# curl ftp://workstation.lab.example.com/pub/glusterfs.ca -o /etc/ssl/glusterfs.ca

    c. 从 serverc，利用 curl 命令将服务器的密钥、证书以及证书认证机构的证书下载到 /etc/ssl。

    [root@serverc ~]# curl ftp://workstation.lab.example.com/pub/serverc.pem -o /etc/ssl/glusterfs.pem
    [root@serverc ~]# curl ftp://workstation.lab.example.com/pub/serverc.key -o /etc/ssl/glusterfs.key
    [root@serverc ~]# curl ftp://workstation.lab.example.com/pub/glusterfs.ca -o /etc/ssl/glusterfs.ca

4. 配置允许访问 dev-vol 的服务器和客户端。

    a. 允许 servera、serverb、serverc 和 workstation 安全访问受信存储池。

    [root@servera ~]# gluster volume set dev-vol auth.ssl-allow 'servera.lab.example.com,serverb.lab.example.com,serverc.lab.example.com'

5. 为卷 dev-vol 启用 SSL。

    a. 从 servera，为 dev-vol 启用服务器 SSL。

    [root@servera ~]# gluster volume set dev-vol server.ssl on

    b. 从 servera，为 dev-vol 启用客户端 SSL。

    [root@servera ~]# gluster volume set dev-vol client.ssl on

6. 为服务器 servera、serverb 和 serverc 启用管理加密。

    a. 停止 servera、serverb 和 serverc 上的 glusterd 服务。

    [root@servera ~]# for I in server{a..c};do ssh ${I} "systemctl stop glusterd";done

    b. 中断 servera、serverb 和 serverc 上的 glusterfs 进程。

    [root@servera ~]# for I in server{a..c};do ssh ${I} "pkill glusterd";done

    c. 在 servera、serverb 和 serverc 上创建 /var/lib/glusterd/secure-access 文件。

    [root@servera ~]# for I in server{a..c};do ssh ${I} "touch /var/lib/glusterd/secure-access";done

    d. 在 servera、serverb 和 serverc 上启动 glusterd。

    [root@servera ~]# for I in server{a..c};do ssh ${I} "systemctl restart glusterd";done

7. 启动 dev-vol。

    a. 从 servera，启动卷 dev-vol。

    [root@servera ~]# gluster volume start dev-vol

8. 准备 servere，使其成为受信存储池的客户端。

    a. 在 servere 上停止并禁用 glusterd 服务。

    [root@servere ~]# systemctl stop glusterd
    [root@servere ~]# systemctl disable glusterd

9. 为新客户端 servere 提供受信存储池的访问权限。

    a. 从 servere，利用命令 curl 将服务器的密钥、证书以及证书认证机构的证书上传到 /etc/ssl。

    [root@servere ~]# curl ftp://workstation.lab.example.com/pub/servere.pem -o /etc/ssl/glusterfs.pem
    [root@servere ~]# curl ftp://workstation.lab.example.com/pub/servere.key -o /etc/ssl/glusterfs.key
    [root@servere ~]# curl ftp://workstation.lab.example.com/pub/glusterfs.ca -o /etc/ssl/glusterfs.ca

10. 为 servere 启用管理加密。

    a. 通过创建目录 /var/lib/glusterd 和文件 /var/lib/glusterd/secure-access，为 servere 启用管理加密。

    [root@servere ~]# mkdir -p /var/lib/glusterd
    [root@servere ~]# touch /var/lib/glusterd/secure-access

11. 为 servere 启用 dev-vol 的访问权限。

    将 servere 添加到被允许访问 dev-vol 的服务器和客户端列表中。

    [root@servera ~]# gluster volume set dev-vol auth.ssl-allow 'servera.lab.example.com,serverb.lab.example.com, serverc.lab.example.com,servere.lab.example.com'
    
12. 从 servere 访问 dev-vol。

    a. 在 servere 上安装 glusterfs-fuse 软件包。

    [root@servere ~]# yum -y install glusterfs-fuse

    b. 使用原生 gluster 客户端，将 dev-vol 挂载到 servere 上的 /mnt 目录。

    [root@servere ~]# mount -t glusterfs servera:/dev-vol /mnt


### 总结

在本章中，您学到了：

*    如何为红帽 Gluster 存储受信存储池配置 I/O 加密。
*    如何为红帽 Gluster 存储受信存储池配置管理加密。
*    如何配置额外的红帽 Gluster 存储节点，以访问现有的加密受信存储池。
*    如何配置额外的红帽 Gluster 存储客户端，以访问现有的加密受信存储池。
*    如何为红帽 Gluster 存储客户端配置管理加密。
*    如何使用 gluster volume set 命令启用 SSL 访问。 
