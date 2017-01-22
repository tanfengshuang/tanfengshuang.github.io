---
layout: post
title:  "图形化报告(442-2)"
categories: Linux
tags: RHCA 442 vmstat iostat mpstat netstat
---

### Units and Unit Conversions

###### How much is how much?

How many bytes in a 2 TB disk?

|kilo- (K) = 10^3 = 1, 000   |Kibi- (Ki) = 2^10 = 1024   |
|mega- (M) = 10^6 = 1,000,000|Mebi- (Mi) = 2^20 = 1048576|
|giga- (G) = 10^9            |Gibi- (Gi) = 2^30          |
|tera- (T) = 10^12           |Tebi- (Ti) = 2^40          |
|peta- (P) = 10^15           |Pebi- (Pi) = 2^50          |
|exa- (E) = 10^18            |Exbi- (Ei) = 2^60          |
|df -H                       |df -h                      |

> 100Mib/s  equals (100/8/1025)*3600 GiB/h
> 43KiB/sec equals (43/1024)*60      MiB/min
> 20GiB/h   equals (20*1024)/3600    MiB/s

```
# bc
bc 1.06.95
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'. 
scale=8
1/6
.16666666
2*10^12             -> 2T
2000000000000
2*10^12/2^40        -> 转化为2Ti
1.81898940
```

```
# df -H
Filesystem            Size  Used Avail Use% Mounted on
/dev/mapper/vg_cloudqe16vm02-lv_root
                       53G  2.3G   48G   5% /
tmpfs                 985M     0  985M   0% /dev/shm
/dev/vda1             500M   41M  433M   9% /boot
/dev/mapper/vg_cloudqe16vm02-lv_home
                       11G   38M  9.9G   1% /home
/dev/mapper/disk1      96M  4.0M   87M   5% /mnt/myencryptdisk

# df -h
Filesystem            Size  Used Avail Use% Mounted on
/dev/mapper/vg_cloudqe16vm02-lv_root
                       50G  2.2G   45G   5% /
tmpfs                 939M     0  939M   0% /dev/shm
/dev/vda1             477M   39M  413M   9% /boot
/dev/mapper/vg_cloudqe16vm02-lv_home
                      9.8G   37M  9.2G   1% /home
/dev/mapper/disk1      91M  3.8M   83M   5% /mnt/myencryptdisk

```


### Profiling Tools

###### vmstat(MEM IO CPU): Virtual Memory Statistics

vmstat命令是最常见的Linux/Unix监控工具，可以展现给定时间间隔的服务器的状态值,包括服务器的CPU使用率，内存使用，虚拟内存交换情况,IO读写情况。这个命令是我查看Linux/Unix最喜爱的命令，一个是Linux/Unix都支持，二是相比top，我可以看到整个机器的CPU,内存,IO的使用情况，而不是单单看到各个进程的CPU使用率和内存使用率(使用场景不一样)。

一般vmstat工具的使用是通过两个数字参数来完成的，第一个参数是采样的时间间隔数，单位是秒，第二个参数是采样的次数

每个参数的意思：

*    r 表示运行队列(就是说多少个进程真的分配到CPU)，我测试的服务器目前CPU比较空闲，没什么程序在跑，当这个值超过了CPU数目，就会出现CPU瓶颈了。这个也和top的负载有关系，一般负载超过了3就比较高，超过了5就高，超过了10就不正常了，服务器的状态很危险。top的负载类似每秒的运行队列。如果运行队列过大，表示你的CPU很繁忙，一般会造成CPU使用率很高。
*    b 表示阻塞的进程,这个不多说，进程阻塞，大家懂的。
*    swpd 虚拟内存已使用的大小，如果大于0，表示你的机器物理内存不足了，如果不是程序内存泄露的原因，那么你该升级内存了或者把耗内存的任务迁移到其他机器。
*    free   空闲的物理内存的大小，我的机器内存总共8G，剩余3415M。
*    buff   Linux/Unix系统是用来存储，目录里面有什么内容，权限等的缓存，我本机大概占用300多M
*    cache cache直接用来记忆我们打开的文件,给文件做缓冲，我本机大概占用300多M(这里是Linux/Unix的聪明之处，把空闲的物理内存的一部分拿来做文件和目录的缓存，是为了提高 程序执行的性能，当程序使用内存时，buffer/cached会很快地被使用。)
*    si  每秒从磁盘读入虚拟内存的大小，如果这个值大于0，表示物理内存不够用或者内存泄露了，要查找耗内存进程解决掉。我的机器内存充裕，一切正常。
*    so  每秒虚拟内存写入磁盘的大小，如果这个值大于0，同上。
*    bi  块设备每秒接收的块数量，这里的块设备是指系统上所有的磁盘和其他块设备，默认块大小是1024byte，我本机上没什么IO操作，所以一直是0，但是我曾在处理拷贝大量数据(2-3T)的机器上看过可以达到140000/s，磁盘写入速度差不多140M每秒
*    bo 块设备每秒发送的块数量，例如我们读取文件，bo就要大于0。bi和bo一般都要接近0，不然就是IO过于频繁，需要调整。
*    in 每秒CPU的中断次数，包括时间中断
*    cs 每秒上下文切换次数，例如我们调用系统函数，就要进行上下文切换，线程的切换，也要进程上下文切换，这个值要越小越好，太大了，要考虑调低线程或者进程的数目,例如在apache和nginx这种web服务器中，我们一般做性能测试时会进行几千并发甚至几万并发的测试，选择web服务器的进程可以由进程或者线程的峰值一直下调，压测，直到cs到一个比较小的值，这个进程和线程数就是比较合适的值了。系统调用也是，每次调用系统函数，我们的代码就会进入内核空间，导致上下文切换，这个是很耗资源，也要尽量避免频繁调用系统函数。上下文切换次数过多表示你的CPU大部分浪费在上下文切换，导致CPU干正经事的时间少了，CPU没有充分利用，是不可取的。
*    us 用户CPU时间，我曾经在一个做加密解密很频繁的服务器上，可以看到us接近100,r运行队列达到80(机器在做压力测试，性能表现不佳)。
*    sy 系统CPU时间，如果太高，表示系统调用时间长，例如是IO操作频繁。
*    id  空闲 CPU时间，一般来说，id + us + sy = 100,一般我认为id是空闲CPU使用率，us是用户CPU使用率，sy是系统CPU使用率。
*    wt 等待IO CPU时间。

```
# rpm -qf `which vmstat`
procps-3.2.8-36.el6.x86_64

# vmstat 1 3
procs -----------memory---------- ---swap-- -----io---- --system-- -----cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 0  0      0 1041928  72100 570676    0    0     2     1   11    7  0  0 100  0  0	
 0  0      0 1041912  72100 570676    0    0     0     0   32   14  0  0 100  0  0	
 0  0      0 1041912  72100 570676    0    0     0     0   23   15  0  0 100  0  0	
 
# vmstat 1 3 -a         -> 显示inactive和active列
procs -----------memory---------- ---swap-- -----io---- --system-- -----cpu-----
 r  b    swpd   free  inact active   si   so    bi    bo   in   cs us sy id wa st
 0  0      0 1041928 410628 309352    0    0     2     1   11    7  0  0 100  0  0	
 0  0      0 1041912 410628 309372    0    0     0     0   44   20  0  0 100  0  0	
 0  0      0 1041912 410628 309372    0    0     0     0   20   14  0  0 99  0  0	

# grep HZ /boot/config-2.6.32-642.el6.x86_64 
CONFIG_NO_HZ=y
# CONFIG_HZ_100 is not set
# CONFIG_HZ_250 is not set
# CONFIG_HZ_300 is not set
CONFIG_HZ_1000=y
CONFIG_HZ=1000
CONFIG_MACHZ_WDT=m

```

```
# free -m
             total       used       free     shared    buffers     cached
Mem:          1877        859       1017          0         70        557
-/+ buffers/cache:        231       1645
Swap:         4031          0       4031

# man free
The -b switch displays the amount of memory in bytes; the -k switch (set by default) displays it in kilobytes; the -m switch displays it in megabytes;  the  -g
       switch displays it in gigabytes

# free
             total       used       free     shared    buffers     cached
Mem:       1922088     880052    1042036        204      72068     570672
-/+ buffers/cache:     237312    1684776
Swap:      4128764          0    4128764
# echo $[880300 - 237540]           -> used column
642760
# echo $[72084 + 570676]            -> buffers + cached
642760
```

```
# cat /proc/meminfo 
MemTotal:        8132592 kB
MemFree:         1038328 kB
MemAvailable:    4030244 kB
Buffers:          325400 kB
Cached:          2673716 kB
SwapCached:            0 kB
Active:          4845568 kB
Inactive:        1638496 kB
Active(anon):    3486060 kB
Inactive(anon):    42244 kB
Active(file):    1359508 kB
Inactive(file):  1596252 kB
Unevictable:           0 kB
Mlocked:               0 kB
SwapTotal:       8261628 kB
SwapFree:        8261628 kB
Dirty:              1816 kB
Writeback:             0 kB
AnonPages:       3484780 kB
Mapped:           442920 kB
Shmem:             43364 kB
Slab:             422772 kB
SReclaimable:     340972 kB
SUnreclaim:        81800 kB
KernelStack:       10704 kB
PageTables:        52404 kB
NFS_Unstable:          0 kB
Bounce:                0 kB
WritebackTmp:          0 kB
CommitLimit:    12327924 kB
Committed_AS:   10053164 kB
VmallocTotal:   34359738367 kB
VmallocUsed:           0 kB
VmallocChunk:          0 kB
HardwareCorrupted:     0 kB
AnonHugePages:    845824 kB
CmaTotal:              0 kB
CmaFree:               0 kB
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
DirectMap4k:      221704 kB
DirectMap2M:     8124416 kB
```


###### iostat(IO) and mpstat(CPU) (sysstat package)

iostat主要用于监控系统设备的IO负载情况，iostat首次运行时显示自系统启动开始的各项统计信息，之后运行iostat将显示自上次运行该命令以后的统计信息。用户可以通过指定统计的次数和时间来获得所需的统计信息。

*    tps：该设备每秒的传输次数（Indicate the number of transfers per second that were issued to the device.）。"一次传输"意思是"一次I/O请求"。多个逻辑请求可能会被合并为"一次I/O请求"。"一次传输"请求的大小是未知的。
*    kB_read/s：每秒从设备（drive expressed）读取的数据量；
*    kB_wrtn/s：每秒向设备（drive expressed）写入的数据量；
*    kB_read：读取的总数据量；
*    kB_wrtn：写入的总数量数据量；这些单位都为Kilobytes。

iostat还有一个比较常用的选项-x，该选项将用于显示和io相关的扩展数据。

*    rrqm/s：每秒这个设备相关的读取请求有多少被Merge了（当系统调用需要读取数据的时候，VFS将请求发到各个FS，如果FS发现不同的读取请求读取的是相同Block的数据，FS会将这个请求合并Merge）；wrqm/s：每秒这个设备相关的写入请求有多少被Merge了。
*    rsec/s：每秒读取的扇区数；
*    wsec/：每秒写入的扇区数。
*    rKB/s：The number of read requests that were issued to the device per second；
*    wKB/s：The number of write requests that were issued to the device per second；
*    avgrq-sz 平均请求扇区的大小
*    avgqu-sz 是平均请求队列的长度。毫无疑问，队列长度越短越好。
*    await：  每一个IO请求的处理的平均时间（单位是微秒毫秒）。这里可以理解为IO的响应时间，一般地系统IO响应时间应该低于5ms，如果大于10ms就比较大了。这个时间包括了队列时间和服务时间，也就是说，一般情况下，await大于svctm，它们的差值越小，则说明队列时间越短，反之差值越大，队列时间越长，说明系统出了问题。
*    svctm    表示平均每次设备I/O操作的服务时间（以毫秒为单位）。如果svctm的值与await很接近，表示几乎没有I/O等待，磁盘性能很好，如果await的值远高于svctm的值，则表示I/O队列等待太长，系统上运行的应用程序将变慢。
*    %util： 在统计时间内所有处理IO时间，除以总共统计时间。例如，如果统计间隔1秒，该设备有0.8秒在处理IO，而0.2秒闲置，那么该设备的%util = 0.8/1 = 80%，所以该参数暗示了设备的繁忙程度。一般地，如果该参数是100%表示设备已经接近满负荷运行了（当然如果是多磁盘，即使%util是100%，因为磁盘的并发能力，所以磁盘使用未必就到了瓶颈）。

```
# rpm -qf `which iostat`
sysstat-9.0.4-31.el6.x86_64

# iostat 1 1                -> 默认监控所有的硬盘设备，可以用-d /dev/sda 指定监控sda
Linux 2.6.32-642.el6.x86_64 (cloud-qe-16-vm-02.idmqe.lab.eng.bos.redhat.com) 	01/21/2017 	_x86_64_	(2 CPU)
avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.02    0.02    0.04    0.01    0.01   99.89
Device:            tps   Blk_read/s   Blk_wrtn/s   Blk_read   Blk_wrtn
vda               0.30         7.34         5.15    1192812     837228
vdb               0.00         0.02         0.02       3802       2512
dm-0              0.89         7.15         5.09    1161682     827552
dm-1              0.00         0.02         0.00       2600          0
dm-2              0.00         0.01         0.02       1698       2512
dm-3              0.03         0.02         0.06       3088       9624
dm-4              0.03         0.01         0.06       2326       9624

# iostat -k 1 1             -> -k某些使用block为单位的列强制使用Kilobytes为单位（Blk_read   Blk_wrtn -> kB_read    kB_wrtn）
Linux 2.6.32-642.el6.x86_64 (cloud-qe-16-vm-02.idmqe.lab.eng.bos.redhat.com) 	01/22/2017 	_x86_64_	(2 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.02    0.02    0.04    0.01    0.01   99.89

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
vda               0.30         3.66         2.57     596406     418826
vdb               0.00         0.01         0.01       1901       1256
dm-0              0.89         3.57         2.54     580841     413988
dm-1              0.00         0.01         0.00       1300          0
dm-2              0.00         0.01         0.01        849       1256
dm-3              0.03         0.01         0.03       1544       4812
dm-4              0.03         0.01         0.03       1163       4812

# iostat -x -k 1 1
Linux 2.6.32-642.el6.x86_64 (cloud-qe-16-vm-02.idmqe.lab.eng.bos.redhat.com) 	01/22/2017 	_x86_64_	(2 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.02    0.02    0.04    0.01    0.01   99.89

Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
vda               0.04     0.60    0.21    0.09     3.66     2.57    41.56     0.00    2.72    0.65    7.68   0.83   0.02
vdb               0.00     0.00    0.00    0.00     0.01     0.01     9.31     0.00    1.14    0.26    2.93   0.57   0.00
dm-0              0.00     0.00    0.23    0.65     3.56     2.54    13.77     0.00    3.60    0.71    4.63   0.28   0.02
dm-1              0.00     0.00    0.00    0.00     0.01     0.00     8.00     0.00    0.38    0.38    0.00   0.31   0.00
dm-2              0.00     0.00    0.00    0.00     0.01     0.01     7.96     0.00    1.93    0.32    3.01   0.64   0.00
dm-3              0.00     0.00    0.00    0.03     0.01     0.03     2.30     0.00    0.69    0.61    0.70   0.10   0.00
dm-4              0.00     0.00    0.00    0.03     0.01     0.03     2.19     0.00   18.05    0.87   20.34   0.11   0.00
```

mpstat是MultiProcessor Statistics的缩写，是实时系统监控工具。其报告与CPU的一些统计信息，这些信息存放在/proc/stat文件中。在多CPUs系统里，其不但能查看所有CPU的平均状况信息，而且能够查看特定CPU的信息

*    %user      在internal时间段里，用户态的CPU时间(%)，不包含nice值为负进程  (usr/total)*100
*    %nice      在internal时间段里，nice值为负进程的CPU时间(%)   (nice/total)*100
*    %sys       在internal时间段里，内核时间(%)       (system/total)*100
*    %iowait    在internal时间段里，硬盘IO等待时间(%) (iowait/total)*100
*    %irq       在internal时间段里，硬中断时间(%)     (irq/total)*100
*    %soft      在internal时间段里，软中断时间(%)     (softirq/total)*100
*    %idle      在internal时间段里，CPU除去等待磁盘IO操作外的因为任何原因而空闲的时间闲置时间(%) (idle/total)*100
*    %steal     Show the percentage of time spent in involuntary wait by the virtual CPU or CPUs while the hypervisor was servicing another virtual processor.
*    %guest     Show the percentage of time spent by the CPU or CPUs to run a virtual processor.


```
total_cur=user+system+nice+idle+iowait+irq+softirq
total_pre=pre_user+ pre_system+ pre_nice+ pre_idle+ pre_iowait+ pre_irq+ pre_softirq
user=user_cur – user_pre
total=total_cur-total_pre
其中_cur 表示当前值，_pre表示interval时间前的值。上表中的所有值可取到两位小数点。  
```


```
# rpm -qf `which mpstat`
sysstat-9.0.4-31.el6.x86_64

# mpstat            -> 当没有参数时，mpstat则显示系统启动以后所有信息的平均值
Linux 2.6.32-642.el6.x86_64 (cloud-qe-16-vm-02.idmqe.lab.eng.bos.redhat.com) 	01/21/2017 	_x86_64_	(2 CPU)

10:40:00 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest   %idle
10:40:00 PM  all    0.02    0.02    0.04    0.01    0.00    0.00    0.01    0.00   99.89

# mpstat 1 3        -> 第一行的信息自系统启动以来的平均信息。从第二行开始，输出为前一个interval时间段的平均信息。
Linux 2.6.32-642.el6.x86_64 (cloud-qe-16-vm-02.idmqe.lab.eng.bos.redhat.com) 	01/21/2017 	_x86_64_	(2 CPU)

10:44:04 PM  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest   %idle
10:44:05 PM  all    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
10:44:06 PM  all    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
10:44:07 PM  all    0.00    0.00    0.50    0.00    0.00    0.00    0.00    0.00   99.50
Average:     all    0.00    0.00    0.17    0.00    0.00    0.00    0.00    0.00   99.83

# LANG=C mpstat 1 3     -> 时间格式为24小时制
Linux 2.6.32-642.el6.x86_64 (cloud-qe-16-vm-02.idmqe.lab.eng.bos.redhat.com) 	01/21/17 	_x86_64_	(2 CPU)

22:49:32     CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest   %idle
22:49:33     all    0.00    0.00    0.50    0.00    0.00    0.00    0.00    0.00   99.50
22:49:34     all    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
22:49:35     all    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
Average:     all    0.00    0.00    0.17    0.00    0.00    0.00    0.00    0.00   99.83

# LANG=C mpstat 1       -> 1s 取一次，一直取下去
Linux 2.6.32-642.el6.x86_64 (cloud-qe-16-vm-02.idmqe.lab.eng.bos.redhat.com) 	01/21/17 	_x86_64_	(2 CPU)

22:50:43     CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest   %idle
22:50:44     all    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
22:50:45     all    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
22:50:46     all    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
22:50:47     all    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
22:50:48     all    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
22:50:49     all    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
^C

# LANG=C mpstat -P ALL 1 1
Linux 2.6.32-642.el6.x86_64 (cloud-qe-16-vm-02.idmqe.lab.eng.bos.redhat.com) 	01/21/17 	_x86_64_	(2 CPU)

23:30:37     CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest   %idle
23:30:38     all    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
23:30:38       0    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
23:30:38       1    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00

Average:     CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest   %idle
Average:     all    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
Average:       0    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
Average:       1    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00

# LANG=C mpstat -P 1 1 1
Linux 2.6.32-642.el6.x86_64 (cloud-qe-16-vm-02.idmqe.lab.eng.bos.redhat.com) 	01/21/17 	_x86_64_	(2 CPU)

23:31:06     CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest   %idle
23:31:07       1    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00
Average:       1    0.00    0.00    0.00    0.00    0.00    0.00    0.00    0.00  100.00

```
```
# cat /proc/cpuinfo 
processor	: 0
vendor_id	: GenuineIntel
cpu family	: 6
model		: 42
model name	: Intel(R) Core(TM) i7-2600 CPU @ 3.40GHz
stepping	: 7
microcode	: 0x29
cpu MHz		: 1680.609
cache size	: 8192 KB
physical id	: 0
siblings	: 8
core id		: 0
cpu cores	: 4
apicid		: 0
initial apicid	: 0
fpu		: yes
fpu_exception	: yes
cpuid level	: 13
wp		: yes
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc aperfmperf eagerfpu pni pclmulqdq dtes64 monitor ds_cpl vmx smx est tm2 ssse3 cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic popcnt tsc_deadline_timer aes xsave avx lahf_lm epb tpr_shadow vnmi flexpriority ept vpid xsaveopt dtherm ida arat pln pts
bugs		:
bogomips	: 6784.57
clflush size	: 64
cache_alignment	: 64
address sizes	: 36 bits physical, 48 bits virtual
power management:
```

```
# rpm -qf `which lscpu`
util-linux-ng-2.17.2-12.24.el6.x86_64
# lscpu 
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                8
On-line CPU(s) list:   0-7
Thread(s) per core:    2
Core(s) per socket:    4
Socket(s):             1
NUMA node(s):          1
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 42
Model name:            Intel(R) Core(TM) i7-2600 CPU @ 3.40GHz
Stepping:              7
CPU MHz:               1692.164
CPU max MHz:           3800.0000
CPU min MHz:           1600.0000
BogoMIPS:              6784.57
Virtualization:        VT-x
L1d cache:             32K
L1i cache:             32K
L2 cache:              256K
L3 cache:              8192K
NUMA node0 CPU(s):     0-7
Flags:                 fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx rdtscp lm constant_tsc arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc aperfmperf eagerfpu pni pclmulqdq dtes64 monitor ds_cpl vmx smx est tm2 ssse3 cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic popcnt tsc_deadline_timer aes xsave avx lahf_lm epb tpr_shadow vnmi flexpriority ept vpid xsaveopt dtherm ida arat pln pts
```

###### NET

Netstat 命令用于显示各种网络相关信息，如网络连接，路由表，接口状态 (Interface Statistics)，masquerade 连接，多播成员 (Multicast Memberships) 等等。

*    -a (all)显示所有选项，默认不显示LISTEN相关
*    -t (tcp)仅显示tcp相关选项
*    -u (udp)仅显示udp相关选项
*    -n 拒绝显示别名，能显示数字的全部转化成数字。
*    -l 仅列出有在 Listen (监听) 的服務状态
*    -p 显示建立相关链接的程序名
*    -r 显示路由信息，路由表
*    -e 显示扩展信息，例如uid等
*    -s 按各个协议进行统计
*    -c 每隔一个固定时间，执行该netstat命令。

> 提示：LISTEN和LISTENING的状态只有用-a或者-l才能看到

```
# rpm -qf `which netstat`
net-tools-1.60-110.el6_2.x86_64

# netstat -antlp
# netstat -r
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
10.16.96.0      *               255.255.252.0   U         0 0          0 eth0
link-local      *               255.255.0.0     U         0 0          0 eth0
default         unused          0.0.0.0         UG        0 0          0 eth0
# netstat -rn
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
10.16.96.0      0.0.0.0         255.255.252.0   U         0 0          0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U         0 0          0 eth0
0.0.0.0         10.16.99.254    0.0.0.0         UG        0 0          0 eth0


1. 列出所有端口 (包括监听和未监听的)
# netstat -a | more     -> 列出所有端口
# netstat -at           -> 列出所有tcp端口
# netstat -au           -> 列出所有udp端口

2. 列出所有处于监听状态的 Sockets
# netstat -l            -> 只显示监听端口 
# netstat -lt           -> 只列出所有监听 tcp 端口 
# netstat -lu           -> 只列出所有监听 udp 端口 
# netstat -lx           -> 只列出所有监听 UNIX 端口

3. 显示每个协议的统计信息

# netstat -s            -> 显示所有端口的统计信息 
# netstat -st           -> 显示 TCP 端口的统计信息
# netstat -su           -> 显示 UDP 端口的统计信息

4. 在 netstat 输出中显示 PID 和进程名称, netstat -p 可以与其它开关一起使用，就可以添加 “PID/进程名称” 到 netstat 输出中，这样 debugging 的时候可以很方便的发现特定端口运行的程序

# netstat -p
# netstat -pt

5. 显示核心路由信息 netstat -r
# netstat -r

6. 在 netstat 输出中不显示主机，端口和用户名 (host, port or user)
当你不想让主机，端口和用户名显示，使用 netstat -n。将会使用数字代替那些名称。
同样可以加速输出，因为不用进行比对查询。

# netstat -an
# netstat -rn

如果只是不想让这三个名称中的一个被显示，使用以下命令

# netsat -a --numeric-ports
# netsat -a --numeric-hosts
# netsat -a --numeric-users

7. 持续输出 netstat 信息, netstat 将每隔一秒输出网络信息。
# netstat -c

10. 显示网络接口列表
# netstat -i
Kernel Interface table
Iface       MTU Met    RX-OK RX-ERR RX-DRP RX-OVR    TX-OK TX-ERR TX-DRP TX-OVR Flg
eth0       1500   0   571473      0      0      0    12797      0      0      0 BMRU
lo        65536   0      156      0      0      0      156      0      0      0 LRU

# netstat -ie
Kernel Interface table
eth0      Link encap:Ethernet  HWaddr 52:54:00:40:58:04
          inet addr:10.16.98.103  Bcast:10.16.99.255  Mask:255.255.252.0
          inet6 addr: fec0:0:a10:6000:5054:ff:fe40:5804/64 Scope:Site
          inet6 addr: 2620:52:0:1060:5054:ff:fe40:5804/64 Scope:Global
          inet6 addr: fe80::5054:ff:fe40:5804/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:571566 errors:0 dropped:0 overruns:0 frame:0
          TX packets:12804 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:53869691 (51.3 MiB)  TX bytes:1564263 (1.4 MiB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:156 errors:0 dropped:0 overruns:0 frame:0
          TX packets:156 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:53875 (52.6 KiB)  TX bytes:53875 (52.6 KiB)
```

###### sar: The system Activity Reporter

sar(System Activity Reporter系统活动情况报告)是一个优秀的一般性能监视工具，它可以输出Linux所完成的几乎所有工作的数据。sar命令在sysstat rpm中提供。
sar可以显示CPU、运行队列、磁盘I/O、分页（交换区）、内存、CPU中断、网络等性能数据。最重要的sar功能是创建数据文件。每一个Linux系统都应该通过cron工作收集sar数据。该sar数据文件为系统管理员提供历史性能信息。这个功能非常重要，它将sar和其他性能工具区分开。我们首先讨论数据收集。

######### sar数据收集器

sar数据收集通过/usr/lib/sa中的一个二进制可执行文件和两个脚本来完成。sar数据收集器是一个位于/usr/lib/sa/sadc的二进制可执行文件。sa1、sa2为脚本，sadc为二进制可执行文件。
*    第一个脚本sa1，是调用sadc将性能数据收集到二进制日志文件中的一个Shell脚本。sa1命令还确保了每天都使用不同的文件。由脚本可得知，通过执行“sadc -F -L 间隔时间 采集次数 保存文件地址+名称”的方式，把采集的数据进行保存；如果不指定间隔时间和采集次数，则只会采集1次；如果不指定存储文件地址和名称，则会使用 sa+日期，存储到/var/log/sa/下；
*    第二个命令sa2，是将当天二进制文件中所有的数据存储到文本文件的另一个Shell脚本，然后它将清除七天之内的所有日志文件。参数-A指定了从二进制文件中提取哪些数据存储到文本文件中

```
# rpm -ql sysstat | grep cron
/etc/cron.d/sysstat

# vim /etc/cron.d/sysstat
# Run system activity accounting tool every 10 minutes
*/10 * * * * root /usr/lib64/sa/sa1 1 1                 -> 10分钟统计一次
# 0 * * * * root /usr/lib64/sa/sa1 600 6 &
# Generate a daily summary of process accounting at 23:53
53 23 * * * root /usr/lib64/sa/sa2 -A

# file /usr/lib64/sa/sa1
/usr/lib64/sa/sa1: POSIX shell script text executable

# file /usr/lib64/sa/sa2
/usr/lib64/sa/sa2: POSIX shell script text executable

# file /usr/lib64/sa/sadc 
/usr/lib64/sa/sadc: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.18, stripped

# vim /usr/lib64/sa/sa1
SADC_OPTIONS="-S DISK"
ENDIR=/usr/lib64/sa
exec ${ENDIR}/sadc -F -L ${SADC_OPTIONS} 1 1 -

# man sadc
/usr/lib64/sa/sadc [ -C comment ] [ -S { INT | DISK | SNMP | IPV6 | POWER | XDISK | ALL | XALL } ] [ -F ] [ -L ] [ -V ] [ interval [ count ] ] [ outfile ]
If outfile is set to -, then sadc uses the standard system activity daily data file, the /var/log/sa/sadd file, where the dd parameter indicates the current day.
```

######### 执行sar命令

sar命令执行模式：
1. 分析系统当前
2. 读文件，分析以前的cron产生的文件

```
# ls /var/log/sa/sa
sa19   sa20   sa21   sa22   sar19  sar20  sar21 
# vim /etc/sysconfig/sysstat
HISTORY=28                  -> 配置产生的saNN文件的个数
```

```
# rpm -qf `which sar`
sysstat-9.0.4-31.el6.x86_64

# sar 1 3
Linux 2.6.32-642.el6.x86_64 (cloud-qe-16-vm-02.idmqe.lab.eng.bos.redhat.com) 	01/22/2017 	_x86_64_	(2 CPU)

12:47:29 AM     CPU     %user     %nice   %system   %iowait    %steal     %idle
12:47:30 AM     all      0.00      0.00      0.00      0.00      0.00    100.00
12:47:31 AM     all      0.50      0.00      0.00      0.00      0.00     99.50
12:47:32 AM     all      0.00      0.00      0.50      0.00      0.00     99.50
Average:        all      0.17      0.00      0.17      0.00      0.00     99.67

# LANG=C sar -u 1 1                 -> 输出CPU使用情况的统计信息,默认是-u
Linux 2.6.32-642.el6.x86_64 (cloud-qe-16-vm-02.idmqe.lab.eng.bos.redhat.com) 	01/22/17 	_x86_64_	(2 CPU)
00:50:36        CPU     %user     %nice   %system   %iowait    %steal     %idle
00:50:37        all      0.00      0.00      0.00      0.00      0.00    100.00
Average:        all      0.00      0.00      0.00      0.00      0.00    100.00

# LANG=C sar -r 1 1                 -> 输出内存和交换空间的统计信息
Linux 2.6.32-642.el6.x86_64 (cloud-qe-16-vm-02.idmqe.lab.eng.bos.redhat.com) 	01/22/17 	_x86_64_	(2 CPU)
00:50:42    kbmemfree kbmemused  %memused kbbuffers  kbcached  kbcommit   %commit
00:50:43      1040524    881564     45.86     72372    571192    170792      2.82
Average:      1040524    881564     45.86     72372    571192    170792      2.82

# LANG=C sar -d 1 1                 -> 输出每一个块设备的活动信息， 硬盘使用报告
Linux 2.6.32-642.el6.x86_64 (cloud-qe-16-vm-02.idmqe.lab.eng.bos.redhat.com) 	01/22/17 	_x86_64_	(2 CPU)
00:50:53          DEV       tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz     await     svctm     %util
00:50:54     dev252-0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
00:50:54    dev252-16      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
00:50:54     dev253-0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
00:50:54     dev253-1      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
00:50:54     dev253-2      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
00:50:54     dev253-3      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
00:50:54     dev253-4      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
Average:          DEV       tps  rd_sec/s  wr_sec/s  avgrq-sz  avgqu-sz     await     svctm     %util
Average:     dev252-0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00       -> dev252-0 (主设备号，辅设备号)
Average:    dev252-16      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
Average:     dev253-0      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
Average:     dev253-1      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
Average:     dev253-2      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
Average:     dev253-3      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00
Average:     dev253-4      0.00      0.00      0.00      0.00      0.00      0.00      0.00      0.00

# ll /dev/vda*
brw-rw----. 1 root disk 252, 0 Jan 20 02:49 /dev/vda            -> (主设备号252，辅设备号0)
brw-rw----. 1 root disk 252, 1 Jan 20 02:49 /dev/vda1
brw-rw----. 1 root disk 252, 2 Jan 20 02:49 /dev/vda2

# sar -n DEV 1 1            -> 网络
Linux 2.6.32-642.el6.x86_64 (cloud-qe-16-vm-02.idmqe.lab.eng.bos.redhat.com) 	01/22/2017 	_x86_64_	(2 CPU)
12:55:48 AM     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
12:55:49 AM        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00
12:55:49 AM      eth0      4.95      0.00      0.37      0.00      0.00      0.00      0.00
Average:        IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
Average:           lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00
Average:         eth0      4.95      0.00      0.37      0.00      0.00      0.00      0.00

# LANG=C sar -q 1 1         -> 负载
Linux 2.6.32-642.el6.x86_64 (cloud-qe-16-vm-02.idmqe.lab.eng.bos.redhat.com) 	01/22/17 	_x86_64_	(2 CPU)
01:32:53      runq-sz  plist-sz   ldavg-1   ldavg-5  ldavg-15
01:32:54            0       134      0.00      0.01      0.05
Average:            0       134      0.00      0.01      0.05
```

```
# sar -f /var/log/sa/sa19
Linux 2.6.32-642.el6.x86_64 (cloud-qe-16-vm-02.idmqe.lab.eng.bos.redhat.com) 	01/19/2017 	_x86_64_	(2 CPU)
10:40:01 PM     CPU     %user     %nice   %system   %iowait    %steal     %idle
10:50:01 PM     all      0.02      0.00      0.01      0.00      0.01     99.95
11:00:01 PM     all      0.02      0.00      0.01      0.00      0.02     99.95
11:10:01 PM     all      0.02      0.00      0.02      0.01      0.01     99.94
11:20:01 PM     all      0.02      0.00      0.01      0.00      0.01     99.96
11:30:01 PM     all      0.02      0.00      0.01      0.00      0.01     99.96
11:40:01 PM     all      0.02      0.00      0.01      0.01      0.01     99.95
11:50:01 PM     all      0.02      0.00      0.01      0.00      0.01     99.96
Average:        all      0.02      0.00      0.01      0.00      0.01     99.95

# LANG=c sar -u -f /var/log/sa/sa22
# LANG=c sar -r -f /var/log/sa/sa22
# LANG=c sar -d -f /var/log/sa/sa22

# file /var/log/sa/sa22
/var/log/sa/sa22: data
```

### Using awk to Format Data


```
# awk -F : '{print $1}' /etc/passwd
# df | awk '{print $1,$5,$6}'
# awk -F : '{print $1,$3}' /etc/passwd
# awk -F : 'NR==1 {print $1,$3}' /etc/passwd
# awk -F : 'NR<=10 {print NR,$1,$3}' /etc/passwd
# awk -F : 'NR<=10 && NR>=5 {print NR,$1,$3}' /etc/passwd
# awk -F : 'NR<=5 || NR>=40 {print NR,$1,$3}' /etc/passwd
# awk -F : '{print $0,NF}' /etc/passwd				
```

> NR: 行号

> $0: 整行

> NF: 以：分隔，列的个数

^abc	-> 以abc开头

abc$	-> 以abc结尾

```
# awk '/^Ave/{print $0}' /file1
# awk -F : '$1 ~ /^r/ {print $0}' /etc/passwd	-> ~: 匹配
# awk '/^r/ {print $0}' /etc/passwd
root:x:0:0:root:/root:/bin/bash
rpc:x:32:32:Rpcbind Daemon:/var/cache/rpcbind:/sbin/nologin
rtkit:x:499:497:RealtimeKit:/proc:/sbin/nologin
rpcuser:x:29:29:RPC Service User:/var/lib/nfs:/sbin/nologin
radvd:x:75:75:radvd user:/:/sbin/nologin
```

```
# awk '^${print $0}' /file1	-> 空行
# awk '^ ${print $0}' /file1	-> 有一个空格的行
# awk '^ +${print $0}' /file1	-> 有至少一个空格的行，+表示至少有一个
# awk '/^[^a-zA-Z]+$/ {print $1, $(NF-2), $(NF-1), $NF}' /file1
```

```
# LANG=c sar -q 1 3 > /file1
# awk '{print $1,$4,$5,$6}' /file1
# awk '/^16/{print $0}' /file1
16:27:14      runq-sz  plist-sz   ldavg-1   ldavg-5  ldavg-15
16:27:15            2       834      1.05      1.09      1.08
16:27:16            6       834      1.05      1.09      1.08
16:27:17            3       834      1.05      1.09      1.08

[[/file1:]]
Linux 2.6.32-504.el6.x86_64 (dhcp-129-221.nay.redhat.com)       03/08/16
_x86_64_        (8 CPU)

16:27:14      runq-sz  plist-sz   ldavg-1   ldavg-5  ldavg-15
16:27:15            2       834      1.05      1.09      1.08
16:27:16            6       834      1.05      1.09      1.08
16:27:17            3       834      1.05      1.09      1.08
Average:            4       834      1.05      1.09      1.08
```

### Plotting Data

###### Plotting Data with gnuplot

```
# yum -y install gnuplot

# LANG=c sar -q -f /var/log/sa/sa22 > /file1
# awk '/^[^a-zA-Z]+$/ {print $1, $(NF-2), $(NF-1), $NF}' /file1 > /file2

# vim /file.gnuplot
sed xdata time
sed timefmt "%H:%M:%S"
set format "%H:%M:%S"
set xlabel "Time"
set ylabel "Load Average"
set terminal png size 1024,768
set output "/tmp/file.png"
plot '/file2' using 1:2 title "1-load" with lines, '/file2' using 1:3 title "5-load" with lines, '/file2' using 1:4 title "15-load" with lines


# gnuplot -persist /file.gnuplot
# eog /tmp/file.png
```

###### Plotting Data with RRDtool

PDP

CDP

--step=60

rrdtool create xxx.rrd

rrdtool update

rrdtool graph xxx

```
# rrdtool create /tmp/loadave.rrd --step=10 DS:1_min_load_average:GAUGE:30:0:U RRA:AVERAGE:0.5:2:60
# rrdtool update /tmp/loadave.rrd $(date +%s):$(uptime | awk '{print $(NF-2)}' | sed 's/,//')
# rrdtool graph /var/www/html/load_average_hour.png -X 0 --start=$(date --date=-1hour +%s) --end=$(date +%s) DEF:v_1_min=/tmp/loadave.rrd:1_min_load_average:AVERAGE LINE=v_1_min#000000:"1-min-load"
```

RRA round robin archive

DS:5_min_load_average:GAUGE:30:0:U

```
# rrdtool create /tmp/loadave.rrd --step=60 --start=$(date +%s) DS:loadavg1:GAUGE:60:0:U DS:loadavg5:GAUGE:60:0:U DS:loadavg15:GAUGE:60:0:U RRA:AVERAGE:0.5:1:60 RRA:AVERAGE:0.5:30:336
# rrdtool update /tmp/loadave.rrd $(date +%s):$(uptime | awk '{print $(NF-2),$(NF-1),$NF}' |  sed 's/,/:/')
# vim /usr/local/bin/update_loadavg.sh
# chmod +x /usr/local/bin/update_loadavg.sh
# crontab -e
*/1 * * * * /usr/local/bin/update_loadavg.sh
# service crond restart
# crontab -l

# rrdtool lastupdate /tmp/loadave.rrd
# rrdtool info /tmp/loadave.rrd

# rrdtool graph /var/www/html/load_avg_hourly.png -X 0 --start=$(date --date=-1hour +%s) --end=$(date +%s) DEF:ldavg1:/tmp/loadavg.rrd:loadavg1:AVERAGE DEF:ldavg2:/tmp/loadavg.rrd:loadavg5:AVERAGE DEF:ldavg3:/tmp/loadvag.rrd:loadavg15:AVERAGE LINE1=ldavg1#FF0000A0:"1_Min_Load" LINE2=ldavg2#00FF00A0:"5_MIN_Load" LINE3=ldavg3#0000FFA0:"15_Min_Load"

# eog /var/www/html/load_avg_hourly.png
# vim /usr/local/bin/plot_loadavg_hourly.sh
# chmod +x /usr/local/bin/plot_loadavg_hourly.sh
# crontab -l
*/10 * * * * /usr/local/bin/plot_loadavg_hourly.sh
```

