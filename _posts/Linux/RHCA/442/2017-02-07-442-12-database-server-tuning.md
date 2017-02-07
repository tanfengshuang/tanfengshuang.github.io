---
layout: post
title:  "Database Server Tuning(442-12)"
categories: Linux
tags: RHCA 442
---

### Database Server Tuning

*    System memory
*    Disk storage
*    Networking

```
# tuned-adm list
Available profiles:
- server-powersave
- laptop-ac-powersave
- laptop-battery-powersave
- enterprise-storage
- virtual-host
- throughput-performance
- spindown-disk
- latency-performance
- desktop-powersave
- default
- virtual-guest
Current active profile: throughput-performance

# tuned-adm profile latency-performance
Calling '/etc/ktune.d/tunedadm.sh stop': [  OK  ]
Reverting to cfq elevator: dm-0 dm-1 dm-2 vda [  OK  ]
Stopping tuned: [  OK  ]
Switching to profile 'latency-performance'
Applying deadline elevator: dm-0 dm-1 dm-2 vda [  OK  ]
Applying ktune sysctl settings:
/etc/ktune.d/tunedadm.conf: [  OK  ]
Calling '/etc/ktune.d/tunedadm.sh start': [  OK  ]
Applying sysctl settings from /etc/sysctl.conf
Starting tuned: [  OK  ]

# cat /proc/sys/net/ipv4/tcp_low_latency 
0

# cat /sys/block/vda/queue/scheduler                -> 数据库服务器适合deadline算法
noop anticipatory [deadline] cfq 
```

### Tune the Network for Latency

*    net.ipv4.tcp_low_latency: 0 -> 1, set this value to optimize for low network latency.
*    sysctl -w net.ipv4.tcp_low_latency=1

```
# yum install qperf
# qperf  -h
Synopsis
    qperf
    qperf SERVERNODE [OPTIONS] TESTS

Description
    qperf measures bandwidth and latency between two nodes.  It can work
    over TCP/IP as well as the RDMA transports.  On one of the nodes, qperf
    is typically run with no arguments designating it the server node.  One
    may then run qperf on a client node to obtain measurements such as
    bandwidth, latency and cpu utilization.

    In its most basic form, qperf is run on one node in server mode by
    invoking it with no arguments.  On the other node, it is run with two
    arguments: the name of the server node followed by the name of the
    test.  A list of tests can be found in the section, TESTS.  A variety
    of options may also be specified.

    One can get more detailed information on qperf by using the --help
    option.  Below are examples of using the --help option:

        qperf --help examples       Some examples of using qperf
        qperf --help opts           Summary of options
        qperf --help options        Description of options
        qperf --help tests          Short summary and description of tests
        qperf --help TESTNAME       More information on test TESTNAME

# qperf --help examples
In these examples, we first run qperf on a node called myserver in server
mode by invoking it with no arguments.  In all the subsequent examples, we
run qperf on another node and connect to the server which we assume has a
hostname of myserver.
    * To run a TCP bandwidth and latency test:
        qperf myserver tcp_bw tcp_lat
    * To run a SDP bandwidth test for 10 seconds:
        qperf myserver -t 10 sdp_bw
    * To run a UDP latency test and then cause the server to terminate:
        qperf myserver udp_lat quit
    * To measure the RDMA UD latency and bandwidth:
        qperf myserver ud_lat ud_bw
    * To measure RDMA UC bi-directional bandwidth:
        qperf myserver rc_bi_bw
    * To get a range of TCP latencies with a message size from 1 to 64K
        qperf myserver -oo msg_size:1:64K:*2 -vu tcp_lat
```

```
[desktop]# qperf 
[server]# qperf 10.16.98.102 tcp_lat
tcp_lat:
    latency  =  108 us


[desktop]# echo 1 > /proc/sys/net/ipv4/tcp_low_latency 



[desktop]# sysctl -w net.ipv4.tcp_low_latency=1
[server]# perf desktopY tcp_lat
[server]# sysctl -w net.ipv4.tcp_low_latency=1
[server]# qperf desktop tcp_lat
```

### Tune SysV IPCs

System V IPC指的是AT&T在System V.2发行版中引入的三种进程间通信工具:(1)信号量，用来管理对共享资源的访问 (2)共享内存，用来高效地实现进程间的数据共享 (3)消息队列，用来实现进程间数据的传递。我们把这三种工具统称为System V IPC的对象，每个对象都具有一个唯一的IPC标识符(identifier)。要保证不同的进程能够获取同一个IPC对象，必须提供一个IPC关键字(IPC key)，内核负责把IPC关键字转换成IPC标识符。

Memory reserved for inter-process communication(IPC) mechanisms, Red Hat Enterprise Linux supports both the older System V (SysV) style of IPC as well as the newer POSIX IPC mechanisms

*   Semaphores(信号量): allow two or more processes to coordinate access to shared resources
*   Message queues(消息队列): allow processes to cooperatively function by exchanging messages
*   Shared memory regions(共享内存段): allow processes to communicate by reading from and writing to the same region of memory 

> /usr/share/doc/kernel-doc-2.6.32/Documentation/sysctl/kernel.txt

The SysV IPC mechanisms are tuned using entries in /proc/sys/kernel/. The sysctl tunables used are:
*    kernel.shmmni specifies the maximum number of shared memory segments systemwide.
*    kernel.shmall specifies the total amount of shared memory, in pages, that can be used at one time on the system. ipcs -l displays this value after converting it to KiB. This should be at least kernel.shmmax/PAGE_SIZE, where PAGE_SIZE is the page size on the system (4 KiB is typical). PAGE_SIZE can be looked up using getconf PAGE_SIZE.
*    kernel.shmmax specifies the maximum size of a shared memory segment that can be created, in bytes.
*    kernel.msgmnb specifies the maximum number of bytes in a single message queue.
*    kernel.msgmni specifies the maximum number of message queue identifiers.
*    kernel.msgmax specifies the maximum size of a message that can be passed between processes. Note that this memory cannot be swapped.
*    kernel.sem contains settings for:
    1. The maximum number of semaphores per semaphore array.
    2. The maximum number of semaphores allowed systemwide.
    3. The maximum number of allowed operations per semaphore system call.
    4. The maximum number of semaphore arrays.

```
# ipcs
------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status

------ Semaphore Arrays --------
key        semid      owner      perms      nsems


# ipcs -l
------ Messages Limits --------
max queues system wide = 3675
max size of message (bytes) = 8192
default max size of queue (bytes) = 16384

------ Shared Memory Limits --------
max number of segments = 4096
max seg size (kbytes) = 18014398509465599
max total shared memory (kbytes) = 18014398442373116
min seg size (bytes) = 1

------ Semaphore Limits --------
max number of arrays = 128
max semaphores per array = 250
max semaphores system wide = 32000
max ops per semop call = 32
semaphore max value = 32767

# ipcs -m                   -> 申请的共享内存段，一行一个
------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status
0x00000000 196608     ftan       600        4194304    2          dest
0x00000000 229377     ftan       600        524288     2          dest
0x00000000 1507330    ftan       600        1282048    2          dest
0x00000000 753667     ftan       600        6627328    2          dest
0x00000000 819204     ftan       600        6627328    2          dest
0x00000000 851973     ftan       600        6627328    2          dest

# ipcs -m -l
------ Shared Memory Limits --------
max number of segments = 4096
max seg size (kbytes) = 67108864
max total shared memory (kbytes) = 17179869184
min seg size (bytes) = 1

# ipcs -s -l
------ Semaphore Limits --------
max number of arrays = 128
max semaphores per array = 250
max semaphores system wide = 32000
max ops per semop call = 32
semaphore max value = 32767

# ipcs -q -l
------ Messages: Limits --------
max queues system wide = 3747
max size of message (bytes) = 65536
default max size of queue (bytes) = 65536



# ls /proc/sys/kernel/
acct                          hostname                ngroups_max                        panic_on_warn                 sched_child_runs_first       shmmni
acpi_video_flags              hotplug                 nmi_watchdog                       perf_cpu_time_max_percent     sched_domain                 shm_next_id
auto_msgmni                   hung_task_check_count   ns_last_pid                        perf_event_max_sample_rate    sched_latency_ns             shm_rmid_forced
bootloader_type               hung_task_panic         numa_balancing                     perf_event_mlock_kb           sched_migration_cost_ns      softlockup_all_cpu_backtrace
bootloader_version            hung_task_timeout_secs  numa_balancing_scan_delay_ms       perf_event_paranoid           sched_min_granularity_ns     softlockup_panic
cad_pid                       hung_task_warnings      numa_balancing_scan_period_max_ms  pid_max                       sched_nr_migrate             stack_tracer_enabled
cap_last_cap                  io_delay_type           numa_balancing_scan_period_min_ms  poweroff_cmd                  sched_rr_timeslice_ms        sysrq
compat-log                    kexec_load_disabled     numa_balancing_scan_size_mb        print-fatal-signals           sched_rt_period_us           tainted
core_pattern                  keys                    numa_balancing_settle_count        printk                        sched_rt_runtime_us          threads-max
core_pipe_limit               kptr_restrict           osrelease                          printk_delay                  sched_schedstats             timer_migration
core_uses_pid                 kstack_depth_to_print   ostype                             printk_ratelimit              sched_shares_window_ns       traceoff_on_warning
ctrl-alt-del                  max_lock_depth          overflowgid                        printk_ratelimit_burst        sched_time_avg_ms            unknown_nmi_panic
dmesg_restrict                modprobe                overflowuid                        pty                           sched_tunable_scaling        usermodehelper
domainname                    modules_disabled        panic                              random                        sched_wakeup_granularity_ns  version
ftrace_dump_on_oops           msgmax                  panic_on_io_nmi                    randomize_va_space            sem                          watchdog
ftrace_enabled                msgmnb                  panic_on_oops                      real-root-dev                 sem_next_id                  watchdog_cpumask
hardlockup_all_cpu_backtrace  msgmni                  panic_on_stackoverflow             sched_autogroup_enabled       shmall                       watchdog_thresh
hardlockup_panic              msg_next_id             panic_on_unrecovered_nmi           sched_cfs_bandwidth_slice_us  shmmax

```

###### Lab SysV IPCs

```
限制共享内存段的个数
# ipcs -m -l
------ Shared Memory Limits --------
max number of segments = 4096
max seg size (kbytes) = 67108864
max total shared memory (kbytes) = 17179869184
min seg size (bytes) = 1

# sysctl -w kernel.shmmni=5         -> echo 'kernel.shmmni = 5' >> /etc/sysctl.conf    (永久生效)
kernel.shmmni = 5                   -> echo 5 > /proc/sys/kernel/shmmni (临时生效)

# ipcs -m -l
------ Shared Memory Limits --------
max number of segments = 5
max seg size (kbytes) = 67108864
max total shared memory (kbytes) = 17179869184
min seg size (bytes) = 1

# cat /proc/sys/kernel/shmmni 
5

# ipcs -m           -> 重新登录系统，查看共享内存段的个数（不会超过5个）

# sysctl -w kernel.shmmni=4096      -> 恢复默认值
kernel.shmmni = 4096
# ipcs -m -l
------ Shared Memory Limits --------
max number of segments = 4096
max seg size (kbytes) = 67108864
max total shared memory (kbytes) = 17179869184
min seg size (bytes) = 1
```

```
限制共享内存的总大小
1. 通过shmall来限制总大小(512M)
# getconf PAGE_SIZE     -> 获得page大小，为4K
4096
# echo $[512*1024/4]    -> 将512M转换成page大小
131072
# sysctl -w kernel.shmall=131072
kernel.shmall = 131072
# ipcs -m -l
------ Shared Memory Limits --------
max number of segments = 4096
max seg size (kbytes) = 67108864
max total shared memory (kbytes) = 524288           -> 已经变为512M
min seg size (bytes) = 1

# echo $[17179869184/4]                             -> 恢复为原来大小
4294967296
# sysctl -w kernel.shmall=4294967296
kernel.shmall = 4294967296
# ipcs -m -l
------ Shared Memory Limits --------
max number of segments = 4096
max seg size (kbytes) = 67108864
max total shared memory (kbytes) = 17179869184
min seg size (bytes) = 1

2. 通过shmmni和shmmax限制最大数量和单个内存段大小来达到限制总大小的效果 - 例如总量想达到512M，每个共享内存段shmmax的大小限制为3M，则shmmni=512/3=170
这种限制方法不如上面那种好，如果申请的每个内存段大小为2M，申请170个，最多才能到340M；如果申请的单个内存段大于3M，也达不到最多数量170
# echo $[3*1024*1024]
3145728
# sysctl -w kernel.shmmni=170
kernel.shmmni=170
# sysctl -w kernel.shmmax=3145728
kernel.shmmax=3145728
# ipcs -m -l
------ Shared Memory Limits --------
max number of segments = 170
max seg size (kbytes) = 3072
max total shared memory (kbytes) = 17179869184
min seg size (bytes) = 1
```

### Huge Pages

The Linux kernel supports large-sized memory pages through the huge pages mechanism(sometimes known as bigpages, largepages or the hugetlbfs file system).

*    Red Hat Enterprise Linux 6 and 7 use a default huge page size of 2 MiB.
*    The requested pages will only be allocated if there is enough free, contiguous memory to satisfy the request.
*    In order to allocate huge memory pages, set the total number of huge pages required using vm.nr_hugepages sysctl parameter

Since huge pages must be allocated as contiguous memory, it can be difficult to allocate them during runtime on a machine, as memory will become more fragmented over time as the
machine runs. For this reason, an alternative way of allocating huge pages is to specify the number of huge pages required at boot time by placing the parameter in /etc/sysctl.conf.

If administrators still find it difficult to allocate the number of huge pages required though the sysctl mechanism, they may alternatively pass the hugepages= argument on the kernel
command line through grub. A kernel command-line argument is also how an administrator may change the huge page size from the default 2 MiB (hugepagesz=).

```
# cat /proc/meminfo  | grep ^Huge
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
# cat /proc/sys/vm/nr_hugepages
0

# sysctl -w vm.nr_hugepages=20      -> In order to allocate huge memory pages, set the total number of huge pages required
vm.nr_hugepages = 20

# cat /proc/meminfo  | grep ^Huge
HugePages_Total:      20
HugePages_Free:       20
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
```

###### mmap shmat shmget

In order to use the large pages, processes must request them using either the mmap system call or the shmat and shmget system calls. If mmap is used, then the large pages must be made available via the hugetlbfs file system:

```
1. mmap
# mkdir /largepage
# mount -t hugetlbfs none /largepage
# mount
/dev/mapper/vg_cloudqe16vm01-lv_root on / type ext4 (rw)
proc on /proc type proc (rw)
sysfs on /sys type sysfs (rw)
devpts on /dev/pts type devpts (rw,gid=5,mode=620)
tmpfs on /dev/shm type tmpfs (rw,rootcontext="system_u:object_r:tmpfs_t:s0")
/dev/vda1 on /boot type ext4 (rw)
/dev/mapper/vg_cloudqe16vm01-lv_home on /home type ext4 (rw)
none on /proc/sys/fs/binfmt_misc type binfmt_misc (rw)
none on /hugepagedir type hugetlbfs (rw)

2. shmat, shmget可以直接在程序中使用，只要系统中开启了大页服务即可
```

###### Transparent Huge Pages

*    Red Hat Enterprise Linux 6.2 introduced
*    Transparent huge pages(THP) are enabled by default and they are managed by the kernel automatically
*    They can also be swapped out of memory, unlike huge pages
*    The khugepaged daemon automatically starts when transparent_hugepage/enabled is set to always or madvise and it automatically shuts down when it is set to never.
*    To disable THP at boot time, append the following to the kernel command line in grub.conf: transparent_hugepage=never
*    ransparent huge pages are not recommended for use with database workloads; instead, if huge pages are desired, they should be preallocated with the sysctl vm.nr_hugepages parameter.

```
# cat /sys/kernel/mm/transparent_hugepage/enabled 
[always] madvise never

# grep AnonHugePages /proc/meminfo
AnonHugePages:      4096 kB

```

> /usr/share/doc/kernel-doc-2.6.32/Documentation/vm/hugetlbpage.txt

> /usr/share/doc/kernel-doc-2.6.32/Documentation/sysctl/vm.txt

[How to use, monitor, and disable transparent hugepages in Red Hat Enterprise Linux 6?](https://access.redhat.com/solutions/46111)
[[RHEL6] How do I allocate hugepages in a single NUMA node?](https://access.redhat.com/solutions/62862)

### Tuning Overcommit

*    Some applications require the kernel to allocate more memory for a program than is available on the system
*    A machine's memory overcommitment policy is set using the vm.overcommit_memory tunable.

vm.overcommit_memory values:
*    0 - a heuristic overcommit algorithm is used. If an application tries to allocate more memory than could obviously be granted, the allocation will be refused. However, the kernel may still overcommit memory if a large number of small allocations are requested by processes.
*    1 - always overcommit memory when it is requested. The kernel will always grant memory allocations, regardless of whether sufficient free memory exists.
*    2 - do not overcommit. The kernel will only commit an amount of memory equal to the amount of swap space plus a percentage (the default is 50) of physical memory. The percentage of physical memory allowed to be overcommitted is specified in the vm.overcommit_ratio tunable.

```
# cat /proc/sys/vm/overcommit_memory 
0

# man proc      -> 搜索 overcommit_memory
        /proc/sys/vm/overcommit_memory
              This file contains the kernel virtual memory accounting mode.  Values are:
                     0: heuristic overcommit (this is the default)
                     1: always overcommit, never check
                     2: always check, never overcommit

              In  mode  0,  calls  of  mmap(2)  with MAP_NORESERVE are not checked, and the default check is very weak, leading to the risk of getting a process "OOM-
              killed".  Under Linux 2.4 any non-zero value implies mode 1. In mode 2 (available since Linux 2.6), the total virtual address space on the system is
              limited to (SS  +  RAM*(r/100)), where SS is the size of the swap space, and RAM is the size of the physical memory, and r is the contents of the file
              /proc/sys/vm/overcommit_ratio.
              
       /proc/sys/vm/overcommit_ratio
              See the description of /proc/sys/vm/overcommit_memory.


the total virtual address space = (SS + RAM*(r/100)) => SS(swap size), RAM(physical memory size), r(/proc/sys/vm/overcommit_ratio)

# sysctl -w vm.overcommit_memory=2
vm.overcommit_memory=2
# cat /proc/sys/vm/overcommit_ratio 
50
# free
             total       used       free     shared    buffers     cached
Mem:       1922096    1592740     329356        308      64212    1168528
-/+ buffers/cache:     360000    1562096
Swap:      4128764          0    4128764

# echo $[4128764 + 1922096/2]
5089812

# grep ^Commit /proc/meminfo 
CommitLimit:     5069332 kB
Committed_AS:     203680 kB


# echo 200 > /proc/sys/vm/overcommit_ratio 
# cat /proc/sys/vm/overcommit_ratio
200
# free
             total       used       free     shared    buffers     cached
Mem:       1922096    1592632     329464        308      64244    1168532
-/+ buffers/cache:     359856    1562240
Swap:      4128764          0    4128764
# echo $[4128764 + 1922096*2]
7972956
# grep ^Commit /proc/meminfo 
CommitLimit:     7891036 kB
Committed_AS:     203680 kB
```

### Tuning Swappiness

*    As discussed in the Large Memory Tuning unit, the vm.swappiness tunable determines how the system balances between swapping pages to disk (or dropping memory-mapped pages) versus reclaiming memory from the page cache.
*    This tunable is also important for database workloads. Force the kernel to be more aggressive in swapping memory. 
*    Set the tunalbe to its maximum value (100)


### Practice

*    Low latency tuned profile
*    I/O scheduler for database
*    Low latency TCP network settings
*    The sum of all share memory sements equals to 2 GiB
*    Allow an application to allocate exactly 256MiB of huge pages
*    Never allow the memory to overcommit more than 100% RAM plus swap size
*    Configure desktopX to also provide a low latency TCP network
