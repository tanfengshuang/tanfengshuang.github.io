---
layout: post
title:  "Software Profiling(442-6)"
categories: Linux
tags: RHCA 442
---

### CPU Scheduling

*    A single CPU can only execute one process at a time.
*    Multitasking shares the CPU among multiple processes by allowing each process to take turns to run on the CPU.
*    The kernel uses its process scheduler to determine which process to run at any given point in time.
*    O(1) Scheduler (kernel version 2.6, RHEL4 and RHEL5)
*    Completely Fair Scheduler(kernel version 2.6.23, RHEL6 and RHEL7)

###### O(1) Scheduler

The O(1) scheduler works by using 2 queues per CPU:

*    run queue
*    expire queue

###### Completely Fair Scheduler(RHEL 6 and RHEL 7)

*    CFS uses a red-black tree based on virtual time
*    This virtual time is based on the time waiting to run and the number of processes vying for CPU time
*    The process with the most virtual time, which is the longest time waiting for the CPU, gets to use the CPU. As it uses CPU cycles, its virtual time decreases.
*    The CFS scheduler can select the next process in O(1) time, but reinserting a task after it has run takes O(log n) time.

```
# yum install -y kernel-doc
# ls /usr/share/doc/kernel-doc-2.6.32/Documentation/scheduler/sched-design-CFS.txt 
/usr/share/doc/kernel-doc-2.6.32/Documentation/scheduler/sched-design-CFS.txt
```

```
# chrt

chrt - manipulate real-time attributes of a process.

Set policy:
  chrt [options] <policy> <priority> {<pid> | <command> [<arg> ...]}

Get policy:
  chrt [options] {<pid> | <command> [<arg> ...]}


Scheduling policies:
  -b | --batch         set policy to SCHED_BATCH
  -f | --fifo          set policy to SCHED_FIFO
  -i | --idle          set policy to SCHED_IDLE
  -o | --other         set policy to SCHED_OTHER
  -r | --rr            set policy to SCHED_RR (default)

Options:
  -h | --help          display this help
  -p | --pid           operate on existing given pid
  -m | --max           show min and max valid priorities
  -v | --verbose       display status information
  -V | --version       output version information

# chrt -m
SCHED_OTHER min/max priority	: 0/0
SCHED_FIFO min/max priority	: 1/99
SCHED_RR min/max priority	: 1/99
SCHED_BATCH min/max priority	: 0/0
SCHED_IDLE min/max priority	: 0/0

# top   
Tasks: 117 total,   1 running, 116 sleeping,   0 stopped,   0 zombie
Cpu(s):  0.0%us,  0.0%sy,  0.0%ni,100.0%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:   1922096k total,   921828k used,  1000268k free,    71592k buffers
Swap:  4128764k total,        0k used,  4128764k free,   607652k cached

  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
   10 root      RT   0     0    0    0 S  0.0  0.0   0:00.57 watchdog/1
    6 root      RT   0     0    0    0 S  0.0  0.0   0:00.60 watchdog/0
  697 root      20   0     0    0    0 S  0.0  0.0   0:00.00 virtio-net
  385 root      20   0     0    0    0 S  0.0  0.0   0:00.00 virtio-blk
  715 root      20   0     0    0    0 S  0.0  0.0   0:00.00 vballoon
   62 root      20   0     0    0    0 S  0.0  0.0   0:00.00 usbhid_resumer
  586 root      16  -4 11120 1304  416 S  0.0  0.1   0:00.13 udevd
26711 root      18  -2 11116 1172  288 S  0.0  0.1   0:00.01 udevd
26712 root      18  -2 11116 1148  264 S  0.0  0.1   0:00.00 udevd  

---
385 PR 20 ni 0
586 PR 16 ni -4
---
# ps -el | grep 385
1 S     0   385     2  0  80   0 -     0 worker ?        00:00:00 virtio-blk
# ps -el | grep 586
5 S     0   586     1  0  76  -4 -  2780 poll_s ?        00:00:00 udevd
# ps -el | head -n 2
F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
4 S     0     1     0  0  80   0 -  4842 poll_s ?        00:00:00 init
---
385 PRI 80 NI 0
586 PRI 76 NI -4
---

# chrt -p 385
pid 385's current scheduling policy: SCHED_OTHER
pid 385's current scheduling priority: 0
# chrt -p 586
pid 586's current scheduling policy: SCHED_OTHER
pid 586's current scheduling priority: 0

# chrt -p 10
pid 10's current scheduling policy: SCHED_FIFO
pid 10's current scheduling priority: 99
# chrt -p 6
pid 6's current scheduling policy: SCHED_FIFO
pid 6's current scheduling priority: 99

# chrt --fifo 20 md5sum /dev/zero &
[1] 27355
# top
Tasks: 118 total,   3 running, 115 sleeping,   0 stopped,   0 zombie
Cpu(s): 48.6%us,  1.5%sy,  0.0%ni, 49.9%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:   1922096k total,   922332k used,   999764k free,    71664k buffers
Swap:  4128764k total,        0k used,  4128764k free,   607660k cached

  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
27355 root     -21   0 98.6m  616  508 R 100.0  0.0   1:24.92 md5sum
27356 root      20   0 15036 1208  900 R  0.3  0.1   0:00.12 top
    1 root      20   0 19368 1512 1192 S  0.0  0.1   0:00.97 init
    2 root      20   0     0    0    0 S  0.0  0.0   0:00.08 kthreadd 

# md5sum /dev/zero &
[2] 27358
# top
Tasks: 119 total,   4 running, 115 sleeping,   0 stopped,   0 zombie
Cpu(s):  0.1%us,  0.0%sy,  0.0%ni, 99.8%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:   1922096k total,   922224k used,   999872k free,    71664k buffers
Swap:  4128764k total,        0k used,  4128764k free,   607660k cached

  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
27355 root     -21   0 98.6m  616  508 R 99.1  0.0   2:43.89 md5sum
27358 root      20   0 98.6m  616  508 R 99.1  0.0   0:05.11 md5sum
    1 root      20   0 19368 1512 1192 S  0.0  0.1   0:00.97 init
    2 root      20   0     0    0    0 S  0.0  0.0   0:00.08 kthreadd 

# vim /a.sh
#/bin/bash
while :
do
        :
done
# chmod +x /a.sh 
# chrt --fifo 99 /a.sh &
[1] 27370
# top
Mem:   1922096k total,   922580k used,   999516k free,    71732k buffers
Swap:  4128764k total,        0k used,  4128764k free,   607708k cached

  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
27370 root      RT   0  103m 1132  968 R 100.0  0.1   0:16.41 sh
    1 root      20   0 19368 1512 1192 S  0.0  0.1   0:00.97 init
    2 root      20   0     0    0    0 S  0.0  0.0   0:00.08 kthreadd  
    3 root      RT   0     0    0    0 S  0.0  0.0   0:01.19 migration/0
    4 root      20   0     0    0    0 S  0.0  0.0   0:00.07 ksoftirqd/0 

# chrt -p 3
pid 3's current scheduling policy: SCHED_FIFO
pid 3's current scheduling priority: 99
# chrt --fifo -p 98 3
# chrt -p 3
pid 3's current scheduling policy: SCHED_FIFO
pid 3's current scheduling priority: 98
# top
Tasks: 117 total,   1 running, 116 sleeping,   0 stopped,   0 zombie
Cpu(s):  0.2%us,  0.0%sy,  0.0%ni, 99.8%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:   1922096k total,   922192k used,   999904k free,    71740k buffers
Swap:  4128764k total,        0k used,  4128764k free,   607708k cached

  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
   12 root      20   0     0    0    0 S  0.3  0.0   0:15.36 events/1
    1 root      20   0 19368 1512 1192 S  0.0  0.1   0:00.97 init
    2 root      20   0     0    0    0 S  0.0  0.0   0:00.08 kthreadd
    3 root     -99   0     0    0    0 S  0.0  0.0   0:01.19 migration/0 

# ps ax -o pid,cmd,class,rtprio,pri,nice,policy
  PID CMD                         CLS RTPRIO PRI  NI POL
    1 /sbin/init                  TS       -  19   0 TS
    2 [kthreadd]                  TS       -  19   0 TS
    3 [migration/0]               FF      98 138   - FF
    4 [ksoftirqd/0]               TS       -  19   0 TS
    5 [stopper/0]                 FF      99 139   - FF
    6 [watchdog/0]                FF      99 139   - FF
    7 [migration/1]               FF      99 139   - FF
    8 [stopper/1]                 FF      99 139   - FF
    9 [ksoftirqd/1]               TS       -  19   0 TS
   10 [watchdog/1]                FF      99 139   - FF
```

```
# nice
0
# renice
```

```
# time ls /tmp
# time ls /tmp
ks-script-HHRJcJ  ks-script-HHRJcJ.log  ks-script-JddDSm  tmp.Nv1049  yum.log

real	0m0.008s
user	0m0.000s
sys	0m0.007s


# type time
time is a shell keyword


# /usr/bin/time ls /tmp
ks-script-HHRJcJ  ks-script-HHRJcJ.log	ks-script-JddDSm  tmp.Nv1049  yum.log
0.00user 0.00system 0:00.00elapsed 0%CPU (0avgtext+0avgdata 896maxresident)k
0inputs+0outputs (0major+261minor)pagefaults 0swaps
# export TIME="\n %s %S %U"
# /usr/bin/time ls /tmp
ks-script-HHRJcJ  ks-script-HHRJcJ.log	ks-script-JddDSm  tmp.Nv1049  yum.log

 0 0.00 0.00

```

### strace and ltrace Usage

*    strace跟踪程序的每个系统调用
*    ltrace能够跟踪进程的库函数调用,它会显现出哪个库函数被调用

###### strace

> -S for strace:

> 	-S sortby   Sort the output of the histogram printed by the -c option by the specified criterion.  Legal values are time, calls, name, and nothing (default time).

```
# strace
usage: strace [-CdffhiqrtttTvVxxy] [-I n] [-e expr]...
              [-a column] [-o file] [-s strsize] [-P path]...
              -p pid... / [-D] [-E var=val]... [-u username] PROG [ARGS]
   or: strace -c[df] [-I n] [-e expr]... [-O overhead] [-S sortby]
              -p pid... / [-D] [-E var=val]... [-u username] PROG [ARGS]
-c -- count time, calls, and errors for each syscall and report summary
-C -- like -c but also print regular output
-d -- enable debug output to stderr
-D -- run tracer process as a detached grandchild, not as parent
-f -- follow forks, -ff -- with output into separate files
-i -- print instruction pointer at time of syscall
-q -- suppress messages about attaching, detaching, etc.
-r -- print relative timestamp, -t -- absolute timestamp, -tt -- with usecs
-T -- print time spent in each syscall
-v -- verbose mode: print unabbreviated argv, stat, termios, etc. args
-x -- print non-ascii strings in hex, -xx -- print all strings in hex
-y -- print paths associated with file descriptor arguments
-h -- print help message, -V -- print version
-a column -- alignment COLUMN for printing syscall results (default 40)
-b execve -- detach on this syscall
-e expr -- a qualifying expression: option=[!]all or option=[!]val1[,val2]...
   options: trace, abbrev, verbose, raw, signal, read, write
-I interruptible --
   1: no signals are blocked
   2: fatal signals are blocked while decoding syscall (default)
   3: fatal signals are always blocked (default if '-o FILE PROG')
   4: fatal signals and SIGTSTP (^Z) are always blocked
      (useful to make 'strace -o FILE PROG' not stop on ^Z)
-o file -- send trace output to FILE instead of stderr
-O overhead -- set overhead for tracing syscalls to OVERHEAD usecs
-p pid -- trace process with process id PID, may be repeated
-s strsize -- limit length of print strings to STRSIZE chars (default 32)
-S sortby -- sort syscall counts by: time, calls, name, nothing (default time)
-u username -- run command as username handling setuid and/or setgid
-E var=val -- put var=val in the environment for command
-E var -- remove var from the environment for command
-P path -- trace accesses to path


# strace -c uname
Linux
% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
  0.00    0.000000           0         1           read
  0.00    0.000000           0         1           write
  0.00    0.000000           0         3           open
  0.00    0.000000           0         5           close
  0.00    0.000000           0         4           fstat
  0.00    0.000000           0        10           mmap
  0.00    0.000000           0         3           mprotect
  0.00    0.000000           0         2           munmap
  0.00    0.000000           0         3           brk
  0.00    0.000000           0         1         1 access
  0.00    0.000000           0         1           execve
  0.00    0.000000           0         1           uname
  0.00    0.000000           0         1           arch_prctl
------ ----------- ----------- --------- --------- ----------------
100.00    0.000000                    36         1 total


# strace -e open iptables -L
open("/etc/ld.so.cache", O_RDONLY)      = 3
open("/lib64/libip4tc.so.0", O_RDONLY)  = 3
open("/lib64/libxtables.so.4", O_RDONLY) = 3
open("/lib64/libm.so.6", O_RDONLY)      = 3
open("/lib64/libc.so.6", O_RDONLY)      = 3
open("/lib64/libdl.so.2", O_RDONLY)     = 3
open("/proc/sys/kernel/modprobe", O_RDONLY) = 3
--- SIGCHLD {si_signo=SIGCHLD, si_code=CLD_EXITED, si_pid=27508, si_status=0, si_utime=0, si_stime=2} ---
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
+++ exited with 0 +++

# strace -fc -S calls elinks -dump http://www.baidu.com > /dev/null
Process 27539 attached
% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
 89.63    0.001910           6       340         3 read
  1.45    0.000031           0       119           mmap
  0.33    0.000007           0        91           gettimeofday
  4.97    0.000106           1        84        18 open
  0.00    0.000000           0        73           close
  0.00    0.000000           0        60           mprotect
  0.00    0.000000           0        56           fstat
  3.61    0.000077           2        50         1 write
  0.00    0.000000           0        31           munmap
  0.00    0.000000           0        28         1 select
  0.00    0.000000           0        27           alarm
  0.00    0.000000           0        18           fcntl
  0.00    0.000000           0        15           brk
  0.00    0.000000           0        13         1 stat
  0.00    0.000000           0         9           lseek
  0.00    0.000000           0         9         5 ioctl
  0.00    0.000000           0         6           socket
  0.00    0.000000           0         5           rt_sigaction
  0.00    0.000000           0         4           poll
  0.00    0.000000           0         4         3 connect
  0.00    0.000000           0         4           umask
  0.00    0.000000           0         4         1 futex
  0.00    0.000000           0         3         1 access
  0.00    0.000000           0         3           sendto
  0.00    0.000000           0         3           recvmsg
  0.00    0.000000           0         2           lstat
  0.00    0.000000           0         2           pipe
  0.00    0.000000           0         2           recvfrom
  0.00    0.000000           0         2         1 wait4
  0.00    0.000000           0         2           uname
  0.00    0.000000           0         2           fsync
  0.00    0.000000           0         2           getcwd
  0.00    0.000000           0         2           rename
  0.00    0.000000           0         2           sysinfo
  0.00    0.000000           0         2           statfs
  0.00    0.000000           0         2           set_robust_list
  0.00    0.000000           0         1           rt_sigprocmask
  0.00    0.000000           0         1         1 rt_sigreturn
  0.00    0.000000           0         1           bind
  0.00    0.000000           0         1           getsockname
  0.00    0.000000           0         1         1 getpeername
  0.00    0.000000           0         1           getsockopt
  0.00    0.000000           0         1           clone
  0.00    0.000000           0         1           execve
  0.00    0.000000           0         1           getrlimit
  0.00    0.000000           0         1           getuid
  0.00    0.000000           0         1           getgid
  0.00    0.000000           0         1           geteuid
  0.00    0.000000           0         1           getegid
  0.00    0.000000           0         1           arch_prctl
  0.00    0.000000           0         1           gettid
  0.00    0.000000           0         1           set_tid_address
  0.00    0.000000           0         1           clock_gettime
------ ----------- ----------- --------- --------- ----------------
100.00    0.002131                  1098        37 total

# strace -fc -e open elinks -dump http://www.baidu.com > /dev/null
Process 27553 attached
% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
  0.00    0.000000           0        84        18 open
------ ----------- ----------- --------- --------- ----------------
100.00    0.000000                    84        18 total

# strace ls
execve("/bin/ls", ["ls"], [/* 38 vars */]) = 0
brk(0)                                  = 0x1c1d000
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f9cf4c4f000
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
open("/etc/ld.so.cache", O_RDONLY)      = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=48663, ...}) = 0
mmap(NULL, 48663, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f9cf4c43000
close(3)                                = 0
open("/lib64/libselinux.so.1", O_RDONLY) = 3
read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0PY\200J9\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0755, st_size=124640, ...}) = 0
mmap(0x394a800000, 2221912, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x394a800000
mprotect(0x394a81d000, 2093056, PROT_NONE) = 0
mmap(0x394aa1c000, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1c000) = 0x394aa1c000
mmap(0x394aa1e000, 1880, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x394aa1e000
close(3)                                = 0
open("/lib64/librt.so.1", O_RDONLY)     = 3
read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\240!\300I9\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0755, st_size=47760, ...}) = 0
mmap(0x3949c00000, 2128816, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x3949c00000
mprotect(0x3949c07000, 2093056, PROT_NONE) = 0
mmap(0x3949e06000, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x6000) = 0x3949e06000
close(3)                                = 0
open("/lib64/libcap.so.2", O_RDONLY)    = 3
read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0000\23\300O9\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0755, st_size=19016, ...}) = 0
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f9cf4c42000
mmap(0x394fc00000, 2111776, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x394fc00000
mprotect(0x394fc04000, 2093056, PROT_NONE) = 0
mmap(0x394fe03000, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x3000) = 0x394fe03000
close(3)                                = 0
open("/lib64/libacl.so.1", O_RDONLY)    = 3
read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\200\36@O9\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0755, st_size=33816, ...}) = 0
mmap(0x394f400000, 2126416, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x394f400000
mprotect(0x394f407000, 2093056, PROT_NONE) = 0
mmap(0x394f606000, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x6000) = 0x394f606000
close(3)                                = 0
open("/lib64/libc.so.6", O_RDONLY)      = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0000\356AI9\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0755, st_size=1930416, ...}) = 0
mmap(0x3949400000, 3750184, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x3949400000
mprotect(0x394958a000, 2097152, PROT_NONE) = 0
mmap(0x394978a000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x18a000) = 0x394978a000
mmap(0x3949790000, 14632, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x3949790000
close(3)                                = 0
open("/lib64/libdl.so.2", O_RDONLY)     = 3
read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\340\r\0I9\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0755, st_size=23088, ...}) = 0
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f9cf4c41000
mmap(0x3949000000, 2109696, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x3949000000
mprotect(0x3949002000, 2097152, PROT_NONE) = 0
mmap(0x3949202000, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x2000) = 0x3949202000
close(3)                                = 0
open("/lib64/libpthread.so.0", O_RDONLY) = 3
read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0000^\200I9\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0755, st_size=146592, ...}) = 0
mmap(0x3949800000, 2212848, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x3949800000
mprotect(0x3949817000, 2097152, PROT_NONE) = 0
mmap(0x3949a17000, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x17000) = 0x3949a17000
mmap(0x3949a19000, 13296, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x3949a19000
close(3)                                = 0
open("/lib64/libattr.so.1", O_RDONLY)   = 3
read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\200\23\200K9\0\0\0"..., 832) = 832
fstat(3, {st_mode=S_IFREG|0755, st_size=21152, ...}) = 0
mmap(0x394b800000, 2113888, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x394b800000
mprotect(0x394b804000, 2093056, PROT_NONE) = 0
mmap(0x394ba03000, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x3000) = 0x394ba03000
close(3)                                = 0
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f9cf4c40000
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f9cf4c3e000
arch_prctl(ARCH_SET_FS, 0x7f9cf4c3e7a0) = 0
mprotect(0x394aa1c000, 4096, PROT_READ) = 0
mprotect(0x3949e06000, 4096, PROT_READ) = 0
mprotect(0x394f606000, 4096, PROT_READ) = 0
mprotect(0x394978a000, 16384, PROT_READ) = 0
mprotect(0x3949202000, 4096, PROT_READ) = 0
mprotect(0x3948e1f000, 8192, PROT_READ) = 0
mprotect(0x3949a17000, 4096, PROT_READ) = 0
mprotect(0x394ba03000, 4096, PROT_READ) = 0
munmap(0x7f9cf4c43000, 48663)           = 0
set_tid_address(0x7f9cf4c3ea70)         = 27564
set_robust_list(0x7f9cf4c3ea80, 24)     = 0
futex(0x7ffcb061778c, FUTEX_WAKE_PRIVATE, 1) = 0
futex(0x7ffcb061778c, FUTEX_WAIT_BITSET_PRIVATE|FUTEX_CLOCK_REALTIME, 1, NULL, 7f9cf4c3e7a0) = -1 EAGAIN (Resource temporarily unavailable)
rt_sigaction(SIGRTMIN, {0x3949805cb0, [], SA_RESTORER|SA_SIGINFO, 0x394980f7e0}, NULL, 8) = 0
rt_sigaction(SIGRT_1, {0x3949805d40, [], SA_RESTORER|SA_RESTART|SA_SIGINFO, 0x394980f7e0}, NULL, 8) = 0
rt_sigprocmask(SIG_UNBLOCK, [RTMIN RT_1], NULL, 8) = 0
getrlimit(RLIMIT_STACK, {rlim_cur=10240*1024, rlim_max=RLIM64_INFINITY}) = 0
statfs("/selinux", {f_type=0xf97cff8c, f_bsize=4096, f_blocks=0, f_bfree=0, f_bavail=0, f_files=0, f_ffree=0, f_fsid={0, 0}, f_namelen=255, f_frsize=4096}) = 0
statfs("/selinux", {f_type=0xf97cff8c, f_bsize=4096, f_blocks=0, f_bfree=0, f_bavail=0, f_files=0, f_ffree=0, f_fsid={0, 0}, f_namelen=255, f_frsize=4096}) = 0
stat("/selinux", {st_mode=S_IFDIR|0755, st_size=0, ...}) = 0
brk(0)                                  = 0x1c1d000
brk(0x1c3e000)                          = 0x1c3e000
open("/usr/lib/locale/locale-archive", O_RDONLY) = 3
fstat(3, {st_mode=S_IFREG|0644, st_size=99164480, ...}) = 0
mmap(NULL, 99164480, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f9ceedab000
close(3)                                = 0
ioctl(1, SNDCTL_TMR_TIMEBASE or SNDRV_TIMER_IOCTL_NEXT_DEVICE or TCGETS, {B38400 opost isig icanon echo ...}) = 0
ioctl(1, TIOCGWINSZ, {ws_row=50, ws_col=185, ws_xpixel=0, ws_ypixel=0}) = 0
open(".", O_RDONLY|O_NONBLOCK|O_DIRECTORY|O_CLOEXEC) = 3
fcntl(3, F_GETFD)                       = 0x1 (flags FD_CLOEXEC)
getdents(3, /* 16 entries */, 32768)    = 600
getdents(3, /* 0 entries */, 32768)     = 0
close(3)                                = 0
fstat(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(136, 2), ...}) = 0
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f9cf4c4e000
write(1, "active-profile\tdesktop-powersave"..., 124active-profile	desktop-powersave   functions		 laptop-battery-powersave  my-performance    spindown-disk	     virtual-guest
) = 124
write(1, "default\t\tenterprise-storage  lap"..., 128default		enterprise-storage  laptop-ac-powersave  latency-performance	   server-powersave  throughput-performance  virtual-host
) = 128
close(1)                                = 0
munmap(0x7f9cf4c4e000, 4096)            = 0
close(2)                                = 0
exit_group(0)                           = ?
+++ exited with 0 +++

# strace -c -S calls ls
active-profile	desktop-powersave   functions		 laptop-battery-powersave  my-performance    spindown-disk	     virtual-guest
default		enterprise-storage  laptop-ac-powersave  latency-performance	   server-powersave  throughput-performance  virtual-host
% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
  0.00    0.000000           0        27           mmap
  0.00    0.000000           0        16           mprotect
  0.00    0.000000           0        13           close
  0.00    0.000000           0        11           open
  0.00    0.000000           0        11           fstat
  0.00    0.000000           0         8           read
  0.00    0.000000           0         3           brk
  0.00    0.000000           0         2           write
  0.00    0.000000           0         2           munmap
  0.00    0.000000           0         2           rt_sigaction
  0.00    0.000000           0         2           ioctl
  0.00    0.000000           0         2           getdents
  0.00    0.000000           0         2           statfs
  0.00    0.000000           0         2         1 futex
  0.00    0.000000           0         1           stat
  0.00    0.000000           0         1           rt_sigprocmask
  0.00    0.000000           0         1         1 access
  0.00    0.000000           0         1           execve
  0.00    0.000000           0         1           fcntl
  0.00    0.000000           0         1           getrlimit
  0.00    0.000000           0         1           arch_prctl
  0.00    0.000000           0         1           set_tid_address
  0.00    0.000000           0         1           set_robust_list
------ ----------- ----------- --------- --------- ----------------
100.00    0.000000                   112         2 total
```

```
# cat >> /a         -> 先用strace -p跟踪，然后输入111，然后Ctrl+C退出
111
^C
# ps aux | grep cat
root     27594  0.0  0.0 100948   544 pts/2    S+   01:52   0:00 cat
root     27596  0.0  0.0 103320   824 pts/1    S+   01:53   0:00 grep cat
# strace -p 27594
Process 27594 attached
read(0, "111\n", 32768)                 = 4
write(1, "111\n", 4)                    = 4
read(0, 0x1197000, 32768)               = ? ERESTARTSYS (To be restarted if SA_RESTART is set)
--- SIGINT {si_signo=SIGINT, si_code=SI_KERNEL} ---
+++ killed by SIGINT +++
# kill -l
 1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL	 5) SIGTRAP
 6) SIGABRT	 7) SIGBUS	 8) SIGFPE	 9) SIGKILL	10) SIGUSR1
11) SIGSEGV	12) SIGUSR2	13) SIGPIPE	14) SIGALRM	15) SIGTERM
16) SIGSTKFLT	17) SIGCHLD	18) SIGCONT	19) SIGSTOP	20) SIGTSTP
21) SIGTTIN	22) SIGTTOU	23) SIGURG	24) SIGXCPU	25) SIGXFSZ
26) SIGVTALRM	27) SIGPROF	28) SIGWINCH	29) SIGIO	30) SIGPWR
31) SIGSYS	34) SIGRTMIN	35) SIGRTMIN+1	36) SIGRTMIN+2	37) SIGRTMIN+3
38) SIGRTMIN+4	39) SIGRTMIN+5	40) SIGRTMIN+6	41) SIGRTMIN+7	42) SIGRTMIN+8
43) SIGRTMIN+9	44) SIGRTMIN+10	45) SIGRTMIN+11	46) SIGRTMIN+12	47) SIGRTMIN+13
48) SIGRTMIN+14	49) SIGRTMIN+15	50) SIGRTMAX-14	51) SIGRTMAX-13	52) SIGRTMAX-12
53) SIGRTMAX-11	54) SIGRTMAX-10	55) SIGRTMAX-9	56) SIGRTMAX-8	57) SIGRTMAX-7
58) SIGRTMAX-6	59) SIGRTMAX-5	60) SIGRTMAX-4	61) SIGRTMAX-3	62) SIGRTMAX-2
63) SIGRTMAX-1	64) SIGRTMAX	

# top
PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
    1 root      20   0 19368 1512 1192 S  0.0  0.1   0:00.99 init
# strace -p 1
Process 1 attached
select(10, [3 5 6 7 9], [], [7 9], NULL

# ps aux | grep flush
root       414  0.0  0.0      0     0 ?        S    Feb03   0:00 [kdmflush]
root       416  0.0  0.0      0     0 ?        S    Feb03   0:00 [kdmflush]
root       924  0.0  0.0      0     0 ?        S    Feb03   0:00 [kdmflush]
root     27487  0.0  0.0      0     0 ?        S    01:29   0:00 [flush-253:0]
root     27605  0.0  0.0 103320   820 pts/2    R+   01:58   0:00 grep flush
# strace -p 414
strace: attach: ptrace(PTRACE_ATTACH, ...): Operation not permitted

```


###### ltrace

> -S for ltrace:
	Display system calls as well as library calls

> -f fork -c count

```
# ltrace
ltrace: too few arguments
Usage: ltrace [option ...] [command [arg ...]]
Trace library calls of a given program.

  -a, --align=COLUMN  align return values in a secific column.
  -c                  count time and calls, and report a summary on exit.
  -C, --demangle      decode low-level symbol names into user-level names.
  -d, --debug         print debugging info.
      --dl            show calls to symbols in dlopened libraries.
  -e expr             modify which events to trace.
  -f                  follow forks.
  -h, --help          display this help and exit.
  -i                  print instruction pointer at time of library call.
  -l, --library=FILE  print library calls from this library only.
  -L                  do NOT display library calls.
  -n, --indent=NR     indent output by NR spaces for each call level nesting.
  -o, --output=FILE   write the trace output to that file.
  -p PID              attach to the process with the process ID pid.
  -r                  print relative timestamps.
  -s STRLEN           specify the maximum string size to print.
  -S                  display system calls.
  -t, -tt, -ttt       print absolute timestamps.
  -T                  show the time spent inside each call.
  -u USERNAME         run command with the userid, groupid of username.
  -V, --version       output version information and exit.
  -x NAME             treat the global NAME like a library subroutine.

```

```
# ltrace -cf grep -r adsf /etc
% time     seconds  usecs/call     calls      function
------ ----------- ----------- --------- --------------------
 43.56   17.591152         280     62690 memchr
 29.56   11.936609    11936609         1 __libc_start_main
  6.15    2.482631        1019      2434 read
  5.49    2.217611         896      2474 malloc
  5.38    2.173711        1098      1978 memcpy
  5.25    2.120744        1652      1283 memrchr
  0.60    0.243770          91      2651 readdir
  0.60    0.241558         124      1934 openat
  0.56    0.225004          91      2472 free
  0.46    0.187234          89      2096 memmove
  0.43    0.173765          87      1975 strlen
  0.38    0.153903         110      1393 close
  0.32    0.129572         107      1202 fcntl
  0.32    0.127844         110      1154 __fxstat
  0.31    0.126474         109      1152 isatty
  0.16    0.064282         134       478 __fxstatat
  0.12    0.047438          87       543 __errno_location
  0.11    0.044475          86       513 mbrtowc
  0.08    0.033983         141       240 fdopendir
  0.06    0.025529         106       240 closedir
  0.05    0.021584          84       255 __ctype_b_loc
  0.01    0.002445          81        30 strncmp
  0.01    0.002069          86        24 realloc
  0.00    0.001300          81        16 memset
  0.00    0.000715         715         1 exit
  0.00    0.000597         597         1 setlocale
  0.00    0.000364         364         1 _obstack_begin
  0.00    0.000357          89         4 wcrtomb
  0.00    0.000247          82         3 calloc
  0.00    0.000221         110         2 strrchr
  0.00    0.000213         106         2 fclose
  0.00    0.000196          98         2 __xstat
  0.00    0.000195         195         1 
  0.00    0.000185         185         1 bindtextdomain
  0.00    0.000167          83         2 getenv
  0.00    0.000166          83         2 __fpending
  0.00    0.000166         166         1 textdomain
  0.00    0.000164          82         2 __ctype_get_mb_cur_max
  0.00    0.000163         163         1 __cxa_atexit
  0.00    0.000144         144         1 SYS_exit_group
  0.00    0.000102         102         1 re_compile_pattern
  0.00    0.000098          98         1 re_set_syntax
  0.00    0.000082          82         1 strchr
  0.00    0.000082          82         1 strchrnul
  0.00    0.000081          81         1 getpagesize
  0.00    0.000081          81         1 strcmp
------ ----------- ----------- --------- --------------------
100.00   40.379473                 89261 total
            
# ltrace -Scf grep -r adsf /etc                 -> syscalls带有前缀SYS_
% time     seconds  usecs/call     calls      function
------ ----------- ----------- --------- --------------------
 55.76   17.290152         275     62690 memchr
 29.53    9.156585     9156585         1 __libc_start_main
  7.19    2.228732         840      2651 readdir
  0.83    0.257891         105      2434 read
  0.67    0.208067         107      1934 openat
  0.65    0.201990          81      2472 free
  0.65    0.201552          81      2474 malloc
  0.55    0.169917          81      2096 memmove
  0.52    0.161866          81      1975 strlen
  0.52    0.160094          80      1978 memcpy
  0.46    0.142012         101      1393 close
  0.41    0.126377         105      1202 fcntl
  0.40    0.123038         106      1154 __fxstat
  0.40    0.122498         106      1152 isatty
  0.35    0.107862          84      1283 memrchr
  0.18    0.054795         114       478 __fxstatat
  0.15    0.046364          90       513 mbrtowc
  0.13    0.040212          74       543 __errno_location
  0.11    0.034729         144       240 fdopendir
  0.10    0.029825          12      2436 SYS_read
  0.08    0.024295         101       240 closedir
  0.08    0.023619          12      1934 SYS_257
  0.07    0.022474          88       255 __ctype_b_loc
  0.05    0.014557           8      1640 SYS_close
  0.04    0.012867           7      1682 SYS_fcntl
  0.04    0.011818           8      1399 SYS_fstat
  0.03    0.009514           8      1152 SYS_ioctl
  0.02    0.007697          16       478 SYS_262
  0.02    0.005746          11       480 SYS_getdents
  0.01    0.002625          87        30 strncmp
  0.01    0.002131          88        24 realloc
  0.00    0.001409          88        16 memset
  0.00    0.000720         720         1 exit
  0.00    0.000672         672         1 setlocale
  0.00    0.000387         193         2 getenv
  0.00    0.000365          91         4 wcrtomb
  0.00    0.000287          95         3 calloc
  0.00    0.000277          21        13 SYS_mmap
  0.00    0.000263         131         2 __ctype_get_mb_cur_max
  0.00    0.000254         127         2 strrchr
  0.00    0.000221         221         1 textdomain
  0.00    0.000218         218         1 
  0.00    0.000210         105         2 fclose
  0.00    0.000202         101         2 __xstat
  0.00    0.000190         190         1 bindtextdomain
  0.00    0.000181         181         1 _obstack_begin
  0.00    0.000176          88         2 __fpending
  0.00    0.000172         172         1 strchr
  0.00    0.000171          34         5 SYS_open
  0.00    0.000171         171         1 re_compile_pattern
  0.00    0.000166         166         1 __cxa_atexit
  0.00    0.000165         165         1 strchrnul
  0.00    0.000154         154         1 SYS_exit_group
  0.00    0.000129          18         7 SYS_brk
  0.00    0.000085          85         1 re_set_syntax
  0.00    0.000083          83         1 getpagesize
  0.00    0.000082          82         1 strcmp
  0.00    0.000080          20         4 SYS_mprotect
  0.00    0.000064          64         1 SYS_arch_prctl
  0.00    0.000029          29         1 SYS_access
  0.00    0.000026          26         1 SYS_munmap
  0.00    0.000019           9         2 SYS_stat
------ ----------- ----------- --------- --------------------
100.00   31.009529                100496 total
```

```
# cat > /a              -> 先用ltrace跟踪cat进程，然后输入222 333，最后Ctrl+C退出
222
333
^C
# ps aux | grep cat
root     27660  0.0  0.0 100948   548 pts/2    S+   02:23   0:00 cat
root     27662  0.0  0.0 103320   824 pts/1    S+   02:23   0:00 grep cat
# ltrace -p 27660
write(1, "222\n", 4)                                                                                              = 4
read(0, "333\n", 32768)                                                                                           = 4
write(1, "333\n", 4)                                                                                              = 4
read(0,  <unfinished ...>
--- SIGINT (Interrupt) ---
+++ killed by SIGINT +++

```

### Use valgrind to Profile Cache Usage

###### Processor Caches

*    Computer systems use different tiers of memory to try to ensure that the processor always has data when it is ready to start processing. 
*    The fastest storage on any system are the memory registers on the processor itself. 
*    Cache memory is much faster than main memory. Whereas main memory has access times of 8+ nanoseconds, cache memory can have access times as little as a few CPU clock cycles.

```
# getconf -a | grep CACHE
LEVEL1_ICACHE_SIZE                 32768
LEVEL1_ICACHE_ASSOC                8
LEVEL1_ICACHE_LINESIZE             64
LEVEL1_DCACHE_SIZE                 32768
LEVEL1_DCACHE_ASSOC                8
LEVEL1_DCACHE_LINESIZE             64
LEVEL2_CACHE_SIZE                  2097152
LEVEL2_CACHE_ASSOC                 8
LEVEL2_CACHE_LINESIZE              64
LEVEL3_CACHE_SIZE                  0
LEVEL3_CACHE_ASSOC                 0
LEVEL3_CACHE_LINESIZE              0
LEVEL4_CACHE_SIZE                  0
LEVEL4_CACHE_ASSOC                 0
LEVEL4_CACHE_LINESIZE              0

# getconf -a | grep CACHE | grep _SIZE
LEVEL1_ICACHE_SIZE                 32768
LEVEL1_DCACHE_SIZE                 32768
LEVEL2_CACHE_SIZE                  2097152
LEVEL3_CACHE_SIZE                  0
LEVEL4_CACHE_SIZE                  0

```

```
# yum install x86info
# x86info -c
x86info v1.25.  Dave Jones 2001-2009
Feedback to <davej@redhat.com>.

Found 2 CPUs
--------------------------------------------------------------------------
CPU #1
EFamily: 0 EModel: 0 Family: 6 Model: 13 Stepping: 3
CPU Model: Pentium M 
Processor name string: QEMU Virtual CPU version (cpu64-rhel6)
Type: 0 (Original OEM)	Brand: 0 (Unsupported)
Number of cores per physical package=1
Number of logical processors per socket=1
Number of logical processors per core=1
APIC ID: 0x0	Package: 0  Core: 0   SMT ID 0
Cache info
 L1 Instruction cache: 32KB, 8-way associative. 64 byte line size.
 L1 Data cache: 32KB, 8-way associative. 64 byte line size.
 L2 cache: 2MB, 8-way associative. 64 byte line size.
TLB info
--------------------------------------------------------------------------
CPU #2
EFamily: 0 EModel: 0 Family: 6 Model: 13 Stepping: 3
CPU Model: Pentium M 
Processor name string: QEMU Virtual CPU version (cpu64-rhel6)
Type: 0 (Original OEM)	Brand: 0 (Unsupported)
Number of cores per physical package=1
Number of logical processors per socket=1
Number of logical processors per core=1
APIC ID: 0x1	Package: 0  Core: 0   SMT ID 0
Cache info
 L1 Instruction cache: 32KB, 8-way associative. 64 byte line size.
 L1 Data cache: 32KB, 8-way associative. 64 byte line size.
 L2 cache: 2MB, 8-way associative. 64 byte line size.
TLB info
--------------------------------------------------------------------------

```

```
# lscpu | grep ^L
L1d cache:             32K
L1i cache:             32K
L2 cache:              4096K

# lscpu | grep ^L
L1d cache:             32K
L1i cache:             32K
L2 cache:              256K
L3 cache:              8192K
```

###### Caches Architecture

*    Cache memory is organized into lines. Each line of cache can be used to cache a specific location in memory.
*    Most computer systems have separate caches for instructions for the processor, the I-cache, and data, the D-cache. This is sometimes referred to as the Harvard memory architecture.
*    On a multiprocessor system, each CPU has its own separate cache. Each cache has an associated cache controller. When a processor makes a reference to main memory, the cache controller first checks to see if the requested address is in cache and will satisfy the processor's request from there if it is. This is referred to as a cache hit. If the requested memory is not in cache, then a cache miss occurs and the requested location must be read from main memory and brought into cache. This is referred to as a cache-line fill.
*    The cache controller contains an array of cache entries for each line of cache. In addition to the cached memory contents, each cache entry consists of a tag and flags describing the status of that cache entry.
*    The processor both reads from and writes to cache memory. In the case of write operations, cache memory can be configured as either write-through or write-back. Write-back caching is more efficient than write-through caching.
*    On multiprocessor systems, the need to maintain cache coherency arises. That is, if a processor updates a line of cache in memory, it must let the other processors on the system know that the location in memory has been updated in case they are caching it also. This is referred to as cache snooping and is performed in the system hardware
*    NUMA is discussed in more detail in later

> cache hit 

> cache miss

> cache-line fill

> cache snoop

> write back/write through

###### Direct mapped cache and Fully associative cache

*    Direct mapped cache is the least expensive type of cache memory. Each line of direct mapped cache can only cache a specific location in main memory.
*    Fully associative cache memory is the most flexible type of cache memory and consequently the most expensive, as it requires the most circuitry to implement. A fully associative cache memory line can cache any location in main memory.
*    Most systems compromise and use set associative cache memory. Set associative cache memory is usually referred to as n-way set associative, where n is some power of 2. Set associative cache memory allows a memory location to be cached into any one of n lines of cache. Set associative memory provides a good compromise between direct mapped cache and fully associative cache.

###### valgrind and perf

```
# yum install valgrind 
# valgrind --tool=cachegrind ls
==27825== Cachegrind, a cache and branch-prediction profiler
==27825== Copyright (C) 2002-2012, and GNU GPL'd, by Nicholas Nethercote et al.
==27825== Using Valgrind-3.8.1 and LibVEX; rerun with -h for copyright info
==27825== Command: ls
==27825== 
anaconda-ks.cfg  install.log  install.log.syslog  NETBOOT_METHOD.TXT  RECIPE.TXT
==27825== 
==27825== I   refs:      460,474
==27825== I1  misses:      1,476
==27825== LLi misses:      1,403
==27825== I1  miss rate:    0.32%
==27825== LLi miss rate:    0.30%
==27825== 
==27825== D   refs:      168,378  (124,475 rd   + 43,903 wr)
==27825== D1  misses:      4,710  (  3,782 rd   +    928 wr)
==27825== LLd misses:      3,159  (  2,336 rd   +    823 wr)
==27825== D1  miss rate:     2.7% (    3.0%     +    2.1%  )
==27825== LLd miss rate:     1.8% (    1.8%     +    1.8%  )
==27825== 
==27825== LL refs:         6,186  (  5,258 rd   +    928 wr)
==27825== LL misses:       4,562  (  3,739 rd   +    823 wr)
==27825== LL miss rate:      0.7% (    0.6%     +    1.8%  )



# yum install -y perf
# perf list
# perf stat -e cache-misses ls
```

###### oProfile

oProfile是Linux平台上的一个功能强大的性能分析工具，支持两种采样(sampling)方式：基于事件的采样(eventbased)和基于时间的采样(timebased)，它可以工作在不同的体系结构上，包括MIPS、ARM、IA32、IA64和AMD。 








