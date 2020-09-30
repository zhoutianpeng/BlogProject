---
title: JavaWeb笔记-Spring D1
date: 2020-07-02 04:00:26
categories:
- [Java开发]
tags:
- [Spring]
---

## 写在前面
最近在重新复习Java Web方面的知识，并且有计划准备接触分布式，大数据以及微服务的知识，看看工作一年后再接触校园时学习的技术有什么感触😂，ps复习计划会持续，但是博客不定时更新，也许是我废话太多，写一篇博客太慢了

<!-- more -->

## Spring介绍
Spring是一个轻量级的Java Web开发框架，既然是框架就是为了简化我们复杂的开发过程。

Spring主要提供了以下支持：
- 控制反转(Inversion of Control,IoC) 方便解耦；
- 面向切面编程(Aspect Oriented Programming, AOP)；（老实说因为工作不怎么接触Java，所以对AOP没有什么深入的理解）
- 事务的支持；
- 便于集成第三方框架；

## Spring demo
我们会实现一个最小的Spring应用demo来体验Spring的使用

在这里我们会用手动的方式搭建一个Spring环境，而非使用Maven等构筑工具，IDE选择的是Eclipse，~~绝对不是因为IDEA过期了，我很喜欢Eclipse的，嗯真的~~，在demo中我们会利用Spring的IoC功能来创建一个对象，而非手动`new`一个

最初学习的时候我始终无法理解IoC的意思，或者是它的意义是什么？我们为什么要费这么大的劲让程序帮我们new对象，我一贯的思考方式是：如果想弄明白一个东西存在的意义，就想象这个东西不存在会发生什么；刚接触时我用这种方式思考得不出结论，因为那时经验不足，没有做过不用框架写web的项目，所以选择带着问题继续往下看，也就是明确现在要通过框架来创建对象而不是自己new一个，所以同样如果看到这里有IoC的疑惑不妨继续读下去，我们现在的目的就是使用IoC，目标就是在程序中不出现`new`关键字然后创建对象。

### 环境搭建

#### 1.下载Spring Jar包
这个下载地址是真的难找（不会就我一个人这么觉得吧，不会吧不会吧），在Spring官网极其隐蔽的位置，也许是因为通常我们不会手动下载jar文件的原因吧，地址如下：
```
https://repo.spring.io/libs-release-local/org/springframework/spring/
```

在仓库中选择好版本后会有三种文件可以下载，分别是
- *-dist.zip 保存的是Spring的源文件
- *-docs.zip 保存的是Spring的文档
- *-schema.zip 保存的是Spring的约束文件

我们需要下载源文件，解压后源文件的目录构成如下
- doc 保存的是Spring的文档
- libs 保存的是Spring的所有jar文件
- schema 保存的是Spring的约束文件
- license.txt 许可协议
- notice.txt 
- readme.txt

#### 2.创建项目并引用jar文件
在Eclipse中创建一个空项目，**不是web项目**

在项目的根目录下创建一个名为`libs`的目录用来存放引用的jar文件

我们需要将以下jar文件copy到`libs`目录中，并不是上一步下载的全部jar文件
- **spring-core.jar** 该jar包中包含框架核心基础的功能比如IoC、AOP
- **spring-beans.jar** 该jar包中包含访问配置文件、创建和管理bean以及进行IoC操作相关的所有类。这个jar包依赖spring-core.jar
- **spring-context.jar** 
- **spring-express.jar**
- **commons-logging.jar** 这个jar包不在Spring源码文件中，需要我们单独下载，[地址点我](http://commons.apache.org/proper/commons-logging/download_logging.cgi)

完成上一步之后只是把jar包复制到一个单独的目录中，而并未告诉工程要依赖这些jar文件，Eclipse的话需要对有所jar文件`右击 - Build Path - Add to build path`，第一次学的时候并不知道这件事，所以闹了不少笑话

#### 2.1题外话聊一聊这个java中依赖这个事情,不感兴趣请直接跳过该节😂

我们都知道java源码文件是`.java`，而java一份代码运行的过程是首先使用javac命令将该源代码编译成`.class`文件，这是一个与平台无关的字节码文件，最后由jvm虚拟机来执行这个字节码文件，假设jvm在执行过程中需要加载一个新的类比如是`com.abc.User`，其实本质就是jvm去寻找`User.class`这个字节码文件。

jvm是通过`classpath`来实现上述功能的，`classpath`是jvm的一个环境变量，用来告诉jvm如何搜索`class`文件；`classpath`里存储了一组目录的集合，jvm在寻找`class`文件时会依次搜索`classpath`中的每一个目录，查询是否有目标`class`文件，若找到就不继续搜索，如果遍历所有路径都没有找到则会报错。

如何为jvm设置`classpath`呢,有两种方式，一是在系统的环境变量中设置`CLASSPATH`环境变量，有没有熟悉的感觉？如果在window中配置java的开发环境通常我们需要设置三个环境变量`JAVA_HOME`、`PATH`以及`CLASSPATH`，这个`CLASSPATH`指向jdk里面的lib目录，也就是java的jdk里的核心库，比如常用的String、List、Map等等，jvm就是通过这个路径找到这些类对应的`class`文件。第二种设置`classpath`的方式就是在启动jvm时传入`-classpath`参数，比如我们可以用如下命令来运行main.class
```
java -classpath .:/usr/local/bin com.abc.main
```
或者用cp命令简写
```
java -cp .:/usr/local/bin com.abc.main
```
>在linux平台下多个路径是通过`:`分割，在windows平台下多个路径是通过`;`来分割

假设main这个类中某一行`new`了一个新的User对象，jvm就会去寻找`User.class`这个文件，此时jvm会去遍历当前目录（.）以及/usr/local/bin这个目录（当然也会遍历系统环境变量里的CLASSPATH）

说到这里需要提一下什么是jar，我是之前学了好久才注意这个问题，拿来这这里复习一下，jar文件本质就是一个zip文件，jar里包含多个class文件，方便对于文件管理，同时jar中还可以包含一个特殊的/META-INF/MANIFEST.MF文件，MANIFEST.MF是纯文本，可以指定一些其它信息。个人感觉可以类比一下c++的动态库的使用。

在Eclipse中运行java代码，Eclipse传入的-cp参数是当前工程的bin目录和引入的jar包。这个是Eclipse通过在工程的根目录下创建一个叫`.classpath`的隐藏文件来实现的，它是一个xml配置文件，我们之前提到的`右击 - Build Path - Add to build path`操作，其实本质就是把这个jar包的路径写入到`.classpath`中去，比如引用了Spring的jar文件后工程里的`.classpath`就会多出这些内容：

```xml
<classpathentry kind="lib" path="libs/spring-beans-5.1.7.RELEASE.jar"/>
<classpathentry kind="lib" path="libs/spring-context-5.1.7.RELEASE.jar"/>
<classpathentry kind="lib" path="libs/spring-core-5.1.7.RELEASE.jar"/>
<classpathentry kind="lib" path="libs/spring-expression-5.1.7.RELEASE.jar"/>
<classpathentry kind="lib" path="libs/commons-logging-1.2.jar"/>
```

#### 3.创建applicationContext.xml文件
回到原来问题进入工程，在根目录下src目录中创建一个名为`applicationContext.xml`目录，这个配置文件是Spring要求的，它会根据这里面的内容来创建对象，放到在src目录是因为编译时需要一同把这个文件也拷贝到输出目录去，通常输出目录是`根目录/bin`

在`applicationContext.xml`的书写是有约束的通常我们需要写入以下内容作为根节点
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p" xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:context="http://www.springframework.org/schema/context"
xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

</beans>
```
第一次看到这个时有点蒙，一直以来书写Spring的配置都是我的噩梦，尤其是在写Hibernate的配置文件时，真正体会到`java能把小项目变成大项目`是什么意思

之前每次在配置文件中看到新标签都会惊叹，wocao这是什么意思，没啥没有介绍这个标签作用

好在SpringBoot把这些配置都剃掉了

好的简单来说一下

第一行`<?xml version="1.0" encoding="UTF-8"?>`是xml必须要有的

第二行的`<beans></beans>`是整个配置文件根节点，它里面那一坨属性是用来校验配置文件格式的，所以每次写它复制一份就好了

另外注意这坨属性中`<beans xmlns="http://www.springframework.org/schema/beans"`这里面的`http://www.springframework.org/schema/beans`不是一个网址，你只需要把它理解问一个ID或者标识符就好了

而真正的网址是这个
`xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">`,这网址里存有xsd的校验文件

最后就是程序不是真的每次都去网上下载xsd文件，实际上在`spring-beans.jar`这个jar中保存了历代版本的xsd，每次程序会先去这里找，没有找到的时候再去网上扒拉一份

`spring-beans.jar`包中名叫
`org.springframework.beans.factory.xml.PluggableSchemaResolver`的类，它用一个静态常量保存了所有xsd的信息，如下

`public static final String DEFAULT_SCHEMA_MAPPINGS_LOCATION = "META-INF/spring.schemas";`

你可以打开这个`spring-beans.jar`中`org.springframework.beans.factory.xml`目录下`META-INF/spring.schemas`这个文件，它使用形如

`http\://www.springframework.org/schema/beans/spring-beans-2.0.xsd=org/springframework/beans/factory/xml/spring-beans.xsd`
的形式记录所有版本的xsd信息

其中`org/springframework/beans/factory/xml/spring-beans.xsd`是这个文件的包路径

#### 4.编写类以及applicationContext.xml文件

好了，想象一下我们是一个很厉害的程序猿，我们现在正在设计一个电商系统，于是设计了一个User类，这个类创建的对象用来保存用户信息，它除了拥有基础成员变量，我们额外添加一个Order类型的成员变量保存这个用户的订单信息，好吧其实User与Order之间应该是一对多的关系，为了简化我们这里设置成一对一，代码如下

User.java
```java
public class User {
	public String username;
	
	public String password;
	
	public Order order;

	public User(String username, String password, Order order) {
		super();
		this.username = username;
		this.password = password;
		this.order = order;
	}

    getter && setter ...
}
```

Order.java
```java
public class Order {
	public String orderId;
	
	public String orderInfo;

    getter && setter ...
}
```
为了能够让Spring可以以IoC的方式创建这个User对象，我们需要编写`applicationContext.xml`在`<beans></beans>`根节点下加入这个节点

```
<bean id="user" class="com.xxn.beans.User"></bean> 
```

代码如上，其实就是编写一个bean节点，id设置为’user‘，并把User的包路径设置给这个bean的class属性即可，这样Spring就可以通过这个配置文件来创建User对象了，在Spring中他所管理的对象被叫做bean对象，注意根据此时这个配置文件创建的user所有属性都是未初始化的默认值，当然我们可以业务逻辑中手动为他赋值，可注意其中User有一个Order类型的属性，如果我们new了一个Order设置给这个user，这样破坏了我们使用IoC的计划，我们的目的就是干掉`new`，所有对象的创建都交给Spring

解决这个问题我们需要使用依赖注入，意思就是Spring在创建对象时为他设置属性（注入，嗯，你品，你细品），也许你注意到了User类里面有一个构造器函数，接下来我们就让Spring使用这个构造函数来为User对象设置值，不过既然Spring需要把一个Order对象赋值给User对象作为成员变量，那么Spring就需要有能够创建Order对象的能力，也就是这里需要把Order也设置为一个bean

```	
<bean id="order" class="com.xxn.beans.Order"></bean> 
```

接下来我们让Spring可以访问User的构造器，这里用`<consructor-arg></constructor-arg>`标签，他的意思就是调用构造器来注入参数，我们把它挂在id="user"的bean节点下，代码如下

```xml
<bean id="user" class="com.xxn.beans.User">
		<constructor-arg name = "username" value = "tom"></constructor-arg>
		<constructor-arg name = "password" value = "123"></constructor-arg>
		<constructor-arg name = "order" ref = "order"></constructor-arg>
</bean> 
```
通过这个配置文件Spring可以创建一个User对象，并且有初始值，不需要我们在手动`new`一个order给他了


#### 5.测试代码

这里测试代码如下，我们创建一个`ApplicationContext`对象ctx，通过`ClassPathXmlApplicationContext("applicationContext.xml")`方法来加载xml配置文件

通过调用ctx的`getBean("user")`方法并传入“user”这个ID，Spring容器就会创建出user对象，这里的user需要亘配置文件的id相同，当然你可以打印出他的属性来检查一下属性有没有被正确的设置

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import com.xxn.beans.User;

public class test01 {
	public static void main(String[] args) {
		ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
		User user = (User) ctx.getBean("user");
	}
}
```


## 总结

这里我们使用了Spring的IoC，说一些我的理解吧，首先我是一个乱七八糟什么都想学的人，当然每一门技术都学的稀烂哈哈，在这里我会比较一些其他语言的web开发，来说明我对于IoC的理解；

首先是借助c#说明，曾经接触过c#游戏服务器，是使用UDP协议来做多人对战的状态同步，这里给我的感觉是，开发角度上我所使用的工具比较开放，你需要定义每个UDP请求的请求以及响应类型（OperationCode）、同样因为UDP是双向的你需要额外定义服务器事件类型（EventCode），这样客户端接到服务器主动发送过来的消息不至于懵逼，服务器接受解析出UDP请求的消息以后要根据OperationCode来了解这个请求的在做什么，比如OperationCode== 1 就是通知服务器我加入场景了，OperationCode== 2 是我在移动哦，请求里携带了我的位置参数，一会你别忘了把这个信息同步给房间中其他人

了解请求后web服务器转发请求给对应的处理类，这里这个处理类有很多，每个处理类都需要在工程启动的时候初始化，并且我把他们保存在了一个容器中，为了后续的调用，这里初始化过程很繁琐；而在Spring中他会帮你干掉这个过程，所以在写C#的时候我满脑子都在惦记着Spring

其次是nodejs的后端开发，nodejs整体用下来的感受就是，我不知道该如何把工程拆分为模块，似乎用这个开发web的流程就是接受http请求，一层层的用中间件处理过滤，当然所有的router也挂在这条链上，若某个router根据规则匹配上url，那么这个请求就交给他处理，接着我就不知道该如何划分业务逻辑的模块，似乎是因为nodejs的所有API都是异步的，当你在router里面调用另一个模块的函数时，router是不会等它结束的就继续往下执行。

异步编程非常小巧灵活，但造就一个严重的问题--大名鼎鼎的回调地狱，如果接下来逻辑需要上面一个异步func返回的结果，正常情况下是拿不到的，因为异步导致这个逻辑与func几乎同时进行，对于这个逻辑而言当他执行时func的返回结果还是null；最原始的解决方案就是回调函数，即你把接下来所有的逻辑写入func的回调函数中，这个函数通常是func的最后一个参数，意思是当这个异步func执行完成后执行回调函数，由于在javaScript中你总是能见到这样的func，所以会出现一个func套另一个func套另一个func....这样就导致你的代码不是竖着往下走的，而是呈套娃状横着走；问题就麻烦了，假设你需要对调两段代码的顺序，非异步很容易做到，回调地狱此时就真的是地狱了，你需要把它....算了我不想描述，似乎到了这里划分函数跟模块不是开发能决定的了，而是取决于这个函数什么时候会出现异步的API，一旦出现异步，你就需要在这里断开逻辑，把后面的逻辑封装成函数当作回调赋值给这个函数。

出现这个现象的原因是javascript是单线程的，由一个事件循环来完成所有的微任务，换句话说，函数的调用不再是通过栈了，而是所有任务一股脑的丢进一个循环中，这样造成了异步函数之间难以同步，更严重的是异常处理也难以进行，好在我们有了Promise以及async&awite，它们可以让你的代码以几乎同步的方式来完成异步的任务

最后我参照了Controller+Service+Dao这种传统的垂直模型重构了那个nodejs的项目，利用Promise将每个模块都包装成异步函数，最后统一调用顺序，当然比较有趣的是其实并不存在Dao层，其实整个项目采用nodejs作为web服务器只是看中了它的io复用模型天然支持高并发，而且内存占用比Java好看多了，主要目的是通过它来调用核心的C++来处理复杂图形信息，不过代码中因为历史原因有两个模块实现相同功能但是难以重构，这完全得益于js没有类型的概念，任何人拿到一个对象都可以随心所欲的为它添加属性，而不需要付出额外的代价，这里就不细谈了

~~发现写多了，但是不想删了~~

