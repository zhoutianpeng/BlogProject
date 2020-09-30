---
title: JavaWeb笔记-SpringBoot D1
date: 2020-07-05 15:22:24
categories:
- [Java开发]
tags:
- [SpringBoot]
---

该笔记包含以下内容：SpringBoot简介、创建SpringBoot工程、SpringBoot搭建MVC实例

<!--more-->

## SpringBoot简介

惭愧惭愧昨天是第一次接触SpringBoot，距离上次我碰JavaWeb的项目已经快两年半了，昨天体验了一下SpringBoot的使用，使用非常顺手，尤其是在项目创建和run的部分。在学习Spring时，框架的搭建对我来说一直是个挑战，而SpringBoot贴心的帮你封装好，并且把复杂的配置统统精简整合，而且在SpringBoot中你可以看见main函数了😂。

好了这里引用一下资料

SpringBoot主要解决的是在微服务架构下简化配置、前后端分离以及快速开发
- 提供了快速启动入门
- 开箱即用，提供默认配置
- 内嵌容器化web项目
- 没有冗余的代码生成以及xml配置要求

感觉SpringBoot这个内嵌容器化web项目的设计非常贴心，也更加适合微服务；在使用Spring时最终项目是打包成war文件丢入web服务器中运行，而SpringBoot直接在项目里塞了一个web服务器，这样在启动方式变为直接用`java`命令运行打出来的jar包就可以了（虽然这个jar包比较大），可以通过写脚本批量的开关服务，运维童鞋的福利呀

## 创建SpringBoot工程

创建SpringBoot工程可以与Spring一样用Maven手动引入依赖

这里介绍官方提供的一种更便捷的方式
```
https://start.spring.io/
```

这是一个快速生成工程的工具，在这里你选择配置，填写工程的信息，最后选择好工程的依赖，生成工具就会帮你创建好工程，从这个网站下载最后导入到你的IDE中就可以

<img src="/img/java/springbootD1-1.png" width="90%" style="text-align:center">

这个页面经常会更新，展示的内容也许会发生变化，但是基础的内容是不会改变的，通常来说如果你是在做基于SpringBoot的web项目，那么图中左侧的部分是不需要更改设置的，你只需要填写好工程的信息就可以

接着来加入SpringBoot的依赖，可以点击右侧的`ADD DEPENDENCIES`按钮来选择依赖，这里你只需要加入一个`Spring web`就可以，不需要额外的其他依赖，这是非常爽的，如果后续你还希望添加新的依赖，STS提供了一个与这个页面一样的添加页面供你选择

这里使用的开发工具依旧是STS

在STS中导入刚刚下载的工程 ： `右击 - import... - Maven/Existing Maven Projects`

我们先来观察一下这个工程

可以看到它的`pom.xml`文件中依赖只有如下配置
```xml
<dependencies>
    
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
        <exclusions>
            <exclusion>
                <groupId>org.junit.vintage</groupId>
                <artifactId>junit-vintage-engine</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
	
	</dependencies>
```
`spring-boot-starter-test`是与单元测试相关的依赖

`spring-boot-starter-web`是SpringBoot需要的最基础的依赖，它是对于各种spring、springMVC还有其他框架的整合，过去我们需要手动搭建，而现在SpringBoot索性直接把这部分也封装了，你可以在工程的`Maven Dependencies`目录中看到之前各种熟悉的jar文件

**Spring Boot引入依赖的方式**

打开`pom.xml`，在文件中`右击-Spring-「Edit Starters」`，你可以看到熟悉的url
```
Service url : https://start.spring.io
```
其实本质还是这个面板去刚刚我们下载模板工程的地方拉取依赖，你在这里可以通过模块的形式管理依赖，这里的模块我理解是对于过去的单独jar文件再做一层封装，优点是可以简化依赖的配置过程，另外降低出现多个jar包间接引用同名不同的版本的jar导致依赖冲突问题，这种问题Maven是不会处理的

**Spring Boot启动**

`右击 - Run As - Spring Boot App `

好了介绍完了，下一条...

哈哈，其实启动这里也挺有讲头，但是上面一句就概括了SpringBoot简洁的启动，首先你会发现你不需要配置服务器了，直接run项目就可以了，这是因为SpringBoot内嵌了一个tomcat服务器，你可以用IDE带有的`Spring Boot App`方式启动或者直接找到工程的入口函数以普通的jar文件方式运行也可以，效果是一样的，在控制台打印出这个经典的logo，是就是通知你项目已经启动了

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.3.1.RELEASE)
```

对了👆上面这个logo是可以被替换的，你只需要在`src/main/resources/`目录下创建一个banner.txt文件就可以，最后这个logo会被替换为`banner.txt`里面的内容

另外你还可以通过`右击 - Run As - Maven install`来打包这个工程，最后会被打包成jar包，如果需要部署直接把这个jar包拷贝到服务器上在terminal中输入`java path/to/XX.jar`命令就可以启动这个服务。部署非常的容易，只需要服务器配有jre环境就行，不需要额外的依赖，因为其他的依赖都在这个jar包里面。

**Spring Boot简单配置**

最后就是SpringBoot把复杂的配置都简化了，并且整合到同一个配置文件里面，在`src/main/resources/applicaiton.properies`中，Spring Boot做简化配置其实是为所有的配置项给了一个默认的值，我们可以在这个配置文件中单独设置，比如如下设置

```
#修改默认端口号
server.port = 8090 

#设置默认虚拟路径
server.servlet.context-path=/app
```

## SpringBoot搭建MVC实例

这部分我不会过多的介绍MVC的形式，因为过于经典，并且是属于业务逻辑相关的部分

这里我会记录两种最简单的配置，一种是`SpringBoot + Spring data jpa + JSP`，另一种是`SpringBoot + Spring data jpa + thymeleaf`，我会主要介绍这两种项目的搭建以及简单的业务流程，主要是作为笔记方便未来忘记时回顾，不会过多的聊具体实现，因为我个人是热爱前后端分离的架构，而对于后端的模板引擎这种方式不是特别感兴趣
 
### SpringBoot + Spring data jpa + JSP

**依赖配置**

这里同样你可以用`右击-Spring-「Edit Starters」`添加`Spring data jpa`与`JSP`的依赖

或者直接在pom.xml加入如下配置

```xml
<!--jsp dependency -->
<!-- jstl -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
</dependency>

<!-- jasper -->
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
    <scope>provided</scope>
</dependency>

<!--jpa dependency -->   
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
</dependency>
```
>  在SpringBoot中使用JSP也是需要依赖的，大概是官方不支持使用JSP了吧，我记得JSP似乎是需要转换成Servlet编译成class的，曾将打开看过它的实现大致就是填充数据最后一行行的打印HTML标签，所以对于大量的表格数据这样实现无疑是低效耗时的。

**配置数据源&视图解析器**

在`src/main/resources/applicaiton.properies`中添加如下配置
```properties
spring.datasource.url=jdbc:mysql://localhost:3306/data?characterEncoding=utf8&useSSL=false&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=123456


spring.mvc.view.prefix=/WEB-INF/jsp/
spring.mvc.view.suffix=.jsp
```

**MVC实例**

controller
```java
@Controller
@RequestMapping("/city")
public class MainController {

	@Autowired
	CityService citySrv;
	
	@RequestMapping("/list")
	public String list(Model map) {
		
		List<City> list =  citySrv.findAll();
		map.addAttribute("list", list);
		System.out.println("list.size():" + list.size());
		return "list";
	}
	
	@RequestMapping("list/{id}")
	public String getOne(@PathVariable("id") Integer id,Model model) {
		
		City city = citySrv.findOne(id);
		model.addAttribute("city", city);
		return "list1";
	}
	
}
```

service
```java
@Service
public class CityService {

	@Autowired

	CityRepository cityRepo;
	
	public List<City> findAll() {
		List<City> findAll = cityRepo.findAll();
		return findAll;
	}

	public City findOne(Integer id) {		
		return cityRepo.getOne(id);
	}
}
```

dao
```java
import org.springframework.data.jpa.repository.JpaRepository;

import com.xxn.springBootMVC.entity.City;

public interface CityRepository extends JpaRepository<City, Integer> {

}

```

entity
```java
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Table(name = "city")
public class City {

	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Integer id;
	private String name;
	getter & setter ...	
}

```

最后我们创建`src/main/webapp/WBE_INF/jsp`目录

创建`list.jsp`写入如下代码

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>

<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
    </head>
    <body>
        <table>
            <tr>
                <th>id</th>
                <th>name</th>
            </tr>
            
            <c:forEach items="${list}" var="item">
                <tr>
                    <td>${item.id }</td>
                    <td>${item.name }</td>
                </tr>
            </c:forEach>
        </table>
    </body>
</html>
```

大致说一下流程

首先我们请求`http://localhost:8080/city/list`这个url，服务器接收到Http请求后内部是通过SpringMVC来转发请求，根据路由器配置传递给`MainController`这个控制器，这个controller会进一步根据路由器配置找到list方法来处理请求，并且传递了一个Model对象作为参数，这个Model对象是保存在Spring的上下文(Context）中，目的保存数据为了后续视图解析器可以获取，list调用`CityService`的方法来具体执行业务逻辑

service会调用Dao层的方法访问数据库，这里是使用了jpa实现的，即这个jpa框架会自动将数据库与对象之间创建一个ORM映射，这是根据City类上的注解完成的，我们编写Dao层接口继承了jpa的接口，就可以访问其中封装的方法来访问数据库获取数据，最后这个数据一层层的返回给Controller，Controller写入Model中，return一个字符串；

剩下的工作就是SpringMVC框架的任务了，SpringMVC拿到字符串，通过视图解析器中的配置拼接成文件路径，找到对应的jsp文件，接着填充数据，渲染视图？（我觉得这里叫渲染不太合适，顶多是生成html代码，我理解的渲染是指生成屏幕上的像素点），最后返回给前端


### SpringBoot + Spring data jpa + thymeleaf

这里几乎过程相同首先是引入thymeleaf视图模板的依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

别忘记把JSP的配置删掉

这里业务逻辑的代码没有变化

我们需要加入thymeleaf的视图文件

在`src/main/resources`目录下创建list.html

```html
<table border="1">

	<tr>
		<th>ID</th>
		<th>Name</th>
	</tr>

	<tr th:each="city : ${list}">
		<td th:text = "${city.id}"></td>
		<td th:text = "${city.name}"></td>
	</tr>

</table>

```

后续会专门看一下thymeleaf的文档，但是不会整理笔记

### 小结

我个人是非常喜欢前后端分离的方案，其实感觉自己从一开始接触javaWeb，看到jsp或者其他视图模板，就不是特别喜欢用，因为觉得这样子使用不灵活；那时候还没接触前后端分离，即使这样做东西时候也是习惯于先配置html页面的路由，先访问html页面，然后后续全部的数据都是html页面里运行的js通过ajax向后端获取的，很少使用后端填充数据返回页面的方式开发东西；

因为感觉前端开发的任务量也越来越大，也就是说前端的开发足以作为一个单独的项目去管理了，再者就是前端的依赖真的不少。只要定义好API的格式，前后端分离两个工程可以同时进行，也非常节约时间。



















