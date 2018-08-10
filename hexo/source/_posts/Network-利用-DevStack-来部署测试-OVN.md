---
title: 利用 DevStack 来部署测试 OVN (Open Virtual Network)
date: 2016-11-06 11:30:28
tags:
categories: Network
---
1. 虚拟机环境搭建
--

目前我都是基于虚拟机来部署测试 Openstack，所以大家先去下载个 [vmware workstation pro 12](https://my.vmware.com/group/vmware/details?downloadGroup=WKST-1251-LX&productId=524&download=true&fileId=62f7511e6475e22f3bed9b3385fa3b25&secureParam=8718cec0b629523d64f88cb7fe26d62d&uuId=b719ce65-406c-473a-a83f-088cb33f5f9b&downloadType=)  以及 [ubuntu server](https://www.ubuntu.com/download/server/thank-you?country=CN&version=16.04.1&architecture=amd64) 镜像。

然后利用 vmware 新建一个 ubuntu server 的虚拟机实例，有一点要注意的是，我们需要给虚拟机分配两张网卡。一张显卡使用 NAT 模式，另一张使用 Host-only 模式。为什么需要新建两张显卡呢？我先给上面两张网卡命个名， NAT 模式的网卡叫 ens33，Host-only 模式的网卡叫 ens34。使用 DevStack 部署 Openstack 的时候，需要单独占用 ens33 这张网卡，并且部署过程中，系统有段时间无法通过 ens33 和外部通讯（也就是断网了）。但是我一般习惯在 Host 上通过 SSH 连接到虚拟机，由于 ens33 会存在掉线的情况，因此我需要一张额外的网卡保证 HOST 和 虚拟机 的网络连接保持通畅，这就是 ens34 网卡的由来。其实如果你放弃使用 SSH 的方式登陆虚拟机进行 Openstack 部署的话，你确实就只需要 ens33 一张网卡就好。

2. DevStack 源码下载
--

**以下所有命令，都是在虚拟机实例上运行的。**

首先去 github 上，下载一份 DevStack 源码，就放在 /home/ 目录下吧：
```sh
root # git clone https://github.com/openstack-dev/devstack /home/devstack
```

然后，使用 DevStack 提供的脚本，新建一个 stack 的系统用户
```sh
root # /home/devstack/tools/create-stack-user.sh
```
为什么要新建一个 stack 用户呢？因此用脚本新建的 stack 用户，是没有用户密码的。后续 DevStack 在部署 Openstack 会多次调用 `sudo` 命令，这个时候就不需要我们额外输入密码（实际上也不可能，鬼知道脚本什么时候要输入密码）。

设置 DevStack 源码的文件权限，保证 stack 用户能正常访问：
```sh
root # chown -R stack:stack /home/devstack/
```

<!-- more -->

3. DevStack 配置
--

DevStack 在部署 Openstack 的时候，回去解析 local.conf 这个配置文件。我们可以通过这个文件来决定，待会需要安装哪些组件(keystone, neutron, nova...)，以及 Openstack 的基本网络拓扑。官网的 [DevStack Doc](http://docs.openstack.org/developer/devstack/configuration.html) 已经对 local.conf 进行了全面的解析，从文章中可以知道，其实 Openstack 就 Neutron 网络组件配置非常复杂（不复杂应该就不能适用灵活多变的网络拓扑吧！）。

本文就不去解析 local.conf 的配置，Networking-OVN 插件已经提供了一个 local.conf.sample，我们直接拿来用就好！

只不过由于我们受到 "GreatFirewall" 无微不至的保护，导致我们无法稳定的访问相关网络资源，因此我们需要稍改一些配置。首先 local.conf 中，需要重新定向 openstack 的源码地址，将其指向到 github 上去。新添加的内容如下：
```
GIT_BASE=http://github.com
NOVNC_REPO=http://github.com/kanaka/noVNC.git
SPICE_REPO=https://github.com/SPICE/spice-html5.git
```

其次 openstack 是基于 python 写的，因此部署过程中，频繁的需要用 pypi 来安装相关包。由于国内访问 python.org 是很不稳定的，因此我们最好是去使用豆瓣的 pypi 源，改动如下：
```
stack $ mkdir ~/.pip
stack $ cat > ~/.pip/pip.conf << EOF
[global]
trusted-host=mirrors.aliyun.com
index-url=http://mirrors.aliyun.com/pypi/simple/
EOF

```

最后 DevStack 在部署的过程中，需要去下载很多文件到 /home/devstack/files 目录下。其中有两个文件，下载的时候经常容易失败。一个是 `get-pip.py` 这个文件，另一个是 `cirros-0.3.4-x86_64-disk.img` 这个文件。因此我们直接手动下载好，放到对应目录就好：
```sh
stack $ cd /home/devstack/files/
stack $ wget https://bootstrap.pypa.io/get-pip.py
stack $ wget http://images.trystack.cn/cirros/cirros-0.3.4-x86_64-disk.img
```

4. 部署 Openstack
--

到这里了，就只需要直接运行 DevStack 的 `stack.sh` 脚本就好了，万幸的话，你会直接成功，看到如下信息：
```sh
stack $ cd /home/devstack/
stack $ ./stack.sh
......
```

要是不幸的话，就只能自己多看 log，多查查了。只不过问题多半是 python 包安装超时，或者组件源码下载超时。手动把对应包安装下，然后重复运行 `./stack.sh` 就好了。

<font color="red"> **转载请注明出处：** http://kyang.cc/ </font>
<br>
