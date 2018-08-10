---
title: 'SDN, SD-WAN, NFV, VNF: What Is All This?'
date: 2017-06-10 10:06:03
tags:
categories: Network
---

Inscrutable alphabet soup. Even the fully expanded terms are only slightly more digestible. But they’re not truly all that confusing: the underlying principles are elegant and illuminating.

**Software Defined Networking (SDN)** is an architecture—first defined circa 1995—that physically separates the network control plane (decisions about traffic flows) from the forwarding plane (moving chunks of data between points A and B). Practically this simply means controlling network hardware devices (think routers, switches, Wi-Fi access points) by using software that is physically hosted elsewhere (think an orchestrator or controller platform).

A SDN architecture provides numerous advantages: it abstracts (and simplifies) the entire network into a “single” entity, programmable from a distant orchestrator or management point; it enables the replication of changes to policies and decisions in the blink of an eye to every outreach of the network, independent of the location, number, configuration or distribution of network devices; it provides resilience if a node—or building, or city, or region, or country!—should go offline. A backhoe through a fiber-optic cable, power failures, hurricanes, earthquakes, and other catastrophes, small and large. SDN enables traffic flows to be securely rerouted and rebuilt via unaffected network paths. Building mobile networks, where things move around constantly, requires this concept—the control of a cellular connection must be independent from the specific cell tower (read: forwarding plane) that the current session is connected to in order to manage the session hand-off to the next cell tower—same thing for communicating devices moving between Wi-Fi access points. 

**Software-Defined Wide Area Network (SD-WAN)** extends the SDN concept—first implemented in local area networks, typically data centers—and applies it to the WAN. SD-WAN achieves the same goals in the WAN that SDN achieved in data centers: **simplified** and **central management** and **visibility**; putting control into software that can be rapidly innovated and separating this from the forwarding hardware; making the installation, configuration, management and upgrades of large numbers of widely scattered branch offices simple, coherent and cost-effective; leveraging general-purpose commodity hardware to host “virtualized” functions.

SD-WAN applies a somewhat arms-length approach to SDN, primarily to allow smooth operation over the often considerable distances separating WAN devices. It is impractical to have a network device go down just because the central controller three thousand miles away is temporarily out of contact. SD-WAN mixes the best of both worlds: a central controller maintaining policies, configurations and priorities, combined with a level of local control by the network device. While the device acts on the most recent set of instructions from the controller, it also takes into consideration local conditions such as link performance and availability. This hybrid approach—distant control instructions colored with local judgment—allows the network device to be resilient, programmable, agile, as well as able to execute split-second decisions imperative to maintaining real-time traffic flows.

<!-- more -->

**Network Functions Virtualization (NFV)** and **Virtualized Network Function (VNF)** are not for the faint of heart, and the terms are frequently used—inaccurately—as interchangeable. The concepts are distinct, yet related: NFV is an ETSI-inspired architecture specifying how to run software-defined network functions independent of any specific hardware platform. A VNF is the implementation of a specific network function (think routing, firewalling, intrusion prevention) as a virtual service.

Listen to Steve Woo taking you through much more details about these concepts in the Demystified: [SD-WAN, SDN, NFV, and VNF][1] webinar. He illustrates how a Cloud-Delivered SD-WAN leverages these architectures with practical implementation examples that deliver tangible benefits—cost reduction, resilience, agility, cloud-readiness and cloud-migration—to your network. He covers many compelling topics including service insertion, service chaining, simplified routing and configuration, assured application performance, link remediation, transport independence and so much more.

Yes, architecture matters.

[1]: https://www.brighttalk.com/webcast/13111/209317


<font color="red"> **本文转载至：http://www.velocloud.com/sd-wan-blog/sdn-sd-wan-nfv-vnf/** </font>
<br>
