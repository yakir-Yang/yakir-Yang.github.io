---
title: 'Openstack: Neutron 深入学习'
date: 2016-10-13 22:46:23
tags:
categories: Network
toc: true
---
本文并未完全转载，具体内容依照学习进度而定，希望了解详细全文的童鞋，请移步至[原文](https://xiaofandh12.github.io/OpenStack-Nutron-Learning-Material)

学习什么
--

- neutron代码的整体架构，消息通知、rpc如何实现，RESTful API如何实现
- neutron的部署，常见问题的定位方法
- neutron的配置文件
- neutron的数据库设计，数据库中各表格的作用及其关联关系
- neutron-server的启动流程及其作用
- neutron-rpc-server的启动流程及其作用
- neutron-openvswitch-agent的启动流程及其作用
- neutron-dhcp-agent的启动流程及其作用
- neutron-l3-agent的启动流程及其作用
- neutron-linuxbridge-agent的启动流程及其作用
- openvswitch、openflow、linuxbridge、iptables，tap device, veth pair的原理及其作用
- plugin, driver, agent的关联关系，及作用
- flat, vlan, gre, vxlan的网络模式是如何实现的
- 如何与keystone交互进行身份认证，policy.json的原理和作用
- nova会调用哪些neutron的API，流程是怎样的
- neutron处理API请求的流程
- firewall as a service, load banalance as a service, vpn as a service， security group
- neutron的HA如何实现
- neutron各种部署方式下，两个虚拟机之间如何通信以及虚拟机如何与外网通信
- 关注邮件列表、IRC、OpenStack Summit，了解neutron最新动态
- SDN/NFV

<!-- more -->

源码
--

- [OpenStack Neutron](https://github.com/openstack/neutron)

- [OpenStack之Neutron入门一](http://www.aboutyun.com/forum.php?mod=viewthread&tid=9523&highlight=Neutron%2B%2B%C8%EB%C3%C5)
- [OpenStack之Neutron入门二](http://www.aboutyun.com/thread-9517-1-1.html)
- [OpenStack之Neutron入门三](http://www.aboutyun.com/thread-9537-1-1.html)
- [openstack源码分析 - Paste Deployment介绍](http://blog.chinaunix.net/uid-20940095-id-4105407.html)
- [openstack neutron分析（1）——neutron-server启动过程分析](http://www.aboutyun.com/thread-9527-1-1.html)
- [openstack Neutron分析（2）—— neutron-l3-agent](http://www.aboutyun.com/thread-9529-1-1.html)
openstack Neutron分析（3）—— neutron-dhcp-agent源码分析
openstack Neutron分析（4）—— neutron-l3-agent中的iptables
openstack Neutron分析（5）-- neutron openvswitch agent
openstack nova 源码分析1-setup脚本
openstack nova 源码分析2之nova-api,nova-compute
openstack nova 源码分析3-nova目录下的service.py、driver.py
openstack nova 源码分析4-1 -nova/virt/libvirt目录下的connection.py
openstack nova 源码分析4-2 -nova/virt/libvirt目录下的connection.py
OpenStack Nova源码结构解析
OpenStack源码探秘（一）——Nova-Scheduler
别以为真懂Openstack: 虚拟机创建的50个步骤和100个知识点(1)
别以为真懂Openstack: 虚拟机创建的50个步骤和100个知识点(2)
别以为真懂Openstack: 虚拟机创建的50个步骤和100个知识点(3)
别以为真懂Openstack: 虚拟机创建的50个步骤和100个知识点(4)
别以为真懂Openstack: 虚拟机创建的50个步骤和100个知识点(5)

文档
--

OpenStack Doc官网
---  

- OpenStack Installation Guide for Red Hat Enterprise Linux and CentOS
OpenStack Networking Guide
OpenStack Operations Guide
Architecture -> Chapter 7. Network Desing
Operations -> Chapter 12. Network Troubleshooting
OpenStack Administrator Guide
Networking
OpenStack High Availability Guide
OpenStack network nodes
OpenStack Security Guide
Networking
OpenStack Architecture Design Guide
Network focused
OpenStack API Complete Reference
Networking API v2.0 (CURRENT)
Networking API v2.0 extensions (CURRENT)
OpenStack End User Guide
OpenStack command-line clients -> Create and manage networks
OpenStack Python SDK -> Networking
OpenStack Command-Line Interface Reference
Networking service command-line client
Networking miscellaneous command-line client
Open source software for application development
OpenStack Developer Documentation
Networking service Developer Documentation(neutron)
BGP-MPLS VPN Networking service Plug-in(networking-bgpvpn)
Calico Networking service Plug-in(networking-calico)
L2GW Netwoking service Plug-in (networking-l2gw)
MidotNet Networking service Plug-in(networking-midonet)
OVN Networking service Plug-in(networking-ovn)
PowerVM Networking service Plug-in(networking-powervm)
Service Function Chaining Networking service Plug-in(networking-sfc)

华为
---

- 华为 - 企业用户
华为 - 交换机
华为 - 路由器
华为 - 云计算
华为 - 网管&SDN控制器
华为 - 软件定义网络（SDN）

RDO Doc
---

- trystack
Networking
OpenStack Networking in Too Much Detail
Using GRE Tenant Networks
Difference between Floating IP and private IP

Product Documentation for Red Hat OpenStack Platform
---

Mirantis Resources
---

- Mirantis OpenStack 7.0 NFVI Deployment Guide
Configuring Floating IP addresses for Networking in OpenStack Public and Private Clouds

Rakspace doc
---

Neutron Networking:详细 Neutron Routers and the L3 Agent

SDN/NFV
---

- Open vSwitch
Open vSwitch Documentation
OpenFlow
OpenFlow Documents
OPEN NETWORKING FOUNDATION
OPEN DAYLIGHT
opnfv
Neutron-Service Function Chaining
vmware NSX
sdnlab
Dragonflow
tacker

书籍
--

- 《TCP/IP详解 卷一：协议》、《TCP/IP详解 卷二：实现》 by W.Richard Stenvens
《UNIX网络编程 卷一：套接字联网API》、《UNIX网络编程 卷二：进程间通信》 by W.Richard Stevens
《UNIX环境高级编程》by W.Richard Stevens
《Computer Networks (Fifth Edition)》by Andrew S. Tanenbaum
《Computer Systems A Programmmer's Perspective (Secone Edition)》 by Randal E. Bryant
《The Hacker's Guide to Python》
The Hitchhiker's Guide to Python
《Learning OpenStack Networking (Neutron)》
《OpenStack设计与实现》
《OpenStack开源云王者归来》
《云计算与OpenStack 虚拟机Nova篇》

博客
--

- 有云博客
Mirants Blog
RDO Community Blogs
RACKSPACE DEVELOPER BLOG
OpenStack Hacker养成指南
孔令贤
Blogs - Red Hat Customer Portal
开发人员必读openstack网络基础1:什么是L2、L3
一系列文章Neutron 理解(1)：Neutron所实现的虚拟化网络(How Neutron Virtualizes Network)
openstack(juno) 入门(总结篇)二十七:openstack排除故障及常见问题记录
Neutron理解(9):How Nova Implements Security Group and How Neutron Implements Virtual Firewall
OpenStack Neutron FWaaS学习(by quqi99)
OpenStack中的防火墙(by quqi99)
初探Openstack Neutron DVR
openstack的虚拟机网卡、网桥等（tap、qbr、qvb、qvo）mtu设置
云计算：openstack neutron(tap、qvb、qvo、qbr详解)
neutron的基本原理
OpenvSwitch完全使用手册
专注云计算-学习OpenStack
技术点详解---IPSec VPN基本原理
计算机网络基础知识总结
NAT技术介绍(转)
IPsec技术介绍(转)
IPSEC与SSL/TLS的比较（转）
IPSec VPN与SSL VPN 比较（转）
项目经理 VS 产品经理 （工作职责和要求）
OpenStack开发过程中常用Git操作场景（转）
centos 7.0 网卡配置及重命名教程（转）
CentOS 7.0，启用iptables防火墙(转)
centos7 VMware workstation 10 添加多网卡及重命名为ethx（eth0，eth1失败）(还想再添加网卡eth1???)
vmware centos7 clone mac地址导致 Failed to start LSB: Bring up/down networking.
centos7将网卡加入到网桥后， Missing config file br-ex，网卡无法正常启动问题解决
OpenStack中的LoadBalancer(负载均衡)功能使用实例
OpenStack Neutron之Load Balance
负载均衡之Haproxy配置详解（及httpd配置）
系统原理分析架构-二-CDN内容分发网络
系统原理分析架构-一-DNS负载均衡
系统原理分析架构-六-负载均衡（定义及介绍及LVS/Nginx/Haproxy比较）
深入理解openstack网络架构（1）
深入理解openstack网络架构（2） ---- Basic Use Cases
深入理解openstack网络架构（3） ---- 路由
深入理解openstack网络架构（4） ---- 连接到publi network
网络虚拟化技术（一）：Linux网络虚拟化
网络虚拟化技术（二）：详细TUN/TAP MACVLAN MACVTAP
Open vSwitch工作原理
OpenStack neutron floating ips与iptables深入分析

Mailing List and IRC
--

- openstack-dev@lists.openstack.org
/#openstack-neutron, /#openstack-meeting-3

<font color="red"> **本文转载至：** https://xiaofandh12.github.io/OpenStack-Nutron-Learning-Material </font>
<br>
