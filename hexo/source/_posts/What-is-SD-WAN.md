---
title: SD-WAN 到底是什么？
date: 2017-06-16 12:09:43
tags:
categories: 网络
---

In a netshull, SD-WAN

> - Virtualizes the network
> - Enables a secure overlay
> - Simplifies services delivery
> - Provides interoperability
> - Leverages cost effective hardware
> - Supports automation with business policy framework
> - Monitors usage and performance
> - Supports interoperable and open networking
> - Enables managed services



## Virtualizes the network

SD‐WAN as a network overlay enables application traffic to be carried independently of the underlying physical or transport layer, offering a transport‐independent overlay. Multiple links, even from different service providers, constitute a unified pool of resources, often referred to as a virtual WAN.
This capability enables SD‐WAN to provide high availability and performance for applications. It also increases the utilization of resources and simplifies the network.

> SD-WAN 能够对底层物理传输层进行抽象，对各种各样的物理链路进行整合统一，为应用程序提供高可用和高性能的虚拟 WAN 服务。

Network operators can add new links and applications easily because no static tie exists between the application and the link it must use – a key benefit of the abstraction principle. The virtualization also provides self‐healing as links experience degraded performance.

> SD-WAN 将应用程序和物理链路之间进行了解耦，这样使得网络运行商可以随意引进新的传输链路或者部署新的应用程序。


<!-- more -->


## Enabling a secure overlay

SD‐WAN provides a secure overlay that is independent of the underlying transport components. SD‐WAN devices are authenticated before they participate in the overlay.
Any combination of circuits and service providers can support secure, encrypted transmission, and the separated control plane enables automated configuration and key management across the multitude of branches. Additionally, a network designer can include segmentation as an overlay that is both independent and consistent across the various underlying components.

> 不管底层走的是何种传输链路，SD-WAN 都能支持对流量进行加密，为其提供安全传输。配置的过程也非常方便，只要通过 SD-WAN 的控制面，及可以远程自动在各子公司间开启流量的加密传输。



## Simplifying services delivery

SD‐WAN programmability does not just cover connectivity policy, it also extends to the insertion of network services, whether on the branch customer premise equipment (CPE), in the cloud or in regional and enterprise data centers.
The business‐level abstraction simplifies configurations to both route the traffic to the service delivery node and to configure the policy. Business‐level abstraction simplifies complex configurations of traffic routing and policy definitions.

> SD-WAN 也能实现在边缘设备（如 CPE，数据中心）上，轻易的添加新的网络服务（如 Firewalls, Policy Route 等）



## Providing interoperability

SD‐WAN provides the ability to incrementally add resources and interoperate with existing devices and circuits. This capability follows directly from the separation and abstraction of the control plane from the data plane.
SD‐WAN also satisfies a key design goal to enable multiple circuits, devices and services to coexist and interoperate. APIs enable integration into existing and different management and reporting systems deployed by enterprises.

> 得益于 SD-WAN 将控制面和数据面的分离，能使得上层控制策略能够和各种底层传输设备进行分工合作（不懂这里想表达什么......）。



## Leveraging cost‐effective hardware

SD‐WAN improves cost effectiveness and flexibility by leveraging commercially available hardware and network appliances or servers. The separation of the control plane from the data plane enables the use of standard hardware for the data plane.

> 在 SD-WAN 的网络架构中，将不再需要 Purposes-build 的网络设备，只需要采购标准的数据层硬件即可。

Virtual appliances can be remotely delivered and take advantage of existing or standard commercial off‐the shelf (COTS) servers. However, the initial installation and configuration of these servers typically requires on‐site IT installations. This form factor is likely well suited to larger branches as well as campuses and/or data centers. Virtual appliances are also deployable in hosted cloud environments.
Custom‐designed network appliances based on standard CPUs, memory and other components can still capture the cost benefits of commercially available silicon, yet provide the advantages of purpose‐built hardware. Custom‐designed appliances will come with just the right configuration out of the box, thus enabling deployment in sites without IT support, which can be a significant advantage for smaller and remote branches without on‐site IT resources.

> 网络服务设备也将以软件的形式进行发布和部署，不再需要让工程师进驻现场进行调试安装。



## Supporting automation with business policy framework

SD‐WAN enables the abstraction of configuration into business‐level policy definitions that span multiple data plane components and also remain stable over time, even as the network changes. The control plane provides the programming flexibility and centralization over a diverse and distributed data plane. Enterprises can expect application awareness and smart defaults to provide further abstraction from the detailed transport level details. Policy definitions can refer to users and groups, the applications they should use and what level of service they should receive.
Notably, this abstraction from the physical layer enables the self‐provisioning delivery model. Devices no longer require pre‐configuration on a per‐device basis; instead, they inherit the configurations and policies based on their assigned role in the network.

> 通过 SD-WAN 的智能网络策略，可以非常灵活便捷地对不同的用户组进行配置，让其享受不同的网络服务和企业应用。



## Monitoring usage and performance

SD‐WAN provides consolidated monitoring and visibility across the variety of physical transports and service providers, as well as across all remote sites. This monitoring capability offers business‐level visibility, such as application usage and network resource utilization. SD‐WAN adds detailed performance monitoring across all components of the data plane. Coupled with the business policies, performance monitoring enables intelligent steering of application traffic across different paths and resources within the virtual WAN network.

> SD-WAN 提供了统一的监听服务，即使流量跑在不同的物理传输层，或者各个网络供应商上。这样一来就可以结合业务策略，就能智能的选择网络路径，提高对网络资源的利用率。



## Supporting interoperable and open networking

SD‐WAN further improves agility, cost effectiveness and incremental migration via its approach of open networking, interoperability and evolving standards. Two organizations at the forefront of SDN and open networking are
- Open Networking Foundation (ONF): The OpenNetworking Foundation champions open, vendor‐neutralSDN architecture, interfaces, protocols and open‐sourcesoftware with the goal of accelerating SDN’s commercialadoption.
- Open Networking User Group (ONUG): The OpenNetworking User Group (ONUG) is a community of ITbusiness leaders who exchange ideas and best practicesfor implementing open networking and SDN designs.There is an ONUG Working Group for SD‐WAN.

> 两大一线社区的大力推进，明确了 SD-WAN 是顺应时代潮流之选。



## Enabling managed services

Many enterprises, even the largest, outsource the managementof their branch networks and WAN to either managed IT providers or to their network service providers. Additionally, some cloud application providers, such as Unified Communi-cations as a Service (UCaaS) providers, provision and managethe circuits needed for accessing their applications.
To address this business requirement, SD‐WAN should enablemanaged service providers (MSPs) to manage the WAN net-works of their clients with a multi‐tenant infrastructure. In addition to the management and orchestration functions, the data center networking components should also be designed for multi‐tenancy and scalable virtual deployment in providers’ cloud data centers.

> SD-WAN 也应该和 SDN 一样，提供多租户模式。


<font color="red"> **本文摘取至：Velocloud's [Software-Defined WAN for Dummies]** </font>
<br>
