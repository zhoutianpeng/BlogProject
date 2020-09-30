---
title:  JavaWeb笔记-SpringBoot D2
date: 2020-07-06 14:46:13
categories:
- [Java开发]
tags:
- [SpringBoot]
---

笔记包含以下内容：SpringMVC使用

<!--more-->

## SpringMVC使用

这部分是我整理了之前学习的笔记，并且因为SpringBoot内部整合了SpringMVC，所以不再整理如何搭建配置SpringMVC环境

当时学习顺序是按照SpringMVC搭建、SpringMVC注解、SpringMVC模型数据、Spring Pojo入参、Spring视图&视图解析器以及自定义视图这个顺序学习的，这里只整理常用的注解，后续会补充

### SpringMVC实例

最基础的Controller实例，这里的实例是配置视图解析器最后返回success.jsp文件，若在SpringBoot中使用JSP需要做单独配置
```java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
//请求处理器
public class Helloworld {
	/*
	 *使用@RequestMapping注解来映射请求的url
	 *返回时会通过视图解析器解析为实际的物理视图，对于org.springframework.web.servlet.view.InternalResourceViewResolver
	 *这个解析器会做如下解析
	 *prefix + return + suffix
	 *这样得到实际的物理视图，然后会做转发
	 **/
	@RequestMapping("/helloworld")
	public String hello() {
		System.out.print("hello world");
		return "/success";
	}
}
```

### SpringMVC常用注解

**@RequestMapping**

- 该注解可以修饰类和方法
    - 修饰类：提供初步的请求映射信息，相对于web应用的根路径
    - 修饰方法：进一步的URL信息

- @RequestMapping的value、method、params以及headers分别对应请求的URL、请求方法、请求参数、以及请求头
	- `@RequestMapping(value = "/testMethod",method = RequestMethod.POST)`指定使用POST方式提交
	- params和headers支持简单的表达式
	`@RequestMapping(value = "/testParams",params = {"username","age != 10"})`表示必须有username参数，并且age不可以是10
- URL支持Ant风格的通配符
	?：匹配文件名中的一个字符
	*：匹配文件名中任意字符
	**：匹配多层路径


**@PathVariable**
- 带有占位符的URL是spring3.0增加的功能，REST
- 通过@PathVariable可以将URL占位符参数绑定到控制器方法的入参中
    ```java
    @RequestMapping("/delete/{id}")
	public String testMethod() {
		System.out.print(@PathVariable("id") Integer id);
		System.out.print("params:"+id );
		return SUCCESS;
	}
    ```

**@RequestParam**
- 绑定请求参数
    - value：值即请求参数的参数名
    - required：是否为必须，默认为true，int等基础类型设置这个参数无效
    - defaultValue：请求参数的默认值""
    ```java
        @RequestMapping(value = "/testRequestParam")
        public String testRequestParam(@RequestParam(value = "username") String nm,@RequestParam(value = "age",required = false,defaultValue = "0") Integer age) {
            System.out.print(nm+age);
            return SUCCESS;
        }
    ```
**@RequestHeader**
- 绑定请求头，使用方式与@RequestParam类似
    - value：值即请求头的参数名
    - required：是否为必须，默认为true，int等基础类型设置这个参数无效
    - defaultValue：请求头的默认值""

**@CookieValue**
- 获取Cookie值，使用方式与@RequestParam类似


**@ResponseBody**
- 作用在方法中，该注解会将方法返回的对象，根据HTTP Request Header的Accept的内容转换为指定格式后，写入到Response对象的body数据区。
- 常用在返回json数据说这xml数据


最后SpringMVC允许使用使用Servlet的API做为参数
```
 HttpServletRequest
 HttpServletResponse
 HttpSession
 java.security.Principal
 Locale
 InputStream
 OutputStream
 Reader
 Writer
```
例如
```java
@RequsetMapping("/testServletAPI")
public String testPojo(HttpServletRequest request,HttpServletResponse response){
    return SUCCESS;
}
```

###  模型数据
springMVC提供以下方法输出模型数据
`ModelAndView`、`Map & Model`、`@SessionAttributes`、`@ModelAttribute`

**ModelAndView**

可以作为方法的返回值，该对象同时存储视图信息和数据信息，springMVC会把ModelAndView中Model的数据放到request域对象中
```java
@RequestMapping(value = "/testModelAndView")
public ModelAndView testModelAndView() {
    ModelAndView mv = new ModelAndView(SUCCESS);
    //添加模型数据到ModelAndView
    mv.addObject("time",new Date());
    return mv;
}
```

**Map & Model**

方法可以添加map类型的参数,也可以是Model类型或者ModelAndMap类型
在请求域中获取map中的参数
```java
@RequestMapping(value = "/testMap")
public String testMap(Map<String,Object> map) {
    map.put("names",Arrays.asList("tom","jerry","Mike"));
    return SUCCESS;
}
```

**@SessionAttributes**

该注解注释到Controller上，将请求域中的数据放到会话域中

通过属性名指定需要放到会话中的属性

通过types指定哪些对象类型的模型属性放到会话中 

@SessionAttributes(value = {"user"},types = {String.class})

**@ModelAttribute**

> 这一节当时笔记记得不是特别清楚，需要重新整理







