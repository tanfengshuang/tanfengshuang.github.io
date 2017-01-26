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

### strace and ltrace Usage

```
# strace -c uname
# strace -e open iptables -L

# strace -fc -S calls elinks -dump http://www.baidu.com
# strace -fc -e open elinks -dump http://www.baidu.com
```

> -S for strace:
> 	-S sortby   Sort the output of the histogram printed by the -c option by the specified criterion.  Legal values are time, calls, name, and nothing (default time).

```
# ltrace -Sfc elinks -dump http://www.baidu.com
```

> -S for ltrace:
	Display system calls as well as library calls

> -f fork -c count

### Use valgrind to Profile Cache Usage






