---
title: JavaWeb笔记-FastDFS D1
date: 2020-07-16 16:12:09
categories:
- [Java开发]
tags:
- [Nginx]
- [FastDFS]
---
该笔记记录了使用FastDFS搭建单节点文件服务器的过程，主要包括FastDFS简介，FastDFS工作流程，单节点安装配置，配置Nginx进行http访问以及使用Java客户端去访问FastDFS文件服务器等操作，注意该笔记所有操作都是在单节点中实现的！

<!-- more -->

## FastDFS简介
FastDFS是使用C语言开发的轻量级分布式文件系统，特别适合于以文件为载体的在线服务，比如视频网站，图片网站；它主要提供对于文件管理的功能，包括但不仅限于文件存储、文件同步、文件上传&下载；FastDFS考虑了冗余备份、负载均衡、线性扩容等机制，并注重高可用、高性能指标，所以使用FastDFS能够快速的搭建出一套高性能的文件服务器集群

FastDFS中主要包含三个角色：客户端（Client）、追踪服务器（Tracker Server）、存储服务器（Storage Server）
 - 客户端（Client）：并不是指浏览器，而是我们指项目所部署的服务器，是上传下载数据的入口
 - 存储服务器（Storage Server）：主要用来提供存储与备份服务，实际文件就存储在该服务器中
 - 追踪服务器（Tracker Server）：主要做调度工作，起到均衡的作用，负责管理所有的存储服务器

首先我们在最基础的单节点的环境来简述FastDFS的整体工作流程，需要在服务器上先安装启动FastDFS的`存储服务`与`追踪服务`，存储服务是负责实际存储文件的程序，而client在上传或者下载文件时需要先访问追踪服务，追踪服务会返回存储服务的信息包括Storage Server的IP与端口，之后client向存储服务发送上传或者下载的请求完成实际的任务，client可以是你的web程序，来发出文件相关的请求

FastDFS的存储服务与追踪服务可以是单机或者是由多台由服务器构成的集群，存储服务与追踪服务的服务器可以随时增加或者下线而不会影响线上服务；

存储服务完成了文件管理的所有功能：存储、同步、存取等接口，存储服务不仅仅保存文件还可以保存文件的元信息(metadata)，metadata是文件的属性列表以键值对的形式保存了文件的属性信息。存储服务为了支持大容量存储，采用了分组的组织方式，整个存储系统由一个或者多个组构成，组就类似于Windows的C、D、F等磁盘的概念，每个组之间的文件是相互独立的，最终整个存储系统的总存储容量是所有组的存储容量之和；一个组可以由一台或者多台服务器构成，这里要提一下如果一个组有多台服务器并不能起到增加这个组存储容量的作用，同组的多台服务器之间存储的内容是相同的，如果你在某组内上传一个文件，系统会自动把这个文件在同组内所有服务器中拷贝一份，所以同组之内的多台服务器是用来做冗余备份和负载均衡的，因此某个组的最大存储容量时等于这个组内最小的那台服务器存储空间；如果你需要在某个组内增加服务器，系统会先在新服务器中自动同步组内的文件，同步完成后系统才会允许新增服务器上线提供服务。为了提高系统的总存储空间我们应该增加新的组，而不是在组内添加服务器，只需要添加一台或者多台服务器，并配置为新的组，这样就可以了，所以在FastDFS中标示一个文件的是由两部分组成：组名+文件名。

因为FastDFS是一个以组为单位的分布式的文件系统，所以在请求文件时需要一个调用的程序来决定实际由哪个服务器来处理请求。这就是追踪服务器的任务，主要是做调度任务并且在访问上起到了负载均衡的作用，例如请求是上传文件到某一个组内，追踪服务器会根据一定的策略查询这个组可用的存储服务器，返回存储服务器的IP与端口，追踪服务器记录了所有的存储服务器状态，是连接client与存储服务器的枢纽。如果在FastDFS中部署了多台存储服务器，存储服务器之间是对等的关系，同时提供服务。存储服务器会定期的向追踪服务器集群发送自己的存储信息，包括磁盘剩余空间、文件同步状态、文件上传下载次数等数据。

## FastDFS上传下载流程

**上传流程**
<img src="/img/java/upload.png" width="90%" style="text-align:center">

- storage定期的向tracker发送自己的存储信息
- client向tracker发送上传文件的请求，不需要附加参数
    - 当集群中不只一个tracker时，client发送上传文件的请求可以任意选择一个trakcer，当tracker接收到上传文件的请求时，会为该文件分配一个可用的group
- tracker返回一台可用的storage，tracker会在group内选择一个storage服务器信息返回给客户端
- client直接和storage通讯完成文件上传，返回文件的存储信息，例如`group1/M00/00/00/wKiWDV0xfqWAFe1OAAAib-i5DLU637.log`，其中`group1`对应的就是组名，其余部分的含义如下：
    - client向storage发送上传文件请求，storage将会为文件分配一个数据存储目录，存储目录是指`/M00/`，这是我们后续配置的虚拟目录，它映射到了磁盘的一个真实目录
    - 选定存储目录之后，storage会为文件生一个Fileid，作为文件在服务器上保存的名称。例如`wKiWDV0xfqWAFe1OAAAib-i5DLU637`
    - 每个存储目录下都有两级的256*256的子目录，storage会根据上一步生成的Fileid选择其中一个二级子目录，然后将文件以Fileid为文件名存储到该子目录下，例如`00/00/`。
    - 当文件存储到目录中就认为该文件存储成功，接着向client返回文件的完整存储信息，文件存储信息由group、存储目录、二级子目录、fileid、文件后缀名（由客户端指定，主要用于区分文件类型）拼接而成。



**下载流程**
<img src="/img/java/download.png" width="90%" style="text-align:center">
- client询问tracker下载文件的storage，参数为文件标识（组名和文件名）；
- tracker返回一台可用的storage；
- client直接和storage通讯完成文件下载。


## 单节点安装配置FastDFS

操作系统：CentOS

### 安装依赖
安装必须使用make、cmake和gcc编译器。

``` 
yum install -y make cmake gcc gcc-c++
```

### 安装FastDFS核心库
libfastcommon是从FastDFS 和FastDHT 中提取出来的公共C函数库

**1、下载源代码**
```
https://github.com/happyfish100/libfastcommon/releases
```

**2、编译安装**

① 解压源代码，进入源代码根目录

② 编译安装
```
./make.sh
./make.sh install
```

**3、创建软连接**

libfastcommon有固定的默认安装位置，在/usr/lib64和/usr/include/fastcommon两个目录中，但是FastDFS主程序设置的lib目录是/usr/local/lib，所以需要创建软链接。
```
ln -s /usr/lib64/libfastcommon.so /usr/local/lib/libfastcommon.so

ln -s /usr/lib64/libfastcommon.so /usr/lib/libfastcommon.so

ln -s /usr/lib64/libfdfsclient.so /usr/local/lib/libfdfsclient.so

ln -s /usr/lib64/libfdfsclient.so /usr/lib/libfdfsclient.so
```

### 安装FastDFS主程序

**1、下载源代码**
```
https://github.com/happyfish100/fastdfs/releases
```


**2、编译安装**

① 解压源代码，进入源代码根目录

② 编译安装
```
./make.sh
./make.sh install
```

**3、安装目录**

FastDFS主程序所在位置：
```
/usr/bin - 可执行文件所在位置
/etc/fdfs - 配置文件所在位置
/usr/lib64 - 主程序代码所在位置
/usr/include/fastdfs - 包含的一些插件组所在位置
```

### 配置启动Tracker服务器

配置文件在`/etc/fdfs/`目录中，作者提供了模版的样例，使用时需要copy一份

- tracker.conf.sample - 跟踪器服务配置文件模板

- storage.conf.sample - 存储服务器配置文件模板

- client.conf.sample - FastDFS提供的命令行客户端配置文件模板。可以通过命令行测试FastDFS有效性。

启动程序脚本安装在了`/etc/init.d/`目录中，脚本文件分别是 **fdfs-storaged**和**fdfs-trackerd**


**1、修改Tracker配置文件**

复制一份模板配置文件

```
cd /etc/fdfs

cp tracker.conf.sample tracker.conf
```

编辑tracker.conf，修改`base_path`路径，Tracker启动后的根目录，用来存放Tracker data和logs

```
base_path=/home/yuqing/fastdfs -> base_path=/var/data/fastdfs-tracker（自定义目录）
```

另外配置中的路径必须存在，否则启动失败

```
mkdir -p /var/data/fastdfs-tracker
```

**2、启动Tracker**

```
/etc/init.d/fdfs_trackerd start
```

启动成功后，配置文件中base_path指向的目录中出现FastDFS服务相关数据目录（data目录、logs目录）

**3、查看Tracker服务状态**

```
ps -ef | grep fdfs
```

**4、停止Tracker服务**

```
/etc/init.d/fdfs_trackerd stop
```
**5、重启Tracker服务**

```
/etc/init.d/fdfs_trackerd restart
```

### 配置启动Storage服务器


**1、修改Storage配置文件**

复制一份模板配置文件
```
cd /etc/fdfs

cp storage.conf.sample storage.conf
```

编辑storage.conf，主要修改三个配置
```
base_path=/home/yuqing/fastdfs -> base_path=/var/data/fastdfs-storage/base（自定义目录）

store_path0=/home/yuqing/fastdfs -> store_path0=/var/data/fastdfs-storage/store（自定义目录）

tracker_server=192.168.150.11:22122 -> tracker_server=tracker服务IP:22122
```

- base_path - 基础路径。用于保存storage基础数据内容和日志内容的目录。
- store_path0 - 存储路径，也就是上节中提到的虚拟路径映射到磁盘目录的配置，是FastDFS用于存储文件的目录，就是要创建256*256个子目录的位置。
- tracker_server - 追踪服务器位置，就是跟踪服务器的ip和端口。这里不能是127.0.0.1，所以我设置成了服务器的公网IP。


另外配置中的路径必须存在，否则启动失败

```
mkdir -p /var/data/fastdfs-storage/base
mkdir -p /var/data/fastdfs-storage/store
```

**2、启动Storage**

要求tracker服务必须已启动

```
/etc/init.d/fdfs_storaged start
```

启动成功后，配置文件中base_path指向的目录中出现FastDFS服务相关数据目录（data目录、logs目录）

配置文件中的store_path0指向的目录中同样出现FastDFS存储相关数据录（data目录）

其中$store_path0/data/目录中默认创建若干子孙目录（两级目录层级总计256*256个目录），是用于存储具体文件数据的。

Storage服务器启动比较慢，因为第一次启动的时候，需要创建256*256个目录。

**3、查看Storage服务状态**

```
/etc/init.d/fdfs_storaged status
```

**4、停止Storage服务**

```
/etc/init.d/fdfs_storaged stop
```
**5、重启Storage服务**

```
/etc/init.d/fdfs_storaged restart
```

### 配置使用Client

上一步成功启动了Tracker和Storage后，单机的FastDFS服务器就已经搭建完成，为了测试可以使用程序中带有的client

**1、配置Client**

```
cd /etc/fdfs
cp client.conf.sample client.conf
```

client.conf配置文件中主要描述客户端的行为，需要进行下述修改：

```
base_path=/home/yuqing/fastdfs -> base_path=/fastdfs/client （自定义目录）

tracker_server=192.168.150.11:22122 -> tracker_server=tracker服务IP:22122

```

base_path - 就是客户端命令行执行过程时临时数据存储位置。

同样需要保证base_path目录是存在的

```
mkdir -p /fastdfs/client
```

**2、上传文件**

/usr/bin/fdfs_upload_file /etc/fdfs/client.conf  /要上传的文件

```
[root@node03 data]# /usr/bin/fdfs_upload_file /etc/fdfs/client.conf /root/install.log
group1/M00/00/00/wKiWDV0xfqWAFe1OAAAib-i5DLU637.log
```

其中返回的`group1/M00/00/00/wKiWDV0xfqWAFe1OAAAib-i5DLU637.log`就是文件的存储信息
- 组名：**group1**文件上传后所在的storage组名称，在文件上传成功后由storage服务器返回，需要客户端自行保存。
-  虚拟磁盘路径：**M00**  storage配置的虚拟路径，与磁盘选项store_path*对应。如果配置了store_path0则是M00，如果配置了store_path1则是M01，以此类推。
- 数据两级目录：**/00/00** storage服务器在每个虚拟磁盘路径下创建的两级目录，用于存储数据文件。
- 文件名：**wKiWDV0xfqWAFe1OAAAib-i5DLU637.log**

**3、删除文件**

```
/usr/bin/fdfs_delete_file /etc/fdfs/client.conf group1/M00/00/00/wKiWDV0xfqWAFe1OAAAi
b-i5DLU637.log
```

## 配置Nginx组件

此时无法通过HTTP去访问文件，比如加载头像等操作，为了可以通过HTTP协议访问文件，通常我们做法是安装配置Nginx，在集群中是在Storage服务器安装Nginx，该笔记是单节点所以我们是直接在单机节点安装

前几天的笔记详细记录了如何安装Nginx，这里只安装FastDFS的Nginx模块

**1、下载fastdfs-nginx-module源代码**
```
https://github.com/happyfish100/fastdfs-nginx-module/releases
```

**2、解压进入根目录**

**3、修改安装配置**
```
vi src/config

CORE_INCS="$CORE_INCS /usr/include/fastdfs /usr/include/fastcommon/"
```

**4、编译安装Nginx**

```
./configure --prefix=/usr/local/tengine
--add-module=/root/fastdfs-nginx-module/src/
```

这里--prefix设置为nginx的安装目录

```
make && make install
```

**5、修改运行配置**

① 拷贝一份`fastdfs-nginx-module`源代码中的配置文件
```
cp /root/fastdfs-nginx-module/src/mod_fastdfs.conf /etc/fdfs/
```

② 修改`/etc/fdfs/mod_fastdfs.conf`配置文件
```
tracker_server=your/IP:22122
url_have_group_name = true
store_path0=/var/data/fastdfs-storage/store
```

③ 拷贝http服务需要的配置

复制fastdfs主程序源代码中的两个配置文件（conf/http.conf和conf/mime.types）到`/etc/fdfs`目录中


④  创建网络访问存储服务的软连接

在上传文件到FastDFS后，FastDFS会返回group1/M00/00/00/xxxxxxxxxx.xxx。其中group1是卷名，在mod_fastdfs.conf配置文件中已配置了url_have_group_name，以保证URL解析正确。

另外的的M00是FastDFS保存数据时使用的虚拟目录，我们需要将这个虚拟目录定位到真实数据目录上
```
ln -s /var/data/fastdfs-storage/store/data/  /var/data/fastdfs-storage/store/data/M00
```

⑤ 最后一步！！修改nginx配置文件，并让运行中的Nginx重新加载配置文件即可

```
    location ~ /group([0-9])/M00 {
        ngx_fastdfs_module;
    }
    
```

哇，终于结束有一丢丢麻烦，完成这些后我们就可以直接通过HTTP来访问文件了，还记得上一节中我们上传到文件服务器中的那个文件吗，现在我们可以直接在浏览器中访问它了，url如下：
```
http://your.ip/group1/M00/00/00/wKiWC10xxc6AfHCKAAAib-i5DLU543_big.log
```
当你在浏览器中输入url后就会下载这个log文件，但是这个文件的名称并不友好，我们如果希望暴露出更加易读的文件名还需要单独的配置😂

只要修改Nginx的配置就可以

```
location ~ /group([0-9])/M00 {
    ngx_fastdfs_module;
    add_header Content-Disposition "attachment;filename=$arg_attname";
}
```

这样我们在访问文件时url需要加上这样的一个参数
```
http://your.ip/group1/M00/00/00/wKiWC10xxc6AfHCKAAAib-i5DLU543_big.log？attachment=a.log
```

## Java客户端

上文中都是通过程序自带的命令作为client，该小节记录实际将JavaWeb工程作为client来访问FastDFS，这里使用了FastDFS_Client这个项目来访问FastDFS，在Java工程中配置可以参看源代码的README，`https://github.com/tobato/FastDFS_Client`，是一个蛮详细的中文文档，这里只简单的记录基本的使用

**1、安装依赖**
<dependency>
   <groupId>net.oschina.zcx7878</groupId>
   <artifactId>fastdfs-client-java</artifactId>
   <version>1.27.0.0</version>
</dependency>

**2、修改SpringBoot配置**
```
fdfs:
  so-timeout: 1500
  connect-timeout: 600
  tracker-list:
  - your.ip:22122
```

**3、上传示例**
```java
// 元数据
Set<MetaData> metaDataSet = new HashSet<MetaData>();
    metaDataSet.add(new MetaData("Author", "ztp"));
    metaDataSet.add(new MetaData("CreateDate", "2020-01-05"));
    
    
try {
    StorePath uploadFile = null;
    uploadFile = fc.uploadFile(filename.getInputStream(), filename.getSize(), getFileExtName(filename.getOriginalFilename()), metaDataSet);

    String FileSavePath = uploadFile.getPath();//获取带group的全路径
    //uploadFile.getPath();不带group的全路径
} catch (FileNotFoundException e) {
    // TODO Auto-generated catch block
    e.printStackTrace();
}
```

代码中获取文件扩展名有以下两种方式
```java
//method1
private String getFileExtName(String name) {
    // TODO Auto-generated method stub
    return (name.substring(name.lastIndexOf(".")+1));
}
	
//method2
FilenameUtils.getExtension();
```

**4、下载示例**

```java
@RequestMapping("/down")
    @ResponseBody
    public ResponseEntity<byte[]> down(HttpServletResponse resp) {
        
        DownloadByteArray cb = new DownloadByteArray();
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_OCTET_STREAM);
        headers.setContentDispositionFormData("attachment", "aaa.xx");
        byte[] bs = fc.downloadFile("group1", "M00/00/00/wKiWDV0vAb-AcOaYABf1Yhcsfws9181.xx", cb);
        
        return new ResponseEntity<>(bs,headers,HttpStatus.OK);
    }
```

**5、其他功能**

自动创建缩略 ，比如根据用户上传的头像图片自动创建其他尺寸的文件

修改SpringBoot配置
```
  thumb-image:
    width: 150
    height: 150
```

```
uploadFile  = fc.uploadImageAndCrtThumbImage(filename.getInputStream(), filename.getSize(), FilenameUtils.getExtension(filename.getOriginalFilename()), metaDataSet);
		
```