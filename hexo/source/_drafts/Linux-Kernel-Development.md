---
title: Linux Kernel Development
tags:
categories: 内核
---

* Linux 的进程调度是基于 "处理器比重" 策略，具体实现通过 vruntime 成员。所有进程以各自的 vruntime 作为 Key，链接成为一个 RBTree。调度核心通过查找最小的 vruntime 对应的进程，作为下一刻需要运行的进程。**vruntime 作为进程调度的核心，那它的计算方法是什么？**

* 应用程序进行系统调用(system call)时，通过 **软中断(soft irq)** 陷入内核态，将 **系统调用号** 以及 **参数指针** 通过 CPU **通用寄存器** 传入(exp. eax, ebx...)，由内核代表应用程序在 *内核空间* 执行系统调用。
