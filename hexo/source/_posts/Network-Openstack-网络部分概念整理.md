---
title: Openstack 网络部分概念整理
date: 2016-12-03 09:30:17
tags:
categories: Network
---
## Openstack 概念

我刚听说要去做 openstack 开发的时候，蛮激动的啊。虽然我不知道 openstack 是什么东西，但是我知道这个东西和云计算有关。云计算这东西，听着就高大上，各大互联网公司都有投人进去搞，所以大方向上是必须肯定的。于是我按捺不住了，想在自己的主机上部署 openstack，来体验下这个是什么东西。经过一顿折腾之后，终于在虚拟机里面用 devstack 把 openstack 给部署出来了（部署过程中，最坑的就是国内的防火墙）。

部署的过程，其实就是把一堆 python 包安装到主机系统的过程。openstack 有一个 python 包叫做 Dashboard，这个东西提供了一个 Web 界面，从那里我真正体验到了 openstack 是个啥。在 Dashboard 上创建一台虚拟机的过程蛮有趣的，这个过程涉及到的步骤，很多都和现实中安装电脑很像。

|Openstack         | Real Life|
|:                 |
|创建 ubuntu 镜像   | 使用 Ultraiso 刻录好一个 ubuntu 系统 的 U 盘|
|创建 provider 网络 | 叫电信的来给家里开通网络|
|创建 Instance 主机 | 买了一台主机，并用刚刚的 U 盘给它装好 Ubuntu 系统|
|启动 Instance 主机 | 把电信的网口插到主机网卡上|

我在 Dashboard 创建了两台 Instance 主机，两台主机跑的好好的。就目前来说，这个效果和 VMware Workstation 软件就很像了，可以在一台物理机上，跑多虚拟主机。只不过 VMware Workstation 软件面向的是单台物理机，而 openstack 却可以 **面对物理机集群** 进行管理和虚拟化。当成百上千的物理机被 openstack 管理着，而终端使用用户只需要面对唯一的一个 Dashboard 界面，这个时候私有云的概念就体现出来了。

有了感性的认识后，再去 [openstack](https://www.openstack.org/) 官网是什么定义自己的：
> **Open source software for creating private and public clouds.**
>
> OpenStack software controls large pools of **compute**, **storage**, and **networking resources** throughout a datacenter, managed through a dashboard or via the OpenStack API. OpenStack works with popular enterprise and open source technologies making it ideal for heterogeneous infrastructure.

>Hundreds of the world’s largest brands rely on OpenStack to run their businesses every day, reducing costs and helping them move faster. OpenStack has a strong ecosystem, and users seeking commercial support can choose from different OpenStack-powered products and services in the Marketplace.

>The software is built by a thriving community of developers, in collaboration with users, and is designed in the open at our Summits.

OpenStack 软件控制整个数据中心的大型计算，存储和网络资源，用户可以通过 Dashboard 或 OpenStack API 进行管理。就我理解，Openstack 更像是一个操作系统，一个强大的云计算操作系统。

<!-- more-->

--------

## Openstack 网络概念

我前面通过 Dashboard 创建的两个 Instance 虚拟机，它们之间是可以互相 ping 通的，因为它们两是处于同一个二层网络的，都是直接链接到 Provider 网络。

```
｜  VM1  |              |  VM2  |
｜-------|              |-------|
    |                      |
    | Eth(172.24.4.2)      | Eth(172.24.4.3)
____|______________________|___________
- - - - - - - - - - - - - - - - - - - - Provider (172.24.4.0/24)
```

上面的拓扑其实已经涵盖了 openstack 中比较核心的三个网络概念：**Network**，**Subnet**, **Port** 。我们把这三个概念和上面的网络拓扑映射一下：

- "Provider" 这个名字，对应的就是 openstack 的 **Network**
- "172.24.4.0/24" 这个 IP 地址池，对应的就是 openstack 的 **Subnet**
- "Eth(172.24.4.2)" 这个 Instance 网卡，对应的就是 openstack 的 **Port**

其实我们还可以让 VM1 和 VM2 不在一个二层网络上，只不过这个时候，就需要引入 openstack 网络的另一核心概念 **Router**。

```
Network Topology with Router:

                        |  VM2  |
                        |-------|
                            |
                            | Eth(10.0.0.2)
                     _______|_______
                     - - - - - - - - Private (10.0.0.0/24)
                            |
                            | If(10.0.0.1)
                            |
 |  VM1  |             |  Router1 |
 |-------|             |----------|
    |                       |
    | Eth(172.24.4.2)       | Gw(172.24.4.3)
____|_______________________|__________
- - - - - - - - - - - - - - - - - - - - Provider (172.24.4.0/24)
```

这样一来 VM1 和 VM2 就不在同一个二层网络了，两个 Instance 虚拟机的网络环境也就实现了隔离。从拓扑上可以看出，这里多出了三样新部件：

- **Router**: 也就是拓扑中的 Router1
- **Router-Gateway**: 也就是拓扑中的 Gw(172.24.4.3)，主要是为了 Router1 后面的虚拟机访问 Provider 网络。
- **Router-Interface**: 也就是拓扑中的 If(10.0.0.1)，让 Router1 作为 Private 网络的 L3 网关。

上面两个网络拓扑，就覆盖 openstack 网络的核心概念了，当然仅仅有以上四个核心概念，是没办法适应实际 Data Center 复杂的网络拓扑需求的，为此有了其他虚拟网络概念如：**FloatingIP**, **SecurityGroup**, **ServiceFunctionChain** 等。

--------

## Openstack 网络组件介绍

Openstack 包含了非常多的子项目，目前几个核心的子项目如下：

- Nova: 提供 compute 计算能力（虚拟机）
- Neutron: 提供 networking 网络连接能力
- Glance: 提供 image 镜像存储能力
- Keystone: 提供 identity 身份认证能力
- Cinder / Swift: 提供 storage 存储能力

网络虚拟化是块发展比较快的领域，到目前 Neutron 已经很好的支持 L2, L3, HA, SecurityGroup, LBaas, FWaas, VPNaas, DVR 等等。只不过我看过一些文章，里面说到 Neutron 的发展方向应该是，专注于 API 标准的定制。让各个网络功能从 Neutron 中剥离出去，让各个网络设备商的 SDN Controller Plugin 去聚焦和实现这部分功能。Neutron 需要设计出一组良好的北向接口规范，让自己成为一个纯粹的 API Server（这些观点的正确性，我也不确定，只不过换个高度看事情，未尝不是件好事）。

Neutron 里面有两个框架比较有趣， ML2 框架和 Extension Plugin框架 。ML2 是对 L2 网络功能的一次抽象，这让不同的 L2 虚拟化技术（OVS, LinuxBridge, OpenvSwitch...）只需统一对接到 ML2 即可。而 Extension Plugin 为一些在开发中的 SDNControler or NetworkFunction，能够方便的对接到 Neutron 中，进行部署测试，待起孵化成熟之后，再合并到 Neutron 项目中。我在主机上部署的 Openstack (Neutron + OVN)，其实就是充分利用了 Neutron ML2 和 ExtensionPlugin 才得以实现，从这点上也看出了 Neutron 的灵活性非常强大。

上面简单介绍了 Neutron，现在说说 OVN (Open Virtual Network for vSwitch)，它是 OpenvSwitch 团队自己孵化 OVS 的子项目，目的是为了让 OVS 更加友好的支持虚拟网络，容我引用 IBM 的一篇博文的精彩观点：

> Pick up from [如何借助 OVN 来提高 OVS 在云计算环境中的性能](http://www.ibm.com/developerworks/cn/cloud/library/1603-ovn-ovs-openvswitch/index.html)
>
>众所周知，OpenvSwitch 以其丰富的功能和不错的性能，已经成为 Openstack 部署中最受欢迎的虚拟交换机。由于 Openstack Neutron 的架构引入了一些性能问题，比如 neutron-server 要与非常多的 agent 通信，RPC 就是一个性能瓶颈，还有 neutron 里面用到非常多的 namespace，namespace 资源有限而且系统开销比较大，这也是一个性能瓶颈。OVS 社区觉得从长远来看，Neutron 应该让一个其它的项目来做虚拟网络的控制平面，Neutron 只需要提供 API 的处理，于是 OVS 社区推出了 OVN（Open Virtual Switch）这个项目，OVN 是 OVS 的控制平面，它给 OVS 增加了对虚拟网络的原生支持，大大提高了 OVS 在实际应用环境中的性能和规模。

如果想用 OVN 和 Neutron 进行集成使用，还需要 Networking-ovn Plugin 的帮助。Networking-ovn 是个比较简单的 Plugin，它的工作是将 Neutron 中对虚拟网络的定义，翻译到 OVN 对虚拟网络的定义中去。它的简单，来源于优美的 Neutron API 接口设计，和精简的 OVN 北向数据库表设计。

```
|      OPENSTACK      |
|                     |
|      (neutron)      |  API Server
-----------------------
         |  |
         v  v
---> networing-ovn <---  Service and Plugin
         |  |
         v  v
     |----------|
     |   OVN    |        SDN Controller for OpenvSwitch
     |----------|
          |
          v
----------------------
|    OpenvSwitch     |   DataPlan support for Virtual Networking Function
|                    |
```

<font color="red"> **转载请注明出处：** http://kyang.cc/ </font>
<br>
