---
title: CoreOS VS Docker容器大战之容器引擎
date: 2017-06-24 16:25:44
tags:
categories: 故事
---

在之前的文章中，我们从容器OS、容器引擎，基础架构的容器网络、存储、安全，容器运行必不可少的镜像仓库、编排，及运维关注的监控、日志，多维度关注并解读了容器生态圈中的各个玩家。总体而言，较为活跃的有Google主导的kubernetes生态，和Docker公司主导的docker生态。选择哪个技术流派从某种意义上，也决定了选择哪种生态，这也是当前使用容器的公司面临的一大难题。

本文将从容器引擎为切入点，说明这两大流派的历史、初衷和技术对比。



## 容器引擎玩家知多少

看到这个标题，你的表情可能是这样滴[惊愕]：纳尼？容器引擎不就是docker么？这时，[CoreOS](https://www.kubernetes.org.cn/tags/coreos)可能在后面默默地流泪，因为CoreOS的[rkt](https://www.kubernetes.org.cn/tags/rkt)也是玩家中的一员。虽然目前Docker的容器使用者相对更多，但rkt的发展也不可忽视。

![docker-and-rtk](images/storages/kubernetes/docker-and-rkt-logos.png)

## Rocket 与 Docker 引发的站队

### 前世

这里让我们先来八一个卦，故事要从2013年开始说起。是的，[Docker](https://www.kubernetes.org.cn/tags/docker)公司在2013年发布了后来红遍大江南北的docker产品，一场新的技术带来一次新的革命，也带来新的市场机遇，CoreOS也是其中的一员，在容器生态圈中贴有标签：专为容器设计的操作系统CoreOS。作为互补，CoreOS+Docker曾经也是容器部署的明星套餐。值得一提的是，CoreOS为Docker的推广和源码社区都做出了巨大的贡献。



然而好景不长，CoreOS认为Docker野心变大，与之前对Docker的期望是“一个简单的基础单元”不同，Docker也在通过开发或收购逐步完善容器云平台的各种组件，准备打造自己的生态圈，而这与CoreOS的布局有直接竞争关系。

<!-- more -->

2014年底，CoreOS的CEO Alex Polvi正式发布了CoreOS的开源容器引擎Rocket（简称rkt），作为一份正式的“分手”声明。当然，Docker的CEO Ben Golub也在官网作出了及时回应，总体意思表明没有违背初心，但这些都是应用户和贡献者的要求而扩展的，希望大家能一起继续并肩作战。

### 今生

当然，我们都知道了后来的事实，作为容器生态圈的一员，Google坚定的站在了CoreOS一边，并将kubernetes支持rkt作为一个重要里程碑，Docker发布的docker 1.12版本开始集成了集群swarm的功能。作为相亲相爱的一家人，Google于2015年4月领投CoreOS 1200万美元，而CoreOS也发布了Tectonic，成为首个支持企业版本kubernetes的公司。**从此，容器江湖分为两大阵营，Google派系和Docker派系。**

![img](/images/storages/kubernetes/google-rkt-docker-battel.png)

而CoreOS除了Rocket和CoreOS外，也提供类似dockerhub的Quay的公共镜像服务，也逐步推出容器网络方案flannel、镜像安全扫描Clair、容器分布式存储系统Torus（2017年2月在对应github项目上表示停止开发）等优质的开源产品，包括早期的etcd，各个产品都和kubernetes有了很好的融合。

### CNI & CNCF

在两大派系的强烈要求（对撕）下，2015年6月，Docker不得已带头成立了OCI组织，旨在“制定并维护容器镜像格式和容器运行时的正式规范（“OCI Specifications”），以达到让一个兼容性的容器可以在所有主要的具有兼容性的操作系统和平台之间进行移植，没有人为的技术屏障的目标 （artificial technical barriers）”。在2016年8月所罗门和Kubernetes 的Kelsey Hightower的Twitter大战中，所罗门透露出对容器标准化消极的态度。

有意思的是，同年（2015年）7月，Google联合Linux基金会成立了最近国内容器厂商陆续加入的CNCF组织，并将kubernetes作为首个编入CNCF管理体系的开源项目。旨在“构建 **云原生** 计算并促进其广泛使用，一种围绕着微服务、容器和应用动态调度的以基础设施架构为中心的方式”。陆续加入CNCF的项目有CoreDNS，Fluentd（[日志](https://www.kubernetes.org.cn/tags/%e6%97%a5%e5%bf%97)），gRPC, Linkerd（服务管理），openTracing，[Prometheus](https://www.kubernetes.org.cn/tags/prometheus)（[监控](https://www.kubernetes.org.cn/tags/%e7%9b%91%e6%8e%a7)）。



<font color="red"> **本文转载至：https://www.kubernetes.org.cn/2250.html** </font>