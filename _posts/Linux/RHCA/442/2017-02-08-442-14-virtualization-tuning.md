---
layout: post
title:  "Virtualization Tuning(442-14)"
categories: Linux
tags: RHCA 442
---

### Virtualization Tuned Profiles

### CPU Pinning

Increase cache-hits and to manually balance CPUs for specific workloads

###### pin cpu with command 'virsh vcpupin'

```
# man virsh
        vcpupin domain [vcpu] [cpulist] [[--live] [--config] | [--current]]
           Query or change the pinning of domain VCPUs to host physical CPUs.  To pin a single vcpu, specify cpulist; otherwise, you can query one vcpu or omit vcpu to list all at
           once.
```

```
# virsh list
 Id    Name                           State
----------------------------------------------------
 2     rhel6.8                        running

1. To view the information on which physical CPUs a guest may use. Configuring the CPU Affinity setting is how a vCPU can be assigned to a physical CPU. In this output, the guest's vCPU can be assigned to any of the four hypervisor CPUs.
# virsh vcpuinfo rhel6.8                
VCPU:           0
CPU:            2
State:          running
CPU time:       1381.6s
CPU Affinity:   yyyyyyyy

VCPU:           1
CPU:            3
State:          running
CPU time:       2711.2s
CPU Affinity:   yyyyyyyy

2. Suppose that an administrator would like to only permit this guest to use the first hypervisor CPU. To accomplish the task, the administrator would use the vcpupin argument to the virsh command
# virsh vcpupin rhel6.8 0 1
# virsh vcpuinfo rhel6.8
VCPU:           0
CPU:            1
State:          running
CPU time:       1381.8s
CPU Affinity:   -y------

VCPU:           1
CPU:            3
State:          running
CPU time:       2712.5s
CPU Affinity:   yyyyyyyy

3. Administrators can set the CPU Affinity to more than one CPU by using a comma-separated list or a range.
# virsh vcpupin rhel6.8 0 1,3       -> set the CPU Affinity to permit the virtual machine to use the hypervisor's first or third CPU
# virsh vcpupin desktop 0 0-2       -> permit the virtual machine to use the zero, first, or second hypervisor CPUs.

4. Configure a memory hard limit for the desktop virtual machine of 1 GiB.
# virsh memtune desktop --hard-limit 1g 

# virsh memtune rhel6.8
hard_limit     : 9007199254740988
soft_limit     : 9007199254740988
swap_hard_limit: 9007199254740988


```

###### pin cpu with modifying xml file

```
1. change the block from
# locate rhel6.8.xml
/etc/libvirt/qemu/rhel6.8.xml
# vim /etc/libvirt/qemu/rhel6.8.xml         -> or virsh edit rhel6.8
<vcpu>2</vcpu>                  -> 开始内容
<vcpu cpuset='0,2'>2</vcpu>     -> 修改后内容

2. pin specific VCPUs to specific physical CPUs
<cputune>
    <vcpupin vcpu='0' cpuset='0'/>
    <vcpupin vcpu='1' cpuset='2'/>
</cputune>
```

### Kernel Samepage Merging (KSM)

When guests are running identical operating systems and/or workloads, there is a high chance that many memory pages will have the exact same content.

KSM: merging those identical pages into one memory page. When a guest writes to that page, it will be converted into a new, separate page on the fly.

ksm can be configured manually in /sys/kernel/mm/ksm/. The following files are used to control ksm, but remember that ksmtuned will adjust this settings as necessary if it is running:

*    run: When set to 1, ksm will actively scan memory; When set to 0, scanning is disabled.
*    pages_to_scan:The number of memory pages to scan in one cycle.
*    sleep_millisecs: The number of milliseconds to sleep between cycles. 

There are also a couple of informative files in this directory, among which are:
*    pages_shared: The number of physical pages being shared.
*    pages_sharing: The number of logical pages being shared.
*    full_scans: How often the entire memory has been scanned so far.
*    merge_across_nodes: Whether pages from different NUMA nodes can be merged.

To tune ksm, it is often more useful to use the ksmtuned service than to tune by hand. To configure the ksmtuned service, use the file /etc/ksmtuned.conf. To see what ksmtuned is
doing, uncomment the LOGFILE and DEBUG lines and restart ksmtuned.

To limit (or increase) the number of pages ksm is allowed to mark as nonswappable, add a line KSM_MAX_KERNEL_PAGES=number to /etc/sysconfig/ksm. When this line is absent or set to 0, ksm will mark up to half the physical memory as unswappable. Unswappable memory can increase performance, but at the cost of less flexibility when it comes to memory assignments.
For example, there might be 16 GiB of free swap, but since most memory is marked unswappable it cannot be used to swap out memory belonging to a VM.

```
# which ksmtuned 
/usr/sbin/ksmtuned
# rpm -qf /usr/sbin/ksmtuned
qemu-kvm-0.12.1.2-2.491.el6.x86_64

# ls /sys/kernel/mm/ksm/*
/sys/kernel/mm/ksm/full_scans          /sys/kernel/mm/ksm/pages_shared   /sys/kernel/mm/ksm/pages_to_scan   /sys/kernel/mm/ksm/pages_volatile  /sys/kernel/mm/ksm/sleep_millisecs
/sys/kernel/mm/ksm/merge_across_nodes  /sys/kernel/mm/ksm/pages_sharing  /sys/kernel/mm/ksm/pages_unshared  /sys/kernel/mm/ksm/run

# cat /etc/ksmtuned.conf
# Configuration file for ksmtuned.

# How long ksmtuned should sleep between tuning adjustments
# KSM_MONITOR_INTERVAL=60

# Millisecond sleep between ksm scans for 16Gb server.
# Smaller servers sleep more, bigger sleep less.
# KSM_SLEEP_MSEC=10

# KSM_NPAGES_BOOST=300
# KSM_NPAGES_DECAY=-50
# KSM_NPAGES_MIN=64
# KSM_NPAGES_MAX=1250

# KSM_THRES_COEF=20
# KSM_THRES_CONST=2048

# uncomment the following if you want ksmtuned debug info

# LOGFILE=/var/log/ksmtuned
# DEBUG=1

# cat /sys/kernel/mm/ksm/pages_shared 
0
# cat /sys/kernel/mm/ksm/pages_sharing 
0

```

### Limit Virtualization Guests Using cgroups

```
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
```

### Virtual Machine Storage



