---
title: Linux TC 常用命令
date: 2017-06-22 10:34:15
tags:
categories: 网络
---



**Bandwith Limit to 12Mbps**

```
# tc qdisc add dev ens4 handle 1: root htb default 11
# tc class add dev ens4 parent 1: classid 1:1 htb rate 1000Mbps
# tc class add dev ens4 parent 1:1 classid 1:11 htb rate 12Mbit
```



**Latency with 300ms and 3% Loss** (*Be careful about the limit argument*)

```
# tc qdisc add dev ens4 root handle 1:0 netem limit 102343600 delay 300ms 0ms loss 3%
```



**Bandwith Limit + Latency + Loss**

```
# tc qdisc add dev ens4 handle 1: root htb default 11
# tc class add dev ens4 parent 1: classid 1:1 htb rate 1000Mbps
# tc class add dev ens4 parent 1:1 classid 1:11 htb rate 12Mbit
# tc qdisc add dev ens4 parent 1:11 handle 10: netem limit 102343600 delay 300ms loss 3%
```



**Metrics**

```
# tc -s qdisc ls dev ens4
```

更多 TC 内部机制细节，请参考 http://perthcharles.github.io/2015/06/12/tc-tutorial/



