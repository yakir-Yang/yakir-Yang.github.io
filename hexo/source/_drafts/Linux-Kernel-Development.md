---
title: Linux Kernel Development
tags:
categories: 内核
---

* Linux 的进程调度是基于 "处理器比重" 策略，具体实现通过 vruntime 成员。所有进程以各自的 vruntime 作为 Key，链接成为一个 RBTree。调度核心通过查找最小的 vruntime 对应的进程，作为下一刻需要运行的进程。**vruntime 作为进程调度的核心，那它的计算方法是什么？**

* 应用程序进行系统调用(system call)时，通过 **软中断(soft irq)** 陷入内核态，将 **系统调用号** 以及 **参数指针** 通过 CPU **通用寄存器** 传入(exp. eax, ebx...)，由内核代表应用程序在 *内核空间* 执行系统调用。

* Linux 内核常用的 **内建数据结构** ： 链表 list， 队列 kfifo， 映射 idr， 二叉树 rbtree

* Linux 中断实现：中断发生时，通过 **中断向量表** 跳转到预定义入口点。初始入口点将 **中断号保存在栈中**，然后跳转调用 ```unsigned int do_IRQ(struct pt_regs regs)``` 进行处理。因为 C 的调用惯例是要把函数参数放在 **栈的顶部**，因此可以从 **pt_regs 结构中提取出中断号** 进行进一步处理。
