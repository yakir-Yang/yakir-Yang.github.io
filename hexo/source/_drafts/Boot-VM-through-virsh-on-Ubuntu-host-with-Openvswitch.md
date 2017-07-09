---
title: Boot VM through virsh on Ubuntu host with Openvswitch
date: 2017-07-10 00:26:18
tags:
---

```
➜  kvm cat /etc/network/interfaces
# interfaces(5) file used by ifup(8) and ifdown(8)
auto lo
iface lo inet loopback

# The primary network interface
auto enp2s0
iface enp2s0 inet manual

auto br-int
allow-ovs br-int
iface br-int inet static
        OVSPort enp2s0
        address 192.168.0.103
        netmask 255.255.255.0
        broadcast 192.168.0.255
        gateway 192.168.0.1
        dns-nameservers 8.8.8.8 114.114.114.114

```

```
➜  kvm cat cloud-config 
#cloud-config

#cloud-config
password: kk
chpasswd: { expire: False }
ssh_pwauth: True
```

```
./create-config-drive.sh --ssh-key ~/.ssh/id_rsa.pub -u cloud-config config-drive.iso
```

```
➜  kvm cat boot-vm.sh 
#! /bin/bash

#       --disk path=/home/kk/kvm/config-drive.iso,device=cdrom \

virt-install --name ubuntu --ram 512 --vcpus 1 \
        --disk path=/home/kk/kvm/kk.iso,device=cdrom \
        --disk path=/home/kk/kvm/zesty-server-cloudimg-amd64.img,format=qcow2 \
        --network network:br-int \
        --accelerate
```

```
virsh destroy ubuntu
virsh undefine ubuntu
```
