---
layout: post
title:  "Limiting Resources(442-4)"
categories: Linux
tags: RHCA 442
---

### Using POSIX resources limits

###### Shell built-in limits the resources: ulimit command

```
# ulimit 
unlimited

# ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 7397
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024                   --> 最大打开文件数
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 10240
cpu time               (seconds, -t) unlimited
max user processes              (-u) 7397                   --> 最大用户进程数
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited

# ulimit -n
1024

# ulimit -m
unlimited

# ulimit -t
unlimited

# ulimit -n 10000
# ulimit -n
10000

# ulimit -v
unlimited
# ulimit -v 0
# ls
Killed

# help ulimit                   -> 对于内建程序，查看帮助用 help <command>
ulimit: ulimit [-SHacdefilmnpqrstuvx] [limit]
    Modify shell resource limits.
    
    Provides control over the resources available to the shell and processes
    it creates, on systems that allow such control.
    
    Options:
      -S	use the `soft' resource limit
      -H	use the `hard' resource limit
      -a	all current limits are reported
      -b	the socket buffer size
      -c	the maximum size of core files created
      -d	the maximum size of a process's data segment
      -e	the maximum scheduling priority (`nice')
      -f	the maximum size of files written by the shell and its children
      -i	the maximum number of pending signals
      -l	the maximum size a process may lock into memory
      -m	the maximum resident set size
      -n	the maximum number of open file descriptors
      -p	the pipe buffer size
      -q	the maximum number of bytes in POSIX message queues
      -r	the maximum real-time scheduling priority
      -s	the maximum stack size
      -t	the maximum amount of cpu time in seconds
      -u	the maximum number of user processes
      -v	the size of virtual memory
      -x	the maximum number of file locks
    
    If LIMIT is given, it is the new value of the specified resource; the
    special LIMIT values `soft', `hard', and `unlimited' stand for the
    current soft limit, the current hard limit, and no limit, respectively.
    Otherwise, the current value of the specified resource is printed.  If
    no option is given, then -f is assumed.
    
    Values are in 1024-byte increments, except for -t, which is in seconds,
    -p, which is in increments of 512 bytes, and -u, which is an unscaled
    number of processes.
    
    Exit Status:
    Returns success unless an invalid option is supplied or an error occurs.
```

###### pam_limits.so /etc/pam.d/system-auth

```
# vim /etc/pam.d/system-auth
...
session     optional      pam_keyinit.so revoke
session     required      pam_limits.so
session     [success=1 default=ignore] pam_succeed_if.so service in crond quiet use_uid
session     required      pam_unix.so
session     optional      pam_sss.so

# vim /etc/security/limits.conf
#<domain>      <type>  <item>         <value>
#@managers       hard    maxlogins       2
u1              soft    maxlogins       2
u1              hard    maxlogins       2

# tail -f /var/log/secure
```

###### rss limit is currently not enforceable on Red Hat Enterprise Linux System

### Using Control Groups (CGroups)

Control groups: RHEL6 introduces a new, more flexible way of limiting access to system resources.    
Control groups subdivided your resources into controllers:

*    blkio - sets limits on the available bandwidth to and from block devices
*    cpu - sets limits to the available CPU time
*    cpuset - sets limits on the available DPUs, and also memory regions in NUMA systems
*    memory - sets limits on the available memory
*    freezer - task in this cgroups are temporarily suspended

These subsystems are also known as resource controllers or controller

###### Setting Limits for Control Cgroups

```
# yum install -y libcgroup
# vim /etc/cgconfig.conf
mount {
        cpuset  = /cgroup/cpuset;
        cpu     = /cgroup/cpu;
        cpuacct = /cgroup/cpuacct;
        memory  = /cgroup/memory;
        devices = /cgroup/devices;
        freezer = /cgroup/freezer;
        net_cls = /cgroup/net_cls;
        blkio   = /cgroup/blkio;
}

# service cgconfig status
Stopped
# ls /cgroup/
# service cgconfig start
Starting cgconfig service: [  OK  ]
# ls /cgroup
blkio  cpu  cpuacct  cpuset  devices  freezer  memory  net_cls
# ls /cgroup/blkio/
blkio.io_merged         blkio.io_service_time  blkio.throttle.io_service_bytes  blkio.throttle.write_bps_device   blkio.weight_device   release_agent
blkio.io_queued         blkio.io_wait_time     blkio.throttle.io_serviced       blkio.throttle.write_iops_device  cgroup.event_control  tasks
blkio.io_service_bytes  blkio.reset_stats      blkio.throttle.read_bps_device   blkio.time                        cgroup.procs
blkio.io_serviced       blkio.sectors          blkio.throttle.read_iops_device  blkio.weight                      notify_on_release
# cat /cgroup/blkio/blkio.throttle.write_bps_device

# man cgconfig.conf
       controller
              Name  of  the  kernel  subsystem. The list of subsystems supported by the kernel can be found in /proc/cgroups file. Named hierarchy can be specified as
              controller "name=<somename>". Do not forget to use double quotes around this controller name (see examples below).

              Libcgroup merges all subsystems mounted to the same directory (see Example 1) and the directory is mounted only once.

       path   The directory path where the group hierarchy associated to a given controller shall be mounted. The directory is created automatically on cgconfig  ser-
              vice startup if it does not exist and is deleted on service shutdown.

       group section has this form:
              group <name> {
                     [permissions]
                     <controller> {
                             <param name> = <param value>;
                             ...
                     }
                     ...
              }

# cat /proc/cgroups 
#subsys_name	hierarchy	num_cgroups	enabled
cpuset	1	1	1
ns	0	1	1
cpu	2	1	1
cpuacct	3	1	1
memory	4	1	1
devices	5	1	1
freezer	6	1	1
net_cls	7	1	1
blkio	8	1	1
perf_event	0	1	1
net_prio	0	1	1

# lssubsys
cpuset
cpu
cpuacct
memory
devices
freezer
net_cls
blkio

# lssubsys -am
ns
perf_event
net_prio
cpuset /cgroup/cpuset
cpu /cgroup/cpu
cpuacct /cgroup/cpuacct
memory /cgroup/memory
devices /cgroup/devices
freezer /cgroup/freezer
net_cls /cgroup/net_cls
blkio /cgroup/blkio

# ls /cgroup/cpu
cgroup.event_control  cgroup.procs  cpu.cfs_period_us  cpu.cfs_quota_us  cpu.rt_period_us  cpu.rt_runtime_us  cpu.shares  cpu.stat  notify_on_release  release_agent  tasks

# vim /etc/cgconfig.conf
group cpulimit_high {
    cpu {
        cpu.shares = 20;
    }
}
group cpulimit_low {
    cpu {
        cpu.shares = 10;
    }
}

# service cgconfig restart
Stopping cgconfig service: [  OK  ]
Starting cgconfig service: [  OK  ]

# ls /cgroup/cpu
cgroup.event_control  cpu.cfs_period_us  cpulimit_high  cpu.rt_period_us   cpu.shares  notify_on_release  tasks
cgroup.procs          cpu.cfs_quota_us   cpulimit_low   cpu.rt_runtime_us  cpu.stat    release_agent

# ls /cgroup/cpu/cpulimit_high
cgroup.event_control  cgroup.procs  cpu.cfs_period_us  cpu.cfs_quota_us  cpu.rt_period_us  cpu.rt_runtime_us  cpu.shares  cpu.stat  notify_on_release  tasks

# cat /cgroup/cpu/cpu.shares
1024

# cat /cgroup/cpu/cpulimit_high/cpu.shares
20

# cat /cgroup/cpu/cpulimit_low/cpu.shares
10
```


###### Assigning Processes to Control Groups - cgred

```
# man cgrules.conf
ESCRIPTION
       cgrules.conf configuration file is used by libcgroups to define control groups to which a process belongs.

       The file contains a list of rules which assign to a defined group/user a control group in a subsystem (or control groups in subsystems).

       Rules have two formats:
           <user>                   <controllers>       <destination>
           <user>:<process name>    <controllers>       <destination>

       Where:
       user can be:
           - a user name
           - a group name with @group syntax
           - the wildcard ’*’, for any user or group
           - ’%’, which is equivalent to "ditto" (useful for
             multi-line rules where different cgroups need to be
             specified for various hierarchies for a single user)

       process name is optional and it can be:
           - a process name
           - a full command path of a process

       controllers can be:
           - comma separated controller names (no spaces) or
           - * (for all mounted controllers)

       destination can be:
           - path relative to the controller hierarchy (ex. pgrp1/gid1/uid1)
           - following strings called "templates" and will get expanded

                 %u     username, uid if name resolving fails
                 %U     uid
                 %g     group name, gid if name resolving fails
                 %G     gid
                 %p     process name, pid if name not available
                 %P     pid

EXAMPLES
       student         devices         /usergroup/students
       Student’s processes in the ’devices’ subsystem belong to the control group /usergroup/students.

       student:cp       devices         /usergroup/students/cp
       When student executes ’cp’ command, the processes in the ’devices’ subsystem belong to the control group /usergroup/students/cp.

       @admin           *              admingroup/
       Processes started by anybody from admin group no matter in what subsystem belong to the control group admingroup/.

       peter           cpu             test1/
       %               memory          test2/
       The  first  line  says Peter’s task for cpu controller belongs to test1 control group. The second one says Peter’s tasks for memory controller belong to test2/
       control group.

       *               *               default/
       All processes in any subsystem belong to the control group default/. Since the earliest matched rule is applied, it makes sense to have this line at the end of
       the list. It will put a task which was not mentioned in the previous rules to default/ control group.

       @students cpu,cpuacct    students/%u
       Processes  in  cpu  and cpuacct subsystems started by anybody from students group belong to group students/name. Where "name" is user name of owner of the process.
```

```
# vim /etc/cgrules.conf
*:a.sh          cpu             cpulimit_high/
*:b.sh          cpu             cpulimit_low/

# service cgred reload
cgred is not running.[FAILED]
# service cgred start
Starting CGroup Rules Engine Daemon: [  OK  ]

# vim a.sh
#!/bin/bash
while :
do
        :
done
# cp a.sh b.sh
# chmod +x a.sh b.sh 

# ./a.sh &              -> 运行一段时间后，a.sh占用cpu达到100%（top命令查看）
[1] 15474

# ./b.sh &              -> 运行一段时间后，a.sh占用cpu一直是b.sh的二倍（top命令查看）
[2] 15475

# jobs
[1]-  Running                 ./a.sh &
[2]+  Running                 ./b.sh &
# kill %1
# kill %2
[1]-  Terminated              ./a.sh
[2]+  Terminated              ./b.sh
```

```
# vim /etc/cgconfig.conf
group cpulimit_low {
    cpu {
        cpu.shares = 10;
    }
    memory {
        memory.max_usage_in_bytes = 100000;
    }
}
# service cgconfig restart
Stopping cgconfig service: [  OK  ]
Starting cgconfig service: [  OK  ]

# vim /etc/cgrules.conf
*:a.sh          cpu             cpulimit_high/
*:b.sh          cpu,memory      cpulimit_low/

# service cgred reload
Reloading rules configuration...
[  OK  ]
```

###### Persistently Create a Control Group

In this exercise you will persistently create a Control group named bigload that will limit its processes to no more than 1MiB/s of read throughput on your primary disk.    
The syntax for blkio.throttle.read_bps_device is MAJOR:MINOR Bytes/s.    
Futhermore any process called dd should automatically be put in your bigload Control Group

```
# cat /proc/partitions 
major minor  #blocks  name

 252        0   83886080 vda
 252        1     512000 vda1
 252        2   83373056 vda2
 252       16   83886080 vdb
 252       17   83885056 vdb1
 253        0   52428800 dm-0
 253        1    4128768 dm-1
 253        2   10485760 dm-2
 253        3     102400 dm-3
 253        4     100352 dm-4

# bc
bc 1.06.95
Copyright 1991-1994, 1997, 1998, 2000, 2004, 2006 Free Software Foundation, Inc.
This is free software with ABSOLUTELY NO WARRANTY.
For details type `warranty'. 
scale=8
1*1024*1024
1048576

# vim /etc/cgconfig.conf
group bigload {
    blkio {
        blkio.throttle.read_bps_device = " 252:0 1048576";
    }
}

# service cgconfig restart
Stopping cgconfig service: [  OK  ]
Starting cgconfig service: [  OK  ]

# vim /etc/cgrules.conf
*:dd            blkio           bigload/

# service cgred reload
Reloading rules configuration...
[  OK  ]

# yum install iotop

# dd if=/dev/vda of=/dev/null       -> 用top查看传输速率，最大在1M左右
```

