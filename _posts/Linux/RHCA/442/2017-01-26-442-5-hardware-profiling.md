---
layout: post
title:  "Hardware(442-5)"
categories: Linux
tags: RHCA 442
---

### Generating a Hardware Profile

*    CPUs
*    Memory
*    Storage
*    Networking

###### getconf

用来获取系统的基本配置信息

```
# getconf -a            -> 输出全部系统配置变量值
# getconf PAGE_SIZE     -> 获取到PAGE SIZE大小
4096
# getconf INT_MAX
2147483647
# getconf INT_MIN
-2147483648
# getconf LONG_BIT

# getconf NAME_MAX /usr
255
# getconf PATH_MAX /usr
4096

```

###### dmesg

###### lscpu

```
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
CPU MHz:               1663.343
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

# lscpu -p
# The following is the parsable format, which can be fed to other
# programs. Each different item in every column has an unique ID
# starting from zero.
# CPU,Core,Socket,Node,,L1d,L1i,L2,L3
0,0,0,0,,0,0,0,0
1,1,0,0,,1,1,1,0
2,2,0,0,,2,2,2,0
3,3,0,0,,3,3,3,0
4,0,0,0,,0,0,0,0
5,1,0,0,,1,1,1,0
6,2,0,0,,2,2,2,0
7,3,0,0,,3,3,3,0
```

###### x86info

```
# x86info 
x86info v1.30.  Dave Jones 2001-2011
Feedback to <davej@redhat.com>.

Found 8 identical CPUs
Extended Family: 0 Extended Model: 2 Family: 6 Model: 42 Stepping: 7
Type: 0 (Original OEM)
CPU Model (x86info's best guess): Unknown model. 
Processor name string (BIOS programmed): Intel(R) Core(TM) i7-2600 CPU @ 3.40GHz

Total processor threads: 8
This system has 1 quad-core processor with hyper-threading (2 threads per core) running at an estimated 3.40GHz

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

```


### Determining SMBIOS/DMI Information

###### dmidecode

```
# ls /sys/class/dmi/id/
bios_date    bios_version     board_name    board_vendor   chassis_asset_tag  chassis_type    chassis_version  power         product_serial  product_version  sys_vendor
bios_vendor  board_asset_tag  board_serial  board_version  chassis_serial     chassis_vendor  modalias         product_name  product_uuid    subsystem        uevent
# ls /sys/class/dmi/id/product_
product_name     product_serial   product_uuid     product_version  
# ls /sys/class/dmi/id/product_*
/sys/class/dmi/id/product_name  /sys/class/dmi/id/product_serial  /sys/class/dmi/id/product_uuid  /sys/class/dmi/id/product_version

# cat /sys/class/dmi/id/product_name
HP Compaq 8200 Elite MT PC
# cat /sys/class/dmi/id/product_serial 
CNG2198GZB
# cat /sys/class/dmi/id/product_uuid 
B34E2800-91D9-11E1-0000-E8393558D59D
# cat /sys/class/dmi/id/bios_vendor 
Hewlett-Packard
# cat /sys/class/dmi/id/chassis_vendor
Hewlett-Packard
```


### Generating a Profile of the whole system

###### lspci

```
# lspci 
00:00.0 Host bridge: Intel Corporation 2nd Generation Core Processor Family DRAM Controller (rev 09)
00:01.0 PCI bridge: Intel Corporation Xeon E3-1200/2nd Generation Core Processor Family PCI Express Root Port (rev 09)
00:16.0 Communication controller: Intel Corporation 6 Series/C200 Series Chipset Family MEI Controller #1 (rev 04)
00:16.3 Serial controller: Intel Corporation 6 Series/C200 Series Chipset Family KT Controller (rev 04)
00:19.0 Ethernet controller: Intel Corporation 82579LM Gigabit Network Connection (rev 04)
00:1a.0 USB controller: Intel Corporation 6 Series/C200 Series Chipset Family USB Enhanced Host Controller #2 (rev 04)
00:1b.0 Audio device: Intel Corporation 6 Series/C200 Series Chipset Family High Definition Audio Controller (rev 04)
00:1c.0 PCI bridge: Intel Corporation 6 Series/C200 Series Chipset Family PCI Express Root Port 1 (rev b4)
00:1c.4 PCI bridge: Intel Corporation 6 Series/C200 Series Chipset Family PCI Express Root Port 5 (rev b4)
00:1c.6 PCI bridge: Intel Corporation 6 Series/C200 Series Chipset Family PCI Express Root Port 7 (rev b4)
00:1c.7 PCI bridge: Intel Corporation 6 Series/C200 Series Chipset Family PCI Express Root Port 8 (rev b4)
00:1d.0 USB controller: Intel Corporation 6 Series/C200 Series Chipset Family USB Enhanced Host Controller #1 (rev 04)
00:1e.0 PCI bridge: Intel Corporation 82801 PCI Bridge (rev a4)
00:1f.0 ISA bridge: Intel Corporation Q67 Express Chipset Family LPC Controller (rev 04)
00:1f.2 SATA controller: Intel Corporation 6 Series/C200 Series Chipset Family SATA AHCI Controller (rev 04)
00:1f.3 SMBus: Intel Corporation 6 Series/C200 Series Chipset Family SMBus Controller (rev 04)
01:00.0 VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Caicos [Radeon HD 6450/7450/8450 / R5 230 OEM]
01:00.1 Audio device: Advanced Micro Devices, Inc. [AMD/ATI] Caicos HDMI Audio [Radeon HD 6400 Series]

# lspci -vv

```

###### lsusb

```
# lsusb
Bus 002 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 
Bus 001 Device 004: ID 17ef:6019 Lenovo 
Bus 001 Device 003: ID 0461:4e04 Primax Electronics, Ltd 
Bus 001 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 
```

###### lsblk

```
# lsblk 
NAME                           MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda                              8:0    0 465.8G  0 disk 
├─sda1                           8:1    0   500M  0 part /boot
└─sda2                           8:2    0 465.3G  0 part 
  ├─fedora_dhcp--129--221-root 253:0    0    50G  0 lvm  /
  ├─fedora_dhcp--129--221-swap 253:1    0   7.9G  0 lvm  [SWAP]
  └─fedora_dhcp--129--221-home 253:2    0 407.4G  0 lvm  /home
sr0                             11:0    1  1024M  0 rom  
```

######

```
# lsscsi
[0:0:0:0]    disk    ATA      ST500DM002-1BD14 HP73  /dev/sda 
[2:0:0:0]    cd/dvd  hp       DVD A  DH16ACSH  JHD5  /dev/sr0
```

###### sosreport


### Accessing NUMA Topologies

### Profiling Storage

### I/O Scheduling

> /usr/share/doc/kernel-doc-2.6.32/Documentation/iostats.txt

> /usr/share/doc/kernel-doc-2.6.32/Documentation/block/deadline-iosched.txt

```
# cat /sys/block/sda/queue/scheduler
noop deadline [cfq] 
# ls /sys/block/sda/queue/iosched/
back_seek_max  back_seek_penalty  fifo_expire_async  fifo_expire_sync  group_idle  low_latency  quantum  slice_async  slice_async_rq  slice_idle  slice_sync  target_latency

# echo deadline > /sys/block/sda/queue/scheduler
# cat /sys/block/sda/queue/scheduler
noop [deadline] cfq
# ls /sys/block/sda/queue/iosched/
fifo_batch  front_merges  read_expire  write_expire  writes_starved
```
