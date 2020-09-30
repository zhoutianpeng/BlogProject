---
title: JavaWeb笔记-Spring D2
date: 2020-07-03 14:06:54
categories:
- [Java开发]
tags:
- [Spring]
---

该笔记包含以下内容：使用Maven构筑Spring开发环境、Spring IoC、基于XML的常用依赖注入（构造器、属性）、工厂方法、Bean作用域、循环引用

<!-- more -->
## 使用Maven构筑Spring开发环境

### Maven简介

Maven是一个专门管理构筑Java工程的工具，它主要功能是：
- 提供一套标准的项目结构
- 提供一套标准的构筑流程
- 提供一套依赖管理机制

使用Maven管理的Java项目，它的目录结构默认如下：
```
maven-project
├── pom.xml // 项目描述文件，包含工程描述，依赖管理
├── src
│   ├── main
│   │   ├── java //源码目录
│   │   └── resources //资源目录，比如xml配置文件
│   └── test
│       ├── java //测试源码目录
│       └── resources //测试资源
└── target //存放编译结果、打包后文件
```
一般主流的开发环境都有自己的依赖管理平台，比如nodejs开发使用`npm`进行包管理，OSX以及iOS使用`CocoaPods`，Python生态里依赖库管理似乎更加丰富，工作中我使用`Anaconda`多一些，当然它不仅仅是一个依赖管理，更多的是对于py环境的管理

C++的话，我在工作中在项目中使用第三方依赖时，是直接在git中下载对应版本的源代码然后在本地手动编译，多数情况会将代码`build`成动态库使用

Maven中央仓库地址：`https://mvnrepository.com/`

### 创建项目

本次开发环境是使用了Spring官网推荐的STS开发工具，对应的下载地址是
`https://spring.io/tools`，看起来是一个安装了插件的Eclipse，对于SpringBoot开发做了友好的支持，同时STS也提供了VS code的插件，我记得之前的展示页中也有Atom的插件，这次访问发现从展示页中移除了

好的，我们来创建项目，~~这里就不截图了，整理图片太麻烦，~~

- `右击 - new - project` 会出现`select a wizard`面板，我们选择`Maven - Maven Project`点击 `next`

- 接着会出现`New Maven project`面板，在这一步中我们需要勾选`Create a simple project(skip archetype selection)`选项，这样会创建一个基础的Maven模板工程，接着点击`next`

- 此时会出现`Configure project`面板,这里是用来声明工程描述信息的，最终你可以在`pom.xml`这个配置文件中找到这些描述信息，简介一下这些信息
    - Group Id:包名，用来描述工程的分组信息，即当前工程是属于哪个组的，同组内不允许出现两个相同Artifact Id的工程
    - Artifact Id:工程的唯一的标识名
    - Version:工程版本信息
    - Packaging: (jar|war) 工程最后会打包的类型
    - Name: 工程的名称

- 填写完描述信息点击`Finih`，完成工程的创建


### 构建Spring开发环境

**引用Jar文件**

在`pom.xml`添加如下依赖，保存后Maven就自动去中央仓库根据你的依赖下载jar包
```xml
<dependencies>
  
	<!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
	<dependency>
	    <groupId>org.springframework</groupId>
	    <artifactId>spring-context</artifactId>
	    <version>5.2.6.RELEASE</version>
	</dependency>
	
	 <!-- https://mvnrepository.com/artifact/commons-logging/commons-logging -->
	<dependency>
	    <groupId>commons-logging</groupId>
	    <artifactId>commons-logging</artifactId>
	    <version>1.2</version>
	</dependency>
	
	<!-- https://mvnrepository.com/artifact/org.apache.commons/commons-lang3 -->
	<dependency>
	    <groupId>org.apache.commons</groupId>
	    <artifactId>commons-lang3</artifactId>
	    <version>3.9</version>
	</dependency>
	
</dependencies>
```

这里其实并没有把Spring基础的jar文件全部引用，当Maven去下载`spring-context.jar`时会把相关起来的其他jar文件也全部下载，最后依赖的`commons-lang3.jar`中封装了常用的工具，后续会使用它进行调试

**创建配置文件applicationContext.xml**

在`src/main/resources/`目录下创建`applicationContext.xml`配置文件，并粘贴以下内容，该部分在上一篇博文中做过详细解释
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p" xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:context="http://www.springframework.org/schema/context"
xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
</beans>
```

此时基础的Spring环境基本搭建完成


## Spring IoC
Spring Ioc是指控制反转，也就是在应用程序中对象的创建交给容器不是由开发写代码new出来，Spring框架的一个目的是解耦，而IoC是解耦的一种方式，也就是程序中出现的对象都交给Spring去管理

Spring所管理的对象被叫做bean对象，配置文件是Spring进行生产、依赖注入以及分发Bean的蓝图，在配置文件中写入一个`<bean>`就是在配置一个bean对象，id是bean的唯一标识，class是对应类的包路径
```xml
<bean id="user" class="com.xxn.User"></bean>
```
为了方便对于bean的管理我们可以创建多个配置文件，用来配置不同类型的bean，在加载过程中，可以让Spring只加载一个配置，而这个配置文件去引用其他的配置文件，比如如下代码就是引用一个外部的配置文件
```
<import resource="application-service.xml"/>
```

## 基于XML的常用依赖注入

依赖注入的意思是，Spring可以为创建的bean对象设置属性，也就是为他们的成员变量赋值，该小节记录了基于XML的常用注入方式，实际在开发过程中我们很少去使用XML方式注入，更多的情况是使用注解的方式自动注入

这里介绍构造器注入、属性注入

还是回到上篇博客的问题中，我们要继续解决那个电商项目，现在让我们把之前定义的`User`类扩展一下，我们为他添加多种类型的成员变量，以存储更多的信息，扩展后的代码如下

```java
public class User {
	public String username;
	public String password;
	public String[] address;
	public List<String> orders;
	public Set<String> favours;
	public Map<Integer,String> orderStatus;
	
	public User() {
		super();
	}
	
	public User(String username, String password, String[] address, List<String> orders, Set<String> favours,
			Map<Integer, String> orderStatus) {
		super();
		this.username = username;
		this.password = password;
		this.address = address;
		this.orders = orders;
		this.favours = favours;
		this.orderStatus = orderStatus;
	}

    getter && setter ...

```

这个为了稍微简化，他的成员变量存储的都是基础类型或者是集合类型

### 基于构造器的依赖注入
构造器注入在上一篇中已经介绍了，是使用`<consructor-arg></constructor-arg>`来注入属性，就不做演示了

需要注意的是构造器注入就是Spring通过反射来调用类的带参数构造器，在配置文件中指定参数时可有以一下几种方法
- 指定参数名称：
```
<constructor-arg name="username"  value="tom"></constructor-arg>
```

- 指定参数类型：
```
<constructor-arg type="String"  value="tom"></constructor-arg>
```
- 指定索引：
```
<constructor-arg index="0" value="tom"/>
```

总而言之就是不常用，所以这里敷衍一下😂


### 基于属性的依赖注入
属性注入是Spring通过反射调用对应成员变量的setter方法实现的，例如我们希望为User的username、password注入属性可以用如下方式：

```xml
<bean name = "user" class="com.xxn.spring01.User">
    <property name="username" value="tom"></property>
    <property name="password" value="123456"></property>
</bean>
```

若为集合类型成员变量注入则用以下方式：
```xml
<property name="address">
    <array>
        <value>北京市XX区XX小区XX号</value>
        <value>上海市XX区XX小区XX号</value>
        <value>天津市XX区XX小区XX号</value>
    </array>
</property>

<property name="orders">
    <list>
        <value>洗手液</value>
        <value>口罩</value>
        <value>图书</value>
    </list>
</property>

<property name="favours">
    <set>
        <value>洗手液</value>
        <value>口罩</value>
        <value>图书</value>
    </set>
</property>

<property name="orderStatus">
    <map>
        <entry key = "1001" value="已签收"></entry>
        <entry key = "1002" value="已发货"></entry>
        <entry key = "1003" value="未发货"></entry>
    </map>
</property>
```
此时可以测试效果
```java
ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");

User user = (User)ctx.getBean("user");

System.out.print(ToStringBuilder.reflectionToString(user, ToStringStyle.MULTI_LINE_STYLE));//ToStringBuilder是之前引入的lang3中工具类，可以直接将对象以字符串形式打印出来
```

运行结果
```
com.xxn.spring01.User@932bc4a[
  username=tom
  password=123456
  address={北京市XX区XX小区XX号,上海市XX区XX小区XX号,天津市XX区XX小区XX号}
  orders=[洗手液, 口罩, 图书]
  favours=[洗手液, 口罩, 图书]
  orderStatus={1001=已签收, 1002=已发货, 1003=未发货}
]
```

## 工厂方式创建Bean

之前我们在配置文件中编写bean时是关联到了一个类上，事实上Spring允许我们关联到一个方法中，准确的说是关联到一个工厂方法中，这个方法会根据条件返回一个对象

例如此时我们电商系统设计了一些货物类，这些货物类要拥有相同的一些方法，并我们会通过工厂方法构造它们的实例，所以首先我们创建了一个名为Goods的接口，如下：

```java
public interface Goods {
	public String getName();
	public String getPrice();
}
```

接着创建第一个接口实现类Book
```java
public class Book implements Goods {
	
	public String name = "游戏引擎架构";
	public String price = "$128.00";

	public String getName() {
		// TODO Auto-generated method stub
		return name;
	}

	public String getPrice() {
		// TODO Auto-generated method stub
		return price;
	}
}
```

然后编写工厂类来创建对象，此时根据工厂方法是否是静态的，我们有两种不同的实现
```java
//动态工厂
public class GoodsDynamicFactory {
	public Goods getGoods(String name) throws Exception {
		if(name.equals("Book")) {
			return new Book();
		}
		else {
			throw new Exception("can not create " + name);
		}
	}
}

//静态工厂
public class GoodsStaticFactory {
	public static Goods getGoods(String name) throws Exception {
		if(name.equals("Book")) {
			return new Book();
		}
		else {
			throw new Exception("can not create " + name);
		}
	}
}
```

最后来编写配置文件
```xml
<!-- Dynamic factory -->
<bean name = "goodsFactory" class = "com.xxn.spring01.GoodsDynamicFactory"></bean>

<bean name = "goods" factory-bean = "goodsFactory" factory-method="getGoods">
	<constructor-arg value="Book"></constructor-arg>
</bean>

<!-- Static factory -->
<bean name = "goodsStatic" class = "com.xxn.spring01.GoodsStaticFactory" factory-method="getGoods">
	<constructor-arg value="Book"></constructor-arg>
</bean>
```
使用起来其实很容易理解，动态的工厂函数是要通过对象来调用的，所以我们需要先把这个工厂类也注册为bean，然后再注册货物的bean，此时货物bean就不再指向某一个类，而是指向工厂方法，至于传递参数是通过`<constructor-arg></constructor-arg>`而不是`<property></property>`，因为property底层就是反射调用了某一个属性的setter方法，所以他它就是用来为属性赋值的

我们来测试一下代码
```java	
//Dynamic factory
Book bookD = (Book)ctx.getBean("goods");
System.out.print(ToStringBuilder.reflectionToString(bookD, ToStringStyle.MULTI_LINE_STYLE));

//Static factory
Book bookS = (Book)ctx.getBean("goodsStatic");
System.out.print(ToStringBuilder.reflectionToString(bookS, ToStringStyle.MULTI_LINE_STYLE));
```

运行结果
```
com.xxn.spring01.Book@10163d6[
  name=游戏引擎架构
  price=$128.00
]
com.xxn.spring01.Book@2dde1bff[
  name=游戏引擎架构
  price=$128.00
]
```

## Bean作用域
Bean的作用域是指，被Spring创建的bean对象的生命周期，Spring为bean提供了六种作用域，分别是`singleton`,`prototype`,`websocket`,`request`,`session`,`application`，我们可以在配置文件`<bean>`标签中设置`scope`属性来指定这个bean的作用域

其中singleton是单例模式，也就是自始至终Spring只会创建一个这个bean对象，这个是默认值，prototype是非单例的，每次该bean被注入或者通过getBean被获取时时，Spring都会创建一个新的对象

后面四种作用域他们的使用场景都是在web服务器中，每个bean作用域与对应的生命周期所绑定，比如request作用域，他的生命周期就是指一个用户发送一次http的请求开始，到这一次请求结束为止，也就是这个bean的作用域从时序上来看就是这次请求开始，到请求结束。

session作用域，因为http请求是无状态的，意思就是服务器收到两个http请求他并不能直接知道这两个请求是否是同一个用户发送过来的，这导致的问题是登录操作没啥用，所以为了实现会话追踪，其中一种解决方案就是session，这是在服务器端实现的一种会话追踪技术，简单来说就是在服务器端记录用户的某些信息，以此来识别每次请求的用户，session作用域是从用户发过来的第一个请求开始，结束时间通常不能直接判断，因为浏览器关闭是不会通知服务器的，一般我们判定如果用户在一定时间内没有反应则会话结束，tomcat默认失效时间是20min，此时bean在这个作用域内生效；另外一种常用的会话追踪是通过token，先不做介绍了

application作用域好理解，是指服务器一开始执行运行，至到服务器关闭为止

另外后面四种作用域，每个bean作用域与对应的生命周期所绑定，并且只在自己的生命周期内是单例的，例如用户A发送一个http请求，此时在request作用域内，不论bean被注入或是被获取，操作的都是同一对象；而用户A与用户B分别发送相同的请求，在这两个请求线程中bean分别是两个不同对象。


## 循环引用
循环引用比较容易理解，就是A里面包含B类型的成员变量，B里又包含A类型的成员变量，这样的结构

此时如果你的代码中不幸出现这种结构，直接看结论

- 若所有循环引用的bean都是通过构造器注入，无论如何Spring都不能成功创建

- 若所有循环引用的bean都是通过属性注入
	- 若所有的bean都是singlton，Spring可以成功创建bean
	- 若所有的bean都是prototype，Spring不能成功创建bean
	- 若bean既有singlton又有prototype，当首先获取singlton的bean时会成功，否则会失败

原理的话后续在看Spring源码时会研究补充，先在这埋个坑




