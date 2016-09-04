---
title: mTCP论文重要观点
date: 2016-09-02 11:27:37
tags:
categories: 网络
---

相对其他 TCP/IP 优化方案，mTCP 重要的核心改动有三点：

- Translates multiple expensive system calls into a single shared memory reference.
- Allows efficient flow-level event aggregation.
- performs batched packet I/O for high I/O efficiency

<br>

Linux 内核 TCP 协议栈存在的缺点：

- **Lack of connection locality** : 在多核系统上，很多应用程序都使用多线程技术来提高处理性能。但是这些线程都共享一个 Listen Socket 去监听特定的端口，这样就导致对 socket's accept queue 的访问需要加锁，从而降低了多核的处理效率。其次，对于同一个 TCP Socket 而言，处理 connection 的内核代码段 和 处理 sends/receives 数据的用户代码段 可能运行在不同的 CPU 上，这样就增大了 CPU cache-misses 和 cache-line sharing 的概率。
- **Shared file descriptor space** : 在兼容 POSIX 标准的操作系统上，一个进程内的所有线程共享一个 file descriptor (fd) space。当为新建立的 socket 申请一个文件描述符时，Linux内核会在 available fd number 中搜索出最小值返回出去。由于 fd space 是共享的，因此多线程访问的时候自然需要加锁防止竟态。这样一来，当系统面临大量并发连接的时候，allocating fd 自然会较大的系统开销。
- **Inefficient per-packet processing** : 内核在处理小包的时候效率比较，主要瓶颈在于 per-packet memory (de)allocation and DMA overhead, NUMA-unaware memory access, and heavy data structures (e.g., sk_buffer)。
- **System call overhead** : 由于 BSD socket API 都是基于系统调用的，当系统中存在大量并发的 short-lived connections 的时候，系统中自然就会由频繁的系统调用。系统调用的代价是很高的，一方面是用户态和内核态之间的切换，另一个是 processor state (e.g., top-level caches, branch prediction table, etc.) pollution，这些都会导致严重的性能下降。
