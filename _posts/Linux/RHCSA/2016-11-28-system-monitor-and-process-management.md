---
layout: post
title:  "系统监控和进程管理"
categories: Linux
tags: RHCSA kill pidof pgrep nice last lastlog pstree jobs fg bg
---

关于linux的基本命令的详细使用可以通过网站 [http://man.linuxde.net/](http://man.linuxde.net/)查看

### 查询系统状况

*    uname
*    hostname
*    last       列出最近的用户登录
*    lastlog    列出每一个用户的最近登录状态
*    free
*    top

### Linux系统进程

*    ps
*    pstree
*    top

#### 进程概述

系统的原始进程是systemd，systemd的PID总是1，除了systemd外，所有的进程都有父进程


#### 控制进程

1. pkill
2. killall

        killall [-signal] <进程名>

3. kill

        kill [-signal] <PID>

+    15     默认为15
+    -9     立即终止
+    -18    继续进程
+    -19    暂停进程

```
# kill -l
 1) SIGHUP   2) SIGINT   3) SIGQUIT  4) SIGILL   5) SIGTRAP
 6) SIGABRT  7) SIGBUS   8) SIGFPE   9) SIGKILL 10) SIGUSR1
11) SIGSEGV 12) SIGUSR2 13) SIGPIPE 14) SIGALRM 15) SIGTERM
16) SIGSTKFLT   17) SIGCHLD 18) SIGCONT 19) SIGSTOP 20) SIGTSTP
21) SIGTTIN 22) SIGTTOU 23) SIGURG  24) SIGXCPU 25) SIGXFSZ
26) SIGVTALRM   27) SIGPROF 28) SIGWINCH    29) SIGIO   30) SIGPWR
31) SIGSYS  34) SIGRTMIN    35) SIGRTMIN+1  36) SIGRTMIN+2  37) SIGRTMIN+3
38) SIGRTMIN+4  39) SIGRTMIN+5  40) SIGRTMIN+6  41) SIGRTMIN+7  42) SIGRTMIN+8
43) SIGRTMIN+9  44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9  56) SIGRTMAX-8  57) SIGRTMAX-7
58) SIGRTMAX-6  59) SIGRTMAX-5  60) SIGRTMAX-4  61) SIGRTMAX-3  62) SIGRTMAX-2
63) SIGRTMAX-1  64) SIGRTMAX
```

#### 搜索进程

```
# pgrep
# pidof
# ps aux | grep xxx    
```

#### 进程的优先级

进程的优先级用nice值来表示, 优先级受到进程的"好心"值(nice value)的影响，这个值的范围是-20到19(其中-20最高，19最低)，默认为0

*    nice   以一个不同的nice值来运行指令

    nice -n command

*    renice 改变一个运行进程的nice值

    renice -n pid

#### 后台运行

*    command &
*    nohup command &
*    ctrl + z   将一个正在运行的前台进程暂时停止，并丢入后台

#### 后台进程的控制

*    jobs   列出系统作业号和名称
*    fg [%作业号]   前台恢复运行
*    bg [%作业号]   后台恢复运行
*    kill [%作业号] 给对应的作业发送终止信号
