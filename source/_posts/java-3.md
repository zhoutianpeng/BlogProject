---
title: JavaWeb笔记-Spring D3
date: 2020-07-04 11:22:12
categories:
- [Java开发]
tags:
- [Spring]
---

该笔记包含以下内容：自动注入、Spring注解、AOP与代理

<!-- more -->

## 自动注入

自动注入是指，若类A引用类B作为成员变量，可以在配置文件中A的bean标签上加入autowire属性，这样Spring就会自动的为A中B类型成员变量赋值，之前我们不使用autowire时是通过`<property name = "b" ref="b">`来初始化的，使用autowire就可以省去这个配置；autowire的可选值有两个分别是`byName`,`byType`；

`byName`是通过名称的方式自动注入，即Spring在初始化A类型的bean时，发现它有一个B类型的成员变量，假设其标识符是b，那么Spring就会去容器中寻找id为b的bean，找到后赋值给A中的b；

`byType`是按照类型的方式自动注入，还是上述的环境，那么Spring就会去容器中寻找类型为B的bean，找到后赋值给A中的b，此时注意Spring容器中不能出现两个类型相同的bean,否则就会报错，也就是通常这个被注入的bean应该是单例的；

```xml
<bean id="user" class="com.xxn.User" autowire="byName">
</bean>

<bean id="person" class="com.xxn.Person"  autowire="byType">
</bean>

<bean id="order" class="com.xxn.Order">
</bean>
```

我们可以在每一个bean标签中单独设置autowire，也可以全局设置autowire，全局设置后Spring容器中所有的bean都会自动注入

全局注入是通过在根节点beans中添加default-autowire属性设置的

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:p="http://www.springframework.org/schema/p"
	xsi:schemaLocation="http://www.springframework.org/schema/beans           http://www.springframework.org/schema/beans/spring-beans.xsd"
	default-autowire="byType">

</beans>
```

## Spring注解

Spring可以通过配置文件来实现对于bean的配置，该方式比较繁琐复杂；除此之外，Spring还提供了基于注解的配置方式，通常在项目开发中注解使用的频率较高

### annotation注解的引入
使用注解，我们可以不在applicationContext.xml文件中做bean的配置，但需要引入annotation的注解

我们需要引入aop.jar文件，然后在applicationContext.xml中如入下配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:p="http://www.springframework.org/schema/p" 
xmlns:mvc="http://www.springframework.org/schema/mvc" 
xmlns:context="http://www.springframework.org/schema/context"
xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

	<context:component-scan base-package="com.xxn.spring01"></context:component-scan>
</beans>
```

此时beans中的属性增加了，主要是增加了对于context的使用约束，其次是配置了context:component-scan这个扫描器标签，它会使Spring去扫描目标包里面的类找到对应的注解，之后注册到Spring容器中；通常Spring中的注解有三种作用，一是注册bean，二是注入值，三是配合AOP做增强用的功能性注解；

### 常用注解
**@Component**

注解在类上，作用是来注册bean的，相当于在配置文件中写入`<bean></bean>`标签

- 在类上添加`@Component`，默认会将首字母小写的类名作为Id注册到Spring容器中
- 可以手动指定Id，`@Component("controller")`
- `@Controller`、`@Service`、`@Repository`是作用与`@Component`相同，只是其语义不同

**@Scope**

注解在类上，作用是来修改bean的作用域，相当于在配置文件设置bean的scope属性

**@Value**

注解在属性上，作用是来为bean注入值，相当于在配置文件设置bean的`<property></property>`
- 通过注解`@Value`注入值时，不需要为属性设置getter、setter方法

**@Autowired**

注解在属性上，作用是来为bean注入对象引用，相当于在配置文件设置bean的autowire属性
- `@Autowired`注解默认是`byType`取值
- 通过在与`@Qualifier("name")`一起使用，可以实现`byName`的效果

## AOP与动态代理
AOP这里并不做过多理解，在后续整理事务的博客中会详细解释AOP的意义，这里主要说一下AOP的实现原理

Spring框架是通过动态代理来实现AOP的，而实现动态代理有两种方式，一种是JDk的动态代理，另一种是CGLIB的动态代理，Spring优先支持JDK的实现，但JDK动态代理依赖接口实现，如果没有接口，Spring会使用CGLIB实现动态代理

### 代理模式

代理模式是指：给某一个对象提供一个代理，并由代理对象来控制对真实对象的访问，代理模式是一种结构型设计模式

代理模式需要包括如下三种角色：
- **Subject**：抽象主题角色

- **RealSubject**：真实主题角色

- **Proxy**：代理主题角色

<img src="/img/java/SpringD3-1.jpg" width="90%" style="text-align:center">

若根据**字节码的创建**时机来分类，代理模式可以分为静态代理和动态代理：
- 静态代理是指程序运行前就已经存在代理类（Proxy代理主题角色）的字节码文件，代理类和真实主题角色的关系在运行前就确定了
- 动态代理是指代理类（Proxy代理主题角色）的字节码在程序运行期间，由JVM根据反射等机制动态的生成，所以在运行前并不存在代理类的字节码文件

### 静态代理的实现
Subject抽象主题角色
```java
public interface UserService {

    public void select();   
    
    public void update();
}
```

RealSubject真实主题角色
```java
public class UserServiceImpl implements UserService {
	public void select() {
		System.out.println("select invoke");
	}

	public void update() {
		System.out.println("update invoke");
	}
}
```
Proxy代理主题角色
```java
public class UserServiceProxy implements UserService {
	
	UserService target;
	
	public UserServiceProxy(UserService target){
		this.target = target;
	}

	public void select() {
		 before();
		 target.select();
		 after();
	}

	public void update() {
		 before();
		 target.update();
		 after();
	}
	
	public void before() {
		System.out.println("before invoke()");
	}
	
	public void after() {
		System.out.println("after invoke()");
	}
}
```

client测试代码
```java
public static void main(String[] args) {
	// 静态代理
	UserService userServiceImpl = new UserServiceImpl();
	
	UserService proxy = new UserServiceProxy(userServiceImpl);
	
	proxy.update();
	
	proxy.select();
}
```

静态代理实现了代理的设计模式，但是缺陷非常明显就是通过硬编码的方式实现的，每次代理一个目标类就需要写一个与它同接口的代理类，而且不易维护；

### 动态代理的实现

首先明确的是类是可以动态生成的，这与JVM的类加载机制有关

JVM类加载过程主要分为五个阶段：加载、验证、准备、解析、初始化。其中加载阶段需要完成以下3件事情：

- 通过一个类的全限定名来获取定义此类的二进制字节流
- 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构
- 在内存中生成一个代表这个类的 java.lang.Class 对象，作为方法区这个类的各种数据访问入口

其中**获取定义此类的二进制字节流**这个步骤比较灵活，这个字节码既可以来自于.class文件，也可以是来自于运行时产生的数据

> 这部分的具体实现后续会专门详细讲解，主要涉及到字节码操作技术、ASM等等

为了让生成的代理类与真实主题角色保持一致性，通常有如下两种方式：
- 通过实现接口的方式：JDK动态代理
- 通过继承类的方式：CGLIB动态代理

**JDK动态代理实现**
```java
//Subject抽象主题角色
public interface UserService {

    public void select();   
    
    public void update();
}

//RealSubject真实主题角色
public class UserServiceImpl implements UserService {
	public void select() {
		System.out.println("select invoke");
	}

	public void update() {
		System.out.println("update invoke");
	}
}
```

这里的Proxy代理主题角色是动态生成的，但是Proxy对RealSubject的调用逻辑需要我们来定义
```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

public class LogHandler implements InvocationHandler {
	
	Object target;
	
	public LogHandler(Object target) {
		this.target = target;
	}
	
	private void before() {
		System.out.println("before invoke()");
	}
	
	private void after() {
		System.out.println("after invoke()");
	}

	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		before();
		Object result = method.invoke(target, args);
		after();
		return result;
	}
}
```

client测试代码
```java
public static void main(String[] args) {
	//JDK动态代理
	//RealSubject
	UserServiceImpl userServiceImpl = new UserServiceImpl();

	//获取RealSubject的ClassLoader，这是为了构造proxy对象
	ClassLoader classLoader = userServiceImpl.getClass().getClassLoader();

	//获取RealSubject所有的接口
	Class[] interfaces = userServiceImpl.getClass().getInterfaces();
	
	//创建一个调用请求处理器logHandler
	//这个logHandler会传递给最终生成的proxy对象
	//logHandler负责处理所有的proxy对象上的方法调用
	InvocationHandler logHandler = new LogHandler(userServiceImpl);
	
	//根据上面提供的信息，创建proxy对象
	//JDK会通过根据传入的参数信息动态地在内存中创建和.class 文件等同的字节码
	//然后根据相应的字节码转换成对应的class，
	//然后调用newInstance()创建代理实例
	UserService proxy = (UserService) Proxy.newProxyInstance(classLoader, interfaces, logHandler);
	
	//调用代理的方法
	proxy.select();
	proxy.update();
}
```

**CGlib动态代理实现**
```java
//RealSubject真实主题角色
public class UserDao {
    public void select() {
        System.out.println("UserDao 查询 selectById");
    }
    public void update() {
        System.out.println("UserDao 更新 update");
    }
}
```
```java
import java.lang.reflect.Method;
import java.util.Date;

public class daoMethodInterceptor implements MethodInterceptor {
    /**
     * @param object 表示要进行增强的对象
     * @param method 表示拦截的方法
     * @param objects 数组表示参数列表，基本数据类型需要传入其包装类型
     * @param methodProxy 表示对方法的代理，invokeSuper方法表示对被代理对象方法的调用
     * @return 执行结果
     * @throws Throwable
     */
    @Override
    public Object intercept(Object object, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        before();
        Object result = methodProxy.invokeSuper(object, objects);   // 注意这里是调用 invokeSuper 而不是 invoke，否则死循环，methodProxy.invokesuper执行的是原始类的方法，method.invoke执行的是子类的方法
        after();
        return result;
    }
    private void before() {
        System.out.println(String.format("log start time [%s] ", new Date()));
    }
    private void after() {
        System.out.println(String.format("log end time [%s] ", new Date()));
    }
}
```
测试
```java
import net.sf.cglib.proxy.Enhancer;

public class CglibTest {
    public static void main(String[] args) {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(UserDao.class);  // 设置超类，cglib是通过继承来实现的
        enhancer.setCallback(new daoMethodInterceptor());

        UserDao dao = (UserDao)enhancer.create();   // 创建代理类
        dao.update();
        dao.select();
    }
}
```













