---
layout: post
title:  "Tuning for a CPU Intensive Workload(442-10)"
categories: Linux
tags: RHCA 442
---

### Limiting CPU Access with cgroups

```
# cat /cgroup/cpu/cpu.shares 
1024
# vim /etc/cgconfig.conf
group limitcpu1 {
    cpu {
        cpu.shares = 256;
    }
}
group limitcpu2 {
    cpu {
        cpu.shares = 512;
    }
}
# service cgconfig restart
Stopping cgconfig service: [  OK  ]
Starting cgconfig service: [  OK  ]

# cat /cgroup/cpu/limitcpu1/cpu.shares 
256
# cat /cgroup/cpu/limitcpu2/cpu.shares 
512


# cgexec -h
Usage: cgexec [-h] [-g <controllers>:<path>] [--sticky] command [arguments] ...
Run the task in given control groups
  -g <controllers>:<path>	Control group which should be added
  --sticky			cgred daemon does not change pidlist and children tasks

# cat /a.sh 
#/bin/bash
while :
do
        :
done
# cp /a.sh /b.sh
# chmod +x /a.sh /b.sh
# cgexec -g cpu:limitcpu1 /a.sh 
# cgexec -g cpu:limitcpu2 /b.sh

# cat /proc/1196/cgroup 
52:blkio:/
51:net_cls:/
50:freezer:/
49:devices:/
48:memory:/
47:cpuacct:/
46:cpu:/limitcpu2
45:cpuset:/
# cat /proc/31174/cgroup 
52:blkio:/
51:net_cls:/
50:freezer:/
49:devices:/
48:memory:/
47:cpuacct:/
46:cpu:/limitcpu1
45:cpuset:/

```

```
limitcpu1 256
limitcpu2 512
parent    1024

1   256/256+512+1024  =1/7
2   512/256+512+1024  =2/7
3   1024/256+512+1024 =4/7
```

### Balancing Interrupts

*    Red Hat Enterprise Linux ships with a daemon named irqbalance that is enabled by default on new installs. 
*    The purpose of  irqbalance is to adjust the smp_affinity of all interrupts every 10 seconds so that an interrupt handler has the highest chance of getting cache hits when executing. 
*    On single-core systems and dual-core systems that share their L2 cache, irqbalance will do nothing.

In /etc/sysconfig/irqbalance, there are two settings that can be made to influence the behavior of the irqbalance daemon.

*    IRQBALANCE_ONESHOT: When set to yes, irqbalance will sleep for one minute after startup, rebalance interrupts once, and then exit. This can be useful for avoiding the overhead of irqbalance waking up every 10 seconds.
*    IRQBALANCE_BANNED_CPUS: If set, irqbalance will not assign any interrupts to the CPUs listed. The value is a hexadecimal bitmask of all CPUs available on the system, calculated in the same way as  smp_affinity. For example, setting a value of fe will mark CPUs 1-15 as not eligible for interrupt handling, with only CPU0 available for interrupt processing (and any CPU with an index of 16 or higher). Setting a value of 3 will ban irqbalance from placing interrupts on CPU0 or CPU1.

```
# /etc/init.d/irqbalance status         => 当irqbalance服务启动后，它会自动调整所有进程的smp_affinity值，不用人工干预/proc/irq/*/smp_affinity
irqbalance (pid  1387) is running...
# cat /proc/interrupts 
           CPU0       CPU1       
  0:        119          1   IO-APIC-edge      timer
  1:          0          6   IO-APIC-edge      i8042
  4:          0          1   IO-APIC-edge    
  8:          0          0   IO-APIC-edge      rtc0
  9:          0          0   IO-APIC-fasteoi   acpi
 10:          0          0   IO-APIC-fasteoi   virtio1
 11:          0         61   IO-APIC-fasteoi   uhci_hcd:usb1, snd_hda_intel
 12:          0        104   IO-APIC-edge      i8042
 14:          0          0   IO-APIC-edge      ata_piix
 15:          0          0   IO-APIC-edge      ata_piix
 24:          0          0   PCI-MSI-edge      virtio2-config
 25:          0      62153   PCI-MSI-edge      virtio2-requests
 26:          0          0   PCI-MSI-edge      virtio0-config
 27:          0     763096   PCI-MSI-edge      virtio0-input.0
 28:          1          1   PCI-MSI-edge      virtio0-output.0
NMI:          0          0   Non-maskable interrupts
LOC:    7560870    6619526   Local timer interrupts
SPU:          0          0   Spurious interrupts
PMI:          0          0   Performance monitoring interrupts
IWI:          0          0   IRQ work interrupts
RES:     583469     582107   Rescheduling interrupts
CAL:        123        164   Function call interrupts
TLB:     220259     201950   TLB shootdowns
TRM:          0          0   Thermal event interrupts
THR:          0          0   Threshold APIC interrupts
MCE:          0          0   Machine check exceptions
MCP:        744        744   Machine check polls
ERR:          0
MIS:          0

# find /proc/irq/*/smp_affinity
/proc/irq/0/smp_affinity
/proc/irq/10/smp_affinity
/proc/irq/11/smp_affinity
/proc/irq/12/smp_affinity
/proc/irq/13/smp_affinity
/proc/irq/14/smp_affinity
/proc/irq/15/smp_affinity
/proc/irq/1/smp_affinity
/proc/irq/24/smp_affinity
/proc/irq/25/smp_affinity
/proc/irq/26/smp_affinity
/proc/irq/27/smp_affinity
/proc/irq/28/smp_affinity
/proc/irq/2/smp_affinity
/proc/irq/3/smp_affinity
/proc/irq/4/smp_affinity
/proc/irq/5/smp_affinity
/proc/irq/6/smp_affinity
/proc/irq/7/smp_affinity
/proc/irq/8/smp_affinity
/proc/irq/9/smp_affinity
```

```
手动修改某个进程的smp_affinity
$[2**0]                 => 在0号CPU上运行
$[2**0+2**1+2**3]       => 在0号，1号，3号CPU上运行
$[2**0+2**5+2**7]       => 在0号，5号，7号CPU上运行


# cat /proc/irq/15/smp_affinity
1

# echo 2 > /proc/irq/15/smp_affinity
# cat /proc/irq/15/smp_affinity
2

# printf '%0x' $[2**0]
1
# printf '%0x' $[2**0] > /proc/irq/15/smp_affinity
# cat /proc/irq/15/smp_affinity
1
```

### Pin Processes to a Specific CPU with cgroups

```
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

# numactl --hardware
available: 1 nodes (0)
node 0 cpus: 0 1
node 0 size: 2047 MB
node 0 free: 451 MB
node distances:
node   0
  0:  10
```

### Real-Time Scheduling

*    chrt command
*    Specify a maximum runtime for realtime processes inside a period(microseconds)
*    cpu group: cpu.rt_period_us cpu.rt_runtime_us 

```
实时进程（内核调度的进程）只能在1000000周期内运行950000时间，剩下的时间要让给非实时进程
# cat /cgroup/cpu/cpu.rt_period_us 
1000000
# cat /cgroup/cpu/cpu.rt_runtime_us 
950000
```

```
实时进程优先级是1-99，数值越大，优先级越高
# chrt -m
SCHED_OTHER min/max priority	: 0/0
SCHED_FIFO min/max priority	: 1/99
SCHED_RR min/max priority	: 1/99
SCHED_BATCH min/max priority	: 0/0
SCHED_IDLE min/max priority	: 0/0

# chrt -p 10
pid 10's current scheduling policy: SCHED_FIFO
pid 10's current scheduling priority: 99
# chrt -p 6
pid 6's current scheduling policy: SCHED_FIFO
pid 6's current scheduling priority: 99
# chrt -p 4
pid 4's current scheduling policy: SCHED_OTHER
pid 4's current scheduling priority: 0

```

