---
title: JavaWeb笔记-LVS负载均衡 D3
date: 2020-07-22 14:56:19
categories:
- [Java开发]
tags:
- [LVS]
---

该笔记是基于keepalived的LVS集群搭建

<!--more-->

## 前置相关术语
防止有童鞋是直接看这节

- `DS` : Director Server 指负载均衡服务器
- `RS` : Real Server 指真实的业务服务器
- `CIP` : Client IP 指客户端IP
- `VIP` : Virtual IP 指用户请求的目标IP地址
- `DIP` : Director IP 指用于集群内通信的主机IP
- `RIP` : Real IP 指后端服务器的真实IP地址


## 需要解决的问题

上节搭建的LVS集群存在一些小问题，首先所有的客户端都会将请求发给DS服务器，DS服务器并未做特殊处理，若DS服务器出了单点故障那么整个系统都无法正常服务；另外就是若某台RS服务器出现故障，DS也是无法知道的它依旧会继续向这台出问题的服务器转发数据包，这会导致部分用户在使用时出现异常。

通常来说为了解决单点故障的问题，我们使用主备模型，即同一时间只有一台服务器在工作，若这台服务器出现故障，我们立即切换到备用服务器，若主服务器排查修复完毕，根据情况再切换回主服务器

主备模型需要准备一个master服务器，以及多台backup服务器，为了知道master服务器是否出现故障，通常是master服务器定期向backup服务器发送通知，若backup发现某段时间没有监听到master发送的通知，表明主服务器出现了故障，此时要快速求换到一台备用服务器来实现整个服务的高可用，使用哪台backup是根据事前配置的权重，我们会优先启用权重高的backup；通常在LVS集群中中，master服务器配置较高，而backup服务器配置稍微低一些，所以当主服务器出现故障又修复完成后，系统应该再切换回主服务器，并且因为LVS只负责转发，而它本身没有记录过多的信息，所以切换的成本不是特别巨大

而针对部分RS服务器故障的情况我们使用健康检查，即DS服务器实时的检测各个RS是否能正常工作，若发现某服务器无法正常工作，就不继续将数据包转发给它，直到这台服务器恢复为止

通常在HTTP服务器中做健康检查就是负载均衡服务器向RS定期发送一个HTTP请求，比如规定返回了200的状态码就认为RS是正常的，还记不记得之前的tengine内部也有健康检查模块，工作原理与这个是一样的

我们上述问题的解决方案已经有了一个开源的软件叫做keepalived，它主要实现的集群中主备模型，以及健康检查模块



## 基于keepalived的LVS集群

先来看一下整个集群的结构图，同样我还并没有实际操作过...

<img src="/img/java/LVS5.png" width="95%" style="text-align:center">

这里与上节笔记类型，所有操作都是在虚拟环境下完成的，内部有四个单独的Linux虚拟机，模拟实际的四台Linux服务器，其中Node01与Node04是DS，而01是主服务器，04是备用机，Node02与Node03是实际处理业务的服务器RS

### RS服务器搭建
修改配置的主机Node02、Node03

这里的RS节点与上一节一毛一样，这里冗余的在写一遍，但是不再做原理的解释

1、修改Linux内核，对外不暴露内部的配置的VIP
```
echo 1 > /proc/sys/net/ipv4/conf/eth0/arp_ignore 
echo 1 > /proc/sys/net/ipv4/conf/all/arp_ignore 
echo 2 > /proc/sys/net/ipv4/conf/eth0/arp_announce 
echo 2 > /proc/sys/net/ipv4/conf/all/arp_announce 
```
2、设置隐藏的vip：
```
ifconfig  lo:2  192.168.150.100  netmask 255.255.255.255
```

3、在RS中启动web服务

### DS服务器搭建
修改配置的主机Node01、Node04

这里与上一节不同，该笔记是使用keepalived搭建的LVS负载均衡服务器

> 如果之前在服务器上已经用ipvsadm做过配置，可以用`ipvsadm -C`命令清除所有的相关配置

1、安装keeplived、ipvsadm
```
yum install keepalived ipvsadm -y
```

2、修改keeplived配置文件
```
cd  /etc/keepalived/
vi keepalived.conf
```
`vrrp_instance`配置块用来做LVS本身相关的配置

主要是配置三个地方：

- 01的state配置改为MASTER、04的state配置改为BACKUP

- 01的priority权重配置改为100、04的priority权重配置改为50

- `virtual_ipaddress`：这个是在配置VIP，即客户访问的IP地址，示例的配置
大致意思是在eth0这块网卡的子网卡eth0:2中配置一个新的IP地址，地址是192.168.150.10，这个IP的子网掩码是255.255.255.0

在主服务器正常工作期间，virtual_ipaddress在backup服务器上是不生效的

```
vrrp_instance VI_1 {
    state MASTER         //  node04  BACKUP
    interface eth0
    virtual_router_id 51
    priority 100		 //	 node04	 50
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.150.10/24 dev eth0 label eth0:2
    }
}
```


`virtual_server`配置块是用来做负载均衡相关的配置

所有的主备服务器这里的配置应该是相同的，这里设置了负载均衡的规则以及所有真实服务器，并且规定了健康检查模块发送Http请求的请求url与返回值；
real_server
```
virtual_server 192.168.150.100 80 {
    delay_loop 6
    lb_algo rr
    lb_kind DR
    nat_mask 255.255.255.0
    persistence_timeout 50 
    protocol TCP

    real_server 192.168.150.2 80 {
        weight 1
        HTTP_GET {
            url {
                path /
                status_code 200
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }   
    }     

    real_server 192.168.150.3 80 {
        weight 1
        HTTP_GET {
            url {
                path /
                status_code 200
            }
            connect_timeout 3
            nb_get_retry 3
            delay_before_retry 3
        }
    }

```

3、启动服务
```
service keepalived start
```

这样就完成了LVS集群的搭建，不需要在单独配置