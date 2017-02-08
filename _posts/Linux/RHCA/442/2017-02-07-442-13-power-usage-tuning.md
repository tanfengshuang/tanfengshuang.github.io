---
layout: post
title:  "Power Usage Tuning(442-13)"
categories: Linux
tags: RHCA 442
---

### Power Saving Strategy

A number of general strategies can be used to reduce power consumption on a computer:
*    Disable unused services.
*    Disable unused hardware devices.
*    Avoid operations that poll the system.
*    Extend the lifetimes of deferred activity.
*    Allow inactive devices to enter power-saving states
*    Reduce the number of wakeups from sleep.

### Profile Power Usage with powertop

```
# yum install powertop

# powertop ls
PowerTOP 2.3      Overview   Idle stats   Frequency stats   Device stats   Tunables
Summary: 11.9 wakeups/second,  0.0 GPU ops/seconds, 0.0 VFS ops/sec and 0.7% CPU use
                Usage       Events/s    Category       Description
              4.0 ms/s       1.0        Process        [flush-253:0]
            366.7 µs/s       2.0        Process        powertop ls
             28.4 µs/s       2.0        Process        [events/0]
             25.6 µs/s       2.0        Process        [events/1]
            450.7 µs/s       1.0        Process        /usr/bin/python /usr/bin/beah-rhts-task
            201.2 µs/s       1.0        Process        sendmail: accepting connections
             31.4 µs/s       1.0        Process        /usr/sbin/httpd
             12.7 µs/s       1.0        Process        [bdi-default]
              9.3 µs/s       1.0        Process        [flush-252:0]
              0.9 ms/s      0.00        Interrupt      [25] virtio2-requests
            284.9 µs/s      0.00        Timer          tick_sched_timer
            274.6 µs/s      0.00        Timer          rh_timer_func
            139.6 µs/s      0.00        Interrupt      [9] RCU(softirq)
            130.9 µs/s      0.00        Process        sshd: root@pts/3
            102.3 µs/s      0.00        Interrupt      [1] timer(softirq)
            102.1 µs/s      0.00        Interrupt      [3] net_rx(softirq)
             86.3 µs/s      0.00        Timer          clocksource_watchdog
             37.2 µs/s      0.00        Timer          hrtimer_wakeup
             32.0 µs/s      0.00        Timer          delayed_work_timer_fn
             26.6 µs/s      0.00        Interrupt      [7] sched(softirq)
             21.5 µs/s      0.00        Interrupt      [27] virtio0-input.0
             12.9 µs/s      0.00        Timer          process_timeout
              2.0 µs/s      0.00        Timer          tcp_write_timer
            100.0%                      Device         USB device: UHCI Host Controller


```


