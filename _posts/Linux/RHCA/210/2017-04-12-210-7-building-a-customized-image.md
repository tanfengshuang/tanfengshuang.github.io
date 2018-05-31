---
layout: post
title:  "构建自定义映像(210-7)"
categories: Linux
tags: RHCA 210
---

### 自定义预构建映像

Diskimage-builder 是一款用于自定义云映像的工具，它为使用红帽和 Fedora 构建云映像提供支持。Diskimage-builder 以 qcow2 格式输出虚拟磁盘映像。diskimage-builder 在构建过程中应用元素来自定义映像。元素是在 chroot 环境内运行的代码集，它可以改变映像的构建方式。例如，docker 元素从指定容器导出 tarball，使其他元素能够在其基础上进行构建，或者 bootloader 元素可在系统的引导分区上安装 grub2。 

###### 架构

Diskimage-builder 在 chroot 环境中绑定 /proc、/sys 和 /dev 挂载。映像构建进程生成最小的系统，它们包含实现其 OpenStack 相关目的需要的所有数据位。映像可以是文件系统映像那样的简单映像，也可以经过自定义提供完整的磁盘映像。在完成文件系统树时，将构建一个带有文件系统（或分区表及文件系统）的环回设备，其中复制了该文件系统树。

###### 元素

元素用于实施放入映像的内容以及所需的任何修改。映像必须至少使用一种基础分发元素，给定的分发具有多个元素。例如，分发元素可以是 rhel7，其他元素则用于修改 rhel7 基础映像。基于多个元素调用脚本并应用到映像。

###### 元素依赖项

每一元素可以使用 element-deps 和 element-provides 定义或影响依赖项。Element-deps 是包含元素列表的纯文本文件，它们将在映像创建时添加到其中构建的元素列表中。Element-provides 是包含由此元素提供的元素列表的纯文本文件。这些特定元素不会与创建时构建到映像中的元素一起包含。

```
[stack@demo ~]$ ls /usr/share/diskimage-builder/elements
apt-conf                         debian-upstart       dracut-ramdisk
apt-preferences                  debootstrap          element-manifest
apt-sources                      deploy               enable-serial-console
architecture-emulation-binaries  deploy-baremetal     epel
baremetal                        deploy-ironic        fedora
base                             deploy-kexec         fedora-minimal
cache-url                        deploy-targetcli     grub2
centos                           deploy-tgtadm        hwburnin
centos7                          devuser              hwdiscovery
centos-minimal                   dhcp-all-interfaces  ilo
cleanup-kernel-initrd            dib-init-system      install-static
cloud-init-datasources           dib-run-parts        install-types
cloud-init-nocloud               disable-selinux      ironic-agent
debian                           dkms                 ironic-discoverd-ramdisk
debian-minimal                   dpkg                 iso
debian-systemd                   dracut-network       local-config
```

每一元素具有在构建时应用于映像的脚本。

```
[stack@demo ~]$ tree /usr/share/diskimage-builder/elements/base
/usr/share/diskimage-builder/elements/base/
├── cleanup.d
│   ├── 01-ccache
│   └── 99-tidy-logs
├── element-deps
├── environment.d
│   └── 10-ccache.bash
├── extra-data.d
│   └── 50-store-build-settings
├── install.d
│   ├── 00-baseline-environment
│   ├── 00-up-to-date
│   ├── 10-cloud-init
│   ├── 50-store-build-settings
│   └── 99-dkms
├── package-installs.yaml
├── pkg-map
├── pre-install.d
│   └── 03-baseline-tools
├── README.rst
└── root.d
    └── 01-ccache

6 directories, 15 files
```

###### 阶段子目录

阶段子目录位于元素目录中。阶段子目录按照 1-8 的顺序处理。阶段子目录包含具有两位数字前缀并按词序执行的脚本。其约定是将数据文件存储在元素目录中，但仅将可执行脚本存储在阶段子目录中。阶段子目录名称和描述如下表所示。

*    阶段子目录 	        详情
*    root.d 	        构建或修改初始的 root 文件系统内容。这是添加自定义项的位置，例如在现有映像基础上构建。一次仅可使用一个元素，除非特别谨慎，不进行盲目的覆盖，而是适应由其他元素提取的上下文。
*    extra-data.d 	    包含来自主机环境的额外数据，构建映像时挂钩可能需要这些数据。这应当将任何数据（如 SSH 密钥、HTTP 代理设置和类似数据）复制到 $TMP_HOOKS_PATH 下的某一位置。
*    pre-install.d 	    在进行任何自定义或安装软件包之前，此代码在 chroot 环境中运行。
*    post-install.d 	此阶段适合用于执行需要在操作系统/应用安装之后、第一次引导映像之前处理的任务；例如，运行systemctl disable来禁用不需要的服务。
*    block-device.d 	自定义块设备，如进行分区。它在 cleanup.d 阶段运行之前、目标树完全填充之后运行。
*    finalize.d 	    完成将 root 文件系统内容复制到挂载的文件系统时，在 chroot 中运行。此处执行对 root 文件系统的调优，因此务必限制为影响文件系统元数据和映像本身所需的操作。大部分操作更适合 post-install.d。
*    cleanup.d 	        清理 root 文件系统内容。

######### Diskimage-builder 环境变量

根据所需的映像自定义，可能需要导出许多环境变量。通常而言，至少需要导出三个变量：DIB_LOCAL_IMAGE、DIB_DIST 和 DIB_YUM_REPO_CONF。

######### Diskimage-builder

在构建映像时，可以指定一些选项；例如：

```
disk-image-create vm rhel7 -n -p
      python-django-compressor -a amd64 -o web.img 2> | tee
      diskimage-build.log
```

Diskimage-builder 选项

*    选项 	详情
*    VM 	引导的映像采用其中一种标准发布版 rhel7 或 fedora
*    -a 	设置映像的架构（默认为 amd64）
*    -n 	跳过基础元素的默认包含
*    -p 	列出映像中要安装的软件包。软件包通过逗号分隔。
*    -o 	设置输出映像文件（默认映像）的映像名称。


每一元素中执行一系列脚本。例如，以下为来自diskimage-build.log文件的输出；请观察root.d的第一个元素：

```
Target: root.d

Script                                     Seconds
---------------------------------------  ----------

01-ccache                                     0.017
10-rhel7-cloud-image                         93.202
50-yum-cache                                  0.045
90-base-dib-run-parts                         0.037
```

将要运行尚未部署的任何脚本，并设置为列出其所需的时间。iscsi-initiator-utils、yum-utils 和 OpenStack 包仅仅是其中几个要安装的软件包，具体取决于映像发现类型、部署或 overcloud。下面列出了驻留在 extra-data.d 子阶段的脚本。

```
Target: extra-data.d

Script                                     Seconds
---------------------------------------  ----------

01-inject-ramdisk-build-files                 0.031
10-create-pkg-map-dir                         0.114
20-manifest-dir                               0.021
50-add-targetcli-module                       0.038
50-store-build-settings                       0.006
75-inject-element-manifest                    0.040
98-source-repositories                        0.041
99-enable-install-types                       0.023
99-squash-package-install                     0.221
99-yum-repo-conf                              0.039
```


### 练习：自定义预构建映像

1. 以 stack 用户身份打开连接 director 的 SSH 会话，然后使用密码 redhat 切换到用户 root。

[student@workstation ~]$ ssh stack@director
[stack@director ~]$ sudo -i
[root@director ~]# 

2. 使用curl命令，从 http://materials.example.com/web.img 检索 Web 映像。将它保存到/home/stack/images目录。

[root@director ~]# cd /home/stack/images/
[root@director images]# curl -s http://materials.example.com/web.img -o /home/stack/images/web.img

3. 通过移除第一元素 cache-url，修改 rhel7 的 element-deps文件。

    a. 使用sed从element-deps移除 cache-url。

    [root@director images]# sed -i 's/cache-url//gi' /usr/share/diskimage-builder/elements/rhel7/element-deps

    b. 访问 /usr/share/diskimage-builder/elements/rpm-distro/pre-install.d/00-fix-requiretty，并使用 sed 在构建过程中更新 sudoer 文件的 chmod。

    [root@director images]# sed -i '13ichmod 0440 /etc/sudoers.d/wheel' /usr/share/diskimage-builder/elements/rpm-distro/pre-install.d/00-fix-requiretty 2>/dev/null

4. 创建一个脚本，该脚本将融入到映像构建中并将 Hello world 打印至 /root/hello-world.txt。

    a. 创建目录 /usr/share/diskimage-builder/elements/rhel7/post-install.d/。

    [root@director images]# mkdir -p /usr/share/diskimage-builder/elements/rhel7/post-install.d

    b. 在新创建的 post-install.d 目录中创建一个下方所示的简单脚本，它将在 diskimage-builder 自定义的安装后阶段中运行。将该文件命名为 01-include-script； 它将追加"Hello world"至 /root/hello-world.txt。

    #!/bin/bash
    echo 'Hello world' >> /root/hello-world.txt

    c. 使用 chmod 设置脚本上的可执行权限。

    [root@director images]# chmod +x /usr/share/diskimage-builder/elements/rhel7/post-install.d/01-include-script

5. 以 stack 用户身份，导出用于分发、映像位置和存储库配置的 diskimage-builder变量。

    a. 导出 diskimage-builder 变量 NODE_DIST、DIB_LOCAL_IMAGE 和 DIB_YUM_REPO_CONF。

    [stack@director ~]$ cd images
    [stack@director images]$ export NODE_DIST=rhel7
    [stack@director images]$ export DIB_LOCAL_IMAGE=web.img
    [stack@director images]$ export DIB_YUM_REPO_CONF="/etc/yum.repos.d/openstack.repo /etc/yum.repos.d/rhel_dvd.repo"

    b. 使用安装的软件包 python-django-compressor 和融入的脚本 01-include-script 构建新映像。等待约 5 分钟，让命令完成。

    [stack@director images]$ disk-image-create vm rhel7 -n -p python-django-compressor -a amd64 -o test-web.img > /dev/null 2>&1 | tee diskimage-build.log


### 验证映像

Guestfish 属于 libguestfs-tools 软件包提供的工具套件的一员，它是一种交互式 shell，云管理员可利用它修改预构建映像。guestfish 可以从命令行使用，也可从 shell 脚本使用。libguestfs API 提供的所有功能均可从 shell 访问。

> 注意: guestfish 不会将映像直接挂载到本地文件系统中，而是提供一个 shell 接口，让管理员可以查看、编辑和删除文件。许多guestfish命令，如touch、chmod 和rm可模拟传统的 Bash 命令。

要开始使用 guestfish 交互式 shell 查看或编辑磁盘映像，请发出下列命令：

```
[stack@demo ~]$ guestfish --ro -a /path/to/image
```

利用 --ro 选项是最安全的使用模式，因为它仅允许只读访问。如果不确定虚拟机的运行状态，则最好先以只读方式挂载映像。修改磁盘映像时，虚拟机绝不能处于运行状态。尝试修改与运行中的虚拟机关联的磁盘映像可以导致灾难性的磁盘损坏。libguestfs 和 guestfish 均不需要 root 权限。不过，如果被访问的磁盘映像需要 root 来读取和/或写入映像文件，则需要 root 权限。guestfish 中不存在当前工作目录这样的概念。无法使用 cd 命令来更改目录，所以只能使用绝对路径。不过，其中提供了用于编辑和创建文件与目录的许多命令，如vim、touch 和 mkdir，以及用于查看文件和目录的命令，如ll、cat、ls 和 more 等。启动 guestfish 交互式 shell 后，可以通过两种方式挂载磁盘映像。

在提示符处，键入 run 来发起附加并初始化磁盘映像。根据系统，第一次执行时最多可能需要 30 秒钟。随后启动时将很快完成。在提示符处，键入list-filesystems来列出由 libguestfs 发现的文件系统，然后运行命令mount查看文件系统的内容。

```
[stack@director images]$ guestfish -a demo.img 
Welcome to guestfish, the guest filesystem shell for
editing virtual machine file systems and disk images.

Type: 'help' for help on commands
      'man' to read the manual
      'quit' to quit the shell

><fs> run
><fs> list-filesystems
/demo/sda1: XFS
><fs> mount /demo/sda1 /
```

也可以让 guestfish 检查和挂载映像，就像它在虚拟机中一样。

```
[stack@director images]$ guestfish -a demo.img -i
      Welcome to guestfish, the guest filesystem shell for
editing virtual machine file systems and disk images.

Type: 'help' for help on commands
      'man' to read the manual
      'quit' to quit the shell

><fs>
```

其他有用的命令有 list-devices、list-partitions、lvs、pvs、vfs-type 和 file。如需更多信息以及任何命令的帮助，可以通过键入help加所需的命令来访问。

```
><fs> help vfs-type
 NAME
    vfs-type - get the Linux VFS type corresponding to a mounted device

 SYNOPSIS
     vfs-type device

 DESCRIPTION
    This command gets the filesystem type corresponding to the filesystem on
    "device".

    For most filesystems, the result is the name of the Linux VFS module
    which would be used to mount this filesystem if you mounted it without
    specifying the filesystem type. For example a string such as "ext3" or
    "ntfs".
```

可以使用 guestfish 修改、创建和移除文件。例如，若要修改resolv.conf，可使用命令edit。edit命令会将文件复制到主机，启动编辑器，然后将文件复制回来。

```
[stack@director images]$ guestfish -a demo.img -i 
Welcome to guestfish, the guest filesystem shell for
editing virtual machine file systems and disk images.

Type: 'help' for help on commands
      'man' to read the manual
      'quit' to quit the shell

><fs> edit /etc/resolv.conf
```


### 练习：验证映像

1. 以 stack 用户身份打开一个连接至 director 的 SSH 会话。跳转到 images 目录。

```
[student@workstation ~]$ ssh stack@director
[stack@director ~]$ cd images
```

2. 使用 yum 安装软件包 libguestfs-tools。

```
[stack@director images]$ sudo yum -y install libguestfs-tools
```

3. 为映像打开一个 guestfish shell。

```
[stack@director images]$ guestfish -a test-web.img.qcow2 -i
      Welcome to guestfish, the guest filesystem shell for
editing virtual machine file systems and disk images.

Type: 'help' for help on commands
      'man' to read the manual
      'quit' to quit the shell

><fs> 
```

4. 使用 tail 命令，验证 python-django-compressor 软件包已经安装。

```
><fs> tail /var/log/yum.log
Mar 02 00:29:47 Installed: python-django-compressor-1.4-3.el7.noarch
...Output omitted...
```

5. 确认 hello-world.txt 文件已创建，并位于 /root 中。

```
><fs> cat /root/hello-world.txt
Hello world
```

6. 退出 guestfish。

```
><fs> exit
```

7. 利用新自定义的映像启动一个实例，验证 test-web.img 仍然完好，没有在自定义过程中损坏。使用 m2.small 作为类别、dev_privnet 作为网络，以及 dev_key1 作为密钥对。

    a. 首先，提供位于 /home/stack/overcloudrc 的 overcloud Keystone 凭据文件。

    [stack@director images]$ source /home/stack/overcloudrc

    b. 将位于 /home/stack/images/ 的 test-web.img.qcow2 文件上传到 Glance。将映像命名为 test-web。

    [stack@director images]$ openstack image create --disk-format qcow2 --container-format bare --public --file test-web.img.qcow2 test-web
    +------------------+------------------------------------------------------+
    | Field            | Value                                                |
    +------------------+------------------------------------------------------+
    | checksum         | ee1eca47dc88f4879d8a229cc70a07c6                     |
    | container_format | bare                                                 |
    | created_at       | 2016-01-11T18:22:33Z                                 |
    | disk_format      | qcow2                                                |
    | file             | /v2/images/7467896f-8c7e-4314-90b8-c9bacbd61aaf/file |
    | id               | 7467896f-8c7e-4314-90b8-c9bacbd61aaf                 |
    | min_disk         | 0                                                    |
    | min_ram          | 0                                                    |
    | name             | test-web                                             |
    | owner            | 535d7127baf840dbb956646bbf892e81                     |
    | properties       | direct_url='rbd://2b6ad-5280143/images/124f-330/snap'|
    | protected        | False                                                |
    | schema           | /v2/schemas/image                                   
    | size             | 13287936                                             |
    | status           | active                                               |
    | updated_at       | 2016-01-11T18:22:34Z                                 |
    | virtual_size     | None                                                 |
    | visibility       | public                                               |
    +------------------+------------------------------------------------------+

    c. 使用上一练习中创建的 test-web 映像，创建名为 dev_instance 的 Nova 实例。

    [stack@director images]$ openstack server create \
    > --flavor m2.small --key-name dev_key1 --nic net-id=dev_privnet \
    > --image test-web dev_instance --wait
    +--------------------------------------+-------------------------------------+
    | Field                                | Value                               |
    +--------------------------------------+-------------------------------------+
    | OS-DCF:diskConfig                    | MANUAL                              |
    | OS-EXT-AZ:availability_zone          | nova                                |
    | OS-EXT-SRV-ATTR:host                 | overcloud-novacompute-0.localdomain |
    | OS-EXT-SRV-ATTR:hypervisor_hostname  | overcloud-novacompute-0.localdomain |
    | OS-EXT-SRV-ATTR:instance_name        | instance-00000003                   |
    | OS-EXT-STS:power_state               | 1                                   |
    | OS-EXT-STS:task_state                | None                                |
    | OS-EXT-STS:vm_state                  | active                              |
    | OS-SRV-USG:launched_at               | 2016-03-02T07:14:31.000000          |
    | OS-SRV-USG:terminated_at             | None                                |
    | accessIPv4                           |                                     |
    ...

    d. 将浮动 IP 172.25.250.51 关联到该实例。该浮动 IP 已由实验脚本添加。

    [stack@director images]$ openstack ip floating add 172.25.250.51 dev_instance

8. 使用 ping，确保可通过上一步中关联的浮动 IP 地址访问该实例。

```
[stack@director images]$ ping -c4 172.25.250.51
PING 172.25.250.51 (172.25.250.51) 56(84) bytes of data.
64 bytes from 172.25.250.51: icmp_seq=1 ttl=63 time=1.75 ms
64 bytes from 172.25.250.51: icmp_seq=2 ttl=63 time=0.686 ms
64 bytes from 172.25.250.51: icmp_seq=3 ttl=63 time=0.726 ms
64 bytes from 172.25.250.51: icmp_seq=4 ttl=63 time=0.716 ms

--- 172.25.250.51 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3009ms
rtt min/avg/max/mdev = 0.686/0.970/1.754/0.453 ms
```

9. 等待一两分钟，这是 cloud-init 将 SSH 密钥注入 dev_instance 实例所需的时间。从 workstation 打开一个新终端。使用 /home/student/dev/dev_key1.pem 下保存的私钥，发起与实例的 SSH 连接。登录之后，确保 python-django-compressor 软件包已安装成功，然后从实例退出。

```
[student@workstation ~]$ ssh -i dev/dev_key1.pem cloud-user@172.25.250.51
[cloud-user@dev-instance ~]$ yum list python-django-compressor
Loaded plugins: langpacks, search-disabled-repos
Installed Packages
python-django-compressor.noarch
[cloud-user@dev-instance ~]$ exit
```


### 实验：构建自定义映像

1. 准备环境

从 director 节点，检索位于 http://materials.example.com/web.img 的 web.img，并将它保存到 /home/stack/images 下。

以 root 用户身份，更新映像构建器所依赖的一些元素。从 /usr/share/diskimage-builder/elements/rhel7/element-deps，删除 cache-url 这一行。在 /usr/share/diskimage-builder/elements/rpm-distro/pre-install.d/00-fix-requiretty 文件中，在第 13 行（以 sed 命令开头的行上方）插入下面这一行：

```
chmod 0440 /etc/sudoers.d/wheel
```

创建一个名为 01-include-script 的可执行脚本，保存在 /usr/share/diskimage-builder/elements/rhel7/post-install.d/ 下。如果该目录不存在，则进行创建。脚本内容应当如下：

```
#!/bin/bash
echo "Hello world" > /root/hello-world.txt
```

    a. 从 workstation，以 stack 用户身份打开一个连接至 director.lab.example.com 的 SSH 会话。登录后，使用密码 redhat 切换为 root 用户。

    [student@workstation ~]$ ssh stack@director
    [stack@director ~]$ sudo -i
    [root@director ~]# 

    b. 使用curl命令，从 http://materials.example.com/web.img 检索 Web 映像。将它保存到/home/stack/images目录。

    [root@director ~]# cd /home/stack/images
    [root@director images]# curl -s http://materials.example.com/web.img -o /home/stack/images/web.img

    c. 通过移除第一元素 cache-url，修改 rhel7 的 element-deps 文件。为此，可使用sed命令。

    [root@director images]# sed -i 's/cache-url//gi' /usr/share/diskimage-builder/elements/rhel7/element-deps

    d. 访问/usr/share/diskimage-builder/elements/rpm-distro/pre-install.d/00-fix-requiretty，并在第 13 行添加chmod 0440 /etc/sudoers.d/wheel。可以使用下列sed命令。

    [root@director images]# sed -i '13ichmod 0440 /etc/sudoers.d/wheel' /usr/share/diskimage-builder/elements/rpm-distro/pre-install.d/00-fix-requiretty

    e. 创建一个脚本，该脚本将融入到映像构建中并将 Success, the script is included! 显示到 /root/hello-world.txt。如果 /usr/share/diskimage-builder/elements/rhel7/post-install.d/ 目录不存在，则先创建该目录。

    [root@director images]# mkdir -p /usr/share/diskimage-builder/elements/rhel7/post-install.d

    f. 如下方所示，在新创建的 post-install.d 目录中创建脚本。该脚本将在 diskimage-builder 自定义的安装后阶段中运行。将文件另存为01-include-script。

    #!/bin/bash
    echo "Hello world" > /root/hello-world.txt

    g. 使用chmod设置脚本上的可执行权限。

    [root@director images]# chmod +x  /usr/share/diskimage-builder/elements/rhel7/post-install.d/01-include-script

    h. 切回 stack 用户。

    [root@director images]# exit
    [stack@director ~]$ 

2. 构建映像

以 stack 用户身份，导出映像构建器需要的三个环境变量。在变量导出后，构建映像并安装 python-django-compressor。将映像命名为 test-web.img。新创建的映像应当为 QCOW2 格式。下表列出了要导出的变量以及它们的值：

环境变量

*    变量	                内容
*    NODE_DIST	        rhel7
*    DIB_LOCAL_IMAGE	    web.img
*    DIB_YUM_REPO_CONF	"/etc/yum.repos.d/openstack.repo /etc/yum.repos.d/rhel_dvd.repo"

    a. 以 stack 用户身份，跳转到 images 目录。导出 diskimage-builder 变量 NODE_DIST、DIB_LOCAL_IMAGE 和 DIB_YUM_REPO_CONF。

    [stack@director ~]$ cd ~/images
    [stack@director images]$ export NODE_DIST=rhel7
    [stack@director images]$ export DIB_LOCAL_IMAGE=web.img
    [stack@director images]$ export DIB_YUM_REPO_CONF="/etc/yum.repos.d/openstack.repo /etc/yum.repos.d/rhel_dvd.repo"

    b. 使用软件包 python-django-compressor 构建新映像。脚本 01-include-script 将自动集成。(注意: 等待约 5 分钟，让命令完成。)

    [stack@director images]$ disk-image-create vm rhel7 -n -p python-django-compressor -a amd64 -o test-web.img

3. 检查映像

从 director 节点，使用 guestfish 检查和验证映像自定义。为此，请确保已安装软件包 libguestfs-tools。检查映像，确保 python-django-compressor 已正确安装并且存在 /root/hello-world.txt 文件。

    a. 从 director.lab.example.com，使用 yum 安装软件包 libguestfs-tools。

```
    [stack@director images]$ sudo yum -y install libguestfs-tools
```

    b. 为映像 /home/stack/images/test-web.img.qcow2 开启一个 guestfish shell。

```
    [stack@director images]$ guestfish -a test-web.img.qcow2 -i
          Welcome to guestfish, the guest filesystem shell for
    editing virtual machine file systems and disk images.

    Type: 'help' for help on commands
          'man' to read the manual
          'quit' to quit the shell

    ><fs>
```

    c. 检查 /var/log/yum.log 文件，确保 python-django-compressor软件包已经安装。

```
    ><fs> tail /var/log/yum.log
    Mar 02 00:29:47 Installed: python-django-compressor-1.4-3.el7.noarch
    ...Output omitted...
```

    d. 确保 hello-world.txt 文件已创建在 /root 下。

```
    ><fs> cat /root/hello-world.txt
    Hello world
```

    e. 退出 guestfish shell。

```
    ><fs> exit
```

4. 生成实例

设置脚本创建了包含下列组件的环境：公共网络 lab_pubnet 及子网 lab_pubsub，专用网络 lab_privnet 及子网 lab_privsub。它也在 default 组创建了两条安全组规则，以允许对实例的 ICMP 和 SSH 访问；它还创建了类别 m2.small，以及密钥对 lab_key1，其中的私钥已存储在 /home/student/lab/lab_key1.pem 上的 workstation 下。

使用新创建的映像，在环境中生成一个实例。从 director 节点，将映像作为 test-web 添加到 Glance 中，然后使用上方所述的信息创建一个名为 lab_instance 的新实例。将实例附加到专用网络 priv_net。

    a. 以 stack 用户身份，提供 overcloud Keystone 凭据文件。

    [stack@director images]$ source ../overcloudrc

    b. 使用 openstack 命令，将映像 test-web.img.qcow2 上传到 Glance。

    [stack@director images]$ openstack image create --disk-format qcow2 \
    > --container-format bare --public --file test-web.img.qcow2 test-web
    +------------------+------------------------------------------------------+
    | Field            | Value                                                |
    +------------------+------------------------------------------------------+
    | checksum         | ee1eca47dc88f4879d8a229cc70a07c6                     |
    | container_format | bare                                                 |
    | created_at       | 2016-01-11T18:22:33Z                                 |
    | disk_format      | qcow2                                                |
    | file             | /v2/images/7467896f-8c7e-4314-90b8-c9bacbd61aaf/file |
    | id               | 7467896f-8c7e-4314-90b8-c9bacbd61aaf                 |
    | min_disk         | 0                                                    |
    | min_ram          | 0                                                    |
    | name             | test-web                                             |
    | owner            | 535d7127baf840dbb956646bbf892e81                     |
    | protected        | False                                                |
    | properties       | direct_url='rbd://2b6ad-5280143/images/124f-330/snap'|   
    | schema           | /v2/schemas/image                                    |
    | size             | 13287936                                             |
    | status           | active                                               |
    | updated_at       | 2016-01-11T18:22:34Z                                 |
    | virtual_size     | None                                                 |
    | visibility       | public                                               |
    +------------------+------------------------------------------------------+

    c. 使用类别 m2.small、密钥对 lab_key1 以及刚上传的映像，创建名为 lab_instance 的实例。将实例附加到 lab_privnet 网络。

    [stack@director images]$ openstack server create \
    > --flavor m2.small --key-name lab_key1 --nic net-id=lab_privnet \
    > --image test-web lab_instance --wait
    +--------------------------------------+-------------------------------------+
    | Field                                | Value                               |
    +--------------------------------------+-------------------------------------+
    | OS-DCF:diskConfig                    | MANUAL                              |
    | OS-EXT-AZ:availability_zone          | nova                                |
    | OS-EXT-SRV-ATTR:host                 | overcloud-novacompute-0.localdomain |
    | OS-EXT-SRV-ATTR:hypervisor_hostname  | overcloud-novacompute-0.localdomain |
    | OS-EXT-SRV-ATTR:instance_name        | instance-00000003                   |
    | OS-EXT-STS:power_state               | 1                                   |
    | OS-EXT-STS:task_state                | None                                |
    | OS-EXT-STS:vm_state                  | active                              |
    | OS-SRV-USG:launched_at               | 2016-03-02T07:14:31.000000          |
    | OS-SRV-USG:terminated_at             | None                                |
    | accessIPv4                           |                                     |
    ...Output omitted...

5. 验证实例

设置脚本在浮动 IP 池 lab_pubnet 中创建了浮动 IP。从 workstation.lab.example.com，提供位于 /home/student/overcloudrc 下的 overcloud Keystone 凭据。向实例添加一个浮动 IP，并确保您可以使用 ping 访问该实例；然后利用位于 /home/student/lab/lab_key1.pem 下的私钥通过 SSH 连接该实例。登录之后，确保软件包 python-django-compressor 已安装成功。

> 注意: 在实例生成后，等待一两分钟，确保 cloud-init 服务有时间将公钥注入到实例中。

    a. 从 workstation，提供 overcloud Keystone 凭据。

    ```
    [student@workstation ~]$ source overcloudrc
    ```

    b. 列出可用的浮动 IP。

    ```
    [student@workstation ~]$ openstack ip floating list
    +--------------------------------------+------------+---------------+----------+-------------+
    | ID                                   | Pool       | IP            | Fixed IP | Instance ID |
    +--------------------------------------+------------+---------------+----------+-------------+
    | f5858cfa-85b7-457f-a784-c527ea9b1192 | lab_pubnet | 172.25.250.51 | None     | None        |
    +--------------------------------------+------------+---------------+----------+-------------+
    ```

    c. 将可用的 IP 关联到实例 lab_instance。

    ```
    [student@workstation ~]$ openstack ip floating add 172.25.250.51 lab_instance
    ```

    d. 在浮动 IP 添加到实例后，使用 ping 命令进行访问。

    ```
    [student@workstation ~]$ ping -c4 172.25.250.51
    PING 172.25.250.51 (172.25.250.51) 56(84) bytes of data.
    64 bytes from 172.25.250.51: icmp_seq=1 ttl=63 time=1.75 ms
    64 bytes from 172.25.250.51: icmp_seq=2 ttl=63 time=0.686 ms
    64 bytes from 172.25.250.51: icmp_seq=3 ttl=63 time=0.726 ms
    64 bytes from 172.25.250.51: icmp_seq=4 ttl=63 time=0.716 ms

    --- 172.25.250.51 ping statistics ---
    4 packets transmitted, 4 received, 0% packet loss, time 3009ms
    rtt min/avg/max/mdev = 0.686/0.970/1.754/0.453 ms
    ```

    e. 等待一两分钟，这是 cloud-init 注入 SSH 密钥所需的时间；接着，使用 SSH 连接实例，运行 yum 命令确保 python-django-compressor 已安装成功，然后退出。若要登录，可使用位于 /home/student/lab/ 下的公钥 lab_key1。

    ```
    [student@workstation ~]$ ssh -i lab/lab_key1.pem cloud-user@172.25.250.51
    [cloud-user@lab_instance ~]$ yum list python-django-compressor
    Loaded plugins: langpacks, search-disabled-repos
    Installed Packages
    python-django-compressor.noarch
    [cloud-user@lab_instance ~]$ exit
    ```


### 总结

在本章中，您可以学到：

*    可以使用 Diskimage-builder 等工具自定义映像
*    映像中包含的配置可以使用工具进行检查，如 guestfish，该工具可以使映像挂载到当前的文件系统中。
