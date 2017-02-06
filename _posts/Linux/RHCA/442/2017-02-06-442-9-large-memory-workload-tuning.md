---
layout: post
title:  "Large Memory Workload Tuning(442-9)"
categories: Linux
tags: RHCA 442
---

### Memory Management

###### Introduction to memory and paging

*    For efficiency, memory on a computer system is organized into fixed-size chunks called pages.
*    The physical RAM on a system is divided into page frames
*    One page frame holds one page of data.
*    Processes do not address physical memory directly. Instead, each process has a virtual address space.
*    The size of a process's virtual address space depends on the processor architecture. On a 32-bit i386 system, a process's virtual address space can hold 2^32 bytes(4 GiB) of memory; on a 64-bit i386 system, a process's virtual address space can hold 2^64 bytes(16 EiB) of memory


> How can you tell how much memory a process is using?

Generally, when a process requests memory, it reserves virtual memory addresses but does not actually map them to physical page frames until they are first used.

Tools such as ps and top distinguish between two statistics: "VIRT" or "VSIZE", the total amount of virtual memory a process has asked for, and "RES" or "RSS", the total amount of virtual memory that a process is currently mapping to physical memory. Normally, "RSS" is the more critical value because it represents memory actually allocated and mapped.


```
# ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND                    -> VSZ RSS
root         1  0.0  0.0  19368  1428 ?        Ss   Feb03   0:01 /sbin/init
root         2  0.0  0.0      0     0 ?        S    Feb03   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        S    Feb03   0:01 [migration/0]

# top
  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND                        -> VIRT RES
    1 root      20   0 19368 1428 1108 S  0.0  0.1   0:01.05 init
    2 root      20   0     0    0    0 S  0.0  0.0   0:00.10 kthreadd
    3 root     -99   0     0    0    0 S  0.0  0.0   0:01.39 migration/0 

# pmap $$           -> pmap命令用于报告进程的内存映射关系，是Linux调试及运维一个很好的工具
26191:   -bash
0000000000400000    852K r-x--  /bin/bash
00000000006d4000     36K rw---  /bin/bash
00000000006dd000     24K rw---    [ anon ]
00000000008dc000     36K rw---  /bin/bash
0000000001a13000    264K rw---    [ anon ]
0000003948c00000    128K r-x--  /lib64/ld-2.12.so
0000003948e1f000      8K r----  /lib64/ld-2.12.so
0000003948e21000      4K rw---  /lib64/ld-2.12.so
0000003948e22000      4K rw---    [ anon ]
0000003949000000      8K r-x--  /lib64/libdl-2.12.so
0000003949002000   2048K -----  /lib64/libdl-2.12.so
0000003949202000      4K r----  /lib64/libdl-2.12.so
0000003949203000      4K rw---  /lib64/libdl-2.12.so
0000003949400000   1576K r-x--  /lib64/libc-2.12.so
000000394958a000   2048K -----  /lib64/libc-2.12.so
000000394978a000     16K r----  /lib64/libc-2.12.so
000000394978e000      8K rw---  /lib64/libc-2.12.so
0000003949790000     16K rw---    [ anon ]
000000394d800000    116K r-x--  /lib64/libtinfo.so.5.7
000000394d81d000   2044K -----  /lib64/libtinfo.so.5.7
000000394da1c000     16K rw---  /lib64/libtinfo.so.5.7
000000394da20000      4K rw---    [ anon ]
00007ff37479a000     52K r-x--  /lib64/libnss_files-2.12.so
00007ff3747a7000   2044K -----  /lib64/libnss_files-2.12.so
00007ff3749a6000      4K r----  /lib64/libnss_files-2.12.so
00007ff3749a7000      4K rw---  /lib64/libnss_files-2.12.so
00007ff3749a8000  96844K r----  /usr/lib/locale/locale-archive
00007ff37a83b000     12K rw---    [ anon ]
00007ff37a841000      8K rw---    [ anon ]
00007ff37a843000     28K r--s-  /usr/lib64/gconv/gconv-modules.cache
00007ff37a84a000      4K rw---    [ anon ]
00007ffec9cdf000     84K rw---    [ stack ]
00007ffec9d30000      4K r-x--    [ anon ]
ffffffffff600000      4K r-x--    [ anon ]
 total           108356K

# pmap $$ | tail -n 1
 total           108356K
# ps aux | grep $$
root      7806  0.0  0.0 103320   824 pts/1    S+   06:35   0:00 grep 26191
root     26191  0.0  0.0 108356  1804 pts/1    Ss   Feb05   0:00 -bash
# ps aux | head -n 1
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
# echo $$
26191
```

###### Page Tables and the TLB(Translation Lookaside Buffer)

Since each process maintains its own virtual address space, each process needs its own table of mappings of the virtual addresses of pages to the physical addresses of page frames in RAM.

When a process tries to access a page of virtual memory for which a physical page has not yet been allocated, it will trigger a page fault.
*    主页缺失(major page fault): If the kernel needs to retrieve a page of memory from disk, it is called a major page fault
*    次页缺失(minor page fault): If a new page of physical memory needs to be allocated, this is called a minor page fault

```
# x86info -c
x86info v1.30.  Dave Jones 2001-2011
Feedback to <davej@redhat.com>.

Found 8 identical CPUs
Extended Family: 0 Extended Model: 2 Family: 6 Model: 42 Stepping: 7
Type: 0 (Original OEM)
CPU Model (x86info's best guess): Unknown model. 
Processor name string (BIOS programmed): Intel(R) Core(TM) i7-2600 CPU @ 3.40GHz

Cache info
TLB info
 Instruction TLB: 4K pages, 4-way associative, 64 entries.
 Data TLB: 4KB or 4MB pages, fully associative, 32 entries.
 Data TLB: 4KB pages, 4-way associative, 64 entries
 Data TLB: 4K pages, 4-way associative, 512 entries.
 Data TLB: 4KB or 4MB pages, fully associative, 32 entries.
 Data TLB: 4KB pages, 4-way associative, 64 entries
 64 byte prefetching.
 Data TLB: 4K pages, 4-way associative, 512 entries.
Found unknown cache descriptors: 76 ff 
Total processor threads: 8
This system has 1 quad-core processor with hyper-threading (2 threads per core) running at an estimated 3.40GHz

# ps o pid,comm,minflt,majflt $$
  PID COMMAND         MINFLT MAJFLT
 7051 bash              1656      0

# ps o pid,comm,minflt,majflt `pidof su`
  PID COMMAND         MINFLT MAJFLT
 6886 su                 624      7
 6920 su                 901      3
 6969 su                 630      0
 7045 su                 904      0

```

```
# cat /proc/cgroups 
#subsys_name	hierarchy	num_cgroups	enabled
cpuset	0	1	1
ns	0	1	1
cpu	0	1	1
cpuacct	0	1	1
memory	0	1	1
devices	0	1	1
freezer	0	1	1
net_cls	0	1	1
blkio	0	1	1
perf_event	0	1	1
net_prio	0	1	1

# ls /cgroup/memory/
cgroup.event_control  memory.force_empty         memory.memsw.failcnt             memory.memsw.usage_in_bytes      memory.soft_limit_in_bytes  memory.usage_in_bytes  release_agent
cgroup.procs          memory.limit_in_bytes      memory.memsw.limit_in_bytes      memory.move_charge_at_immigrate  memory.stat                 memory.use_hierarchy   tasks
memory.failcnt        memory.max_usage_in_bytes  memory.memsw.max_usage_in_bytes  memory.oom_control               memory.swappiness           notify_on_release

# vim /etc/cgconfig.conf
group bigmem {
    memory {
        memory.limit_in_bytes = 256m;           # 只限制了物理内存，当实际使用时，如果物理内存不够用了，会去用交换分区
    }
}
group bigmem2 {
    memory {
        memory.limit_in_bytes = 256m;           # 仅物理内存
        memory.memsw.limit_in_bytes = 256m;     # 物理内存加交换区
    }
}
# service cgconfig restart
Stopping cgconfig service: [  OK  ]
Starting cgconfig service: [  OK  ]
# cat /cgroup/memory/bigmem/memory.limit_in_bytes 
268435456
# cat /cgroup/memory/bigmem/memory.memsw.limit_in_bytes 
9223372036854775807
# cat /cgroup/memory/bigmem2/memory.limit_in_bytes 
268435456
# cat /cgroup/memory/bigmem2/memory.memsw.limit_in_bytes 
268435456
# watch -n .1 free -m

# cgexec -g memory:bigmem bigmem 512        -> 限制内存在cgroup bigmem下，使用bigmem申请物理内存512m，成功
# cgexec -g memory:bigmem2 bigmem 512       -> 限制内存在cgroup bigmem2下，使用bigmem申请物理内存512m，失败

# cgexec -g memory:bigmem bigmem -v 512        -> 限制内存在cgroup bigmem下，使用bigmem申请虚拟内存512m，成功
# cgexec -g memory:bigmem2 bigmem -v 512       -> 限制内存在cgroup bigmem2下，使用bigmem申请虚拟内存512m，成功
# cgexec -g memory:bigmem2 bigmem -v 10240       -> 限制内存在cgroup bigmem2下，使用bigmem申请虚拟内存10240m，成功
```

### Finding Memory leaks

###### Two different type of Memory Leaks

*    In the first case a program request memory with a system call link malloc, but doesn't actually use this memory - virutal size goes up(VIRT in top, The Committed_AS line in /proc/meminfo will also increase, but no actual physical memory is used), resident size stays almost the same(RSS in top)
*    In the second case the program actually uses the memory it allocates - This causes the resident size to go up in step with the virtual size, causing a actual memory shortage

```
# watch -n 1 'free -m ; grep Committed_AS /proc/meminfo' 
# watch -d -n1 'free -m; grep -i commit /proc/meminfo'

# valgrind --tool=memcheck bigmem -v 256 -> 申请虚拟内存
# valgrind --tool=memcheck bigmem 256 -> 申请物理内存

# bigmem -v 256 -> 申请虚拟内存
# bigmem 256    -> 申请物理内存
```

```
# valgrind --tool=memcheck ls
==14337== Memcheck, a memory error detector
==14337== Copyright (C) 2002-2012, and GNU GPL'd, by Julian Seward et al.
==14337== Using Valgrind-3.8.1 and LibVEX; rerun with -h for copyright info
==14337== Command: ls
==14337== 
ks-script-HHRJcJ      vgdb-pipe-from-vgdb-to-14337-by-root-on-cloud-qe-16-vm-01.idmqe.lab.eng.bos.redhat.com
ks-script-HHRJcJ.log  vgdb-pipe-shared-mem-vgdb-14337-by-root-on-cloud-qe-16-vm-01.idmqe.lab.eng.bos.redhat.com
ks-script-JddDSm      vgdb-pipe-to-vgdb-from-14337-by-root-on-cloud-qe-16-vm-01.idmqe.lab.eng.bos.redhat.com
systop.ko	      yum.log
tmp.Nv1049
==14337== 
==14337== HEAP SUMMARY:
==14337==     in use at exit: 21,676 bytes in 15 blocks
==14337==   total heap usage: 49 allocs, 34 frees, 58,216 bytes allocated
==14337== 
==14337== LEAK SUMMARY:
==14337==    definitely lost: 0 bytes in 0 blocks
==14337==    indirectly lost: 0 bytes in 0 blocks
==14337==      possibly lost: 0 bytes in 0 blocks
==14337==    still reachable: 21,676 bytes in 15 blocks
==14337==         suppressed: 0 bytes in 0 blocks
==14337== Rerun with --leak-check=full to see details of leaked memory
==14337== 
==14337== For counts of detected and suppressed errors, rerun with: -v
==14337== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 6 from 6)

# valgrind --leak-check=full ls
==16355== Memcheck, a memory error detector
==16355== Copyright (C) 2002-2012, and GNU GPL'd, by Julian Seward et al.
==16355== Using Valgrind-3.8.1 and LibVEX; rerun with -h for copyright info
==16355== Command: ls
==16355== 
ks-script-HHRJcJ      vgdb-pipe-from-vgdb-to-16355-by-root-on-cloud-qe-16-vm-01.idmqe.lab.eng.bos.redhat.com
ks-script-HHRJcJ.log  vgdb-pipe-shared-mem-vgdb-16355-by-root-on-cloud-qe-16-vm-01.idmqe.lab.eng.bos.redhat.com
ks-script-JddDSm      vgdb-pipe-to-vgdb-from-16355-by-root-on-cloud-qe-16-vm-01.idmqe.lab.eng.bos.redhat.com
systop.ko	      yum.log
tmp.Nv1049
==16355== 
==16355== HEAP SUMMARY:
==16355==     in use at exit: 21,676 bytes in 15 blocks
==16355==   total heap usage: 49 allocs, 34 frees, 58,216 bytes allocated
==16355== 
==16355== LEAK SUMMARY:
==16355==    definitely lost: 0 bytes in 0 blocks
==16355==    indirectly lost: 0 bytes in 0 blocks
==16355==      possibly lost: 0 bytes in 0 blocks
==16355==    still reachable: 21,676 bytes in 15 blocks
==16355==         suppressed: 0 bytes in 0 blocks
==16355== Reachable blocks (those to which a pointer was found) are not shown.
==16355== To see them, rerun with: --leak-check=full --show-reachable=yes
==16355== 
==16355== For counts of detected and suppressed errors, rerun with: -v
==16355== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 6 from 6)
```

### Tuning Swap

The vmstat utility can provide information on whether a system is paging to swap ("swapping") or not. The critical columns in the output of vmstat are si and so, pages swapped in per second and pages swapped out per second.

```
# vmstat 1
procs -----------memory---------- ---swap-- -----io---- --system-- -----cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 0  0      0 485604  55376 1123548    0    0     2     8   15   13  0  0 100  0  0	
 0  0      0 485340  55376 1123576    0    0     0     0  451  125  2  3 95  0  0	
 0  0      0 485356  55376 1123576    0    0     0     0  462  119  3  3 95  0  0	
```

###### System Memory and Page Cache

Processes are not the only consumer of system memory. The kernel may use memory for its own code or to speed up the system in other ways. One of these ways is the page cache.

```
# free
             total       used       free     shared    buffers     cached
Mem:       1922096    1436600     485496        216      55384    1123576
-/+ buffers/cache:     257640    1664456
Swap:      4128764          0    4128764
```

###### Swappiness

> swap_tendency = mapped_ratio/2 + distress(内核释放内存的难度，难度越大数值越大) + vm_swappiness

If swap_tendency is below 100, the kernel will reclaim a page from the page cache; if it is 100 or above, pages which are part of a process memory space will become eligible for swap.

*    mapped_ratio is the percentage of physical memory in use. 
*    distress(0 ~ 100) is a measure of how much trouble the kernel has in freeing memory. It will start out at 0, but if more attempts are necessary to free memory, it will be increased (to a maximum of 100). 
*    The vm_swappiness value comes from the sysctl parameter, vm.swappiness.

*    swap_tendency < 100  使用page cache - 1123576
*    swap_tendency >= 100 使用swap - 4128764
*    所以，可以通过设置vm_swappiness的值，来影响swap_tendency，如果想使用page cache，可以将vm_swappiness的值设置的小一些，这样子swap_tendency的值可以不容易超过100；如果想使用swap分区，可以将vm_swappiness的值设置的大一些
*    对于I/O操作比较的频繁的系统，不建议使用page cache


```
# free
             total       used       free     shared    buffers     cached
Mem:       1922096    1436600     485496        216      55384    1123576
-/+ buffers/cache:     257640    1664456
Swap:      4128764          0    4128764

# bc
bc 1.06.95
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'. 
scale=8
1436600/1922096
.74741324

74%/2 + distress + vm_swappiness

# cat /proc/sys/vm/swappiness 
60
# sysctl -w vm.swappiness=70
vm.swappiness = 70
# cat /proc/sys/vm/swappiness 
70


# echo vm.swappiness=80 >> /etc/sysctl.conf
# grep vm /etc/sysctl.conf
vm.swappiness=80
# sysctl -p
net.ipv4.ip_forward = 0
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.default.accept_source_route = 0
kernel.sysrq = 0
kernel.core_uses_pid = 1
net.ipv4.tcp_syncookies = 1
kernel.msgmnb = 65536
kernel.msgmax = 65536
kernel.shmmax = 68719476736
kernel.shmall = 4294967296
vm.swappiness = 80
```

###### Optimizing Swap Spaces


Swap performance is heavily infulenced by the location and number of swap spaces. On a spinning hard dirve, placing a swap partition at the outer edge of a platter will give better throughput than placing it near the hub, due to the ZCAV(区域恒定角速度) effects. Placing a swap space on SSD storage may result in better performance due to its lower latency and higher throughput per device.

If using an SSD for swap space, and the swap space is frequently used, ensure that the device supports appropriate wear leveling and can sustain a large number of writes before failure. (SLC-based storage may be preferred over MLC-based storage because of this.) Otherwise, the device may experience premature wear.

When multiple swap spaces are in use, the mount option pri=value can be used to specify the priority of use for each space. Swap spaces with a higher pri value will be filled up first before moving on to one with a lower priority. This allows a faster disk to be prioritized over a slower one.

When multiple swap spaces are activated with the same priority, they will be used in a round-robin fashion. This reduces the visit count per individual swap space and results in better performance.

```
# swapon 

Usage:
 swapon [options] [<special>]
 -a, --all                enable all swaps from /etc/fstab
 -d, --discard[=<policy>] enable swap discards, if supported by device
 -e, --ifexists           silently skip devices that do not exist
 -f, --fixpgsz            reinitialize the swap space if necessary
 -p, --priority <prio>    specify the priority of the swap device
 -s, --summary            display summary about used swap devices
 -h, --help               display help
 -V, --version            display version
 -v, --verbose            verbose output

The <special> parameter:
 {-L label | LABEL=label}             LABEL of device to be used
 {-U uuid  | UUID=uuid}               UUID of device to be used
 <device>                             name of device to be used
 <file>                               name of file to be used

# swapon -s
Filename				Type		Size	Used	Priority
/dev/dm-1                               partition	4128764	0	-1

# vim /etc/fstab
/dev/mapper/vg_cloudqe16vm01-lv_swap swap                    swap    defaults        0 0

# swapon -p 1 /dev/vdb1
# swapon -p 2 /dev/vdb2

```

### Memory Reclamation

Physical memory needs to be reclaimed from time to time to stop memory from filling up, rendering a system unusable.

Memory page states:

*    Free: The page is available for immediate allocation.
*    Inactive clean: The page is not in active use and the content corresponds to the content on disk because it has already been written back or has not changed since being read.
*    Inactive dirty: The page is not in active use but the page content has been modified since being read from disk and has not yet been written back.
*    Active: The page is in active use and not a candidate for being freed.

Pages that are marked as Inactive clean can be treated as free pages when a new page needs to be allocated, but if the process that owns the page later needs it again, a major page fault will occur.


```
# grep -i active /proc/meminfo 
Active:           363200 kB
Inactive:         902536 kB
Active(anon):      12436 kB
Inactive(anon):    73608 kB
Active(file):     350764 kB
Inactive(file):   828928 kB
```

###### pdflush

In older kernels, writing out dirty pages to disk was handled by kernel threads called pdflush that would be created when necessary. In newer kernels (including those in Red Hat Enterprise Linux 6), pdflush has been replaced with per-BDI flush threads (BDI = Backing Device Interface). Per-BDI flush threads will show up in the process list as flush-MAJOR:MINOR.

*    vm.dirty_expire_centisecs: How old (in 1/100ths of a second) dirty data must be before it is eligible for being written out to disk. This stops the kernel from writing out the same page multiple times in quick succession just because a process modifies another byte of memory.
*    vm.dirty_writeback_centisecs: How often (in 1/100ths of a second) the kernel will wake up the flushing threads to write out data. Setting this to 0 will disable periodic writeback completely.
*    vm.dirty_background_ratio: The percentage of total system memory being dirty at which the kernel will start writing out data in the background.
*    vm.dirty_ratio: The percentage of total system memory being dirty at which a process generating writes will block and write out dirty pages.
*    There are two more tunables, vm.dirty_bytes and vm.dirty_background_bytes, which will replace the accompanying ratio tunables when set.

```
# ps aux | grep flush
root       414  0.0  0.0      0     0 ?        S    Feb03   0:00 [kdmflush]
root       416  0.0  0.0      0     0 ?        S    Feb03   0:00 [kdmflush]
root       924  0.0  0.0      0     0 ?        S    Feb03   0:00 [kdmflush]
root     23198  0.0  0.0 103324   824 pts/0    S+   08:38   0:00 grep flush

# cat /proc/sys/vm/dirty_expire_centisecs 
3000            -> 30s, 如果到了30s，脏数据还没有写入，会被丢弃
# cat /proc/sys/vm/dirty_writeback_centisecs 
500             -> 5s，每5s会执行一次写操作，写入脏数据
# cat /proc/sys/vm/dirty_background_ratio 
10              -> 当所有进程使用的内存达到10%时，虽然没有达到5s的写时间，仍然会执行写操作
# cat /proc/sys/vm/dirty_ratio 
40              -> 当某个进程使用的内存达到40%时，虽然没有达到5s的写时间，仍然会执行写操作

# cat /proc/sys/vm/dirty_bytes 
0
# cat /proc/sys/vm/dirty_background_bytes 
0
```

###### Memory Zones and the "OOM Killer"

It is possible for the system to be in an out-of-memory condition while free still reports memory available. This is because sometimes memory is needed from a particular memory zone that is not available.

When a process or the kernel requests a memory allocation, it may need it to be located in "low" memory, at a particular physical memory address or lower.

*    64-bit x86_64 physical memory: ZONE_DMA   ZONE_DMA32      ZONE_NORMAL
*    32-bit i386 physical memory: ZONE_DMA   ZONE_NORMAL    ZONE_HIGHMEM
 
The /proc/buddyinfo file shows how many contiguous pages are available in each size for each memory zone. If a zone has all zeros for its entire line, that zone is out of memory, and there is a risk of the OOM killer firing if the system cannot reclaim memory from other sources such as the page cache and swapping. 

```
# cat /proc/buddyinfo
Node 0, zone      DMA     11      1      2      3      1      0      2      1      3      2      0 
Node 0, zone    DMA32     74   1023    905    291    164    147     71     39     50     35     60 

# cat /proc/buddyinfo           -> ample output of a /proc/buddyinfo very short on ZONE_NORMAL
Node 0, zone      DMA      3      1      3      1      3      2      2      1      2      2      2 
Node 0, zone   Normal      0      2      0      0      0      0      0      0      0      0      0 
Node 0, zone  Highmem   1822    143      0      0      0      0      1      0      0      0      0
```

当次缺页发生时，没有空闲内存可以分配时，发生内存超出

```
# cat /proc/sys/vm/panic_on_oom             -> 为0时，如果内存超出，OOM会杀进程，为1，当内存超出时，直接内核恐慌，不会杀进程
0
# echo 1 > /proc/sys/vm/panic_on_oom

# cat /proc/1/oom_adj                       -> oom_adj(-17 ~ 15), 0是默认值，数值越小，越不会被杀(-17永远不会被杀)；数值越大，越会优先被杀
0

# ps -aux | grep httpd
Warning: bad syntax, perhaps a bogus '-'? See /usr/share/doc/procps-3.2.8/FAQ
root     21107  0.0  0.2 177404  3848 ?        Ss   11:34   0:00 /usr/sbin/httpd
apache   21109  0.0  0.1 177404  2456 ?        S    11:34   0:00 /usr/sbin/httpd
apache   21110  0.0  0.1 177404  2472 ?        S    11:34   0:00 /usr/sbin/httpd
apache   21111  0.0  0.1 177404  2456 ?        S    11:34   0:00 /usr/sbin/httpd
apache   21112  0.0  0.1 177404  2456 ?        S    11:34   0:00 /usr/sbin/httpd
apache   21113  0.0  0.1 177404  2456 ?        S    11:34   0:00 /usr/sbin/httpd
apache   21114  0.0  0.1 177404  2456 ?        S    11:34   0:00 /usr/sbin/httpd
apache   21115  0.0  0.1 177404  2456 ?        S    11:34   0:00 /usr/sbin/httpd
apache   21116  0.0  0.1 177404  2456 ?        S    11:34   0:00 /usr/sbin/httpd
root     21175  0.0  0.0 103320   824 pts/4    S+   11:34   0:00 grep httpd


# echo 15 > /proc/21107/oom_adj
# echo f > /proc/sysrq-trigger          -> 强制让OOM Killer工作，这时OOM Killer会至少杀一个进程
# dmesg
[ pid ]   uid  tgid total_vm      rss cpu oom_adj oom_score_adj name
[  586]     0   586     2780      326   0     -17         -1000 udevd
[ 1262]     0  1262     2281      247   1       0             0 dhclient
[ 1319]     0  1319     6900      202   1     -17         -1000 auditd
[ 1353]     0  1353    62272      404   1       0             0 rsyslogd
[ 1387]     0  1387     4565      174   1       0             0 irqbalance
[ 1405]    32  1405     4746      211   1       0             0 rpcbind
[ 1427]    29  1427     5839      317   1       0             0 rpc.statd
[ 1461]    81  1461     7954      299   1       0             0 dbus-daemon
[ 1483]     0  1483    47245      788   1       0             0 cupsd
[ 1515]     0  1515     1021      160   0       0             0 acpid
[ 1527]    68  1527     9474     1465   0       0             0 hald
[ 1528]     0  1528     5100      331   1       0             0 hald-runner
[ 1561]     0  1561     5630      318   1       0             0 hald-addon-inpu
[ 1570]    68  1570     4502      280   0       0             0 hald-addon-acpi
[ 6628]     0  6628    16560      309   1     -17         -1000 sshd
[ 6655]    38  6655     7686      502   1       0             0 ntpd
[ 6676]     0  6676    23263      681   0       0             0 sendmail
[ 6686]    51  6686    20082      532   0       0             0 sendmail
[ 6714]     0  6714    45759      615   1       0             0 abrtd
[ 6726]     0  6726    29219      342   0       0             0 crond
[ 6741]     0  6741     5278      126   0       0             0 atd
[ 6757]     0  6757    27088      176   0       0             0 rhsmcertd
[ 6772]     0  6772    59990     4197   1       0             0 beah-srv
[ 6801]     0  6801    81668     5398   1       0             0 beah-beaker-bac
[ 6822]     0  6822    55720     3664   0       0             0 beah-fwd-backen
[ 7330]     0  7330    37553     4860   0       0             0 beah-rhts-task
[ 7537]     0  7537     1017      145   0       0             0 mingetty
[ 7539]     0  7539     1017      144   0       0             0 mingetty
[ 7541]     0  7541     1017      145   0       0             0 mingetty
[ 7543]     0  7543     1017      145   0       0             0 mingetty
[ 7545]     0  7545     1017      144   0       0             0 mingetty
[ 7547]     0  7547     1017      145   0       0             0 mingetty
[26158]     0 26158    25522     1033   1       0             0 sshd
[26164]     0 26164    27089      445   1       0             0 bash
[26187]     0 26187    25523     1034   1       0             0 sshd
[26191]     0 26191    27089      455   1       0             0 bash
[26711]     0 26711     2779      293   0     -17         -1000 udevd
[26712]     0 26712     2779      287   0     -17         -1000 udevd
[27731]     0 27731    25523     1040   0       0             0 sshd
[27735]     0 27735    27089      445   0       0             0 bash
[ 6886]     0  6886    40851      495   1       0             0 su
[ 6887]   501  6887    27089      452   1       0             0 bash
[ 6920]   501  6920    40884      679   1       0             0 su
[ 6926]     0  6926    27089      434   1       0             0 bash
[ 6969]     0  6969    40851      496   0       0             0 su
[ 6970]   501  6970    27089      447   1       0             0 bash
[ 7045]   501  7045    40884      679   0       0             0 su
[ 7051]     0  7051    27089      450   1       0             0 bash
[ 7762]     0  7762    44286     1523   1       0             0 tuned
[ 8305]     0  8305    27055      483   0       0             0 watch
[12326]     0 12326    34870     1320   1       0             0 vim
[18885]     0 18885    25519     1039   1       0             0 sshd
[19078]     0 19078    27089      448   1       0             0 bash
[19125]     0 19125    25519     1037   1       0             0 sshd
[19177]     0 19177    27089      438   0       0             0 bash
[21107]     0 21107    44351      962   1      15          1000 httpd
[21109]    48 21109    44351      614   1       0             0 httpd
[21110]    48 21110    44351      618   0       0             0 httpd
[21111]    48 21111    44351      614   0       0             0 httpd
[21112]    48 21112    44351      614   1       0             0 httpd
[21113]    48 21113    44351      614   0       0             0 httpd
[21114]    48 21114    44351      614   1       0             0 httpd
[21115]    48 21115    44351      614   1       0             0 httpd
[21116]    48 21116    44351      614   1       0             0 httpd
Out of memory: Kill process 21107 (httpd) score 1000 or sacrifice child
Killed process 21109, UID 48, (httpd) total-vm:177404kB, anon-rss:1780kB, file-rss:676kB

# ps -aux | grep httpd
Warning: bad syntax, perhaps a bogus '-'? See /usr/share/doc/procps-3.2.8/FAQ
root     21107  0.0  0.2 177404  3848 ?        Ss   11:34   0:00 /usr/sbin/httpd
apache   21110  0.0  0.1 177404  2472 ?        S    11:34   0:00 /usr/sbin/httpd
apache   21111  0.0  0.1 177404  2456 ?        S    11:34   0:00 /usr/sbin/httpd
apache   21112  0.0  0.1 177404  2456 ?        S    11:34   0:00 /usr/sbin/httpd
apache   21113  0.0  0.1 177404  2456 ?        S    11:34   0:00 /usr/sbin/httpd
apache   21114  0.0  0.1 177404  2456 ?        S    11:34   0:00 /usr/sbin/httpd
apache   21115  0.0  0.1 177404  2456 ?        S    11:34   0:00 /usr/sbin/httpd
apache   21116  0.0  0.1 177404  2456 ?        S    11:34   0:00 /usr/sbin/httpd
root     22487  0.0  0.0 103320   820 pts/4    R+   11:37   0:00 grep httpd
# echo f > /proc/sysrq-trigger
# dmesg
Out of memory: Kill process 21107 (httpd) score 1000 or sacrifice child
Killed process 21110, UID 48, (httpd) total-vm:177404kB, anon-rss:1780kB, file-rss:692kB

# tail /var/log/messages
Feb  6 11:37:41 cloud-qe-16-vm-01 kernel: [21107]     0 21107    44351      962   1      15          1000 httpd
Feb  6 11:37:41 cloud-qe-16-vm-01 kernel: [21110]    48 21110    44351      618   0       0             0 httpd
Feb  6 11:37:41 cloud-qe-16-vm-01 kernel: [21111]    48 21111    44351      614   0       0             0 httpd
Feb  6 11:37:41 cloud-qe-16-vm-01 kernel: [21112]    48 21112    44351      614   1       0             0 httpd
Feb  6 11:37:41 cloud-qe-16-vm-01 kernel: [21113]    48 21113    44351      614   0       0             0 httpd
Feb  6 11:37:41 cloud-qe-16-vm-01 kernel: [21114]    48 21114    44351      614   1       0             0 httpd
Feb  6 11:37:41 cloud-qe-16-vm-01 kernel: [21115]    48 21115    44351      614   1       0             0 httpd
Feb  6 11:37:41 cloud-qe-16-vm-01 kernel: [21116]    48 21116    44351      614   1       0             0 httpd
Feb  6 11:37:41 cloud-qe-16-vm-01 kernel: Out of memory: Kill process 21107 (httpd) score 1000 or sacrifice child
Feb  6 11:37:41 cloud-qe-16-vm-01 kernel: Killed process 21110, UID 48, (httpd) total-vm:177404kB, anon-rss:1780kB, file-rss:692kB

```

### Non-Uniform Memory Access(NUMA)

On older i386 and x86-64 systems, all memory is equally accessible by the CPUs. That means that access times for all memory addresses are the same, no matter which CPU is performing the operation. The CPUs were connected by a shared front-side bus (FSB) to main memory. This memory architecture is called Uniform Memory Access (UMA).

On more recent x86-64 processors, this is no longer the case. On a NUMA, or Non-uniform Memory Access system, system memory is divided into zones that are connected directly to particular CPUs or sockets. In this case, accessing memory that is local to the CPU is going to be faster than accessing memory that is connected to a remote CPU on that system.

Examples:

|numactl --interleave=all bigdatabase          |Run bigdatabase with its memory interleaved across all CPUs.|
|numactl --cpunodebind=0 --membind=0,1 process |Run process on node 0 with all memory allocated on node 0 and node 1.|
|numactl --preferred=1; numactl --show         |Set node 1 as preferred, and show the resulting state.|
|numactl --localalloc /dev/shm/file            |Reset the policy for the shared memory file to the default localalloc policy.|

```
# numactl --show
policy: default
preferred node: current
physcpubind: 0 1 
cpubind: 0 
nodebind: 0 
membind: 0 

# cat /proc/cpuinfo | grep "physical id"
physical id	: 0
physical id	: 1
# cat /proc/cpuinfo | grep "cpu cores" | uniq
cpu cores	: 1
# cat /proc/cpuinfo | grep "core id" | sort -u
core id		: 0
# cat /proc/cpuinfo | grep processor 
processor	: 0
processor	: 1
# cat /proc/cpuinfo | grep processor | wc -l        -> 如果这个数值等于上面 物理cpu个数*cpu_cores, 说明没有超频，否则说明超频了（例如此数值是4）
2

# numactl --hardware
available: 1 nodes (0)
node 0 cpus: 0 1
node 0 size: 2047 MB
node 0 free: 451 MB
node distances:
node   0 
  0:  10 


# numastat 
                           node0
numa_hit                35869592
numa_miss                      0
numa_foreign                   0
interleave_hit             20488
local_node              35869592
other_node                     0

# vim /etc/cgconfig.conf
group group1 {
    cpuset {
        cpuset.cpus=0;
        cpuset.mems=0;
    }
}
# service cgconfig restart
# chkconfig cgconfig on
# vim /etc/cgrules.conf
*:top           cpuset          group1/
# service cgred restart

# ps o psr,comm,pid
PSR COMMAND           PID
  1 su               6886
  1 su               6920
  1 bash             6926
  0 su               6969
  0 su               7045
  1 bash             7051
  0 mingetty         7537
  0 mingetty         7539
  0 mingetty         7541
  0 mingetty         7543
  0 mingetty         7545
  0 mingetty         7547
  1 watch            8305
  0 ps               9987
  1 vim             12326
  1 bash            19078
  1 bash            19177
  1 bash            26164
  1 bash            26191
  0 bash            27735
# ps efo psr,comm,pid
PSR COMMAND           PID
  0 bash            27735
  1  \_ su           6886
  1 bash            26191
  1  \_ vim         12326
  1 bash            26164
  1 bash            19177
  0 bash            19078
  1  \_ ps          10484
  0 su               7045
  1  \_ bash         7051
  1      \_ watch    8305
  1 su               6920
  1  \_ bash         6926
  0      \_ su       6969
  0 mingetty         7547
  0 mingetty         7545
  0 mingetty         7543
  0 mingetty         7541
  0 mingetty         7539
  0 mingetty         7537

# watch -n 1 ps o psr,comm,pid

```

```
top 翻页 < >
top 按数字1， 显示所有cpu0 cpu1或者cpus(s)

top 设置显示列 - 例如，按f，出现下面选项， 按j则选中（* J: P          = Last used cpu (SMP)），最后按回车键enter，top命令会多出一列 P
* A: PID        = Process Id
* E: USER       = User Name
* H: PR         = Priority
* I: NI         = Nice value
* O: VIRT       = Virtual Image (kb)
* Q: RES        = Resident size (kb)
* T: SHR        = Shared Mem size (kb)
* W: S          = Process Status
* K: %CPU       = CPU usage
* N: %MEM       = Memory usage (RES)
* M: TIME+      = CPU Time, hundredths
  b: PPID       = Parent Process Pid
  c: RUSER      = Real user name
  d: UID        = User Id
  f: GROUP      = Group Name
  g: TTY        = Controlling Tty
  j: P          = Last used cpu (SMP)
  p: SWAP       = Swapped size (kb)
  l: TIME       = CPU Time
  r: CODE       = Code size (kb)
  s: DATA       = Data+Stack size (kb)
  u: nFLT       = Page Fault count
  v: nDRT       = Dirty Pages count
  y: WCHAN      = Sleeping in Function
  z: Flags      = Task Flags <sched.h>
* X: COMMAND    = Command name/line

```



