---
title: JavaWeb笔记-LVS负载均衡 D2
date: 2020-07-21 20:10:19
categories:
- [Java开发]
tags:
- [LVS]
---

这一节记录在Linux环境下搭建LVS负载均衡集群的过程，由于一些设备的原因目前笔记中的内容还没有实际操作过，但是会尽量补全教材中的环境

<!--more-->

## 前置相关术语
防止有童鞋是直接看这节

- `DS` : Director Server 指负载均衡服务器
- `RS` : Real Server 指真实的业务服务器
- `CIP` : Client IP 指客户端IP
- `VIP` : Virtual IP 指用户请求的目标IP地址
- `DIP` : Director IP 指用于集群内通信的主机IP
- `RIP` : Real IP 指后端服务器的真实IP地址

## 教材使用环境

由于整理这个笔记时我还没有完整搭建过LVS集群所以这里尽量的还原教材使用的环境

<img src="/img/java/LVS4.png" width="95%" style="text-align:center">

先介绍一下集群结构，首先教材中所有的服务器都是在一个虚拟机环境中模拟的也就是你看到的那一大坨乱七八糟线的那块，其实上面整张图都是在一台计算机中，实际每个Node都应该是一台Linux服务器

这里搭建的是DR模型，Node01是LVS负载均衡服务器，它在集群内的IP地址是192.168.150.1，这个IP对应上个笔记提到概念中的DIP，我们又在Node01节点的网卡上配置了一个新的IP地址是192.168.150.10，这个IP对应VIP，也就是用户直接访问的IP

Node 02、03、04都是RS，就是实际的服务器，它们各自在局域网内IP （192.168.150.2|3|4）对应是RIP，即LVS内部通信的IP，还记得之前介绍DR时，我们在RS中动了点手脚，在每台RS中配置了一个特殊的IP地址，就就是之前提到的VIP，并且这个地址要满足只有本机可见、其他机器不可见的条件，这个IP地址其实是配置在了每台机器的虚拟网卡中，就是你每次本地搭建web开发环境，访问的127.0.0.1这个地址对应的虚拟网卡，因为这个虚拟网卡可以满足外界不能访问前提，所以VIP会配置在这个网卡上

接下来我们只需要做到不让其他机器知道这个VIP地址就可以，这里是修改Linux内核改变计算机接受外部主机ARP询问结果实现的；首先每个主机内部都有一个ARP高速缓存，用来记录本局域网内所有主机&路由器的IP到它们物理地址的映射，之前提到数据包确定下一跳的IP后会再封装一层，在数据包中添加源/目标mac地址字段，目标mac地址就是下一跳IP对应的物理地址字段，这是通过查询ARP表得到的，那么主机是如何凭空构造出ARP表的？这是通过ARP协议实现的，简单来说，~~这里没法复杂说，内容太多了~~，就是计算机A需要与本局域网内B通信，但如果A只知道B的IP不知道B的mac物理地址，A会先在局域网内广播一个ARP请求包，里面记录如下信息
```
源IP：A@IP
目标IP:B@IP
源mac:A@mac
目标mac:FFFFFFFF
```
这个数据包就是用来询问B的mac地址的，所有计算机都能收到，只有当B收到时才会响应返回ARP响应包，即B告诉A自己的mac地址，同时B也会更新自己的ARP高速缓存表添加上A的IP到MAC的映射

回到刚刚的问题为不让其他机器知道RS服务器配置有VIP地址，我们需要修改RS服务器对于ARP请求与响应的策略。一台计算机可以有多个网卡，每个网卡也是可以配置多个IP地址的，但服务器是很实诚的，当它的某一个网卡收到某一个IP的ARP请求时，只要这个IP是自己这台服务器上任意一个网卡上配置的IP地址他都会作出响应，就像陌生人对你说出了你爸爸的手机号码，然后你回应了陌生人：嗯这是我爸爸的手机号，他的名字是布拉布拉布拉布拉...，Linux内核中有两个参数是用来分别控制ARP协议应答行为的，分别是`arp_ignore`与`arp_announce`

- arp_ignore：参数的作用是控制系统在收到外部的arp请求时，是否要返回arp响应
    - 0: 响应任意网卡上接收到的对本机IP地址的arp请求（包括虚拟网卡上的地址），而不管该目标IP是否在接收的那块网卡上。就是接收网卡会响应目标地址是任意本机上的网卡的arp请求
    - 1: 只响应目标IP地址为接收网卡上的所有IP地址的arp请求。即只响应找我的，本机其他网卡的不回应

- arp_announce：计算机在对外发送arp请求时，如何选择arp请求数据包的源IP地址
    - 0:允许使用任意网卡上的任意IP地址作为arp请求的源IP
    - 1:避免使用不属于该发送网卡子网的本地地址作为发送arp请求的源IP地址
    - 2:仅发送本网卡的IP地址作为数据包的源IP地址

回到上图的模型，Node 02中虚拟网卡配置了VIP，这个虚拟网卡是不会被外界访问到的，所以为了不让其他主机知道Node 02配置了VIP，只需要将Node 02中网卡eth01的`arp_ignore`设置为1，`arp_announce`设置为2，如果Node还有其他网卡依旧做相同配置

好了基本的环境介绍完

## 搭建LVS集群

### 搭建LVS负载均衡服务器

搭建是非常容易的因为之前说过LVS已经是Linux内核的一部分了，模块名叫做ipvs不需要单独安装模块，为了能够让系统调用需要暴露出调用模块，这个调用模块通常需要安装

```
yum install ipvsadm -y
```

`ipvsadm`命令如下：

**集群服务配置**

- 添加： `-A -t|u|f service-address [-s scheduler]`
    - -t:使用TCP协议集群
    - -u:使用UDP协议集群
    - -f:防火墙标记
    - service-address：VIP地址
    - -s:轮询策略
        - 静态调度: rr轮询、wrr权重+轮询、dh、sh
        - 动态调度: lc最少连接、wlc权重+最少连接、sed最短期望延迟 nq Never queue、LBLC基于本地最少连接、DH、LBLCR
- 修改： -E
- 删除： `-D -t|u|f service-address`

例如 `ipvsadm -A  -t 192.168.150.10:80  -s rr`

**集群RS服务配置**

- 添加： `-a -t|u|f service-address -r server-address [-g|i|m] [-w weighr]`
    - -t|u|f service-address:实现配置好的集群服务
    - -r server-address:某RS服务器IP地址，DIP
    - [-g|i|m] : LVS类型 `g - DR`、`i - TUN`、`m - NAT`
    - [-w weighr] : 服务器权重
- 修改：-e
- 删除：`-d -t|u|f service-address -r server-address`
- 查询：-L|l

例如 `ipvsadm -a  -t 192.168.150.10:80  -r  192.168.150.12 -g -w 1`

示例：将Node 01搭建为LVS

1、在eth0网卡添加一个IP配置，这个IP作为VIP即用户直接访问的IP地址使用
```
ifconfig  eth0:8 192.168.150.10/24
```

2、安装 ipvsadm
```
yum install ipvsadm 
```

3、配置集群服务，可以理解为做入方向的配置
```
ipvsadm -A  -t 192.168.150.10:80  -s rr
```

4、配置RS集群服务，可以理解为出方向的配置
```
ipvsadm -a  -t 192.168.150.10:80  -r  192.168.150.2 -g -w 1
ipvsadm -a  -t 192.168.150.10:80  -r  192.168.150.3 -g -w 1
ipvsadm -a  -t 192.168.150.10:80  -r  192.168.150.4 -g -w 1
```

5、查询
```
ipvsadm -ln
```


### 搭建RS服务器
示例:将node02~node04搭建为RS服务器

1、修改内核参数，Linux一切皆文件，`/proc`这个虚拟目录映射里系统进程，即修改该目录里的文件就是在修改系统进程参数，修改这个目录的文件是不能用vi，只能用echo做重定向
```
    echo 1 > /proc/sys/net/ipv4/conf/eth0/arp_ignore 
    echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore 
    echo 2 > /proc/sys/net/ipv4/conf/eth0/arp_announce 
    echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce 
```
2、在虚拟网卡中配置IP
```
ifconfig  lo:2  192.168.150.100  netmask 255.255.255.255
```

3、在node02~node04中启动一些web服务，比如启动Nginx

### 验证
浏览器访问 `192.168.150.10`多次刷新可以看到效果

