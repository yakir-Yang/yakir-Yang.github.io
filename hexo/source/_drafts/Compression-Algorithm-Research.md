---
title: Compression Algorithm Research
date: 2017-06-21 23:40:06
tags:
categories: 工具
---

Snappy 是一个 C++ 的用来压缩和解压缩的开发包。其目标不是最大限度压缩或者兼容其他压缩格式，而是旨在提供高速压缩速度和合理的压缩率。

Snappy 比 zlib 文件相对要大 20% 到 100%，但是 Snappy 却能够在特定的压缩率下拥有惊人的压缩速度，压缩普通文本文件的速度是其他库的1.5-1.7倍，HTML能达到2-4倍，但是对于JPEG、PNG以及其他的已压缩的数据，压缩速度不会有明显改善


> - 在 64位模式的 Core i7 处理器上，可达每秒 250~500兆的压缩速度。
>
>
> - Snappy 在 Google 内部被广泛的使用，从 BigTable 到 MapReduce 以及内部的 RPC 系统
> - Snappy 从一开始就被设计**即便遇到损坏或者恶意的输入文件都不会崩溃**，而且被Google在生产环境中用于压缩PB级的数据。其健壮性和稳定程度可见一斑。



[Snappy by google](http://google.github.io/snappy/)



### Performance

Typical compression ratios (based on the benchmark suite) are about **1.5-1.7x** for **plain text**, about **2-4x** for **HTML**, and of course **1.0x** for **JPEGs**, **PNGs** and **other already-compressed data**. Similar numbers for zlib in its fastest mode are 2.6-2.8x, 3-7x and 1.0x, respectively. More sophisticated algorithms are capable of achieving yet higher compression rates, although usually at the expense of speed. Of course, compression ratio will vary significantly with the input.
