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

Linux 内核 TCP 协议栈的缺点：
- 
