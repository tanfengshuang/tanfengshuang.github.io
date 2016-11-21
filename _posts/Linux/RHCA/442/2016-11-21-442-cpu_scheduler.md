---
layout: post
title:  "442-cpu_scheduler"
categories: Linux
tags: RHCA 442
---

```
# chrt -p 1
# chrt -p 3
# chrt -m
SCHED_OTHER min/max priority	: 0/0
SCHED_FIFO min/max priority	: 1/99
SCHED_RR min/max priority	: 1/99
SCHED_BATCH min/max priority	: 0/0
SCHED_IDLE min/max priority	: 0/0
# chrt --fifo 1 ls

# ps ax -o pid,cmd,class,rtprio,pri,nice,policy
#


# yum install -y kernel-doc
# ls /usr/share/doc/kernel-firmware-2.6.32/Documetation/scheduler/sched-design-CFS.txt


# nice
# renice

# time ls /tmp
C125406BB39FE0419D38064D061FFB0D84E7AA12.31.0.1650.63_service_ipc  gedit.ftan.3203504713  kde-ftanzUnFbU  ksocket-ftan  OSL_PIPE_500_SingleOfficeIPC_ec5428ff05f318b88ac4840ad1b6d
clipboardcache                                                     gedit.root.3942809567  keyring-PkJqAt  lu3waj9x.tmp  plugtmp
clipboardcache-1                                                   hsperfdata_ftan        keyring-Rn1GQt  orbit-ftan    pulse-eEDKY6M0JDiv
evince-4843                                                        kde-ftan               krb5cc_500      orbit-gdm     staplJERbf

real	0m0.025s
user	0m0.000s
sys	0m0.004s

# type time
time is a shell keyword


 
# /usr/bin/time ls /tmp
C125406BB39FE0419D38064D061FFB0D84E7AA12.31.0.1650.63_service_ipc  gedit.ftan.3203504713  kde-ftanzUnFbU  ksocket-ftan	OSL_PIPE_500_SingleOfficeIPC_ec5428ff05f318b88ac4840ad1b6d
clipboardcache							   gedit.root.3942809567  keyring-PkJqAt  lu3waj9x.tmp	plugtmp
clipboardcache-1						   hsperfdata_ftan	  keyring-Rn1GQt  orbit-ftan	pulse-eEDKY6M0JDiv
evince-4843							   kde-ftan		  krb5cc_500	  orbit-gdm	staplJERbf
0.00user 0.00system 0:00.00elapsed 33%CPU (0avgtext+0avgdata 3584maxresident)k
0inputs+0outputs (0major+270minor)pagefaults 0swaps

# export TIME="\n %s %S %U"
# /usr/bin/time ls /tmp
C125406BB39FE0419D38064D061FFB0D84E7AA12.31.0.1650.63_service_ipc  gedit.ftan.3203504713  kde-ftanzUnFbU  ksocket-ftan	OSL_PIPE_500_SingleOfficeIPC_ec5428ff05f318b88ac4840ad1b6d
clipboardcache							   gedit.root.3942809567  keyring-PkJqAt  lu3waj9x.tmp	plugtmp
clipboardcache-1						   hsperfdata_ftan	  keyring-Rn1GQt  orbit-ftan	pulse-eEDKY6M0JDiv
evince-4843							   kde-ftan		  krb5cc_500	  orbit-gdm	staplJERbf

 0 0.00 0.00
```
