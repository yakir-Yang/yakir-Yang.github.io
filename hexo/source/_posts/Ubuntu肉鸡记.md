---
title: Ubuntu肉鸡记
date: 2016-08-13 15:17:46
categories: 网络
tags:
---

今天来和伙伴说说我这两天，亲历的一次网络安全问题。我之前对基础网络方面的知识确实没多少了解，并且正好在假期没什么事情，于是想着借着这次机会好好玩把。虽然已经抓到了木马程序，但是还没理解黑客是怎么黑进来的，估计主机里面还有黑客留的后门，但是中间的调试过程比较有趣。
<br>

### **- 发现问题**
最近由于家里的网络环境速度突然非常糟糕，路由经常掉线，直接导致家里没法上网。主要表现在路由能正常拿到联通的 IP 地址，但是没法正常上网。于是直接更新路由固件至官网回信更新路由固件解决问题。掉线问题解决后，立马用 BOX 测下网速如何，1.2M/s 不错，没什么问题了。

到晚上了，准备 SSH 连上家里的 Ubuntu 搞点东西。先是发现 SSH 建立连接的时间比以前久多了，并且 SSH 连接上后，时不时的 shell 会卡住。这就奇怪了，这局域网内直连，怎么这么不稳定。于是我就用 Ubuntu 去 PING 下百度，发现要 150ms 多，这就不对了，家里 10M 带宽，怎么要怎么久。于是我再用 Notebook 去 PING 了下百度，却只要 50ms 左右。明显 Ubuntu 出了问题，于是马上登到路由管理页面，检查下 Ubuntu 主机的带宽情况。这一看吓一跳，就发现主机有 16M/s 的上行流量！
<br>

### **- 认识问题**
家里的 Ubuntu 主机确实是安装了 Apache，但是并没有真正对外提供 WEB 服务，仅仅只是局域网内测试用。因此这 16M/s 的上行流量肯定是不正常的，SSH 卡顿应该就是因为主机的上行带宽都被占用了，没有多余的带宽给正常的服务。

<!-- more -->

但是当时对网络安全没什么概念，根本不知道这代表什么问题。关键字不对，Google 出来的结果也就都不太靠边。因此只能根据书上看到的东西，估计问题应该这样解决：先确认异常流量确实存在，找到主机异常流量的端口，然后找到使用此端口的进程，接着就卸载掉对应的木马应用。思路明确了，那就开始玩玩吧。
<br>

### **- 调试问题（流量确认）**
首先确认异常流量，Google 下首先发现个 ***iptables*** 端口流量统计，由于我仅仅只要确认 IP 流量，因此小改如下：
```shell
$sudo iptables -I OUTPUT -s 192.168.1.101
$sudo iptables -n -v -L -t filter
```
上面的命令结果，确实有一栏 **Chain OUTPUT** 的 bytes 确实非常大，但是不能直接看出带宽占用情况。本想试试 iptables 的其他参数，但是 `iptables -h` 一看，参数多的吓死人。命令明显很强大，但是也很难看懂怎么用。

因此就接着 Google 了， 接着又找到了 [***ifstat***][ifstat] 命令，看起来是直接输出网卡总的流量情况，并且命令也非常简单，非常合适。由于 Ubuntu 并没有内置这个命令，因此需要用 APT 安装下 `sudo apt-get install ifstat`。
```shell
$ sudo ifstat
       eth0               docker0      
 KB/s in   KB/s out   KB/s in  KB/s out
    0.07   11954.57      0.00      0.00
    0.41   11955.00      0.00      0.00
    0.13   11954.78      0.00      0.00
```
这就对了，直接看出来是 eth0 的 OUT 流量达到了 11 M/s 多，和路由那边检测到相差无几。
<br>

### **- 调试问题（端口确认）**
按照之前的思路，现在应该找异常流量的端口了，于是接着 Google 喽。搜索出来的结果，主要有 ***nethogs*** *&* ***iptraf*** *&* ***iftop*** 。**nethogs** 是可以用来实时检测所有进程占用的带宽情况，**iptraf** 比较强大，拥有一个图形操作界面，可以检测出所有 src/port -> dst:port 的带宽占用情况，甚至统计每个端口的流量情况。**iftop** 也比较强大，专注于实时检测 src:port -> dst:port 的带宽情况。下面逐个举例下，也算作简单的 Mark 备忘下。

[***nethogs***][nethogs] 命令 Ubuntu 是没有集成的，因此继续 APT 安装下 `sudo apt-get install nethogs`。这个命令使用比较简单，没多少可选参数，其实直接 `sudo nethogs` 运行就好。但是 **nethogs** 的输出里并没有看到有那个线程流量很大，基本都只有 1~2 KB/s 。但是伙伴们应该也发现了，貌似主机 192.168.1.101 有非常多的端口，再和 203.66.120.95 的 80 (http) 端口通信，可能有问题。
![这里写图片描述](http://img.blog.csdn.net/20151007001712670)
<br>

[***iptraf***][iptraf] 也需要先用 APT 安装下 `sudo apt-get install iptraf`，**iptraf** 完全是图形操作界面，因此运行命令 `sudo iptraf` 即可。进入菜单界面后，直接去探测所有 TCP/UDP 端口的流量情况，很明显发现 TCP 80 的流量高达 11M/s。这个结果和 nethogs 的有点出入，nethogs 发现的是 本机 很多端口被拿去和 203.66.120.95 的 80 端口通信，如果说 **iptraf** 表示的 TCP/80 指的是 Dest 的端口还差不多，有点奇怪这。
![这里写图片描述](http://img.blog.csdn.net/20151007003506931)
<br>

[***iftop***][iftop] 也需要先用 APT 安装下 `sudo apt-get install iftop` ，**iftop** 的输出界面和 nethogs 类似，但是在输出界面有一些快捷键，可以完成 Src_IP/Dst_IP 是否显示或者 DNS 解析、Src_Port/Dst_Port 是否显示或者解析。直接运行下命令 `sudo iftop`, 统计结果一看发现 主机 (192.168.1.101) 和 203.66.120.95 的上行流量直接高达 10M 多，并且把 Dst_Port / Src_Port 都打开后，结果和 **nethogs** 一致，主机无数多个端口被用于和 203.66.120.95 的 80 端口通信了。（NOTE: 由于这次截图和上次截图属于不同的攻击时间，所以目标的 IP 地址变了 203.66.120.95 -> 203.66.120.97）
![这里写图片描述](http://img.blog.csdn.net/20151007002604102)
<br>

结合上面的情况，应该确定这个巨大的上行流量就是用于和 203.66.120.95 的 80 (http) 端口交互了。虽然主机开了非常多的端口，但是我想其实这些端口应该都是一个木马程序开出来的，随便挑个端口检查对应的进程应用就好了。
<br>

### **- 调试问题（进程确认）**
接着就 Google 如何通过端口确认进程应用程序了，搜索出来的基本都是 ***netstat*** 。这个命令稍微有点复杂，但是值得研究（毕竟没找到别的工具），下面直接贴出些有用的可选参数
```shell
$ sudo netstat -h
  ......
  -n, --numeric            don't resolve names
  -p, --programs           display PID/Program name for sockets
  -a, --all, --listening   display all sockets (default: connected)
  <Socket>={-t|--tcp} {-u|--udp} {-w|--raw}
  ......


$ sudo netstat -nap -t -u
```
![这里写图片描述](http://img.blog.csdn.net/20151007005403742)

从上面的 TCP/UDP 端口信息可以看出，里面并没有找到 *nethogs* & *iftop* 出现的任何主机端口，甚至联 Dst_IP [*203.66.120.95*] 都完全没有出现，这不正常啊，按我的理解所有的 Socket 通信都一定会在 **netstat** 里面体现出来啊。

这个时候我就困惑了，按照我之前的思路调试不下去了。根据端口完全找不到对应的进程，也就找不到木马应用。于是就整理了下前面的现象，到群里去问问大神们，结果立马就有大神 @Flody 说出了 "肉鸡" 这个名词。第一次听说这个 "肉鸡"，很亲切啊，下面贴下 [WIKI][wiki_zombie] 的描述 :

>僵尸电脑（Zombie computer），简称“僵尸（zombie）”或称“肉鸡”，是指接入互联网受恶意软件感染后，受控于黑客的电脑。其可以随时按照黑客的指令展开拒绝服务（DoS）攻击或发送垃圾信息。通常，一部被侵占的电脑只是僵尸网络里面众多中的一个，会被用来去运行一连串的或远端控制的恶意程序。一般电脑的拥有者都没有察觉到自己的系统已经被“僵尸化”，就仿佛是没有自主意识的僵尸一般。

Ah，这基本就是在描述我的主机嘛，看来前面提到的 IP 203.66.120.95 应该就是黑客利用我主机去攻击的对象。哈哈，虽然现在没思路了，但是重要的关键字对了，接着就看 Google 搜索了。
<br>

#### **- 重新认识问题（肉鸡）**
这次搜索出来的文章，大部分和我主机表现一致，都是 SSH 连接服务器卡顿，并且都有巨大的上行流量。但是根据感染源头，我看到的主要分成三种： SSHD 被暴力破解登陆、系统正常程序被替换成木马程序、Apache 中存在 PHP 木马，下面贴些代表性的文章：

[《一次服务器被肉鸡的经历》][一次服务器被肉鸡的经历]，这篇文章主要说了通过 `/var/log/auth.log` 日志文件，分析出黑客通过 SSHD 暴力猜解 **弱密码** 和 **弱用户名**，从而完成服务器登录。
于是我也去分析了主机的 `auth.log`，但是没有发现任何异常 IP 登录，因此我这边不太像是此问题。

[《Linux服务器沦陷为肉鸡的全过程实录》][Linux服务器沦陷为肉鸡的全过程实录]，这篇文章主要从 `.bash_history` 的角度，研究黑客登录到机器后的所有 Shell 行为，从而直接找到木马文件。同时文章的感染原因和上面文章一样，SSHD 被暴力猜解了，删除木马文件后，作者也加强了 SSHD 的权限，关闭密码登录，仅仅允许公钥方式。
我也查看了下各个用户的 `.bash_history` 文件，但是文件中全部都是我的操作，没有任何黑客操作的痕迹。这让我心里一惊，很可能是入侵的黑客对 `.bash_history` 进行了修改，删掉了黑客的记录，此法还是行不通。

[《我的服务器攻击别人了！！ (已解决，把肉鸡抓住了！)》][我的服务器攻击别人了！！ (已解决，把肉鸡抓住了！)]，这篇文章比较贴近我的实际情况。文章中的服务器也是非常多端口被用去攻击别人，并且 `netstat` 里面没有留下 DDOS 之类攻击所产生的任何端口。但是幸运的是他发现停掉 `apache`  服务能够停止主机发送攻击，从而明确是 PHP DDOS 攻击木马，因此接着用 `apache server-status` 内置管理界面，监控所有 Apache 执行的脚本资源信息，从而找出木马真凶。
我也尝试直接 `sudo service apache2 stop` 停止 Apache 服务，并且 `sudo pkill httpd` 杀掉 httpd 进程，但是我发现主机还在发起 DDOS 攻击。并且我也开启了 Server Status 检测 `http://192.168.1.101/server-status`，并 Apache2 没有运行任何脚本，所以确定问题也不在这里。  

[《CentOS服务器上查找肉鸡》][CentOS服务器上查找肉鸡]，这篇文章的解决思路和我想的差不多，主要是通过异常流量端口，来定位木马应用。但是文章中是通过 `tcpdump` 命令对网卡进行抓包，然后用 `wireshark` 拆包找出协议类型及端口号，接着就用 `netstat` 幸运的找到木马应用。
虽然这次方法明显不适合我，但是我对抓包比较感兴趣，想看看我主机到底对外发送的是何种 DDOS 攻击。通过 `sudo tcpdump -i eth0 -s 0 -c 1000 -w rootkit.cap` 抓出数据包，然后 APT 安装 wireshark ，接着就发现主机一直在对远程 IP 疯狂的发送 TCP SYN 数据包。SYN 是 TCP 三次握手的首次数据包，也就是说攻击过程没有建立完整的 TCP 通道，这也是为什么 `net_stat` 里面找不到端口的原因。Google 一搜，发现这就非常流行的 [SYN Flood][syn_flood_wiki] 。顺便也看了下常用的 DDOS 攻击方式，大家有兴趣可以了解下 [Analyzing attacks]、 [CentOS下DDoS攻击防御和分析][CentOS下DDoS攻击防御和分析]

上面几种常见的肉鸡问题，都不适合我的情况，非常悲伤啊。实在是没找了，又去找群里的大神们聊聊天，讨教下接下来怎么搞。啊哈，还真有大神 @carlos 支招，可以尝试挨个挨个应用杀过去，总可以找到是那个混蛋。但是一看 `ps -aux` 出来的进程，尽管大部门都是内核进程，但是应用进程还是有点多。而且很多杀了，又会被复活，感觉行不通。

但是这个建议让我想到了一个点子，主机这么多端口被拿去发起 DDOS 攻击，那么对应的进程肯定是占用了非常多的系统资源，抛开文件句柄不说，CPU 的占用率肯定也非常高，这样以来我直接就可以用 `sudo top` 命令找出那个进程啊。
<br>

### **- 重新调试问题（ TOP 命令）**
`sudo top` 一看，哎呦，一下就看到了，有个 JB 进程，名字一顿乱码 gqjlyovkan，占用资源排名第一，肯定就是这个王八蛋了。急躁的直接先 `sudo kill -7 pid` 下，哈哈，攻击立马立马就没了。刚刚开心一下，NND，又有一个鬼进程出来了，名字也是乱七八糟。

这样看来明显应该有个木马程序，专门 fork 出用于 DDOS 攻击的进程。因此这次我没有直接 kill 掉它，我想看下这 DDOS 进程的父进程是谁。`ps -ef` 找下发现，这 DDOS 进程完全就是个僵尸进程，PPID = 1 。这下可不好办了，杀了又被复活，然后又找不到源头。

正好这时请教了下同事关于进程环境列表问题，发现 FORK 出来的僵尸进程一定会继承父进程的 cmdline 参数，也就是说根本不用找到父进程，我就可以看出木马应用是谁。好兴奋，好兴奋。于是里面执行 :
```shell
$ top
 7580 root      20   0 66752  804  208 S   22  0.0  17:45.56 gqjlyovkan

$ cat /proc/7580/cmdline
who

$ whereis who
who: /usr/bin/who  /usr/share/man/man1/who.1.gz
```
个混蛋，居然把我的系统应用都替换了，贱人！重新安装了这个应用了，接着中午用脚本连续监测了半个小时，NND 终于没有异常流量了。但是由于黑客是替换掉的我系统工具的，所以有足够理由怀疑黑客肯定还在我的主机里面放了后门，说不准以后又会出幺蛾子。由于目前检测这么久，也没有异常流量了，并且后门大多数是 SSHD 的问题，因此首先强化了下 SSH 验证方式，关闭了密码登录，并且重置了所有用户的密码。所以这个问题就缓一下，以后有问题了在抄家伙。
<br>

### **- 结束语**
随便贴出下我的检测脚本，主要是发现异常流量的时候，会分别抓取 网络端口信息 `netstat` ＆ `iftop` ，系统进程信息 `ps` & `lsof`，网卡数据包 `tcpdump`
```shell
$ tree rootkit/

rootkit
├── logs
│   ├── iftop_log
│   │   └── iftop_log.2015-10-07--22:38:57
│   ├── iftop_log.LATEST -> /home/home/proj/rootkit/logs/iftop_log/iftop_log.2015-10-07--22:38:57
│   ├── lsof_log
│   │   └── lsof_log.2015-10-07--22:38:57
│   ├── lsof_log.LATEST -> /home/home/proj/rootkit/logs/lsof_log/lsof_log.2015-10-07--22:38:57
│   ├── netstat_log
│   │   └── netstat_log.2015-10-07--22:38:57
│   ├── netstat_log.LATEST -> /home/home/proj/rootkit/logs/netstat_log/netstat_log.2015-10-07--22:38:57
│   ├── ps_log
│   │   └── ps_log.2015-10-07--22:38:57
│   ├── ps_log.LATEST -> /home/home/proj/rootkit/logs/ps_log/ps_log.2015-10-07--22:38:57
│   ├── tcpdump1_log
│   │   └── tcpdump1_log.2015-10-07--22:38:57
│   ├── tcpdump1_log.LATEST -> /home/home/proj/rootkit/logs/tcpdump1_log/tcpdump1_log.2015-10-07--22:38:57
│   ├── tcpdump2_log
│   │   └── tcpdump2_log.2015-10-07--22:38:57
│   └── tcpdump2_log.LATEST -> /home/home/proj/rootkit/logs/tcpdump2_log/tcpdump2_log.2015-10-07--22:38:57
├── rootkit.log
└── rootkit.sh
```


```shell
[~/proj/rootkit]$ cat rootkit.sh
#! /bin/bash

SPEEDY_LIMIT=1024
LOOP_TIME_SEC=3

record_logs()
{
        # usage: record_logs [log file name] [log command]
        LOG_NAME=$1
        COMMAND=$2
        TIME=$3
        HACK=$4

        PWD=`pwd`
        LOG_FILE=`echo "$PWD/logs/$LOG_NAME/$LOG_NAME.$TIME"`
        LOG_LATEST_FILE=`echo "$PWD/logs/$LOG_NAME.LATEST"`

        # creat the log dir
        if [ ! -d "logs/$LOG_NAME/" ]; then
                mkdir -p "logs/$LOG_NAME/"
        fi

        # run comamnd
        if [ -z $HACK ]; then
                $COMMAND > $LOG_FILE
        else
                $COMMAND $HACK $LOG_FILE
        fi

        # re-link the latest log file
        if [ -f "$LOG_LATEST_FILE" ]; then
                rm $LOG_LATEST_FILE
        fi
        ln -s $LOG_FILE $LOG_LATEST_FILE
}

loop_time=20
while (($loop_time > 0)); do
        TIME=`date +%F--%H:%M:%S`

        # record ifstat results
        IN_OUT=`sudo ifstat -T $LOOP_TIME_SEC 1 | sed -n '3p'`
        IN=`echo $IN_OUT | awk {'print $1'}`
        OUT=`echo $IN_OUT | awk {'print $2'}`

        # print speed to log file
        echo "[DEBUG]: $TIME ||  KB/s in: $IN   KB/s out: $OUT" | tee -a rootkit.log

        # judge whether speedy is normal, if normal just continue.
        IS_SPEEDY_UNORMAL=`echo "$OUT > $SPEEDY_LIMIT" | bc`
        if [ $IS_SPEEDY_UNORMAL -eq 0 ]; then
                continue
        fi

        echo "[ERROR]: $TIME || Detect unormal speedy, start to record more info" | tee -a rootkit.log

        record_logs "netstat_log" "sudo netstat -lnp -t -u" $TIME

        # note the iftop version should be or higher then version 1.0pre4, then we can use -t to use text interface without ncurses.
        record_logs "iftop_log" "sudo iftop -n  -i eth0 -P -B -t -s 5" $TIME

        record_logs "ps_log" "sudo ps -ef" $TIME

        record_logs "lsof_log" "sudo lsof" $TIME

        record_logs "tcpdump1_log" "sudo tcpdump -i eth0 -s 0 -c 2000" $TIME

        record_logs "tcpdump2_log" "sudo tcpdump -i eth0 -s 0 -c 10000" $TIME "-w"

        loop_time=$(($loop_time - 1))
        echo "[DEBUG]: $TIME || loop_time = $loop_time" | tee -a rootkit.log
done        

sync
```
<br>
这次就到此结束了，这次博文太长了，写了我好几天。只不过实话说这次 “肉鸡” 经验挺好的，让我很好的调试了一把网络，了解了很多基础的网络知识，也算比较难得。就先这样了，有机会再接着写点东西，再会。
<br>

[iptables]: http://govfate.iteye.com/blog/2086475
[nethogs]: http://os.51cto.com/art/201108/288014.htm
[iptraf]: http://www.cnblogs.com/chuyuhuashi/archive/2011/11/28/2266478.html
[ifstat]: http://www.cnblogs.com/ggjucheng/archive/2013/01/13/2858923.html
[iftop]: http://www.cnblogs.com/ggjucheng/archive/2013/01/13/2858923.html
[wiki_zombie]: https://zh.wikipedia.org/wiki/%E6%AE%AD%E5%B1%8D%E9%9B%BB%E8%85%A6
[一次服务器被肉鸡的经历]: http://blog.low-it.com/2015/08/an-experience-about-server-compromised/
[Linux服务器沦陷为肉鸡的全过程实录]: http://blog.csdn.net/smstong/article/details/44411993
[CentOS服务器上查找肉鸡]: http://www.centoscn.com/CentOS/Intermediate/2014/0302/2483.html
[我的服务器攻击别人了！！ (已解决，把肉鸡抓住了！)]: http://www.oschina.net/question/17_12655
[CentOS下DDoS攻击防御和分析]:http://www.centoscn.com/CentosSecurity/CentosSafe/2014/0414/2793.html
[Analyzing attacks]: http://help.fortinet.com/fddos/4-1-5/index.html#page/FortiDDoS_Handbook/analyzing_attacks.html
[syn_flood_wiki]: https://zh.wikipedia.org/zh/SYN_flood

<font color="red"> **转载请注明出处：** http://kyang.cc/ </font>
<br>
