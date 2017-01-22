---
layout: post
title:  "性能优化介绍(442-1)"
categories: Linux
tags: RHCA 442
---

### Performance Tuning is

*    Black art
*    Both of the hardware and Software
*    Tuning goals: Pesponse time or Throughout
*    Performance tuning is undertaken to ...
*    Performance management is the process of ...

### Performance Level Agreements

*    Concrete, realistic goals
*    Reproducibly measurable criterial and normal variation
*    Minimum acceptable performance standards
*    Operational windows
*    Notification procedures

### Disable Unused Services

```
# service $service_name stop
# chkconfig $server_name off
# service $server_name status
# yum remove -y $package_name
```

### Monitoring vs. Profiling

###### Monitoring

*    Tracking performance metrics
*    Alerting when performance is out-of-specification

###### Profiling

*    Establishing baseline performance criteria
*    Finding hot spots in application runtime
*    Identifying runtime characteristics as a basis 

优化步骤：
1. OS: 针对os做优化，而不是针对数据库本身等，所以需要懂OS的工作原理
2. 监控 + 分析
3. 调整
