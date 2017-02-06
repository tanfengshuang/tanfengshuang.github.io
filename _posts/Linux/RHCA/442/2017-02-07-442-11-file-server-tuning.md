---
layout: post
title:  "File Server Tuning(442-11)"
categories: Linux
tags: RHCA 442
---

### File System Journaling

The ext3/ext4 journal can work in three different modes, chosen by passing the data=mode option to the file system at mount time. The default mode is ordered. The three modes are:

*    ordered: Only metadata is recorded in the journal. Journal entries will only be committed after all file data has been flushed to disk.
*    writeback: Only metadata is recorded in the journal, but data ordering is not preserved. This is rumored to be the fastest method, and internal file system integrity is guaranteed, but after a crash and recovery, old data may show up inside files.
*    journal: All data is first stored in the journal before being written out to disk. This gives the best reliability for the file system but can be the worst performer in most workloads. In workloads with many small writes, this could help increase performance since the disk elevator gets a chance to group writes together.

Both the XFS and ext4 file systems turn on barriers by default, while ext3 has barriers disabled by default.  To enable or disable barriers at mount time on XFS file systems, use either the barrier or the 
nobarrier option, respectively. For ext3 and ext4 file systems, use either the barrier=1 or barrier=0 mount option, respectively. On devices with a battery-backed cache, barriers can be disabled for a performance improvement while still guaranteeing data integrity.

```

```

### Selecting a tuned Profile for a File Server Workload

The two most important subsystems in a file server are disk and network.
The network connection will, in all probability, be heavily used but not latency-sensitive. The disk load can vary from file server to file server, and depends mostly on the type of files being served and the
type of clients accessing the service. For example, a file server that serves out virtual machine images to hypervisors will have different disk needs than one serving out spreadsheets and word processor documents to an office. For this unit, the discussion will focus on file servers, which perform more read operations than write operations on large files.

Given the requirements, the choice of tuned profiles on RHEL 7 can be narrowed down to four options:

*    throughput-performance: This profile sets the CPU governor and energy performance bias to performance. Disk readahead value is increased to 4096. The throughput-performance profile also enables sysctl settings that improves the throughput performance of disk and network I/O.
*    latency-performance: This profile is geared for low-latency performance by disabling power-saving mechanisms. Like the throughput-performance profile, CPU governor and energy performance bias are set to performance to guarantee the highest clock frequency. By statically setting the clock frequency, latency-performance prevents the latency which occurs when CPU frequency switches in response to workload fluctuations.
*    network-throughput: This profile is designed for throughput network tuning. It inheritsproperties from the throughput-performance profile and increases kernel network buffers to boost network performance.
*    network-latency: This profile is aimed at decreasing network latency. It inherits properties from the latency-performance profile and also disables transparent hugepages and NUMA balancing, as well as tunes several other network-related sysctl parameters to lower network latency.

### Network Performance Tuning

Network throughput can depend on a number of factors(While some of these are properties of the hardware being used, others can also be influenced from a system.): 

*    the network cards used
*    the type of cabling
*    number of hops in a connection
*    the size of the packets being sent

###### ethtoll

One of the most drastic settings to change is the link speed of the network card being used. To query the current setting, use a tool called ethtool


###### qperf

One of the newer and more extensive tools is qperf. When using an Ethernet network, qperf can measure TCP, UDP, RDS, SCTP, and SDP socket throughputs and latencies. qperf can also perform measurements on RDMA and Infiniband networks.

qperf needs to run on two machines. One (the listener) runs qperf without any options. The other (the sender) invokes qperf with the name of the first host as the first argument, followed by the test options. For example, to run TCP and UDP bandwidth tests between desktopX and desktopY, start a qperf listener on desktopY:

###### Network data send

The following outline shows the steps of network transmission.

*    Data is written to a socket (a file-like object) and is then put in the transmit buffer.
*    The kernel encapsulates the data into a protocol data unit (PDU).
*    The PDUs move to the per-device transmit queue.
*    The network device driver copies the PDU from the head of the transmit queue to the NIC.
*    The NIC sends the data and raises an interrupt when transmitted.

###### Network data reception

The following outline shows the steps of network reception.

*    The NIC receives a frame and uses DMA to copy the frame into the receive buffer.
*    The NIC raises a hard interrupt.
*    The kernel handles the hard interrupt and schedules a soft interrupt to handle the packet. This move from hard interrupt to soft interrupt prevents a phenomenon known as livelock. A hard interrupt will preempt everything, including other interrupt handlers. This could cause the receive buffers to fill up during heavy receive loads if packets are not moved out of the receive buffers quickly enough. The kernel processes the hard interrupt as fast as it can, and when the soft interrupt is handled later, the entire receive buffer is processed.
*    The soft interrupt is handled and moves the packet to the IP layer.
*    If the packet is destined for local delivery, the PDU will be decapsulated and put in a socket receive buffer. If a process was waiting on this socket, it will now process the data from the receive buffer.

###### Network Buffers tunables

The buffers (or queues) used in this process consist of the core networking read and write buffers (used for UDP and TCP), per-socket TCP read and write buffers, fragmentation buffers, and DMA buffers for the network card. 

The kernel automatically adjusts the size of these buffers based on the current network utilization, but within the limits specified by the kernel tunables mentioned next.

sysctl tunables:

*    net.core.rmem_max, net.core.wmem_max: The core networking maximum socket receive/send (read/write) buffers. Values are in bytes.
*    net.ipv4.tcp_mem, net.ipv4.udp_mem: The systemwide memory limits for TCP and UDP, respectively. These settings consist of three fields: min, pressure, and max.
*    net.ipv4.tcp_rmem, net.ipv4.tcp_wmem: The receive/send TCP socket buffers. Values are in bytes. A socket buffer will start at the size of default bytes (second value) and be automatically adjusted between min (first value) and max (third value) based on need.

### BDP and Window Scaning 


### Bonding and Link Aggressive


### Jumbo Frames
