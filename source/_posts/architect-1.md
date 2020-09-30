---
title: JavaWeb笔记-Nginx D1
date: 2020-07-09 17:10:45
categories:
- [Java开发]
tags:
- [Nginx]
---

好的，我们来搞web服务器，这一节主要包括一下内容：Nginx（Tengine）安装&脚本化操作、Nginx使用简介、Nginx配置文件分析

<!--more-->

前三天的博客整理的有些水，主要因为东西是很久以前学的，而且发现自己在CSDN上有整理笔记，就直接CV大法照搬了，但是以前记的东西真的烂，过一段时间拿出来就感觉已经读不懂当初在写些什么了😂

好了开始整理今天的笔记，这里的博客后来我调整过小节的顺序并做过内容修改，因为我觉得现在的顺序更加便于理解

## Nginx（Tengine）安装&脚本化操作

Nginx是一款轻量级的Web服务器、反向代理服务器、电子邮件代理服务器，由俄罗斯的程序设计师Igor Sysoev所开发；去年年底有关于Nginx之父Sysoev被捕的新闻，大致内容是Sysoev的老东家Rambler公司认为Nginx是Sysoev在公司任职期间开发的，所有权也应该是公司的，那么问题来了，程序猿在业余时间开发的东西也属于公司吗... 

这里是使用的是Tengine，Tengine是由淘宝发起的Web服务器项目，它在Nginx的基础上，针对大访问量网站的需求，添加了很多高级功能和特性，完全兼容Nginx的配置，继承了Nginx的特性；所这里算是利用Tengine学习Nginx，Tengine提供的许多功能非常便于开发与调试，后续为了方便理解除了使用Tengine的特性以外的地方我都会把Tengine称为Nginx。其实有点怪哈，就像是你把TypeScript叫做JavaScript一样😂

### 环境&依赖

操作系统： centOS

依赖组件： gcc openssl-devel pcre-devel zlib-devel

这里要检查一下你的服务器或者虚拟机有没有上述的组件，直接在Terminal里敲一敲他们的名字就知道了，如果依赖不全的情况下尝试安装，在执行configure时会报错

如果你的服务器是RedHat的Linux，可以使用以下命令安装依赖
```
yum install gcc openssl-devel pcre-devel zlib-devel 
```

### 下载&安装

**下载&解压Tengine**

http://tengine.taobao.org/

随意找一个服务器目录解压Tengine源代码，解压后进入源代码根目录

**编译安装**
``` shell
./ configure --prefix=/usr/local/tengine

make && make install
```
通常将tengine安装在`/usr/local/tengine`目录下

完成以上两步Nginx就已经安装完成，可以进入到`/usr/local/tengine`目录中查看编译后的文件，这个目录有conf、html、logs以及sbin四个文件夹，其中sbin里保存所有可执行文件，我们可以运行其中的nginx这样nginx服务器就启动了，打开浏览器访问服务器的公网ip就能看到Nginx默认的主页。

直接启动nginx的话关闭服务器比较麻烦，需要用传统的kill -9 杀掉对应的所有进程
```
ps -ef | grep nginx
kill -9 [pid]
```

### 补充

后来又在mac系统下安装Nginx遇到了些小小的问题，所以在这里也记录一下安装过程，mac系统也是内置了openssl，但是在生成makefile步骤中还是遇到了找不到openssl依赖的问题，似乎是因为macOS内置的版本过低？

在mac系统中，可以使用`brew install nginx`安装，这样的问题是对于软件的版本不太可控，所以这里采用手动编译源码的的方式

首先需要的下载源代码，到同一个目录中
```
http://nginx.org/download/nginx-1.17.8.tar.gz
https://ftp.pcre.org/pub/pcre/pcre-8.41.tar.gz
https://www.openssl.org/source/openssl-1.1.0g.tar.gz
```

进入nginx源码目录，运行configure，并设置依赖的路径，这里设置安装目录依旧是在/usr/local/nginx
```shell
./configure --with-pcre=../pcre-8.41/ --with-http_ssl_module --with-openssl=../openssl-1.1.0g --prefix=/usr/local/nginx

make && make install
```


### 脚本化操作
直接使用命令开关Niginx服务比较麻烦，可以搞个shell脚本来封装一下直接一个全局命令启动关闭，这里突然想起到自己的毕设，当时程序在部署运行的时候特麻烦，所以把每个节点复杂的参数命令封装成了一个个脚本，最后发现一个个的启动脚本太折腾了，所以又写了一个启动脚本的脚本...

这里提供一个控制Nginx服务的脚本，这个脚本不是官方提供的，做开发时逛git找到好用的脚本时不要直接拿来跑，最好先浏览一下脚本的逻辑

创建`/etc/init.d/nginx`文件中，将后面提供的脚本拷贝到里面，并修改可执行权限
```
chmod 777 nginx
```

#### 服务命令

`service Nginx start`  启动服务

`service Nginx stop` 停止

`service Nginx status` 状态

`service Nginx reload` 动态重载配置文件

#### shell脚本源代码

```shell
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemon
#
# chkconfig:   - 85 15 
# description:  Nginx is an HTTP(S) server, HTTP(S) reverse \
#               proxy and IMAP/POP3 proxy server
# processname: nginx
# config:      /etc/nginx/nginx.conf
# config:      /etc/sysconfig/nginx
# pidfile:     /var/run/nginx.pid
 
# Source function library.
. /etc/rc.d/init.d/functions
 
# Source networking configuration.
. /etc/sysconfig/network
 
# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0
 
nginx="/usr/local/tengine/sbin/nginx"
prog=$(basename $nginx)
 
NGINX_CONF_FILE="/usr/local/tengine/conf/nginx.conf"
 
[ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx
 
lockfile=/var/lock/subsys/nginx
 
make_dirs() {
   # make required directories
   user=`nginx -V 2>&1 | grep "configure arguments:" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`
   options=`$nginx -V 2>&1 | grep 'configure arguments:'`
   for opt in $options; do
       if [ `echo $opt | grep '.*-temp-path'` ]; then
           value=`echo $opt | cut -d "=" -f 2`
           if [ ! -d "$value" ]; then
               # echo "creating" $value
               mkdir -p $value && chown -R $user $value
           fi
       fi
   done
}
 
start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    make_dirs
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}
 
stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}
 
restart() {
    configtest || return $?
    stop
    sleep 1
    start
}
 
reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $nginx -HUP
    RETVAL=$?
    echo
}
 
force_reload() {
    restart
}
 
configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}
 
rh_status() {
    status $prog
}
 
rh_status_q() {
    rh_status >/dev/null 2>&1
}
 
case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
```
> 注意脚本中有两个变量需要设置对你的Nginx服务器对应路径<br>
nginx="/usr/local/tengine/sbin/nginx" <br>
NGINX_CONF_FILE="/usr/local/tengine/conf/nginx.conf"`


## Nginx使用简介

Nginx作为一个web服务器，以内存占有少、并发能力强而出名，有非常多国内的互联网公司在使用；首先是因为高并发且低内存占用特性，官方测试Nginx能够支撑5万并发链接，运行非常稳定，其次是我们经常用它来做负载均衡，动静分离等操作

### Web服务器与应用服务器
第一次接触 Nginx 的时候有一个小小的疑问，哎我不是已经有了一个 tomcat 服务器了吗，这个服务器可以对外提供服务呀，为什么还要使用 Nginx ；

我对于这个问题的理解是这样的，首先你需要区分Web服务器与应用服务器在功能上的不同，所谓 Web 服务器，或者叫做 HTTP 服务器，它主要关心的是在 HTTP 协议层面的传输和访问控制，比如常见的Web服务器有Nginx、Apache、IIS，它们主要提供的功能是让客户端通过HTTP协议访问服务器上的静态资源，另外就是做代理、负载均衡等；而对于应用服务器，比如tomcat，则是一个应用执行的容器，同时为了方便应用服务器往往也会集成一些HTTP服务器功能，但是不如HTTP服务器那么强大，所以应用服务器一般是运行在HTTP服务器的背后，执行应用程序业务逻辑，最后将动态内容转化为静态的内容后，由HTTP服务器返回客户端；所以你可以看见Nginx+tomcat或者Nginx+nodejs等组合

### 代理
经常听到正向代理与反向代理的概念，其中代理的意思与设计模式中的代理其实是一个意思：给某一个真实对象（realsubject）提供一个代理（proxy），并由代理对象（proxy）来控制对真实对象（realsubject）的访问，我们在基于HTTP协议的web场景中来看一下，这里就俩对象客户端（client）和服务器端（Server），如果你对client做了代理就是正向代理，比如科学上网？我们都知道有些网站是不存在的比如Google、油管，我们上不去，那就由代理服务器来帮我们访问，行了不能再写了...，接着如果你对于Server做了代理那就是反向代理，比如你有非常多的服务器组成了一个集群，通常我们是使用Nginx为这些服务器做代理，用户访问的是Nginx服务器，并不知道自己真实访问的是哪台物理服务器。

### 虚拟主机
Nginx可以做虚拟主机，意思就是将一个服务器当做多个服务器用，Nginx部署在一个物理服务器上，却通过ip、端口、域名对外实现多个访问入口，让客户端以为是多个服务器，后面会讲解如何配置虚拟主机。

### 负载均衡
一台服务器能处理的并发是有上限的，通常我们为了提高服务器的处理并发的能力可以采用负载均衡的方式，简单理解为就是在多台服务器上部署了相同的程序，在Nginx做好配置，用户访问Nignx服务器，由Nginx控制用户请求由哪一台服务器处理，所以负载均衡可以理解为是反向代理 + 为每个代理设置weight权重 


## Nginx配置文件解析
Nginx使用比较容易，上节提到的功能我们直接在Nginx的配置文件中做相应的配置就可以使用，而不需要做单独的编程

nginx配置文件在安装目录的conf目录中
```
/usr/local/tengine/conf/nginx.conf 
```

### Nginx配置文件结构
一般来说，Nginx配置文件包含一下几个部分
```shell
# 全局块
...              
# events块
events {         
   ...
}
# http块
http      
{
    # http全局块
    ...   
    # 虚拟主机server块
    server        
    { 
        # server全局块
        ...       
        # location块
        location [PATTERN]   
        {
            ...
        }
        location [PATTERN] 
        {
            ...
        }
    }
    server
    {
      ...
    }
    # http全局块
    ...     
}
```
#### 全局块
配置影响nginx全局的指令。

#### events块
该配置影响nginx服务器与client的网络连接

#### http块
主要用来配置代理、缓存、日志定义等绝大多数功能和第三方模块的配置，可以嵌套多个server，每一个server就是一个虚拟主机


#### server块
配置虚拟主机的相关参数，一个http中可以有多个server

虚拟主机：一种特殊的软硬件技术,它将一台服务器分成多个虚拟的服务器,每个虚拟主机可以独立对外提供服务,这样就可以实现一台主机对外提供多个web服务

#### location块
配置具体请求的路由，以及各种页面的处理情况


### Nginx配置文件具体解析

这里我会把工程自带的配置文件分段的展示在下方，并整理每个常用配置的作用

#### 全局块 - fragment1
```shell
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
#error_log  "pipe:rollback logs/error_log interval=1d baknum=7 maxsize=2G";

#pid        logs/nginx.pid;
```

**user**

定义Nginx运行的用户和用户组，默认被注释即默认配置为nobody，这个配置代表哪个用户以及用户组运行的Nginx如果在配置时省略了用户组名，那么用户组名默认等于用户名；通常遇到权限问题是这个参数设置的问题，一般不要设置为root
    
`user zhou zhou;`

**worker_processes**

进程数，即Nginx可以开启多少进程，建议设置为等于CPU总核心数。

`worker_processes 4;`

**error_log**

全局错误日志，格式为`error_log <FILE> <LEVEL>;`，其中错误日志类型包括`[ debug | info | notice | warn | error | crit ]`

`error_log /var/log/nginx/error.log info;`

**pid**

设置进程文件
    
`pid /var/run/nginx.pid;`


#### events块 - fragment2
```
events {
    worker_connections  1024;
}
```

events模块包含nginx中所有处理连接的设置.

**use**

events有一个事件模型的配置，默认配置文件中没有设置（后续补充每种设置的作用）

`use [ kqueue | rtsig | epoll | /dev/poll | select | poll ];`

**worker_connections**

单个进程最大连接数 `worker_connections 1024;` 

所以Nginx最大并发总数是 worker_processes 和 worker_connections 的乘积，即 max_clients = worker_processes * worker_connections；

设置了反向代理的情况下，max_clients = worker_processes * worker_connections / 4，反向代理除以4是一个经验结论

另外该设置只是理论值，worker_connections值的设置跟内存大小有关，而且因为受到I/O约束，所以max_clients的值要小于系统可以打开的最大文件句柄数，也就是说我的系统最大同时能打开这么多文件，Nginx这边能同时创建的连接应该小于这个值，若查看系统打开的最大文件句柄数使用`–$ cat /proc/sys/fs/file-max`命令

worker_connections 的值需根据worker_processes进程数目和系统可以打开的最大文件总数进行适当地进行设置,使得并发总数小于操作系统可以打开的最大文件数目，当然，理论上的并发总数可能会和实际有所偏差，因为主机还有其他的工作进程需要消耗系统资源。

最后一点是，linux对于一个进程能打开的最大文件句柄也有限制，配置返回如下：系统总限制： `/proc/sys/fs/file-max`，当前使用句柄数：`/proc/sys/fs/file-nr `，修改句柄数：`ulimit -SHn 65535`

#### http块 - fragment3
```shell
http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;
    #access_log  "pipe:rollback logs/access_log interval=1d baknum=7 maxsize=2G"  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
    ...
    }

    server {
    ...
    }
```
**include**

`include mime.types;` 表示引用了./mime.types这个文件，它存储了文件扩展名与文件类型映射表，内容如下：

```
types {
text/html                                        html htm shtml;
text/css                                         css;
text/xml                                         xml;
image/gif                                        gif;
image/jpeg                                       jpeg jpg;
application/javascript                           js;
application/atom+xml                             atom;
application/rss+xml                              rss;
.
.
.
```
第一列对应HTTP协议中Content-Type类型，第二列是该类型实际的文件扩展名，Nginx通过服务器端文件的后缀名来判断这个文件属于什么类型，再将该数据类型写入HTTP头部的Content-Type字段中，返回客户端

**default_type**

表示为mime.types文件中不存在的文件类型，使用该默认类型
`default_type application/octet-stream;` 

**charset**

`charset utf-8;` 默认编码

**client_header_buffer_size**

`client_header_buffer_size 32k;` 上传文件大小限制

**sendfile**

`sendfile` sendfile指令指定nginx是否调用sendfile函数来输出文件

sendfile on; 开启高效文件传输模式，对于普通应用设为 on，对于下载等IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载，如果图片显示不正常把这个改成off。

这个参数的工作原理是这样的，若设置为off时，进行下载时文件首先会传递给Nginx应用，接着Nginx会通知系统内核，并将文件传递给内核，最后由内核通过网卡将文件传输出去，整个过程有两次耗时的IO操作；而若设置为on时，文件不再被传递给Nginx，而是Nginx向内核发送指令，操作内核去获取文件通过网卡将文件传输出去

**tcp_nopush**   

`tcp_nopush` 在linux/Unix系统中优化tcp数据传输，仅在sendfile开启时有效

**autoindex**

`autoindex on; `#开启目录列表访问，合适下载服务器，默认关闭。

**keepalive_timeout**

`keepalive_timeout 120; `长连接超时时间，单位是秒

**gzip**

`gzip on;` 开启gzip压缩输出

`gzip_min_length 1k;`  设置允许压缩的页面最小字节数，页面字节数从header头得content-length中进行获取。默认值是0，不管页面多大都压缩。建议设置成大于2k的字节数，小于2k可能会越压越大。  

`gzip_buffers 4 16k;` 用于向系统申请缓存，来存储gzip的压缩结果数据流。 例如 4 4k 代表以4k为单位，按照原始数据大小以4k为单位的4倍申请内存。 4 8k 代表以8k为单位，按照原始数据大小以8k为单位的4倍申请内存。如果没有设置，默认值是申请跟原始数据相同大小的内存空间去存储gzip压缩结果。

`gzip_http_version 1.0; `压缩版本（默认1.1，前端如果是squid2.5请使用1.0）

`gzip_comp_level 2;` 压缩级别，1-10，数字越大压缩的越好，也越占用CPU时间  

`gzip_types text/plain application/x-javascript text/css application/xml;`压缩类型，默认值: gzip_types text/html，默认不对js/css文件进行压缩


~~哇太多了剩下的明天写~~
#### server块 - fragment4
```
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;

    #access_log  logs/host.access.log  main;
    #access_log  "pipe:rollback logs/host.access_log interval=1d baknum=7 maxsize=2G"  main;

    location / {
        root   html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
    .
    .
    .
    其他location略
}
```
每一个server块就对应一个虚拟主机配置，在server中有location配置用来表示该虚拟主机对应的磁盘目录

nginx支持三种类型的虚拟主机配置
- 基于ip的虚拟主机， （一块主机绑定多个ip地址）
- 基于域名的虚拟主机（servername）
- 基于端口的虚拟主机（listen如果不写ip端口模式）

**listen**

` listen       80;`当前虚拟主机监听的端口

**server_name**

`server_name www.abc.com abc.com; ` 当前虚拟主机映射的域名，可以有多个用空格隔开

**charset**

`charset koi8-r;` 编码集

**access_log** 

访问日志

 ```access_log  logs/host.access.log  main;
access_log  "pipe:rollback logs/host.access_log interval=1d baknum=7 maxsize=2G"  main;
 ```

**location**
```
location / {
    root   html;
    index  index.html index.htm;
}
```
一个url分为主机名以及资源部分，例如
```
http://www.abc.com/account/login.html
```
此时`http://www.abc.com`这部分就是主机名，在Nginx对应server块的配置，`/account/login.html`是资源名对应server块中的location配置

#### location块 - fragment5
```
    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}
    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
```

**location格式**

location [ = | ~ | ~* | ^~ ] uri { ... }

- `location URI {}` 对当前路径及子路径下的所有对象都生效；

- `location = URI {}` 注意URL最好为具体路径。  精确匹配指定的路径，不包括子路径，因此，只对当前资源生效；

- `location ~ URI {}   location ~* URI {} `  模式匹配URI，此处的URI可使用正则表达式，~区分字符大小写，~*不区分字符大小写；

- `location ^~ URI {}` 禁用正则表达式

**优先级**：= > ^~ > ~|~* >  /|/dir/

这里展示一个demo
```
# =是指匹配与条件完全相同的路径，此时只匹配/的资源路径，形如/account等路径不会被匹配
location = / {
    [ configuration A ]
}

# 因为所有的地址都以 / 开头，所以这条规则将匹配到所有请求，但还会继续往下匹配，正则和最长字符串会优先匹配
location / {
    [ configuration B ]
}

# 匹配任何以 /documents/ 开头的路径，匹配符合以后，还会继续往下搜索，只有后面的正则表达式没有匹配到时，这一条才会采用这一条
location /documents/ {
    [ configuration C ]
}

# 匹配任何以 /images/ 开头的地址，如果匹配符合以后，则不需要后续进行正则匹配
location ^~ /images/ {
    [ configuration D ]
}

# 匹配所有以 gif,jpg或jpeg 结尾的请求
location ~* \.(gif|jpg|jpeg)$ {
    [ configuration E ]
}
```
<!-- 
结论
- location会从头开始顺序遍历到最后
    - 如果location 是=精确匹配指定的路径，若找到对应匹配直接结束
    - 如果location 是^~，后续不会再匹配任何的正则表达式location，继续遍历但不是顺序而是跳序
    - 其他location 如果匹配上记录并继续向下，未匹配上也继续向下
- 结束后找到最长字符串最为匹配结果 -->







