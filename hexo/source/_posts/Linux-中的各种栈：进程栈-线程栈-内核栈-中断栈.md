---
title: Linux 中的各种栈：进程栈 线程栈 内核栈 中断栈
categories: 内核
date: 2016-08-26 23:24:35
tags:
---


## 栈是什么？栈有什么作用？
首先，栈 (stack) 是一种串列形式的 **数据结构**。这种数据结构的特点是 **后入先出** (LIFO, Last In First Out)，数据只能在串列的一端 (称为：栈顶 top) 进行 **推入** (push) 和 **弹出** (pop) 操作。根据栈的特点，很容易的想到可以利用数组，来实现这种数据结构。但是本文要讨论的并不是软件层面的栈，而是硬件层面的栈。

![栈结构](/images/kernel/stack/栈结构.png)

大多数的处理器架构，都有实现硬件栈。有专门的栈指针寄存器，以及特定的硬件指令来完成 入栈/出栈 的操作。例如在 ARM 架构上，R13 (SP) 指针是堆栈指针寄存器，而 PUSH 是用于压栈的汇编指令，POP 则是出栈的汇编指令。

> 【扩展阅读】：[ARM 寄存器简介](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.dui0204ic/Babefbce.html)
>
> ARM 处理器拥有 37 个寄存器。 这些寄存器按部分重叠组方式加以排列。 每个处理器模式都有一个不同的寄存器组。 编组的寄存器为处理处理器异常和特权操作提供了快速的上下文切换。
>
> 提供了下列寄存器：
> - 三十个 32 位通用寄存器：
>   - 存在十五个通用寄存器，它们分别是 r0-r12、sp、lr
>   - sp (r13) 是堆栈指针。C/C++ 编译器始终将 sp 用作堆栈指针
>   - lr (r14) 用于存储调用子例程时的返回地址。如果返回地址存储在堆栈上，则可将 lr 用作通用寄存器
> - 程序计数器 (pc)：指令寄存器
> - 应用程序状态寄存器 (APSR)：存放算术逻辑单元 (ALU) 状态标记的副本
> - 当前程序状态寄存器 (CPSR)：存放 APSR 标记，当前处理器模式，中断禁用标记等
> - 保存的程序状态寄存器 (SPSR)：当发生异常时，使用 SPSR 来存储 CPSR

上面是栈的原理和实现，下面我们来看看栈有什么作用。栈作用可以从两个方面体现：**函数调用** 和 **多任务支持** 。

#### 一、函数调用

我们知道一个函数调用有以下三个基本过程：
- 调用参数的传入
- 局部变量的空间管理
- 函数返回

函数的调用必须是高效的，而数据存放在 **CPU通用寄存器** 或者 **RAM 内存** 中无疑是最好的选择。以传递调用参数为例，我们可以选择使用 CPU通用寄存器 来存放参数。但是通用寄存器的数目都是有限的，当出现函数嵌套调用时，子函数再次使用原有的通用寄存器必然会导致冲突。因此如果想用它来传递参数，那在调用子函数前，就必须先 **保存原有寄存器的值**，然后当子函数退出的时候再 **恢复原有寄存器的值** 。

函数的调用参数数目一般都相对少，因此通用寄存器是可以满足一定需求的。但是局部变量的数目和占用空间都是比较大的，再依赖有限的通用寄存器未免强人所难，因此我们可以采用某些 RAM 内存区域来存储局部变量。但是存储在哪里合适？既不能让函数嵌套调用的时候有冲突，又要注重效率。

这种情况下，栈无疑提供很好的解决办法。一、对于通用寄存器传参的冲突，我们可以再调用子函数前，将通用寄存器临时压入栈中；在子函数调用完毕后，在将已保存的寄存器再弹出恢复回来。二、而局部变量的空间申请，也只需要向下移动下栈顶指针；将栈顶指针向回移动，即可就可完成局部变量的空间释放；三、对于函数的返回，也只需要在调用子函数前，将返回地址压入栈中，待子函数调用结束后，将函数返回地址弹出给 PC 指针，即完成了函数调用的返回；

于是上述函数调用的三个基本过程，就演变记录一个栈指针的过程。每次函数调用的时候，都配套一个栈指针。即使循环嵌套调用函数，只要对应函数栈指针是不同的，也不会出现冲突。

![函数栈结构](/images/kernel/stack/函数栈.png)


> 【扩展阅读】：[函数栈帧 (Stack Frame)](http://www.cnblogs.com/clover-toeic/p/3755401.html)
>
> 函数调用经常是嵌套的，在同一时刻，栈中会有多个函数的信息。每个未完成运行的函数占用一个独立的连续区域，称作栈帧(Stack Frame)。栈帧存放着函数参数，局部变量及恢复前一栈帧所需要的数据等，函数调用时入栈的顺序为：
>
> 实参N~1 → 主调函数返回地址 → 主调函数帧基指针EBP → 被调函数局部变量1~N
>
> 栈帧的边界由 **栈帧基地址指针 EBP** 和 **栈指针 ESP** 界定，EBP 指向当前栈帧底部(高地址)，在当前栈帧内位置固定；ESP指向当前栈帧顶部(低地址)，当程序执行时ESP会随着数据的入栈和出栈而移动。因此函数中对大部分数据的访问都基于EBP进行。函数调用栈的典型内存布局如下图所示：
>
> ![函数调用栈的典型内存布局](/images/kernel/stack/函数调用栈的典型内存布局.png)

<br>


#### 二、多任务支持

然而栈的意义还不只是函数调用，有了它的存在，才能构建出操作系统的多任务模式。我们以 main 函数调用为例，main 函数包含一个无限循环体，循环体中先调用 A 函数，再调用 B 函数。

```
func B():
  return;

func A():
  B();

func main():
  while (1)
    A();
```

试想在单处理器情况下，程序将永远停留在此 main 函数中。即使有另外一个任务在等待状态，程序是没法从此 main 函数里面跳转到另一个任务。因为如果是函数调用关系，本质上还是属于 main 函数的任务中，不能算多任务切换。**此刻的 main 函数任务本身其实和它的栈绑定在了一起，无论如何嵌套调用函数，栈指针都在本栈范围内移动。**

由此可以看出一个任务可以利用以下信息来表征：
1. main 函数体代码
2. main 函数栈指针
3. 当前 CPU 寄存器信息

假如我们可以保存以上信息，则完全可以强制让出 CPU 去处理其他任务。只要将来想继续执行此 main 任务的时候，把上面的信息恢复回去即可。有了这样的先决条件，多任务就有了存在的基础，也可以看出栈存在的另一个意义。**在多任务模式下，当调度程序认为有必要进行任务切换的话，只需保存任务的信息（即上面说的三个内容）。恢复另一个任务的状态，然后跳转到上次运行的位置，就可以恢复运行了。**

可见每个任务都有自己的栈空间，正是有了独立的栈空间，为了代码重用，不同的任务甚至可以混用任务的函数体本身，例如可以一个main函数有两个任务实例。至此之后的操作系统的框架也形成了，譬如任务在调用 sleep() 等待的时候，可以主动让出 CPU 给别的任务使用，或者分时操作系统任务在时间片用完是也会被迫的让出 CPU。不论是哪种方法，只要想办法切换任务的上下文空间，切换栈即可。

![多任务模型](/images/kernel/stack/多任务模型.png)

> 【扩展阅读】：任务、线程、进程 三者关系

> 任务是一个抽象的概念，即指软件完成的一个活动；而线程则是完成任务所需的动作；进程则指的是完成此动作所需资源的统称；关于三者的关系，有一个形象的比喻：
> - 任务 = 送货
> - 线程 = 开送货车
> - 系统调度 = 决定合适开哪部送货车
> - 进程 = 道路 + 加油站 + 送货车 + 修车厂

-------
## Linux 中有几种栈？各种栈的内存位置？

介绍完栈的工作原理和用途作用后，我们回归到 Linux 内核上来。内核将栈分成四种：

- 进程栈
- 线程栈
- 内核栈
- 中断栈

#### **一、进程栈**

进程栈是属于用户态栈，和进程 **虚拟地址空间 (Virtual Address Space)** 密切相关。那我们先了解下什么是虚拟地址空间：在 32 位机器下，虚拟地址空间大小为 4G。这些虚拟地址通过页表 (Page Table) 映射到物理内存，页表由操作系统维护，并被处理器的内存管理单元 (MMU) 硬件引用。**每个进程都拥有一套属于它自己的页表，因此对于每个进程而言都好像独享了整个虚拟地址空间。**

Linux 内核将这 4G 字节的空间分为两部分，将最高的 1G 字节（0xC0000000-0xFFFFFFFF）供内核使用，称为 **内核空间**。而将较低的3G字节（0x00000000-0xBFFFFFFF）供各个进程使用，称为 **用户空间**。因为每个进程可以通过系统调用进入内核，因此内核空间由系统内的所有进程共享。虽然说内核和用户态进程占用了这么大地址空间，但是并不意味它们使用了这么多物理内存，仅表示它可以支配这么大的地址空间。它们是根据需要，将物理内存映射到虚拟地址空间中使用。

![Linux虚拟地址空间](/images/kernel/stack/Linux虚拟地址空间.png)

Linux 对进程地址空间有个标准布局，地址空间中由各个不同的内存段组成 (Memory Segment)，主要的内存段如下：
- 程序段 (Text Segment)：可执行文件代码的内存映射
- 数据段 (Data Segment)：可执行文件的已初始化全局变量的内存映射
- BSS段 (BSS  Segment)：未初始化的全局变量或者静态变量（用零页初始化）
- 堆区 (Heap) : 存储动态内存分配，匿名的内存映射
- 栈区 (Stack) : 进程用户空间栈，由编译器自动分配释放，存放函数的参数值、局部变量的值等
- 映射段(Memory Mapping Segment)：任何内存映射文件

![Linux标准进程内存段布局](/images/kernel/stack/Linux标准进程内存段布局.png)

**而上面进程虚拟地址空间中的栈区，正指的是我们所说的进程栈**。进程栈的初始化大小是由编译器和链接器计算出来的，但是栈的实时大小并不是固定的，Linux 内核会根据入栈情况对栈区进行动态增长（其实也就是添加新的页表）。但是并不是说栈区可以无限增长，它也有最大限制　 RLIMIT_STACK (一般为 8M)，我们可以通过 `ulimit` 来查看或更改 RLIMIT_STACK 的值。

> 【扩展阅读】：如何确认进程栈的大小
>
> 我们要知道栈的大小，那必须得知道栈的起始地址和结束地址。**栈起始地址** 获取很简单，只需要嵌入汇编指令获取栈指针 esp 地址即可。**栈结束地址** 的获取有点麻烦，我们需要先利用递归函数把栈搞溢出了，然后再 GDB 中把栈溢出的时候把栈指针 esp 打印出来即可。代码如下：
>
>``` C
void *orig_stack_pointer;

void blow_stack() {
	blow_stack();
}

int main() {
	__asm__("movl %esp, orig_stack_pointer");

	blow_stack();
	return 0;
}
```
> ``` shell
$ g++ -g stacksize.c -o ./stacksize
$ gdb ./stacksize
(gdb) r
Starting program: /home/home/misc-code/setrlimit

Program received signal SIGSEGV, Segmentation fault.
blow_stack () at setrlimit.c:4
4		blow_stack();
(gdb) print (void *)$esp
$1 = (void *) 0xffffffffff7ff000
(gdb) print (void *)orig_stack_pointer
$2 = (void *) 0xffffc800
(gdb) print 0xffffc800-0xff7ff000
$3 = 8378368    // Current Process Stack Size is 8M
```


上面对进程的地址空间有个比较全局的介绍，那我们看下 Linux 内核中是怎么体现上面内存布局的。内核使用内存描述符结构体来表示进程的地址空间，改结构体中包含了进程地址空间有关的全部信息。内存描述符由 mm_struct 结构体表示，下面给出内存描述符的结构和各个域的描述，请大家结合前面的 进程内存段布局 图一起看：

```
struct mm_struct {
	struct vm_area_struct *mmap;           /* 内存区域链表 */
	struct rb_root mm_rb;                  /* VMA 形成的红黑树 */
	...
	struct list_head mmlist;               /* 所有 mm_struct 形成的链表 */
	...
	unsigned long total_vm;                /* 全部页面数目 */
	unsigned long locked_vm;               /* 上锁的页面数据 */
	unsigned long pinned_vm;               /* Refcount permanently increased */
	unsigned long shared_vm;               /* 共享页面数目 Shared pages (files) */
	unsigned long exec_vm;                 /* 可执行页面数目 VM_EXEC & ~VM_WRITE */
	unsigned long stack_vm;                /* 栈区页面数目 VM_GROWSUP/DOWN */
	unsigned long def_flags;
	unsigned long start_code, end_code, start_data, end_data;    /* 代码段、数据段 起始地址和结束地址 */
	unsigned long start_brk, brk, start_stack;                   /* 栈区 的起始地址，堆区 起始地址和结束地址 */
	unsigned long arg_start, arg_end, env_start, env_end;        /* 命令行参数 和 环境变量的 起始地址和结束地址 */
	...
	/* Architecture-specific MM context */
	mm_context_t context;                  /* 体系结构特殊数据 */

	/* Must use atomic bitops to access the bits */
	unsigned long flags;                   /* 状态标志位 */
	...
	/* Coredumping and NUMA and HugePage 相关结构体 */
};
```
![mm_struct 内存段](/images/kernel/stack/mm_struct内存段.png)

> 【扩展阅读】：进程栈的动态增长实现
>
> 进程中的每一个线程都有属于自己的栈，通过不断向栈中压入数据，超出其容量就会耗尽栈所对应的内存区域，这将触发一个 **缺页异常 (page fault)**，而被 Linux 的 `expand_stack()` 处理，它会调用 `acct_stack_growth()` 来检查是否还有合适的地方用于栈的增长。
>
> 如果栈的大小低于 RLIMIT_STACK（通常为8MB），那么一般情况下栈会被加长，程序继续执行，感觉不到发生了什么事情。这是一种将栈扩展到所需大小的常规机制。然而，如果达到了最大栈空间的大小，就会 **栈溢出（stack overflow）**，程序收到一个 **段错误（segmentation fault）**。
>
> 动态栈增长是唯一一种访问未映射内存区域而被允许的情形，其他任何对未映射内存区域的访问都会触发页错误，从而导致段错误。一些被映射的区域是只读的，因此企图写这些区域也会导致段错误。

<br>

#### **二、线程栈**


------
## Linux 为什么需要区分这些栈？


-------
## Reference
[栈的作用](http://www.cnblogs.com/scnutiger/articles/3770991.html)

[C语言函数调用栈(一)](http://www.cnblogs.com/clover-toeic/p/3755401.html)

[函数调用栈 剖析＋图解](http://blog.csdn.net/wangyezi19930928/article/details/16921927)

[进程地址空间分布](http://blog.csdn.net/wangxiaolong_china/article/details/6844325)

[Linux 进程栈和线程栈的区别](http://blog.csdn.net/daniel_ice/article/details/8146003)

[stackoverflow - How Process Size is determined?](http://stackoverflow.com/questions/12490929/how-process-size-is-determined)

[stackoverflow - Relation between stack limit and threads]( http://stackoverflow.com/questions/4369078/relation-betwee-stack-limit-and-threads)

[多线程与多任务的区别](http://www.ni.com/white-paper/8992/zhs/)

[对 Linux 的进程内核栈的认识](http://blog.chinaunix.net/uid-20543672-id-2996319.html)

[Linux中的栈：用户态栈/内核栈/中断栈](http://blog.chinaunix.net/uid-14528823-id-4136760.html)

[What Every Programmer Should Know About Memory? - Ulrich Drepper (Red Hat, Inc.)](http://people.freebsd.org/~lstewart/articles/cpumemory.pdf)
