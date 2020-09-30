---
title: JavaWeb笔记-Nginx D2
date: 2020-07-12 23:03:40
categories:
- [Java开发]
tags:
- [Nginx]
---

这节笔记包含如下内容：Nginx反向代理、负载均衡、配置session共享、动静分离以及其他简单配置
<!--more-->

## 反向代理

反向代理是指以代理服务器来接受浏览器的请求，然后将请求转发给内部网络上的服务器，并从服务器上得到返回结果返回给浏览器；Nginx的优势是在于它的异步阻塞模型，可以通过基于事件的方式同时处理和维护多个请求，而应用服务器只需要关心核心逻辑程序的执行，节约等待时间去处理更多请求，这样可以有效的提高网站性能；

Nginx反向代理的配置如下
```
location /some/path/ {
    proxy_pass http://www.example.com/;
}
```

## 负载均衡
负载均衡可以快速的提高服务的性能，它主要解决的是防止单节点压力过大造成的Web服务响应慢，或者服务直接崩溃的问题

### 负载均衡配置

在http代码块中添加`upstream`配置

```
#tomcat是一个标识符可以自定义命名
upstream tomcat {
    server 47.97.203.186:8080;
    server 47.98.203.186:8090;
}
```
接着修改`server`块中`local`配置
```
location / {
    proxy_pass http://tomcat/;     # tomcat是在上面命名的
}
```
这样我们就实现了一个最简单的负载均衡，直接重新加载配置文件就可以工作了；

实验效果时，你可以在一台电脑启动2个tomcat监听不同的端口，接着在Nginx配置`upstream`配置本机的对应端口，最后实验效果

### 负载均衡参数

**Weight**

该参数用来配置轮询权重，权重大的服务器访问的频率会更高，实际开发可以为配置高的服务器设置更高的权重
```
upstream tomcat {
    server 127.0.0.1:8050       weight=10 down;
    server 127.0.0.1:8060       weight=1;
    server 127.0.0.1:8090       weight=1 backup;
}
```

- down：表示当前的server暂时不参与负载 
- weight：weight越大，负载权重就越大，默认值为1
- backup： 其它所有的非backup服务器down或者忙的时候，才会请求backup服务器


**max_conns**

可以根据服务的性能来设置最大连接数，防止挂掉

```
upstream tomcat {
    server 127.0.0.1:8050    weight=5  max_conns=800;
    server 127.0.0.1:8060    weight=1;
}
```

**max_fails、 fail_timeout**

max_fails:失败多少次认为主机已挂掉则，踢掉，公司资源少的话一般设置2~3次，多的话设置1次

max_fails=3 fail_timeout=30s代表在30秒内请求某一应用失败3次，认为该应用宕机，之后后等待30秒，这期间内不会再把新请求发送到宕机应用，时间到后再有请求进来继续尝试连接宕机应用且仅尝试1次，如果还是失败，则继续等待30秒，以此循环，直到恢复；这感觉与curl库中设置超时时间是一样的效果；

```
upstream tomcat {
    server 127.0.0.1:8050    weight=1  max_fails=1 fail_timeout=20;
    server 127.0.0.1:8060    weight=1;
}
```

**负载均衡策略**

我们常用的负载均衡策略有6种，分别是`轮询`、`轮询+weight`、`ip_hash`、`url_hash`、`least_conn`、`least_time`

- 默认轮询方式

    未在`upstream`中设置策略时，默认是轮询方式，即每个请求按时序逐一分配到不同的服务器

- 轮询+weight

    即权重越大的服务器访问概率越大，用在各个服务器性能不一样的情况   
    ```
    upstream tomcat {
        server 192.168.0.14 weight=1;
        server 192.168.0.15 weight=2;
    }
    ```

- ip_hash

    ip_hash是指若客户已经访问了某个服务器，当用户再次访问时，会自动定位到该服务器

    使用ip_hash后是无法保证后端服务器的负载均衡，可能导致部分服务器接收的请求多，有的服务器收到的请求少，而且权重等方法将不起作用
    
    ip_hash可以避免些别的问题例如session问题，但是建议采用后端服务的session共享方式代替ip_hash方式
    ```
    upstream tomcat {
        ip_hash;
        server 192.168.0.14;
        server 192.168.0.15;
    }
    ```
- url_hash

    根据每次请求的url地址不同，将会访问到不同的服务器节点，hash后访问到的服务器节点将会固定。

    ```
    upstream tomcat {
        #使用ip_hash作负载均衡
        ip_hash;
        #获取到请求的ip地址
        hash $request_uri;
        server 192.168.0.14;
        server 192.168.0.15;
    }
    ```

- least_conn

    会找到连接数最少的服务器中，并且配合权重来处理，如果有多台候选服务器的话，会接着使用轮询的方式
    
    ```
    upstream tomcat {
        least_time; 
        server 192.168.0.14;
        server 192.168.0.15;
    }
    ```

- least_time

    找到平均响应时间最少并且连接数最少的机器，并结合权重，如何有多台候选服务器的话，会接着使用轮询的方式；配置方式与least_conn类似



### 健康检查模块
该模块是Tengine提供的主动式后端服务器健康检查功能，该模块在Tengine-1.4.0版本以前没有默认开启，它可以在配置编译选项的时候开启：./configure --with-http_upstream_check_module

我们需要配置location，专门设置一个url来显示健康检查模块，当你配置好并访问时会显示如下页面

<img src="/img/java/nginxD2.png" width="90%" style="text-align:center">

这个页面会展示所有配置的服务器，但不会显示设置了down关键字的服务器，如果某些服务器在运行过程中挂掉，对应的记录会显示红色
- server number是后端服务器的数量
- generation是Nginx reload的次数
- Index是服务器的索引
- Upstream是在配置中upstream的名称
- Name是服务器IP
- Status是服务器的状态
- Rise是服务器连续检查成功的次数
- Fall是连续检查失败的次数
- Check type是检查的方式，
- Check port是后端专门为健康检查设置的端口

下面是一个配置健康检查模块的Examples

```
http {
    upstream cluster1 {
        # simple round-robin
        server 192.168.0.1:80;
        server 192.168.0.2:80;

        check interval=3000 rise=2 fall=5 timeout=1000 type=http;
        check_http_send "HEAD / HTTP/1.0\r\n\r\n";
        check_http_expect_alive http_2xx http_3xx;
    }

    upstream cluster2 {
        # simple round-robin
        server 192.168.0.3:80;
        server 192.168.0.4:80;

        check interval=3000 rise=2 fall=5 timeout=1000 type=http;
        check_keepalive_requests 100;
        check_http_send "HEAD / HTTP/1.1\r\nConnection: keep-alive\r\n\r\n";
        check_http_expect_alive http_2xx http_3xx;
    }

    server {
        listen 80;

        location /1 {
            proxy_pass http://cluster1;
        }

        location /2 {
            proxy_pass http://cluster2;
        }

        location /status {
            check_status;

            access_log   off;
            allow SOME.IP.ADD.RESS;
            deny all;
        }
    }
}
```

首先需要配置location，为我们提供检查模块入口，这里使用allow/deny通过IP限制了该模块的访问，

接着是在upstream块中配置check指令打开后端服务器的健康检查功能，其参数意义是：
- interval：向后端发送的健康检查包的间隔。
- fall(fall_count): 如果连续失败次数达到fall_count，服务器就被认为是down。
- rise(rise_count): 如果连续成功次数达到rise_count，服务器就被认为是up。
- timeout: 后端健康请求的超时时间。
- default_down: 设定初始时服务器的状态，如果是true，就说明默认是down的，如果是false，就是up的。默认值是true，也就是一开始服务器认为是不可用，要等健康检查包达到一定成功次数以后才会被认为是健康的。
- typehttp：发送HTTP请求，通过后端的回复包的状态来判断后端是否存活

check_http_send指令：该指令可以配置http健康检查包发送的请求内容。为了减少传输数据量，推荐采用"HEAD"方法

check_http_expect_alive指令：该指令指定HTTP回复的成功状态，默认认为2XX和3XX的状态是健康的


## Session共享

当我们使用多台服务器做负载均衡时会出现session问题，用户在第一台服务器登录，再次访问假设被Nginx代理到了第二台服务器中，此时用户的session信息被保存到第一台服务器中，就会出现需要用户重新登录的问题，上面提到Nginx可以用ip_hash方式来避免这个问题，但通常我们会用Session共享的方案来解决问题

### Memcached
#### 安装

- 安装libevent
- 安装memcached

可以用yum方式安装 `yum –y install memcached`

#### 启动memcached

```
memcached -d -m 128 -u root -l 192.168.43.151 -p 11211 -c 256 -P /tmp/memcached.pid
memcached-tool 192.168.2.51:11211
参数解释：
-d:后台启动服务
-m:缓存大小
-p：端口
-l:IP
-P:服务器启动后的系统进程ID，存储的文件
-u:服务器启动是以哪个用户名作为管理用户
```

### Nginx配置
```
upstream tomcat{
    server 192.168.2.52:8080;
    server 192.168.2.53:8080;
}

location /tomcat {
    proxy_pass http://tomcat/;
} 

```

### Tomcat配置

到tomcat的lib下，jar包见附件

每个tomcat里面的context.xml中加入

```
<Manager className="de.javakaffee.web.msm.MemcachedBackupSessionManager" 
	memcachedNodes="n1:1192.168.2.52:11211" 
    sticky="false" 
    lockingMode="auto"
    sessionBackupAsync="false"
	requestUriIgnorePattern=".*\.(ico|png|gif|jpg|css|js)$"
    sessionBackupTimeout="1000" transcoderFactoryClass="de.javakaffee.web.msm.serializer.kryo.KryoTranscoderFactory" 
/>
```

在使用session共享时要注意每个服务器的本机时间，如果时间不一致会导致session离奇的失效

## Nginx动静分离

动静分离是指对于请求静态资源比如html、js、css、图片，该请求由Nginx处理，而不再交给应用服务器，这样会有效的提高应用服务器的负载，让应用服务器专心处理业务请求；比如在部署前后端分离工程时，由webpack打包出来的前端工程可以通过Nginx来配置访问，但需要解决好跨域问题

动静分离的配置如下，似乎此时这个配置还有很多细节问题未注意，后续遇到时会补充
```
location / {
    proxy_pass http://192.168.150.11:803;

}



location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|html|htm|css|js)$ {
    root  /var/data/;   
}

```

## Nginx其他配置

### 设置开机自动启动
```
chkconfig --list
chkconfig --add nginx
chkconfig nginx on
```

### 配置虚拟目录
```
location /www {
    alias  /var/data/www1;
    index  index.html index.htm a.html;
}
```

### 配置自动索引
```
location /dir {
    alias  /var/data/www1/;
    autoindex on;
}
```