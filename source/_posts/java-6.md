---
title: JavaWeb笔记-SpringBoot D3
date: 2020-07-06 14:46:13
categories:
- [Java开发]
tags:
- [SpringBoot]
---

该笔记包含以下内容：SpringBoot整合Mybatis、Mybatis XML映射文件编写、Mybatis插件

<!--more-->

## Mybatis简介
MyBatis是一个优秀的持久层框架，它对jdbc的操作数据库的过程进行封装，使开发者只需要关注SQL本身，而不需要花费精力去处理例如注册驱动、创建connection、创建statement、手动设置参数、结果集检索等jdbc繁杂的过程代码。

## SpringBoot整合Mybatis
### 引入依赖
可以使用SpringBoot的stater安装Mybatis的依赖

或者直接加入如下依赖
```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.0.1</version>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

### 编写Mybatis配置
applicaiton.properties
```
spring.datasource.url=jdbc:mysql://localhost:3306/test?characterEncoding=utf8&useSSL=false&serverTimezone=UTC
##数据库用户名
spring.datasource.username=root
##数据库密码
spring.datasource.password=123456

# 用来自动补全实体类的包名
mybatis.type-aliases-package=com.xxn.springboot.pojo
# 对应的sql映射
mybatis.mapper-locations=classpath:mybatis/mapper/*.xml
#MyBatis显示SQL
logging.level.com.masxxnhibing.springboot.mapper=debug
```
其中`classpath:`是指`src/main/resources`路径

### Mybatis实例

创建Account类
``` java
public class Account {
	private int id;
	private String loginName;
	private String password;
	private String nickName;
	private int age;
	private String location;
	private int banlance;
	public int getId() {
```

创建AccounterMapper
```java
import org.apache.ibatis.annotations.Mapper;

@Mapper
public interface AccountMapper {
    List<Account> findAll();
	void save(Account account);
}
```

编写xml文件，实现方法的实体
创建AccountMapper.xml
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.xxn.springboot.mapper.AccountMapper">
   
   <resultMap type="com.xxn.springboot.mapper.Account" id="BaseResultMap">
   	<result column="login_name" property="loginName"/>
   	<result column="password" property="password"/>
   </resultMap>
   
   
    <insert id="save" parameterType="Account">
        INSERT INTO account(login_name,password)
        VALUES
        (
        #{loginName},#{password}
        )
    </insert>
    
    <select id="findAll" resultMap="BaseResultMap">
        select * from account
    </select>
</mapper>
```

在Service层使用Mapper
```java
@Service
public class AccountService {

	@Autowired
	AccountMapper mapper;
	
	public void add() {	
		Account account = new Account();
		account.setAge(19);
		account.setLocation("beijing");
		account.setLoginName("xiaoming");
		account.setPassword("123");
		mapper.add(account);
	}
}
```

## Mybatis XML映射文件编写

该部分是整理之前的笔记

### 简单的CRUD
```xml
<mapper namespace="com.qut.mybatis.dao.EmployeeMapper">
 
  <select id="getEmpId" resultType="employee" >
    select * from tbl_employee where id = #{id}
  </select>
  
  <insert id="addEmp" parameterType = "com.qut.mybatis.bean.Employee" useGeneratedKeys = "true" keyProperty = "id">
  	insert into tbl_employee(lastName,email,gender) 
  	values(#{lastName},#{email},#{gender})
  </insert>
  
  <update id = "updateEmp">
   update tbl_employee
   set lastName = #{lastName},email = #{email},gender = #{gender}
   where id = #{id}
  </update>
  
  <delete id = "deleteEmpById">
  delete from tbl_employee where id = #{id}
  </delete>
  
</mapper>
```
### insert获取自增、非自增主键

insert获取自增主键
```xml
<insert id="addEmp" parameterType = "com.qut.mybatis.bean.Employee" useGeneratedKeys = "true" keyProperty = "id">
      insert into tbl_employee(lastName,email,gender) 
      values(#{lastName},#{email},#{gender})
  </insert>
```
keyProperty属性指获取主键赋值到employee对象的id属性中

这里我们使用的是oracle数据库的序列生成主键
```xml
  <insert id="addEmp_oracle_before" parameterType = "com.qut.mybatis.bean.Employee">

         <selectKey keyProperty = "id" order= "BEFORE" resultType = "Integer">

                  select EMPLOYEE_SEQ.nextval from dual

         </selectKey> 

        insert into tbl_employee(id,lastName,email,gender)  values(#{id},#{lastName},#{email},#{gender})

  </insert>
```
### Mybatis处理输入参数
1）单个参数：mybatis不会做特殊处理，使用 #{参数名}取出参数；即使参数名不对应也无所谓。

2）多个参数：mybatis对于多个参数进行处理，自动将多个参数封装成一个map，#{}就是从map中取值。#{param1}、#{param2}

我们推荐使用使用命名参数来明确指定封装成map的key，在xml文件中可以使用#{id}来获取参数值，方式如下：
```java
 public Employee getTmpByIdAndLastName(@Param("id") Integer id,@Param("lastName") String lastName);
```

若传入非常多的参数

- 如果多个参数是业务逻辑的数据模型，直接传递pojo对象，#{属性名} 是pojo的属性名。

- 如果多个参数不是业务逻辑的数据模型
    - 如果这些参数不经常一起出现，为了方便可以传入map，#{属性名} 取出map中对应的值
        ```java
         public Employee getTmpByMap(Map<String,Integer> map)
        ```
    - 如果这些参数经常使用，不是业务逻辑的数据模型，推荐创建一个TO(Transfer Object)类封装参数

### 获取输入参数
常用`#{}` 与 `${}`操作符

区别：`#{}` 以预编译的形式将参数设置到语句中，`${}`取出的值后直接拼装在sql语句中，会有安全问题不能防止sql注入，大多数情况我们使用`#{}` 取得参数的值，但是某些情况我们需要使用`${}`去获取参数的值

分表操作：select * from salary_2016   ==> 不能预编译

原生sql不支持占位符的时候就可以使用$进行取值

分表操作：select * from salary_${year}

### 处理返回值
- select返回List，不需要指定返回值类型为List
    ```xml
    public List<Employee> getEmpsByLastName(String lastName);

     <select id = "getEmpsByLastName" resultType = "com.qut.mybatis.bean.Employee">
            select * from tbl_employee where lastName like #{id}
     </select>
    ```

- select返回Map
    ```xml

    //Map key：字段名 value：属性值
    public Map<String ,Object>  getEmpByIdReturnMap(Integer id);

    <select id = "getEmpByIdReturnMap" resultType = "map">
        select * from tbl_employee where id = #{id}
    </select>

    //Map key：id  value：employee对象

    @MapKey("id")//告诉mybatis封装map使用那个属性作为主键
    public Map<Integer,Employee> getEmpByLastNameReturnMap(String lastName);

    <select id = "getEmpByLastNameReturnMap" resultType = "com.qut.mybatis.bean.Employee">
        select * from tbl_employee where lastName like #{id}
    </select>

    ```

- select使用resultMap自定义返回映射规则
    ```xml
    public Employee getEmpById(Integer id);

    <!-- 自定义javaBean的封装规则,type:自定义的java类型,id：-方便引用 -->
    <resultMap type="com.qut.mybatis.bean.Employee" id="MyEmp">
        <!-- 定义主键 -->
        <id column = "id" property = "id"/>
        <result column = "lastName" property = "lastName"/>
        <!--略-->
    </resultMap>

    <select id ="getEmpById" resultMap = "MyEmp">   
        select * from tbl_employee where id = #{id}
    </select>
    ```
### 连接查询
```java
public class Employee {
	private Integer id;
	private String lastName;
	private String email;
	private String gender;
    private Department dept;
    
    setter&getter...
}

public class Department {
    private Integer id;
    private String DepartmentName;
    setter&getter...
}

public interface EmployeeMapperPlus {
	public Employee getEmpAndDept1(Integer id);
	public Employee getEmpAndDept2(Integer id);
}
```

```xml
<!-- 方式1：使用级联属性进行结果的封装 -->
<resultMap type="com.qut.mybatis.bean.Employee" id="MyEmpPlus">
    <id column = "id" property = "id"/>
    <result column = "lastName" property = "lastName"/>
    <result column ="gender" property = "gender"/>
    <result column = "d_id" property = "dept.id"/>
    <result column = "dept_name" property = "dept.DepartmentName"/>
</resultMap>

    <!-- 方式2：使用association指定联合的javabean对象 -->
<resultMap type="com.qut.mybatis.bean.Employee" id="MyEmpAss">
    <id column = "id" property = "id"/>
    <result column = "lastName" property = "lastName"/>
    <result column ="gender" property = "gender"/>
    <association property = "dept" javaType = "com.qut.mybatis.bean.Department">
        <id column = "d_id" property = "id"/>
        <result column = "dept_name" property = "DepartmentName"/>
    </association>
</resultMap>

<!-- 自定义结果集映射规则 -->
<select id = "getEmpAndDept1" resultMap = "MyEmpPlus">
    select * from tbl_employee ,tbl_dept where tbl_employee.d_id = tbl_dept.id AND tbl_employee.id = #{id};
</select>


<!-- 查询employee的同时查出对应的部门 -->
<select id = "getEmpAndDept2" resultMap = "MyEmpAss">
    select * from tbl_employee ,tbl_dept where tbl_employee.d_id = tbl_dept.id AND tbl_employee.id = #{id};
</select>
```
### MyBatis动态SQL标签
- if标签
    ```xml
    <!-- 根据条件查询员工，要求携带哪个字段就查询哪个字段-->
    <select id = "getEmpsByIf" resultType = "com.qut.mybatis.bean.Employee" >
        select * from tbl_employee
        where 1 = 1
        <!-- test 判断表达式(OGNL) ,遇见特殊符号应该去写转义字符-->
        <if test = "id != null">
            id = #{id}
        </if>
        <if test = "lastName != null and lastName != ''">
            and lastName like #{lastName}
        </if>
        <if test = "email != null and email.trim() != ''">
            and email = #{email}
        </if>
        <!-- OGNL会进行字符串与数字的转换 -->
        <if test="gender == 0 or gender == 1">
        and gender = #{gender}
        </if>
    </select>
    ```
- where标签
    ```xml
    <!-- where标签会去掉多余的and or，只能去掉第一个 -->
    <select id = "getEmpsByIf" resultType = "com.qut.mybatis.bean.Employee" >
        select * from tbl_employee
        <where>
        <if test = "id != null"> 
            id = #{id}
        </if>
        <if test = "lastName != null and lastName != ''">
            and lastName like #{lastName}
        </if>
        <if test = "email != null and email.trim() != ''">
            and email = #{email}
        </if>
        <if test="gender == 0 or gender == 1">
        and gender = #{gender}
        </if>
        </where>
    </select>
    ```
- Trim标签 
    ```xml
    <!-- Trim -->
    <select id = "getEmp" resultType = "com.qut.mybatis.bean.Employee">
        select * from tbl_employee
        <!-- 后面多出的and or
        prefix:前缀,给拼接后的字符串加一个前缀
        prefixOverrides：前缀覆盖，去掉前缀
        suffix:后缀，给拼接后的字符串加一个后缀
        suffixOverrides：后缀覆盖，去掉后缀
        -->
        <trim prefix ="where" suffixOverrides = "and">
        <if test = "id != null"> 
            id = #{id} and
        </if>
        <if test = "lastName != null and lastName != ''">
            lastName like #{lastName} and
        </if>
        <if test = "email != null and email.trim() != ''">
            email = #{email} and
        </if>
        <if test="gender == 0 or gender == 1">
        gender = #{gender}
        </if>
        </trim>
    </select>
    ```
- choose标签
    ```xml
    <!-- choose 如果带了id就用id去查、如果带了lastName就用lastName去查 -->
    <select id = "getEmp" resultType = "com.qut.mybatis.bean.Employee">
        select * from tbl_employee 
        <where>
            <choose >
                <when test = "id != null"> id = #{id}</when>
                <when test = "lastName != null"> lastName like #{lastName}</when>
                <when test = "email != null">email  = #{email}</when>
                <otherwise>
                    gender = 0
                </otherwise>
            </choose>
        </where>
    </select>
    ```

- set标签
    ```xml
    <!--set封装修改条件,不需要管多余的逗号问题) -->
    <update id = "updateEmp" >
        update tbl_employee
        <set>
            <if test = "lastName != null">
                lastName = #{lastName},
            </if>
            <if test = "email != null">		
                email = #{emil},
            </if>
            <if test = "gender != null">
                gender = #{gender}
            </if>
        </set>
        <where>
            id = #{id}
        </where>
    </update>
    ```
- foreach标签
    ```xml
    <!-- foreach -->
    <select id = "getEmp" resultType = "com.qut.mybatis.bean.Employee">
        select  * from tbl_employee 
        <!-- 
            collection：指定要遍历的集合
            item：当前遍历的元素
            separator：每个元素之间的分隔符
            open：遍历所有结果拼接一个开始字符
            close：遍历所有结果拼接一个结束字符
            index：遍历list是索引，item就是值；遍历map是key，item就是值
            #{变量名}:取出当前的值 
        -->
        <foreach collection = "ids" item ="item_id" separator = "," open = " where id (" close = ")" index = "">
            #{item_id}
        </foreach>
    </select>

    <!-- foreach批量保存 -->
    <insert id ="addEmps">
        insert into tbl_employee(lastName,email,gender,d_id)
        values
        <foreach collection = "emps" item = "emp" separator = ",">
            (#{emp.lastName},#{emp.email},#{emp.gender},#{emp.dept.id})
        </foreach>	
    </insert>
    ```
    
哇以前记的笔记好烂...
## Mybatis插件
### mybatis-generator
用来根据数据库中的表自动生成Mapper、Xml文件
https://github.com/zouzg/mybatis-generator-gui

### PageHelper分页插件
**依赖**

```xml
<!-- https://mvnrepository.com/artifact/com.github.pagehelper/pagehelper-spring-boot-starter -->
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.2.12</version>
</dependency>

```

**使用**
```java
	public Object findPage(int pageNum, int pageSize) {
		PageHelper.startPage(pageNum, pageSize);
		AccountExample example = new AccountExample();
		return mapper.selectByExample(example );
	}
```



