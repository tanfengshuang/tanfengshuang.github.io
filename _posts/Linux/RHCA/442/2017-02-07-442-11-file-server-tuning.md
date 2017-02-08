---
layout: post
title:  "File Server Tuning(442-11)"
categories: Linux
tags: RHCA 442 tune2fs barrier qperf
---

### File System Journaling

The ext3/ext4 journal can work in three different modes, chosen by passing the data=mode option to the file system at mount time. The default mode is ordered. The three modes are:

*    ordered: Only metadata is recorded in the journal. Journal entries will only be committed after all file data has been flushed to disk.
*    writeback: Only metadata is recorded in the journal, but data ordering is not preserved. This is rumored to be the fastest method, and internal file system integrity is guaranteed, but after a crash and recovery, old data may show up inside files.
*    journal: All data is first stored in the journal before being written out to disk. This gives the best reliability for the file system but can be the worst performer in most workloads. In workloads with many small writes, this could help increase performance since the disk elevator gets a chance to group writes together.

###### tune2fs

tune2fs是调整和查看ext2/ext3文件系统的文件系统参数，Windows下面如果出现意外断电死机情况，下次开机一般都会出现系统自检。Linux系统下面也有文件系统自检，而且是可以通过tune2fs命令，自行定义自检周期及方式。

常用选项说明：
*    -l 查看文件系统信息
*    -c max-mount-counts 设置强制自检的挂载次数，如果开启，每挂载一次mount conut就会加1，超过次数就会强制自检
*    -i interval-between-checks[d|m|w] 设置强制自检的时间间隔[d天m月w周]
*    -m reserved-blocks-percentage 保留块的百分比
*    -j 将ext2文件系统转换为ext3类型的文件系统
*    -L volume-label 类似e2label的功能，可以修改文件系统的标签
*    -r reserved-blocks-count 调整系统保留空间
*    -o [^]mount-option[,...] Set or clear the indicated default mount options in the filesystem. 设置或清除默认挂载的文件系统选项

例子

*    tune2fs -c 30 /dev/hda1 设置强制检查前文件系统可以挂载的次数
*    tune2fs -c -l /dev/hda1 关闭强制检查挂载次数限制。
*    tune2fs -i 10 /dev/hda1 10天后检查
*    tune2fs -i 1d /dev/hda1 1天后检查
*    tune2fs -i 3w /dev/hda1 3周后检查
*    tune2fs -i 6m /dev/hda1 半年后检查
*    tune2fs -i 0 /dev/hda1 禁用时间检查
*    tune2fs -j /dev/hda1 添加日志功能，将ext2转换成ext3文件系统
*    tune2fs -r 40000 /dev/hda1 调整/dev/hda1分区的保留空间为40000个磁盘块
*    tune2fs -o acl,user_xattr /dev/hda1 设置/dev/hda1挂载选项，启用Posix Access Control Lists和用户指定的扩展属性


```
# man tune2fs
        -O [^]feature[,...]
              Set  or  clear the indicated filesystem features (options) in the filesystem.  More than one filesystem feature can be cleared or set by separating fea-
              tures with commas.  Filesystem features prefixed with a caret character (’^’) will be cleared in the filesystem’s superblock; filesystem features  with-
              out a prefix character or prefixed with a plus character (’+’) will be added to the filesystem.

              The following filesystem features can be set or cleared using tune2fs:
                    has_journal
                          Use a journal to ensure filesystem consistency even across unclean shutdowns.  Setting the filesystem feature is equivalent to using the -j option.

# man mkfs.ext4
NAME
       mke2fs - create an ext2/ext3/ext4 filesystem
SYNOPSIS
    mke2fs -O journal_dev [ -b block-size ] [ -L volume-label ] [ -n ] [ -q ] [ -v ] external-journal [ blocks-count ]
OPTIONS
        -O feature[,...]
              Create a filesystem with the given features (filesystem options), overriding the default filesystem options.
               journal_dev
                      Create an external ext3 journal on the given device instead of a regular ext2 filesystem. Note that external-journal must be created with the same block size as the filesystems that will be using it.
        -J journal-options
              Create the ext3 journal using options specified on the command-line.
               device=external-journal
                      Attach the filesystem to the journal block device located on external-journal. The external journal must already have been created using the command
                          mke2fs -O journal_dev external-journal
```

###### journal

```
为了提高写日志和数据的效率，可以将日志单独设置在另外一块磁盘，自己写自己的，这样子可以加快寻址的速度

1. 格式化磁盘时指定日志分区
# mkfs -t ext4 -O journal_dev -b 4096 /dev/sdd1             -> 格式化磁盘/dev/sdd1为日志分区
# mkfs -t ext4 -J device=/dev/sdd1 -b 4096 /dev/sdc1        -> 在格式化数据磁盘/dev/sdc1的同时，挂载日志分区/dev/sdd1

2. 对于已经存在的磁盘，重新指定日志分区
# mkfs -t ext4 -O journal_dev -b 4096 /dev/sdd1             -> 格式化磁盘/dev/sdd1为日志分区
# tune2fs -o '^has_journal' /dev/sdc1                       -> 取消默认的日志存储方式
# tune2fs -j -J device=/dev/sdd1 /dev/sdc1                  -> 为数据磁盘/dev/sdc1指定新的日志分区
```


###### Barrier

Both the XFS and ext4 file systems turn on barriers by default, while ext3 has barriers disabled by default.  To enable or disable barriers at mount time on XFS file systems, use either the barrier or the nobarrier option, respectively. For ext3 and ext4 file systems, use either the barrier=1 or barrier=0 mount option, respectively. On devices with a battery-backed cache, barriers can be disabled for a performance improvement while still guaranteeing data integrity.

要理解barrier，首先需要理解文件系统日志功能。常用的文件系统使用日志功能来保证文件系统的完整性。该功能背后的思路很简单：在写入新的数据块到磁盘之前，会先将元数据写入日志。预先将元数据写入日志可以保证在写入真实数据前后一旦发生错误，日志功能能很容易地回滚到更改之前的状态。这个方法确保了不会发生文件系统崩溃的情况。

单独使用日志功能不能保证没有任何差错。现在的磁盘大都有大容量的缓存，数据不会立即写入到磁盘中，而是先写入到磁盘缓存中。到这一步，磁盘控制器就能更加高效地将其复制到磁盘中。这对性能来说是有好处的，但是对日志功能来说则相反。为了保证日志百分之百可靠，它必须绝对保证元数据在真实数据写入之前被预先写入。这就是我们要介绍文件系统barrier的原因。 

```
被写入的数据在缓存中有一个排序的过程（将邻近的块操作放在一起，加快写的速度），但是如果进行排序后，就不能保证每次都能将数据索引信息优先于数据写入磁盘，从而保证不了journal的正确写入，为了保证数据索引信息总是先于数据写入，可以启用barrier来禁止缓存中重排序，虽然写入的性能有所下降，但是保证了日志的完整性。
如果磁盘带有备用电池组，可以不考虑索引信息和数据的写入顺序，因为即使断电后，仍然能保证所有数据的完整写入，这时可以关掉barrier

用挂载选项-o barrier=0来关闭barrier功能
# mount /dev/sda2 -o barrier=0 /data

可以通过/proc/mounts文件来监控文件系统barrier的当前状态;对于每一个挂载的文件系统，打开这个文件都能看到所有的挂载选项。如果你看到barrier=1，那么你的文件系统就正在使用barrier功能。
# cat /proc/mounts | grep barrier
/dev/mapper/vg_cloudqe16vm01-lv_root / ext4 rw,seclabel,relatime,barrier=1,data=ordered 0 0
/dev/vda1 /boot ext4 rw,seclabel,relatime,barrier=1,data=ordered 0 0
/dev/mapper/vg_cloudqe16vm01-lv_home /home ext4 rw,seclabel,relatime,barrier=1,data=ordered 0 0
```


### Selecting a tuned Profile for a File Server Workload

The two most important subsystems in a file server are disk and network.

The network connection will, in all probability, be heavily used but not latency-sensitive. The disk load can vary from file server to file server, and depends mostly on the type of files being served and the type of clients accessing the service. For example, a file server that serves out virtual machine images to hypervisors will have different disk needs than one serving out spreadsheets and word processor documents to an office. For this unit, the discussion will focus on file servers, which perform more read operations than write operations on large files.

Given the requirements, the choice of tuned profiles on RHEL 7 can be narrowed down to four options:

*    throughput-performance: This profile sets the CPU governor and energy performance bias to performance. Disk readahead value is increased to 4096. The throughput-performance profile also enables sysctl settings that improves the throughput performance of disk and network I/O.
*    latency-performance: This profile is geared for low-latency performance by disabling power-saving mechanisms. Like the throughput-performance profile, CPU governor and energy performance bias are set to performance to guarantee the highest clock frequency. By statically setting the clock frequency, latency-performance prevents the latency which occurs when CPU frequency switches in response to workload fluctuations.
*    network-throughput: This profile is designed for throughput network tuning. It inheritsproperties from the throughput-performance profile and increases kernel network buffers to boost network performance.
*    network-latency: This profile is aimed at decreasing network latency. It inherits properties from the latency-performance profile and also disables transparent hugepages and NUMA balancing, as well as tunes several other network-related sysctl parameters to lower network latency.

### Network Performance Tuning

Network throughput can depend on a number of factors(While some of these are properties of the hardware being used, others can also be influenced from a system.): 

*    the network cards used
*    the type of cabling
*    number of hops in a connection
*    the size of the packets being sent

###### ethtool

One of the most drastic settings to change is the link speed of the network card being used. To query the current setting, use a tool called ethtool

ethtool命令用于获取以太网卡的配置信息，或者修改这些配置。

*    -a 查看网卡中 接收模块RX、发送模块TX和Autonegotiate模块的状态：启动on 或 停用off。 
*    -A 修改网卡中 接收模块RX、发送模块TX和Autonegotiate模块的状态：启动on 或 停用off。 
*    -c display the Coalesce information of the specified ethernet card。 
*    -C Change the Coalesce setting of the specified ethernet card。 
*    -g Display the rx/tx ring parameter information of the specified ethernet card。 
*    -G change the rx/tx ring setting of the specified ethernet card。 
*    -i 显示网卡驱动的信息，如驱动的名称、版本等。 -d 显示register dump信息, 部分网卡驱动不支持该选项。 
*    -e 显示EEPROM dump信息，部分网卡驱动不支持该选项。 
*    -E 修改网卡EEPROM byte。 
*    -k 显示网卡Offload参数的状态：on 或 off，包括rx-checksumming、tx-checksumming等。 
*    -K 修改网卡Offload参数的状态。 
*    -p 用于区别不同ethX对应网卡的物理位置，常用的方法是使网卡port上的led不断的闪；N指示了网卡闪的持续时间，以秒为单位。 
*    -r 如果auto-negotiation模块的状态为on，则restarts auto-negotiation。 
*    -S 显示NIC- and driver-specific 的统计参数，如网卡接收/发送的字节数、接收/发送的广播包个数等。 
*    -t 让网卡执行自我检测，有两种模式：offline or online。 
*    -s 修改网卡的部分配置，包括网卡速度、单工/全双工模式、mac地址等。


```
# ethtool eno1
Settings for eno1:
	Supported ports: [ TP ]
	Supported link modes:   10baseT/Half 10baseT/Full 
	                        100baseT/Half 100baseT/Full 
	                        1000baseT/Full 
	Supported pause frame use: No
	Supports auto-negotiation: Yes
	Advertised link modes:  10baseT/Half 10baseT/Full 
	                        100baseT/Half 100baseT/Full 
	                        1000baseT/Full 
	Advertised pause frame use: No
	Advertised auto-negotiation: Yes
	Speed: 1000Mb/s
	Duplex: Full
	Port: Twisted Pair
	PHYAD: 1
	Transceiver: internal
	Auto-negotiation: on
	MDI-X: on (auto)
	Supports Wake-on: pumbg
	Wake-on: g
	Current message level: 0x00000007 (7)
			       drv probe link
	Link detected: yes


# ethtool -s eth0 speed 100             -> 将千兆网卡的速度降为百兆
# ethtool -p eth0 10                    -> 如果机器上安装了两块网卡，查看eth0对应着哪块网卡

# ethtool -s eth0 autoneg off speed 1000 duplex full        -> 关掉自动协商功能，自己设置网卡速率

也可以在/etc/sysconfig/network-scripts/ifcfg-eth0配置文件中，加入变量ETHTOOL_OPTS="OPTIONS"
# vim /etc/sysconfig/network-scripts/ifcfg-eth0
ETHTOOL_OPTS="autoneg off speed 1000 duplex full"
```

```
# man ethtool
        autoneg on|off
              Specifies whether autonegotiation should be enabled. Autonegotiation is enabled by default, but in some network devices may have trouble with it, so you can disable it if really necessary.
        duplex half|full
              Sets full or half duplex mode.
        speed N
              Set speed in Mb/s.  ethtool with just the device name as an argument will show you the supported device speeds.
```

###### qperf

One of the newer and more extensive tools is qperf. When using an Ethernet network, qperf can measure TCP, UDP, RDS, SCTP, and SDP socket throughputs and latencies. qperf can also perform measurements on RDMA and Infiniband networks.

qperf needs to run on two machines. One (the listener) runs qperf without any options. The other (the sender) invokes qperf with the name of the first host as the first argument, followed by the test options. For example, to run TCP and UDP bandwidth tests between desktopX and desktopY, start a qperf listener on desktopY:

qperf既可以测试网络带宽也可测试延迟

```
[server]# qperf

[client]# qperf 10.16.98.103 tcp_bw udp_bw
tcp_bw:
    bw  =  2.32 GB/sec
udp_bw:
    send_bw  =  2.46 GB/sec
    recv_bw  =  1.87 GB/sec

# qperf -oo msg_size:1:64K:*2 -v 10.16.98.103 tcp_lat
tcp_lat:
    latency        =  104 us
    msg_rate       =  9.6 K/sec
    msg_size       =    1 bytes             ---> msg_size
    loc_cpus_used  =   15 % cpus
    rem_cpus_used  =   13 % cpus
tcp_lat:
    latency        =   103 us
    msg_rate       =   9.7 K/sec
    msg_size       =     2 bytes            ---> msg_size
    loc_cpus_used  =  16.5 % cpus
    rem_cpus_used  =    13 % cpus
...
tcp_lat:
    latency        =   137 us
    msg_rate       =   7.3 K/sec
    msg_size       =    16 KiB (16,384)         ---> msg_size
    loc_cpus_used  =  22.6 % cpus
    rem_cpus_used  =    17 % cpus
tcp_lat:
    latency        =   172 us
    msg_rate       =  5.81 K/sec
    msg_size       =    32 KiB (32,768)         ---> msg_size
    loc_cpus_used  =  22.5 % cpus
    rem_cpus_used  =    19 % cpus
tcp_lat:
    latency        =   316 us
    msg_rate       =  3.17 K/sec
    msg_size       =    64 KiB (65,536)         ---> msg_size
    loc_cpus_used  =  23.5 % cpus
    rem_cpus_used  =    11 % cpus

```

```
# man qperf
        -oo, --loop Var:Init:Last:Incr
              Run  a  test  multiple  times sequencing through a series of values.  Var is the loop variable; Init is the initial value; Last is the value it must not
              exceed and Incr is the increment.  It is useful to set the --verbose_used (-vu) option in conjunction with this option.

```

###### Network data send

The following outline shows the steps of network transmission.

*    Data is written to a socket (a file-like object) and is then put in the transmit buffer.
*    The kernel encapsulates the data into a protocol data unit (PDU).
*    The PDUs move to the per-device transmit queue.
*    The network device driver copies the PDU from the head of the transmit queue to the NIC.
*    The NIC sends the data and raises an interrupt when transmitted.

###### Network data reception

The following outline shows the steps of network reception.

*    The NIC receives a frame and uses DMA to copy the frame into the receive buffer.
*    The NIC raises a hard interrupt.
*    The kernel handles the hard interrupt and schedules a soft interrupt to handle the packet. This move from hard interrupt to soft interrupt prevents a phenomenon known as livelock. A hard interrupt will preempt everything, including other interrupt handlers. This could cause the receive buffers to fill up during heavy receive loads if packets are not moved out of the receive buffers quickly enough. The kernel processes the hard interrupt as fast as it can, and when the soft interrupt is handled later, the entire receive buffer is processed.
*    The soft interrupt is handled and moves the packet to the IP layer.
*    If the packet is destined for local delivery, the PDU will be decapsulated and put in a socket receive buffer. If a process was waiting on this socket, it will now process the data from the receive buffer.

###### Network Buffers tunables

The buffers (or queues) used in this process consist of the core networking read and write buffers (used for UDP and TCP), per-socket TCP read and write buffers, fragmentation buffers, and DMA buffers for the network card. 

The kernel automatically adjusts the size of these buffers based on the current network utilization, but within the limits specified by the kernel tunables mentioned next.

sysctl tunables:

*    net.core.rmem_max, net.core.wmem_max: The core networking maximum socket receive/send (read/write) buffers. Values are in bytes.
*    net.ipv4.tcp_mem, net.ipv4.udp_mem: The systemwide memory limits for TCP and UDP, respectively. These settings consist of three fields: min, pressure, and max.
*    net.ipv4.tcp_rmem, net.ipv4.tcp_wmem: The receive/send TCP socket buffers. Values are in bytes. A socket buffer will start at the size of default bytes (second value) and be automatically adjusted between min (first value) and max (third value) based on need.

*    net.core.rmem_max: TCP+UDP的读buffer总大小 - BDP(in bytes)
*    net.core.wmem_max: TCP+UDP的写buffer总大小 - BDP(in bytes)
*    net.ipv4.tcp_mem: 所有TCP socket的读buffer和写buffer总大小 - min pressure max(所有值比下面的tcp_rmem都要大即可)(in pages)
*    net.ipv4.udp_mem: 所有UDP socket的读buffer和写buffer总大小 - min pressure max
*    net.ipv4.tcp_rmem: 单个TCP的读buffer最大值 - min default(BDP/2) max(BDP) (in bytes)
*    net.ipv4.tcp_wmem: 单个TCP的写buffer最大值 - min default(BDP/2) max(BDP) (in bytes)

```
# sysctl -a | grep rmem 
net.core.rmem_max = 124928
net.core.rmem_default = 124928
net.ipv4.tcp_rmem = 4096	87380	4194304
net.ipv4.udp_rmem_min = 4096

# cat /proc/sys/net/core/rmem_max 
124928

# cat /proc/sys/net/ipv4/tcp_mem 
177504	236672	355008

# cat /proc/sys/net/ipv4/tcp_rmem 
4096	87380	4194304
```

### BDP and Window Scaning 

###### BDP(Bandwidth Delay Product)

To calculate the buffer size needed for maximum throughout

BDP = network speed * RTT

> RTT(round trip time), 可以通过ping命令得到, 例如 20ms = 0.2s

> network speed: 如果4M bps, 则 4*1024*1024/8 Bytes

###### Window Scaning 

If the BDP goes above 64 KiB, TCP connections can utilize window scaling. A TCP window is the amount of data sent to the remote system that has not yet been acknowledged. If unacknowledged data grows to the window size, the sender will stop sending until previous data has been acknowledged.

By default, the window is limited to a maximum of 64 KiB. Window scaling can be negotiated, if both sides support it, allowing the window to grow.

If the sysctl net.ipv4.tcp_window_scaling parameter is set to 1, the kernel will attempt to negotiate window scaling. Window scaling will be disabled if this value is 0.


###### 网络流量控制工具 Netem

Netem 是 Linux 2.6 及以上内核版本提供的一个网络模拟功能模块。该功能模块可以用来在性能良好的局域网中，模拟出复杂的互联网传输性能，诸如低带宽、传输延迟、丢包等等情况。使用 Linux 2.6 (或以上) 版本内核的很多发行版 Linux 都开启了该内核功能，比如Fedora、Ubuntu、Redhat、OpenSuse、CentOS、Debian等等。tc 是 Linux 系统中的一个工具，全名为traffic control（流量控制）。tc 可以用来控制 netem 的工作模式，也就是说，如果想使用 netem ，需要至少两个条件，一个是内核中的 netem 功能被包含，另一个是要有 tc 。

######   tc(traffic control 流量控制)

在Linux中，流量控制都是通过TC这个工具来完成的

tc 是Linux 系统中的一个工具,全名为 traffic control(流量控制)。tc 可以用来控制 netem 的工作模式,也就是说,如果想使用 netem ,需要至少两个条件,一个是内核中的 netem 功能被包含,另一个是要有 tc 。

```
1. 模拟延迟传输
# tc qdisc add dev eth0 root netem delay 100ms          -> 该命令将 eth0 网卡的传输设置为延迟100ms发, 更真实的情况下,延迟值不会这么精确,会有一定的波动,我们可以用下面的情况来模拟出带有波动性的延迟值:
# tc qdisc add dev eth0 root netem delay 100ms 10ms     -> 该命令将 eth0 网卡的传输设置为延迟 100ms ± 10ms (90 ~ 110 ms 之间的任意值)发送
# tc qdisc add dev eth0 root netem delay 100ms 10ms 30% -> 还可以更进一步加强这种波动的随机性, 该命令将 eth0 网卡的传输设置为 100ms ,同时,大约有 30% 的包会延迟 ± 10ms 发送

2、模拟网络丢包
　　# tc  qdisc  add  dev  eth0  root  netem  loss  1%        -> 该命令将 eth0 网卡的传输设置为随机丢掉 1% 的数据包。
　　# tc  qdisc  add  dev  eth0  root  netem  loss  1%  30%   -> 也可以设置丢包的成功率, 该命令将 eth0 网卡的传输设置为随机丢掉 1% 的数据包，成功率为 30% 
　　
3、模拟包重复
　　# tc  qdisc  add  dev  eth0  root  netem  duplicate 1%    -> 该命令将 eth0 网卡的传输设置为随机产生 1% 的重复数据包 。
　　
4、模拟包损坏
　　# tc  qdisc  add  dev  eth0  root  netem  corrupt  0.2%   -> 该命令将 eth0 网卡的传输设置为随机产生 0.2% 的损坏的数据包 。 (内核版本需在2.6.16以上）
　　
5、模拟包乱序
　　# tc  qdisc  change  dev  eth0  root  netem  delay  10ms   reorder  25%  50%  -> 该命令将 eth0 网卡的传输设置为:有 25% 的数据包（50%相关）会被立即发送，其他的延迟 10 秒。  
　　# tc  qdisc  add  dev  eth0  root  netem  delay  100ms  10ms      -> 新版本中，这个命令也会在一定程度上打乱发包的次序
```

###### Practice: Tuning Network Queues

```
1. 设置网络延时
# tc qd add dev eth0 root netem delay 2s        -> 设置2s延时
### tc qd del dev eth0 root                       -> 删除延时设置

# ping 192.168.0.254
PING 192.168.0.254 (192.168.0.254) 56(84) bytes of data.
64 bytes from 192.168.0.254: icmp_seq=1 ttl=64 time=2000 ms
64 bytes from 192.168.0.254: icmp_seq=2 ttl=64 time=2000 ms
64 bytes from 192.168.0.254: icmp_seq=3 ttl=64 time=2000 ms

2. 计算BDP
Assuming a 100 Mbps bandwidth, calculate BDP based on your measured delay.
100 Megabits/s * 2s × 1/8 Byte/bits = 100 * 10^6 * 2 ÷ 8= 25000000 Bytes.

3. 
# time wget http://192.168.0.254/bigfile

4. Add the current core networking and TCP receive (read) buffer tunables (with their current values) to /etc/sysctl.conf. Make a backup of /etc/sysctl.conf in /root/.
# sysctl -a | grep rmem >> /etc/sysctl.conf
# cp /etc/sysctl.conf /root/

5. In /etc/sysctl.conf, change the values for net.core.rmem_max and the last two values for net.ipv4.tcp_rmem as follows:
# vim /etc/sysctl.conf
net.core.rmem_max = 25000000
net.ipv4.tcp_rmem = 4096 12500000 25000000

6. Apply your changes and run the time wget command again. What happened to your transfer time?
# sysctl -p
# time wget http://192.168.0.254/bigfile

7. Disable TCP window scaling and run the test again. What effect did this have?
# sysctl -w net.ipv4.tcp_window_scaling=0
# time wget http://192.168.0.254/bigfile
```

### Bonding and Link Aggressive

> /usr/share/doc/kernel-doc-2.6.32/Documentation/networking/bonding.txt

###### Bonding mode

*    balance-rr (round robin) or 0
*    active-backup or 1
*    802.3ad or 4

###### Configure a bonding interface

```
# /etc/modprobe.d/bonding.conf
alias bond0 bonding
options bond0 -o bond0 mode=balance-rr miimon=100

# vim /etc/sysconfig/network-scripts/ifcfg-bond0
DEVICE=bond0
IPADDR=192.168.1.1
NETMASK=255.255.255.0
NETWORK=192.168.1.0
BROADCAST=192.168.1.255
ONBOOT=yes
BOOTPROTO=none
USERCTL=no

# vim /etc/sysconfig/network-scripts/ifcfg-eth0
DEVICE=eth0
USERCTL=no
ONBOOT=yes
MASTER=bond0
SLAVE=yes
BOOTPROTO=none

# vim /etc/sysconfig/network-scripts/ifcfg-eth1
DEVICE=eth1
USERCTL=no
ONBOOT=yes
MASTER=bond0
SLAVE=yes
BOOTPROTO=none
```


### Jumbo Frames

maximum transmission unit(MTU)

Whenever a packet is transmitted across the network, the packet consists of headers and a payload. A typical TCP/IP packet header includes an Ethernet header, a IP header and a TCP header. All of these headers are included in the maximum transmission unit(MTU) used on the internet, the maximum size a single packet can have.

For instance, a normal TCP connection using TCP timestamps uses 52 bytes of the protocal headers. With a default MTU of 1500 bytes, that's almost 3.5% of the total capacity lost to overhead.

One method to reduce the overhead incurred due to headers is to switch the protocol to another with less overhead. For example, switching from TCP to UDP will reduce the headers from 52 bytes to 28 bytes (less than 1.9% overhead with an MTU of 1500). However, this may not always be feasible.

> tcp: 52/1500

> udp: 28/1500 
 
 
Another method is to increase the size of the packets that can be sent (the MTU). When increasing the MTU above the Ethernet standard of 1500 bytes, the resulting packets are called
jumbo frames.

*    The official maximum size for a jumbo frame is 9000 bytes, but some equipment supports even larger frames.
*    Make sure that every piece of networking equipment on the network supports jumbo frames, including but not limited to the NICs, switches, and routers.
*    To configure a higher MTU, add MTU line to /etc/sysconfig/network-scripts/ ifcfg-name: MTU=size
*    or ifconfig mtu=9000 (临时生效)

> tcp: 52/9000

> udp: 28/9000

```
# vim /etc/sysconfig/network-scripts/ifcfg-eth0:
DEVICE="eth0"
BOOTPROTO="static"
DNS1="192.168.0.254"
HWADDR="00:1A:A0:D7:41:92"
IPADDR="192.168.0.254"
IPV6INIT="yes"
NETMASK="255.255.255.0"
NM_CONTROLLED="yes"
ONBOOT="yes"
BRIDGE=br0
MTU=9000
```
