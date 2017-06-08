---
title: How to test SD-WANs
date: 2017-06-08 12:06:41
tags:
categories: 网络
---

When determining the best SD-WAN solution for your company, there are some basic things you should test, such as scalability, failover, application performance and usability

----

Whenever I speak with companies starting to research SD-WANs, the question about testing invariably comes up. Like probably any enterprise device, SD-WANs are filled with features. And as with any major WAN acquisition, testing those features prior to purchase is incredibly important. SD-WAN vendors have their own nuances and strengths. You need to be sure those strengths align with your environment.

As an edge device, there’s very little in terms of packet processing that needs to be tested in an SD-WAN node. But that doesn’t mean SD-WAN node testing isn’t important. Here are some tips for what you can look for when running your proof of concept (POC) from my buddy DC Palter, CEO at network testing simulator company Apposite Technologies, and our experiences here at SD-WAN Experts.

## Path selection

One of the primary benefits of SD-WAN is being able to split traffic between expensive,  dedicated links and lower-cost internet VPNs. Being able to differentiate mission-critical and latency- or jitter-sensitive traffic from less important or less time-sensitive traffic is key to success, but it isn't easy to accomplish.

Each vendor has proprietary algorithms to determine which traffic should go over which link. Test these algorithms during your POC to ensure they work as expected and manually adjust as needed.

## Scalability

If you have a larger network, scalability will be an important consideration. Will the network be hub and spoke or full mesh. Full-mesh networks will require a more robust SD-WAN solution provider than smaller networks.

<!-- more -->

## Failover

SD-WAN offers failover capabilities in the event of a link outage.

Detecting a link that's completely down is easy. But other than a wire cut or hardware failure, most network outages are highly dynamic, with frequent momentary pauses or a high level of packet drops. This presents a far more difficult challenge to decide when to failover traffic from one link to another and determine when it's safe to switch back, as with network brownouts.

Different products, of course, handle this differently, and some are far more sophisticated than others. If failover is an important use case, testing needs to recreate not only the hard outage case, but also momentary outages, periods of excessive congestion, and a range of packet loss rates. This is important in order to understand what to expect from your system and to optimize any tunable parameters to your specific application needs. 

## Application performance

Application acceleration technology is being integrated into some SD-WAN products, while traditional WAN optimization devices are adding SD-WAN or hybrid WAN features. Application acceleration benefits in particular are highly dependent on the interplay between application, acceleration technique, and network conditions, particularly bandwidth, latency, jitter, and loss.

If you expect the SD-WAN system to also provide application acceleration, the applications must be tested under real-world network conditions to understand the end-user experience and determine which SD-WAN product will work best for your network conditions.

## Usability testing

Usability testing is incredibly important with SD-WAN. It is critical to choose a product that works well for your particular needs and tune the parameters to optimize performance for your particular network. Simulate a multi-site deployment in your POC and then see what degree of management and control the SD-WAN gives you. Test the usability of tasks you will conduct during the course of the day, such as adding or removing users, changing application parameters, installing new SD-WAN locations, and reconfiguring nodes in existing locations.  

If you already have an MPLS network, you can tweak settings to get an objective sense of what happens if you reduce your reliance on MPLS, since you can effectively switch it on and off without a problem.

Testing can be done on the real network or using a WAN emulator to reproduce real-world conditions. Either way, your network, applications, and end-user requirements are different from anyone else's and are different from vendors' assumptions, so it is imperative to test prior to selecting a vendor and optimize prior to deployment.
