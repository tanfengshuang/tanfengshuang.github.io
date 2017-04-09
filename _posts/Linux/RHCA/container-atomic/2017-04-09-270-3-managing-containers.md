---
layout: post
title:  "管理容器(270-3)"
categories: Linux
tags: RHCA 270
---


### 从映像注册表中提取容器映像

###### 下载容器映像

容器映像是用于创建容器的只读模板。可从公共映像注册表（如 registry.hub.docker.com）或基于订阅的红帽注册表 (registry.access.redhat.com) 中下载基准映像。

docker search 命令用于搜索特定映像注册表。在以下示例中，fedora 是搜索词。利用 -s N 选项搜索至少有 N 颗星的映像。添加 --no-trunc=true 选项会打印完整的映像描述。

```
# docker search -s 1 fedora
NAME               DESCRIPTION                       STARS  OFFICIAL AUTOMATED
fedora             Official Fedora 21 base image...  120    [OK]
fedora/apache                                        28              [OK]
fedora/couchdb                                       24              [OK]
fedora/mariadb                                       22              [OK]
fedora/memcached                                     19              [OK]
...Output omitted...
```

找到容器映像后，docker pull 命令用于将映像下载到本地 RHEL Atomic 主机中。以下示例说明如何下载 fedora 映像。

```
# docker pull fedora
834629358fe2: Download complete
6cece30db4f9: Download complete
aecd12627ded: Download complete
511136ea3c5a: Download complete
00a0c78eeb6d: Download complete
782cf93a8f16: Download complete
```

将容器映像加载到 RHEL Atomic 主机本地上的另一种方法是使用 docker load 命令。必须将包含映像的 tar 存档下载并复制到 RHEL Atomic 主机中。例如，以下命令将加载红帽企业 Linux 7.0 容器映像。

```
# ls -l rhel-server-docker-7.0-23.x86_64.tar.gz
-rw-r--r--.  1 root root  51749758 Jan 11 21:50 rhel-server-docker-7.0-23.x86_64.tar.gz
# docker load -i rhel-server-docker-7.0-23.x86_64.tar.gz
```

###### 显示本地可用的映像

docker images 命令显示本地可用的容器映像列表。

```
# docker images
REPOSITORY            TAG           IMAGE ID       CREATED        VIRTUAL SIZE
fedora                20            6cece30db4f9   11 days ago    374.1 MB
fedora                heisenbug     6cece30db4f9   11 days ago    374.1 MB
fedora                rawhide       aecd12627ded   11 days ago    379.5 MB
fedora                21            834629358fe2   11 days ago    250.2 MB
fedora                latest        834629358fe2   11 days ago    250.2 MB
```

docker images 输出的第一个字段是映像名称。有时它还包含映像下载位置的映像注册表名称。第二个字段是标签，通常包含版本信息。第三个字段不是完整的映像 ID，而是唯一的映像 ID 的第一部分。请注意，上一个示例输出中的 fedora 映像的最后两个映像 ID 相同，但它们具有不同的标签。在 /var/lib/docker/repositories-devicemapper 中查找这些映像的原始元数据。


###### 保存容器映像

使用 docker save 命令可将容器映像保存到单个本地文件中。该命令将映像名称用作参数并在标准输出中写入 tar 存档数据流。通过文件重定向可将该数据写入文件中。

```
# docker save fedora:21 > /var/tmp/fedora-21-20150112.tar
# ls -l /var/tmp/fedora-21-20150112.tar
-rw-r--r--. 1 root root 259442688 Jan 12 06:01 /var/tmp/fedora-21-20150112.tar
# tar tvf /var/tmp/fedora-21-20150112.tar
drwx------ 0/0          0 2015-01-12 06:01 ./
drwxr-xr-x 0/0          0 2015-01-12 06:01 00a0c78eeb6d81442efcd1d7c02e8b141745e3a06f1ee3458e1bae628e0067d3/
-rw-r--r-- 0/0          3 2015-01-12 06:01 00a0c78eeb6d81442efcd1d7c02e8b141745e3a06f1ee3458e1bae628e0067d3/VERSION
-rw-r--r-- 0/0       1556 2015-01-12 06:01 00a0c78eeb6d81442efcd1d7c02e8b141745e3a06f1ee3458e1bae628e0067d3/json
-rw-r--r-- 0/0       1024 2015-01-12 06:01 00a0c78eeb6d81442efcd1d7c02e8b141745e3a06f1ee3458e1bae628e0067d3/layer.tar
drwxr-xr-x 0/0          0 2015-01-12 06:01 511136ea3c5a64f264b78b5433614aec563103b4d4702f3ba7d4d2698e22c158/
-rw-r--r-- 0/0          3 2015-01-12 06:01 511136ea3c5a64f264b78b5433614aec563103b4d4702f3ba7d4d2698e22c158/VERSION
-rw-r--r-- 0/0        483 2015-01-12 06:01 511136ea3c5a64f264b78b5433614aec563103b4d4702f3ba7d4d2698e22c158/json
-rw-r--r-- 0/0       1536 2015-01-12 06:01 511136ea3c5a64f264b78b5433614aec563103b4d4702f3ba7d4d2698e22c158/layer.tar
drwxr-xr-x 0/0          0 2015-01-12 06:01 834629358fe214f210b0ed606fba2c17827d7a46dd74bd3309afc2a103ad0e89/
-rw-r--r-- 0/0          3 2015-01-12 06:01 834629358fe214f210b0ed606fba2c17827d7a46dd74bd3309afc2a103ad0e89/VERSION
-rw-r--r-- 0/0       1671 2015-01-12 06:01 834629358fe214f210b0ed606fba2c17827d7a46dd74bd3309afc2a103ad0e89/json
-rw-r--r-- 0/0  259425280 2015-01-12 06:01 834629358fe214f210b0ed606fba2c17827d7a46dd74bd3309afc2a103ad0e89/layer.tar
-rw-r--r-- 0/0         84 2015-01-12 06:01 repositories
```

将映像保存到文件中可用于将本地容器映像从一个 RHEL Atomic 主机复制到另一个主机，而无需使用映像注册表。它还支持自定义映像备份。

###### 移除本地映像副本

docker rmi 命令用于从 RHEL Atomic 主机中移除本地容器映像。这不会影响映像下载位置的映像注册表。以下示例说明如何使用 docker rmi 从本地删除带有特定标签的容器映像。

```
# docker images
REPOSITORY            TAG           IMAGE ID       CREATED        VIRTUAL SIZE
fedora                20            6cece30db4f9   11 days ago    374.1 MB
fedora                heisenbug     6cece30db4f9   11 days ago    374.1 MB
fedora                rawhide       aecd12627ded   11 days ago    379.5 MB
fedora                21            834629358fe2   11 days ago    250.2 MB
fedora                latest        834629358fe2   11 days ago    250.2 MB

# docker rmi fedora:rawhide
Untagged: fedora:rawhide
Deleted: aecd12627ded593a207e32e3537661d1fae1cdc8e0f2e074aa4a730213e5a953

# docker images
REPOSITORY            TAG           IMAGE ID       CREATED        VIRTUAL SIZE
fedora                20            6cece30db4f9   11 days ago    374.1 MB
fedora                heisenbug     6cece30db4f9   11 days ago    374.1 MB
fedora                21            834629358fe2   11 days ago    250.2 MB
fedora                latest        834629358fe2   11 days ago    250.2 MB
```

###### 标记映像

Docker 格式化容器映像支持标记系统，该系统适用于管理源于同一基准映像但运行不同应用的多个映像。目前的惯例是使用映像标签进行版本控制。docker tag 命令为本地映像创建新标签。

```
# docker images
REPOSITORY            TAG           IMAGE ID       CREATED        VIRTUAL SIZE
fedora                20            6cece30db4f9   11 days ago    374.1 MB
fedora                heisenbug     6cece30db4f9   11 days ago    374.1 MB
fedora                21            834629358fe2   11 days ago    250.2 MB
fedora                latest        834629358fe2   11 days ago    250.2 MB

# docker tag 834629358fe2 fedora/rht-training:21-training

# docker images
REPOSITORY            TAG           IMAGE ID       CREATED        VIRTUAL SIZE
fedora                20            6cece30db4f9   11 days ago    374.1 MB
fedora                heisenbug     6cece30db4f9   11 days ago    374.1 MB
fedora                21            834629358fe2   11 days ago    250.2 MB
fedora                latest        834629358fe2   11 days ago    250.2 MB
fedora/rht-training   21-training   834629358fe2   11 days ago    250.2 MB
```

新的图像标签创建后，它会指向示例 docker tag 命令中指定的原始映像 ID。 


### 创建实现服务的容器映像


###### 启动容器

docker run 命令用于启动容器。在 RHEL Atomic 主机上执行此命令时，系统将采取以下步骤：

*    如果本地尚不存在映像，将其提取到主机中。
*    创建新的容器，包括名称空间。
*    分配文件系统并挂载读写层。
*    使用 docker0 桥分配网络/桥接口。
*    从池中查找并附加可用的 IP 地址。
*    执行指定的进程。 

使用之前提取的 fedora 映像创建一个运行交互式 Bash shell 的容器。-i 选项提示 docker 命令提供交互式 shell。-t 选项提示 docker 命令分配 pseudo-tty。fedora 是映像名称，可用作此容器的模板。/bin/bash 命令用于在容器中运行

```
# docker run -i -t fedora /bin/bash
# 
```

1. RHEL Atomic 主机存储. RHEL Atomic 主机上设备映射器使用的存储池依赖于一个逻辑卷。

```
# lvs
  LV          VG   Attr       LSize  Pool Origin Data%  Move Log Cpy%Sync Convert
  docker-pool rah  twi-aotz-- 14.38g             8.70   3.78
...Output omitted...
```

上游 Docker 项目实施过程中，由 Docker 服务提供的设备映射器后备存储器使用名为 /var/lib/docker/devicemapper/devicemapper/data 的 100 GiB 稀疏文件。这种存储实施过程称为“设备映射器环回”或 loop-lvm。该稀疏文件的有限大小对性能有影响。

RHEL Atomic 主机针对企业环境中的容器性能进行了优化。它所使用的后备存储器是配置设备映射器驱动程序的精简池，或 thin-lvm。

```
# cat /etc/sysconfig/docker-storage
DOCKER_STORAGE_OPTIONS=--storage-opt dm.fs=xfs --storage-opt
dm.thinpooldev=/dev/mapper/rah-docker--pool 
```

2. 容器进程. 在 RHEL Atomic 主机上运行的所有进程均对 RHEL Atomic 主机可见。容器只能看到容器中启动的原始进程及其子进程。ps 命令基于它的执行环境显示不同的输出。

在容器中执行 shell 进程的 /bin/bash 进程具有 PID 1。在标准的 Red Hat Enterprise Linux 7 环境中，PID 1 将为 systemd 进程。

```
# ps -efZ
LABEL                                           UID PID PPID    C STIME     TTY          TIME   CMD
system_u:system_r:svirt_lxc_net_t:s0:c924,c928 root  1  0       0 10:53     ?       00:00:00    /bin/bash
system_u:system_r:svirt_lxc_net_t:s0:c924,c928 root 11  1       0 15:27     ?       00:00:00    ps -efZ
```

在 RHEL Atomic 主机上运行 ps 并搜索容器进程的 SELinux 上下文将显示进程在 RHEL Atomic 主机 PID 表中的表示方式。在以下示例中，PID 2486 是在容器中运行的 Bash 进程。

```
# ps -efZ | grep 's0:c924,c928'
system_u:system_r:svirt_lxc_net_t:s0:c924,c928          root 2486 849   0 07:53 pts/1 00:00:00 /bin/bash
unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023   root 2884 2551  0 12:30 pts/2 00:00:00 grep --color=auto s0:c924,c928
```

> ps 命令未在红帽客户门户提供的默认 rhel7 映像中安装。如果未安装 ps 实用工具，可使用 yum 命令在容器中安装 procps-ng 软件包。 


3. 容器 SELinux 安全.  RHEL Atomic 主机上的 docker run 进程在 unconfined_t SELinux 上下文中运行，进程具有的 MCS 标签与在容器中运行命令的 MCS 标签不同。

```
# ps -efZ | grep 'docker run'
unconfined_u:system_r:unconfined_t:s0-s0:c0.c1023 root 2455 930  0 Jan12 pts/0 00:00:01 docker run -i -t fedora /bin/bash
```

容器化的 /bin/bash 进程基于其类型及 MCS 等级受 SELinux 限制。RHEL Atomic 主机内核仅允许 /bin/bash 进程对具有相同 MCS 等级的文件执行操作。以下命令显示当容器中的进程创建文件时会发生什么。在此示例中，新文件将继承在容器中运行的 Bash 进程的 SELinux MCS 等级。

```
# touch newfile
# ls -lZ newfile
-rw-r--r--. root root system_u:object_r:svirt_sandbox_file_t:s0:c924,c928 newfile
# ps -efZ
LABEL                           UID         PID   PPID  C STIME TTY          TIME CMD
system_u:system_r:svirt_lxc_net_t:s0:c924,c928 root 1 0  0 Jan12 ?       00:00:00 /bin/bash
```

4. 容器卷.  有时最好使用容器共享 RHEL Atomic 主机上的工具软件或文件。相比 RHEL Atomic 主机上存在的映像，标准的 Red Hat Enterprise Linux 7 裸映像具有少量的工具软件子集，如 /usr/sbin 目录中文件数所示。以下输出显示 Fedora 基准映像中的文件数：

```
# ls /usr/sbin | wc -l
168
```

比较 RHEL Atomic 主机中的以下输出。它显示了 RHEL Atomic 主机可用的更多 /usr/sbin 实用工具：

```
# ls /usr/sbin | wc -l
412
```

docker run 命令使用 -v 选项将卷从 RHEL Atomic 主机绑定挂载到特定容器中： docker run -v HOST_PATH:CONTAINER_PATH[:ro] ...

可选的 :ro 表示该卷将被只读共享。:rw 是一种明确指定读写共享卷的方式。可将多个 -v 选项传递给 docker run 命令以绑定挂载多个目录。

以下命令在新容器中运行 /bin/bash，并在容器中将 RHEL Atomic 主机中的 /usr/sbin 挂载为 /usr/sbin。ls /usr/sbin | wc -l 命令确认在容器中可见 RHEL Atomic 主机内容。

```
# docker run -v /usr/sbin:/usr/sbin -i -t fedora /bin/bash -c 'ls /usr/sbin | wc -l'
412
```

5. 通过 docker 命令配置网络.  RHEL Atomic 主机建立名为 docker0 的 Linux 桥被容器的 NIC 所用。

```
# brctl show
bridge name	bridge id		STP enabled	interfaces
docker0         8000.56847afe9799       no              veth1ee5
# ip addr show docker0
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP
    link/ether 56:84:7a:fe:97:99 brd ff:ff:ff:ff:ff:ff
    inet 172.17.42.1/16 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::5484:7aff:fefe:9799/64 scope link
       valid_lft forever preferred_lft forever
```

运行容器时，Docker 服务将容器的 NIC 附加到 docker0 桥上。如果在容器内部运行 ip addr show 命令，分配给 eth0 的 IP 地址将与 RHEL Atomic 主机的 docker0 接口在同一 IP 子网上。以下示例输出确认，eth0 的 IP 地址 (172.17.0.4/16) 与 RHEL Atomic 主机的 docker0 桥 (172.17.42.1/16) 在同一网络上。

```
# ip addr show eth0
8: eth0: <BROADCAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 22:64:6f:03:31:54 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.4/16 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::2064:6fff:fe03:3154/64 scope link
       valid_lft forever preferred_lft forever
```

经仔细检查，docker0 桥上附加有新的接口。RHEL Atomic 主机上的 brctl show 命令显示创建了名为 veth9923 的接口并附加到 docker0 桥上。RHEL Atomic 主机上的 veth9923 接口和容器中的 eth0 接口是对等接口；它们用作管道的两端。

```
# brctl show
bridge name	bridge id		STP enabled	interfaces
docker0         8000.56847afe9799       no              veth9923
-bash-4.2# ip addr show veth9923
9: veth9923: <BROADCAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast master docker0 state UP qlen 1000
    link/ether 7e:ff:80:b2:29:c3 brd ff:ff:ff:ff:ff:ff
    inet6 fe80::7cff:80ff:feb2:29c3/64 scope link
       valid_lft forever preferred_lft forever
```

为保持容器实现外部网络连接，对 RHEL Atomic 主机进行几项设置。默认情况下，启用内核可调 net.ipv4.ip_forward。这样，RHEL Atomic 主机可从容器中转发数据包。

```
# sysctl -a | grep ip_forward
net.ipv4.ip_forward = 1
```

此外，RHEL Atomic 主机基于 Netfilter 规则伪装从容器中输出的数据包。此规则确保了从 RHEL Atomic 主机路由出的数据包的源地址将更改为主机的 IP 地址，而不是容器的源地址。

```
# iptables -v -t nat -L POSTROUTING
Chain POSTROUTING (policy ACCEPT 578 packets, 48625 bytes)
 pkts bytes target     prot opt in     out     source               destination
    0     0 MASQUERADE  all  --  any    !docker0  172.17.0.0/16        anywhere
```

6. ping 命令通常用于测试网络连通性。要在容器中使用 ping，创建容器时必须指定 --privileged 选项。

```
# docker run -v /usr/bin:/usr/bin -i -t fedora /bin/bash
# ping www.google.com
bash: /usr/bin/ping: Operation not permitted
# exit
# docker run --privileged -v /usr/bin:/usr/bin -i -t fedora /bin/bash
# ping www.google.com
PING www.google.com (216.58.216.164) 56(84) bytes of data.
64 bytes from sea15s02-in-f4.1e100.net (216.58.216.164): icmp_seq=1 ttl=54 time=36.5 ms
64 bytes from sea15s02-in-f4.1e100.net (216.58.216.164): icmp_seq=2 ttl=54 time=39.4 ms
^C
--- www.google.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 2012ms
rtt min/avg/max/mdev = 36.542/38.007/39.472/1.465 ms
```

7. 将外部端口映射到容器中.  若要将 RHEL Atomic 主机外部的网络流量输入到容器中，可将 RHEL Atomic 主机上的网络端口映射到容器中的端口上。这一操作可通过使用 docker run 命令的 -p HOST:CONTAINER 选项实现。以下示例将 RHEL Atomic 主机的端口 8080 映射到所创建容器的端口 80 上。

```
# docker run -p 8080:80 --rm -i -t fedora /bin/bash
```

典型示例包括 Apache。如之前示例所述创建容器后，安装和启动用于监听容器内端口 80 上的请求的 Apache Web 服务器。

```
# yum install -y httpd
...Output omitted...

Dependency Installed:
  apr.x86_64 0:1.5.1-3.fc21
  apr-util.x86_64 0:1.5.4-1.fc21
  fedora-logos-httpd.noarch 0:21.0.5-1.fc21
  httpd-filesystem.noarch 0:2.4.10-9.fc21
  httpd-tools.x86_64 0:2.4.10-9.fc21
  mailcap.noarch 0:2.1.43-1.fc21

# /usr/sbin/httpd -DFOREGROUND
AH00558: httpd: Could not reliably determine the server's fully qualified
domain name, using 172.17.0.8. Set the 'ServerName' directive globally
to suppress this message
```

通过对 RHEL Atomic 主机的外部 IP 地址的端口 8080 执行 HTTP 请求,可对 Apache 服务器进行测试。

```
# curl --silent http://172.25.0.10:8080 | head -n5
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">
   <head>
          <title>Test Page for the Apache HTTP Server on Fedora</title>
```


###### 管理容器

docker ps 命令可列出所有运行的容器。

```
# docker ps
CONTAINER ID        IMAGE                             COMMAND             CREATED             STATUS              PORTS                  NAMES
afcfb75d1fa5        fedora/rht-training:21-training   "/bin/bash"         22 minutes ago      Up 22 minutes       0.0.0.0:8080->80/tcp   condescending_engelbart
```

docker ps -a 命令列出了所有容器，包括以往运行的容器。有关已退出的容器的信息包括退出状态和容器已退出多长时间。

```
# docker ps -a
CONTAINER ID        IMAGE                             COMMAND                CREATED             STATUS                        PORTS                  NAMES
afcfb75d1fa5        fedora/rht-training:21-training   "/bin/bash"            22 minutes ago      Up 22 minutes                 0.0.0.0:8080->80/tcp   condescending_engelbart
0eaa243eb199        fedora/rht-training:21-training   "/bin/bash"            2 hours ago         Exited (127) 23 minutes ago                          dreamy_ardinghelli
962a39ddd687        fedora/rht-training:21-training   "/bin/bash"            2 hours ago         Exited (130) 2 hours ago                             backstabbing_pike
...Output omitted...
```

docker rm 命令从 docker ps -a 产生的列表中移除已停止的容器。

```
# docker ps -a | grep backstabbing_pike
962a39ddd687        fedora/rht-training:21-training   "/bin/bash"            2 hours ago         Exited (130) 2 hours ago                             backstabbing_pike
# docker rm backstabbing_pike
backstabbing_pike
# docker ps -a | grep backstabbing_pike
```

docker ps -a 命令的 -q 选项仅用于列出容器 ID。该命令可配合 docker rm 使用以移除所有已停止的容器。

```
# docker ps -a -q
afcfb75d1fa5
0eaa243eb199
7a64aa3553b4
0415046d2657
aabbcc6de6ed
7a65cfefaba3
# docker rm $(docker ps -a -q)
Error response from daemon: You cannot remove a running container. Stop
the container before attempting removal or use -f
0eaa243eb199
7a64aa3553b4
0415046d2657
aabbcc6de6ed
7a65cfefaba3
2015/01/15 15:45:23 Error: failed to remove one or more containers
```

###### 从容器中创建映像

docker commit 命令从容器中创建映像。定制了容器并需要保存更改时，这一命令非常有用。指定的 ID 是容器 ID，使用 docker ps -a 命令可显示该 ID。新映像名称应为容器最初基于的映像的名称。可向映像分配标签以便确定它与原始映像之间的区别。 

> docker commit ID NAME:TAG

以下示例说明了如何从映像中启动容器，在容器中创建用户，然后将更新的容器保存为名为 fedora:example-user 的新映像。

```
# docker run -p 8080:80 -i -t fedora /bin/bash
# id example
id: example: no such user
# useradd -p 'locked' example
# grep example /etc/passwd
example:x:1000:1000::/home/example:/bin/bash
# grep example /etc/shadow
example:locked:16451:0:99999:7:::
# id example
uid=1000(example) gid=1000(example) groups=1000(example)
# exit

# id example
id: example: no such user

# docker commit 6fd8f5207002 fedora:example-user
c9b4c7ef8cf3772859158577d88d141bb90211170874f6a00c699da47216e1f4
# docker run -i -t fedora:example-user /bin/bash -c 'grep example /etc/shadow'
example:locked:16451:0:99999:7:::
```

对容器进行更改时，须记住，一些文件系统（如 /run）作为 tmpfs 挂载在容器内，因此当容器退出时，对其内容所作更改将会丢失。

很多守护进程将运行时数据（如进程 ID）保存到 /run 下的子目录文件中。例如，Apache 服务器将运行时数据保存在 /run/httpd 下。通常情况下，这些运行时子目录在系统启动时由 systemd-tmpfiles 刷新和重新创建。由于容器中缺少 systemd 服务，可能需要编写自定义脚本来手动执行这些步骤。下面的示例脚本刷新和重新创建必要的子目录，然后启动 Apache 进程。

```
#!/bin/bash
rm -rf /run/httpd
install -m 710 -o root -g apache -d /run/httpd
install -m 700 -o apache -g apache -d /run/httpd/htcacheclean
exec /usr/sbin/apachectl -D FOREGROUND
```

###### nsenter 实用工具

由于容器化的应用在隔离的名称空间内运行，很难浏览并解决应用级别问题。nsenter 实用工具是一个有用的工具，用于为给定的进程输入名称空间。

nsenter 命令需要容器化进程的 PID。必须采取一些步骤以获取 nsenter 所需的进程 ID：
1. 使用 docker ps 命令列出当前运行的容器。确定相关容器的容器 ID。
2. 使用 docker top 命令显示容器化进程的 RHEL Atomic 主机 PID。

```
# docker ps
CONTAINER ID        IMAGE                             COMMAND             CREATED              STATUS              PORTS                  NAMES
33aeb9b6c7fa        fedora/rht-training:21-training   "/bin/bash"         About a minute ago   Up About a minute   0.0.0.0:8080->80/tcp   condescending_brown
-bash-4.2# docker top 33aeb9b6c7fa
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                20040               849                 0                   22:51               pts/1               00:00:00            /bin/bash
```

将此 PID 传递给 nsenter 以指定要输入哪个进程的名称空间。-t 选项指定要输入的应用。

```
# nsenter -m -u -n -i -p -t 20040 /bin/bash
# ps -efZ
LABEL                           UID         PID   PPID  C STIME TTY          TIME CMD
system_u:system_r:svirt_lxc_net_t:s0:c178,c964 root 1 0  0 01:51 ?       00:00:00 /bin/bash
unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 root 6 0  0 01:54 ? 00:00:00 /bin/bash
unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 root 7 6  0 01:54 ? 00:00:00 ps -efZ
#
```

-m、-u、-n、-i 和 -p 选项定义要输入的名称空间。它们分别输入 mount、UTS、network、IPC 和 PID 名称空间。/bin/bash 参数指定要在应用的名称空间内运行的命令。在上一个示例中，容器运行的命令与 nsenter 和 bash 所用的相同。请注意，用于输入容器名称空间的进程在无限制的 SELinux 上下文中运行。

通过 docker top 命令，容器名称可以替代容器 ID 的使用。在上一个示例中，可能使用了名称 condescending_brown。使用 docker run 命令的 --name 选项指定一个特定名称。

通过使用 docker exec 并指定容器 ID，可向运行的容器发出命令。一旦发出，将在容器内执行该命令。以下命令在运行的容器内执行 echo 命令。

```
# docker exec -i -t b5db1fcb904c echo "Hello world from inside container!"
Hello world from inside container!
```

### Summary

从映像注册表中提取容器映像 .  在本节，您学到：

*    使用 docker search 命令显示可用映像的列表。
*    使用 docker pull imagename 命令下载供本地使用的映像。
*    使用 docker load image.tar.gz 命令加载供本地使用的映像。
*    使用 docker images 命令显示本地可用映像的列表。
*    使用 docker save imagename > image.tar.gz 命令创建映像 tar 存档。
*    使用 docker rmi imagename 命令删除本地映像。
*    使用 docker tag imageid repo/imagename:tag 命令向映像分配标签。

练习：创建实现服务的容器映像 .  在本节，您学到：

*    使用 docker run 命令启动容器。
*    使用 -v HOST_PATH:CONTAINER_PATH 选项将外部目录绑定挂载到容器中。
*    使用 -p HOST_PORT:CONTAINER_PORT 选项将外部端口连接到容器端口上。
*    使用 docker ps 命令列出容器。
*    使用 docker rm ID 命令删除停止的容器。
*    使用 docker commit ID NAME:TAG 命令从容器中创建容器映像。 
