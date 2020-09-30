---
title: JavaWeb笔记-SpringBoot D4
date: 2020-07-08 15:52:44
categories:
- [Java开发]
tags:
- [SpringBoot]
---

该笔记包含以下内容：SpringBoot整合Servlet&Filter&Listener、SpringBoot文件上传

<!-- more -->
最近在做一个小的单体CRUD Demo，所以没有太多的东西去整理

## SpringBoot整合Servlet&Filter&Listener

### SpringBoot整合Servlet
准备一个Servlet
```java
@WebServlet(name = "myServlet",urlPatterns = "/srv",loadOnStartup = 1)
public class MyServlet extends HttpServlet {

	@Override
	protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doGet(req, resp);
		System.out.println("111");
	}
}
```
在启动类中添加注解
```java
@SpringBootApplication
@ServletComponentScan
public class SpringbootApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringbootApplication.class, args);
	}
}
```

### SpringBoot整合Filter
准备一个Filter

这里copy了一个过滤url的Filter，用来判断用户访问的url是否需要做权限校验

```java
/**
 * 用户权限处理
 * @author Administrator
 *
 */
@Component
@WebFilter(urlPatterns = "/*")
public class AccountFilter implements Filter {

	// 不需要登录的 URI
	private final String[] IGNORE_URI = {"/index","/css/","/js/","/login","/images"};
	
	
	@Override
	public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain)throws IOException, ServletException {
		
		HttpServletRequest request =  (HttpServletRequest)req;
		HttpServletResponse response = (HttpServletResponse)resp;
		
		String uri = request.getRequestURI();
		// 当前访问的URI是不是 在Ignore列表里  
		boolean pass = canPassIgnore(uri);
		if (pass) {
			chain.doFilter(request, response);
			return;
		}
		
		Object account = request.getSession().getAttribute("account");
		if (null == account) {
			response.sendRedirect("/login");
			return;
		}
		chain.doFilter(request, response);	
	}

	private boolean canPassIgnore(String uri) {
        for (String val : IGNORE_URI) {
			if(uri.startsWith(val)) {
				return true;
			};
		}
		return false;
	}

	@Override
	public void init(FilterConfig filterConfig) throws ServletException {
		// 加载 Filter 启动之前需要的资源
		Filter.super.init(filterConfig);
	}
}
```
同样在启动类中添加注解
```java
@SpringBootApplication
@ServletComponentScan
public class SpringbootApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringbootApplication.class, args);
	}
}
```

### SpringBoot整合Listener
准备一个Listener
```java
@WebListener
public class FirstListener implements ServletContextListener{
	@Override
	public void contextInitialized(ServletContextEvent sce) {
		System.out.println("init .. ");
	}
	@Override
	public void contextDestroyed(ServletContextEvent sce) {
		System.out.println("desroyed .. ");
	}
}
```
同样在启动类中添加注解
```java
@SpringBootApplication
@ServletComponentScan
public class SpringbootApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringbootApplication.class, args);
	}
}
```

## SpringBoot文件上传

### 相关配置
application.properties
```
# Spring Boot 1.4.x and 1.5.x
# 配置单个文件最大限制
spring.http.multipart.maxFileSize=200MB 
# 配置请求总数据大小限制
spring.http.multipart.maxRequestSize=200MB

# Spring Boot 2.x
# 配置单个文件最大限制
spring.servlet.multipart.max-file-size = 200MB
# 配置请求总数据大小限制
spring.servlet.multipart.max-request-size = 200MB
```

### html示例
```html
<form action="fileUploadController" method="post" enctype="multipart/form-data">
    上传文件：<input type="file" name="filename"/><br/>
    <input type="submit"/>
</form>
```

### Controller示例
```java
@RequestMapping("/fileUploadController")
public String fileUpload(MultipartFile filename) throws Exception{
    System.out.println(filename.getOriginalFilename());
    filename.transferTo(new File("/"+filename.getOriginalFilename()));
    return "ok";
}
```

通常在使用SpringBoot时，我们会将工程打包为jar文件部署，此时上传文件可能会出现路径问题

有一种解决方案就是我们在application配置文件中设置好静态文件的目录，并将上传后的文件保存到该目录中
```
spring.resources.static-locations=classpath:/META-INF/resources/,classpath:/resources/,classpath:/static/,classpath:/public/,file:c:/upload/
```
```java
/**
    * @param filename
    * @return
    */
@RequestMapping("/fileUploadController")
public String fileUpload (MultipartFile filename) {
    System.out.println("file:" + filename.getOriginalFilename());
    try {      
        File path = new File(ResourceUtils.getURL("classpath:").getPath());
        File upload = new File(path.getAbsolutePath(), "static/upload/");
        filename.transferTo(new File(upload+"/"+filename.getOriginalFilename()));
    } catch (IllegalStateException e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
    } catch (IOException e) {
        // TODO Auto-generated catch block
        e.printStackTrace();
    }
    return "SUCCESS";
}
```

