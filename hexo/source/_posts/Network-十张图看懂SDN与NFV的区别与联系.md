---
title: 十张图看懂SDN与NFV的区别与联系
date: 2017-06-12 20:38:54
tags:
categories: Network
---

专业的人说的很准确但是普通人难以理解，常常记不住，分不清，不专业的人往往又说的差点意思。无意间，笔者在领英上看到一个介绍SDN/NFV区别的公开文档，内容详实，简明扼要。 这里我将这个文档精彩的部分分享给大家。

开篇鸣谢：原作者是Riverbed的产品市场经理JustynaBak。


### Page1：核心

![SDN-NFV-Friends-or-Enemies](/images/network/sdn-nfv/SDN-NFV-Friends-or-Enemies-2.jpeg)

> SDN的三个核心要点有三个：
> - 将控制平面和数据平面分离，这是最核心的部分，现在经常提到的SDS其核心也是控制和转发分离，这是SDS设计重要原则之一，可见SDN是先于SDS的；
> - SDN使用的都是商用化，通用的路由器和交换机，这是相对于专有的芯片，专有的架构，专有的设备而言的；
> - 控制面可编程；
> 
> 对应的NFV的三大关键点是：
> - 将网络设备的功能从网络硬件中解耦出来；
> - 将电信硬件设备从专用产品转为商业化产品；
> - 数据平面可编程；


<!-- more -->

### Page2：起源

![SDN-NFV-Friends-or-Enemies](/images/network/sdn-nfv/SDN-NFV-Friends-or-Enemies-3.jpeg)

> SDN起源于园区网，成熟于数据中心；
>
> NFV始于运营商，最初主要是大型运营商在搞的；


### Page3：代言人

![SDN-NFV-Friends-or-Enemies](/images/network/sdn-nfv/SDN-NFV-Friends-or-Enemies-4.jpeg)

> 加州大学伯利克分校教授，Nicira创始人，美国工程院院士，SDN运动的主要开创者之一，计算机网络世界最著名的人物之一。熟悉SDN的人都知道的这人，几年前来过一次中国。
>
> 爱立信执行副总裁，现任沃达丰CTO。（说实话，笔者对这人不熟）


### Page4：适用范围

![SDN-NFV-Friends-or-Enemies](/images/network/sdn-nfv/SDN-NFV-Friends-or-Enemies-5.jpeg)

> SDN跟NFV最明显的区别是，SDN处理的是OSI模型中的2-3层，NFV处理的是4-7。
>
> - SDN主要是优化网络基础设施架构，比如以太网交换机，路由器和无线网络等。
>
> - NFV主要是优化网络的功能，比如负载均衡，防火墙，WAN网优化控制器等。


### Page5： 类似的转变

![SDN-NFV-Friends-or-Enemies](/images/network/sdn-nfv/SDN-NFV-Friends-or-Enemies-6.jpeg)

> 这里做了一个特别有意思的类比，类似于C语言面向过程到C++的面向对象的变化。熟悉编程基础的朋友对此非常熟悉，以对象为思考的元素，更容易建立编程逻辑。
>
> - SDN的转变是从分布式的，采用复杂协议的专用网络设备，用低级的管理工具管理，转变为用高级管理工具管理商用设备组成的集中式架构系统。
>
> - NFV的转变是，从现场工程师配置专用设备，变为远程工程师配置虚拟化设备。


### Page6：带来的好处

![SDN-NFV-Friends-or-Enemies](/images/network/sdn-nfv/SDN-NFV-Friends-or-Enemies-7.jpeg)

> SDN带来的好处：
> - 简化由成千上万来自不同供应商，API接口的物理路由器交换机组成的整个网络的配置过程。
> - 从应用或者策略管理的来看，整个网络大大简化，从而简化了操作。
> - 减少成本，不用再为一些功能强大的贵的硬件花冤枉钱了。
>
> NFV带来的好处：
> - 加快产品和新业务推向市场的速度，因为无需改变硬件，要知道，硬件修改要费尽的多，开发测试周期太长。
> - 由于标准化的作用，带来采购，设计，集成和基础设施的维护的过程大大简化；
> - 由于有了动态分配硬件资源的能力，可以在确定的时间增加网络功能，从而增加了灵活性/扩展；


### Page7：标准的制定者

![SDN-NFV-Friends-or-Enemies](/images/network/sdn-nfv/SDN-NFV-Friends-or-Enemies-8.jpeg)

> SDN是ONF，NFV是ETSI；


### Page8：SDN为数据中心网络架构带来的变化

![SDN-NFV-Friends-or-Enemies](/images/network/sdn-nfv/SDN-NFV-Friends-or-Enemies-9.jpeg)


### Page9：NFV对运营商网络架构带来的变化

![SDN-NFV-Friends-or-Enemies](/images/network/sdn-nfv/SDN-NFV-Friends-or-Enemies-10.jpeg)


### Page10：NFV为终端用户网络架构带来的变化

![SDN-NFV-Friends-or-Enemies](/images/network/sdn-nfv/SDN-NFV-Friends-or-Enemies-11.jpeg)


### Page11：SDN和NFV以及相关技术组成在网络架构中所处的位置

![SDN-NFV-Friends-or-Enemies](/images/network/sdn-nfv/SDN-NFV-Friends-or-Enemies-12.jpeg)


<font color="red"> **本文转载至：http://www.doit.com.cn/p/255330.html** </font>
<br>
