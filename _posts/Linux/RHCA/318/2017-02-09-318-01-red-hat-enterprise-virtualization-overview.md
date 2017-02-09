---
layout: post
title:  "VIRTUALIZATION OVERVIEW(318-1)"
categories: Linux
tags: RHCA 318 VirtIO
---

### VirtIO

virtio 是 KVM 虚拟环境下针对 I/O 虚拟化的最主要的一个通用框架。virtio 提供了一套有效、易维护、易开发、易扩展的中间层 API

Linux Kernel 支持很多 Hypervisor，比如 KVM、Xen 和 VMware 的 VMI 等。每个 Hypervisor 都有自己独特的 block、network、console 等设备模型，设备驱动多样化的特性和优化方式使得各个平台共有性的东西越来越少,亟需提供一种通用的框架和标准接口来减少各 Hypervisor 虚拟化设备之间的差异，从而减少驱动开发的负担。

虚拟化主要包括处理器的虚拟化, 内存的虚拟化以及 I/O 的虚拟化等，从 2006 年开始，KVM 上设备 I/O 虚拟化的性能问题也显现了出来，此时由 Rusty Russell 开发的 virtio 引起了开发者们的注意并逐渐被 KVM 等虚拟化平台接纳并作为了其 I/O 虚拟化最主要的一个通用框架。

Virtio 使用 virtqueue 来实现其 I/O 机制，每个 virtqueue 就是一个承载大量数据的 queue。vring 是 virtqueue 的具体实现方式，针对 vring 会有相应的描述符表格进行描述。

