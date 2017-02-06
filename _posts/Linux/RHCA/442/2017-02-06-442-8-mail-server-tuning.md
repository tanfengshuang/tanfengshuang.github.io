---
layout: post
title:  "Mail Server Tuning/Small Files Tuning(442-8)"
categories: Linux
tags: RHCA 442
---

### Analyzing a Mail Server Workload

Analy a mail server-like workload and look out different factors impacting preformance on a mail server

*    Disk: Many small read and write requests from multiple processes/threads at the same time
*    Memory: Most of the system's memory would be automatically used for disk caches to improve disk I/O.
*    CPU: very CUP-insentive. Due to the disk-intensive nature of a mail server workload, the system could spend time in I/O wait as as running processes block while waiting for disk requests to complete.
*    Network: Although the network on a mail server is used quite frequently, generally the bottleneck on a mail server is throughput to the disk and not network throughput

### Rotation Delay and Disk Elevators

*    rotational delay
*    seek time
*    reranged and merge the requests

###### Disk Elevators

```
# cat /sys/block/vda/queue/scheduler 
noop anticipatory [deadline] cfq 
```

### Selecting a tuned Profile for Mail Server Workload

*    noop: is normally selected when the back-end storage device can also reorder and merge requests and has a better understanding of the actual disk layout behind it, e.g., an enterprise-class SAN or a hypervisor.
*    deadline: The deadline scheduler tries to provide a guaranteed latency for requests. The read_expire and the write_expire parameters can be used to specify the number of milliseconds within which requests should be scheduled for read and write operations, respectively. Is best suited to disks where multiple processes will perform large I/O operations, for example, database server or file servers
*    cfq: was designed for disks where a lot of processes will be reading and writing at the same time, both small and large requests. To take advantage of those classes and priorities, use either the ionice command or the blkio cgroup controller. CFQ is best suited for systems where there will be a lot of concurrent reads and writes from different processes; for example, a user's desktop or a Usenet server.

```
# yum -y install tuned; chkconfig tuned on; chconfig ktune on; service tuned start; service ktune start

# cat /sys/block/sda/queue/scheduler
noop anticipatory [deadline] cfq 
# echo cfq > /sys/block/vda/queue/scheduler
# cat /sys/block/vda/queue/scheduler 
noop anticipatory deadline [cfq] 

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
Current active profile: default
# cat /sys/block/vda/queue/scheduler 
noop anticipatory deadline [cfq] 

# tuned-adm profile throughput-performance
Stopping tuned: [  OK  ]
Switching to profile 'throughput-performance'
Applying deadline elevator: dm-0 dm-1 dm-2 vda [  OK  ]
Applying ktune sysctl settings:
/etc/ktune.d/tunedadm.conf: [  OK  ]
Calling '/etc/ktune.d/tunedadm.sh start': [  OK  ]
Applying sysctl settings from /etc/sysctl.conf
Starting tuned: [  OK  ]
# cat /sys/block/vda/queue/scheduler 
noop anticipatory [deadline] cfq 

```


Elevators can also be selected in custom tuned profiles by using the elevator parameter in the profile's ktune.sysconfig file.

```
# vim /etc/tune-profiles/throughput-performance/ktune.sysconfig 
# scheduler settings.
ELEVATOR="cfq"

# These are the devices, that should be tuned with the ELEVATOR
ELEVATOR_TUNE_DEVS="/sys/block/{sd,cciss,dm-,vd,dasd,xvd}*/queue/scheduler"

# cat /sys/block/vda/queue/scheduler 
noop anticipatory [deadline] cfq
# tuned-adm profile throughput-performance
Stopping tuned: [  OK  ]
Switching to profile 'throughput-performance'
Applying cfq elevator: dm-0 dm-1 dm-2 vda [  OK  ]
Applying ktune sysctl settings:
/etc/ktune.d/tunedadm.conf: [  OK  ]
Calling '/etc/ktune.d/tunedadm.sh start': [  OK  ]
Applying sysctl settings from /etc/sysctl.conf
Starting tuned: [  OK  ]
# cat /sys/block/vda/queue/scheduler
noop anticipatory deadline [cfq] 
```

```
# yum install -y iotop
# iotop -o -P
Total DISK READ: 0.00 B/s | Total DISK WRITE: 0.00 B/s
  PID  PRIO  USER     DISK READ  DISK WRITE  SWAPIN     IO>    COMMAND
```

### Further Mail Server Tips

*    MTA: sendmail gmail postfix exchange
*    SMTP/IMAP: ssl aes-ni speed-up encryption/decryption
*    IMAP clustered
*    SMTP multiple machine
*    ext4 ext3
*    mount -o remount,noatime /


