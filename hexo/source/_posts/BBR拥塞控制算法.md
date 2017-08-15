---
title: BBR拥塞控制算法
date: 2017-07-02 16:31:09
tags:
categories: 网络
---


目前在移植 BBR 拥塞控制的算法，这部分文章就不直接转载。把开发过程中，看过的几篇好文章链接直接贴在下面。


- [TCP 的那些事儿（上）](http://coolshell.cn/articles/11564.html)
- [TCP 的那些事儿（下）](http://coolshell.cn/articles/11609.html)

### BBR Congestion Control
- [来自Google的TCP BBR拥塞控制算法解析](http://blog.csdn.net/dog250/article/details/52830576?locationNum=6&fps=1)
- [Google's BBR TCP拥塞控制算法的四个变速引擎](http://blog.csdn.net/dog250/article/details/52879298)
- [深夜聊聊Bufferbloat以及TCP BBR](http://blog.csdn.net/dog250/article/details/54999332)


### Congestion Avoidance Mechanism
- [TCP拥塞状态机的实现（下）](http://blog.csdn.net/zhangskd/article/details/8283689)


### Fast Retransmit
- [关于TCP快速重传的细节-重传优先级与重传触发条件](http://blog.csdn.net/dog250/article/details/52548345)
- [TCP拥塞控制图解(不包括RTO，因为它太简单了)](http://blog.csdn.net/dog250/article/details/51287078)


### ACK
- [(TCP的核心系列 — SACK和DSACK的实现（六）)](http://blog.csdn.net/zhangskd/article/details/9768519)
- [RACK为TCP BBR提供动力源](http://blog.csdn.net/dog250/article/details/52879457)
- [RACK: a time-based fast loss detection algorithm for TCP](https://tools.ietf.org/html/draft-tcpm-rack-00)


### RTO Timer
- [Computing TCP's Retransmission Timer](https://tools.ietf.org/html/rfc2988)
- [SCTP: Management of Retransmission Timer](https://tools.ietf.org/html/rfc4960#section-6.3)


> [Bomb250](http://my.csdn.net/dog250) 这个人写的 TCP 相关的东西，是写的真好。剖析的很深入，但居然很易懂 [赞]。
