# javaNote
ssm框架，springboot等学习笔记
# SpringMCV总结

## SpringMVC的概述



##### MVC模型

**Model模型**：可以认为是领域模型或JavaBean组件（包含数据和行为），不过现在一般都分离开来：Value Object（数据） 和 服务层（行为）。也就是模型提供了**模型数据查询和模型数据的状态更新等功能**，**包括数据和业务**

**View视图：**负责进行模型的展示，一般就是我们见到的用户界面，客户想看到的东西。

**Controller（控制器）**：接收用户请求，委托给模型进行处理（状态改变），处理完毕后把返回的模型数据返回给视图，由视图负责展示。 也就是说控制器做了个调度员的工作。接收和分发请求

**三层架构**

**表现层：** 

也就是我们常说的web层。它负责接收客户端请求，向客户端响应结果，通常客户端使用http协议请求 web 层，web 需要接收 http 请求，完成 http 响应。 

表现层包括展示层和控制层：控制层负责接收请求，展示层负责结果的展示。 

表现层依赖业务层，接收到客户端请求一般会调用业务层进行业务处理，并将处理结果响应给客户端。 

表现层的设计一般都使用 MVC 模型。（MVC 是表现层的设计模型，和其他层没有关系） 

**业务层：** 

也就是我们常说的 service 层。它负责业务逻辑处理，和我们开发项目的需求息息相关。web 层依赖业 务层，但是业务层不依赖 web 层。 

业务层在业务处理时可能会依赖持久层，如果要对数据持久化需要保证事务一致性。（也就是我们说的， 

事务应该放到业务层来控制） 

**持久层：** 

也就是我们是常说的 dao 层。负责数据持久化，包括数据层即数据库和数据访问层，数据库是对数据进行持久化的载体，数据访问层是业务层和持久层交互的接口，业务层需要通过数据访问层将数据持久化到数据库中。通俗的讲，持久层就是和数据库交互，对数据库表进行曾删改查的。 

## 入门项目

```java
1.创建maven工程
2.在Pom.xml写入springmvc所需依赖
3.完善工程结构
4.配置web.xml
5.配置springmvc.xml
```

## 使用 @RequestMapping 映射请求

**映射方式**

```
映射单个URL，映射多个URL，通配符映射（Ant通配符）
/?  匹配任何单字符
/*  匹配任意数量的字符（含 0 个）
/** 匹配任意数量的目录（含 0 个）
```

**映射请求参数**

```
value:映射地址
param:请求参数约束
params= {"username","age!=10"}, headers = { "Accept-Language=en-US,zh;q=0.8" })
表示请求参数必须带username，且年龄不等于10，逗号为与，以及请求头必须为括号中内容
method：请求方式
```

**PathVariable注解**

```
带占位符的注解，可以解析路径中的占位符，可以用来传递参数
在方法的形参表中用PathVariable注解进行绑定占位符中传递的参数
@RequestMapping(value="/testPathVariable/{id}")
public String testPathVariable(@PathVariable("id") Integer id){
System.out.println("testPathVariable...id="+id);
return "success";
```

## REST风格

```
1.配置过滤器HiddenHttpMethodFilter
作用：Spring 根据请求中的 _method 参数进行转化，因此如果想发起 REST 风格的 DELETE 或者 PUT 请求，只需要在表单中带上 _method 参数，并且把 _method 的值设置为 DELETE 或者 PUT（大写） 即可。
2.过程
在 web.xml 中配置 HiddenHttpMethodFilter
编写 handler 代码
编写页面

3.区别
普通CRUD  
查询 getEmp   	emp--get
添加 addEmp  		emp--post
修改 updateEmp?id=1	emp--put
删除 delete?id=1      emp--delete
```

## 请求数据传入

**@RequestParam注解**

```java
/**
 * @RequestParam 注解用于映射请求参数
 *         value 用于映射请求参数名称
 *         required 用于设置请求参数是否必须的
 *         defaultValue 设置默认值，当没有传递参数时使用该值
 */
@RequestMapping(value="/testRequestParam")
public String testRequestParam(
@RequestParam(value="username") String username,
@RequestParam(value="age",required=false,defaultValue="0") int age){
System.out.println("testRequestParam - username="+username +",age="+age);
return "success";
}
测试页面
<!--测试 请求参数 @RequestParam 注解使用 -->
<a href="springmvc/testRequestParam?username=atguigu&age=10">testRequestParam</a>
```

**@RequestHeader 注解**

```java
使用 @RequestHeader 绑定请求报头的属性值
请求头包含了若干个属性，服务器可据此获知客户端的信息，通过 @RequestHeader 即可将请求头中的属性值绑定到处理方法的参数表中，处理方法就可以获取到请求头的属性信息

//了解: 映射请求头信息 用法同 @RequestParam
@RequestMapping(value="/testRequestHeader")
public String testRequestHeader(@RequestHeader(value="Accept-Language") String al){
System.out.println("testRequestHeader - Accept-Language："+al);
return "success";
}
<!-- 测试 请求头@RequestHeader 注解使用 -->
<a href="springmvc/testRequestHeader">testRequestHeader</a>
```

**@CookieValue 注解**

使用 @CookieValue 绑定请求中的 Cookie 值

@CookieValue** 可让处理方法入参绑定某个 Cookie 值

```java
//了解:@CookieValue: 映射一个 Cookie 值. 属性同 @RequestParam
@RequestMapping("/testCookieValue")
public String testCookieValue(@CookieValue("JSESSIONID") String sessionId) {
System.out.println("testCookieValue: sessionId: " + sessionId);
return "success";
}
<!--测试 请求Cookie @CookieValue 注解使用 -->
<a href="springmvc/testCookieValue">testCookieValue</a>
```

**使用POJO作为参数**

```java
使用 POJO 对象绑定请求参数值
Spring MVC 会按请求参数名和 POJO 属性名进行自动匹配，自动为该对象填充属性值。支持级联属性。只需要保证pojo名和参数名一致
如：dept.deptId、dept.address.tel 等
/**
 * Spring MVC 会按请求参数名和 POJO 属性名进行自动匹配， 自动为该对象填充属性值。
 * 支持级联属性
 *                 如：dept.deptId、dept.address.tel 等
 */
@RequestMapping("/testPOJO")
//自动填充到user对象的属性中去
public String testPojo(User user) {
System.out.println("testPojo: " + user);
return "success";
}

<!-- 测试 POJO 对象传参，支持级联属性 -->
<form action=" testPOJO" method="POST">
username: <input type="text" name="username"/><br>
password: <input type="password" name="password"/><br>
email: <input type="text" name="email"/><br>
age: <input type="text" name="age"/><br>
city: <input type="text" name="address.city"/><br>
province: <input type="text" name="address.province"/>
<input type="submit" value="Submit"/>
</form>	
 

```

## 中文乱码问题

如果中文有乱码，需要配置字符编码过滤器，且配置其他过滤器之前，

如（HiddenHttpMethodFilter），否则不起作用。（思考method=”get”请求的乱码问题怎么解决的）

```
配置字符编码过滤器--解决中文乱码问题
	<!-- 配置字符集 -->
	<filter>
		<filter-name>encodingFilter</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>UTF-8</param-value>
		</init-param>
		<init-param>
			<param-name>forceEncoding</param-name>
			<param-value>true</param-value>
		</init-param>
	</filter>
	<filter-mapping>
		<filter-name>encodingFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>

```

## 响应数据输出--后端数据输出到页面

1.除了request，session，application可以在方法中传入**Map，Model，ModelMap**，在这三个参数里面保存的**数据都会放在request域中**，可以在页面获取。

**Map，Model，ModelMap**数据输出

```java
@RequestMapping("input")
public String input(Map map, ModelMap modelMap, Model model){
    map.put("msg","hello") ;
    modelMap.addAttribute("name","word") ;
    model.addAttribute("age",17) ;
    return "message/index" ;
}
可以在页面中用requestScope、sessionScope、applicationScope
request.setAttribute(“name”，“b”)    //一次请求中，可以请求转发
session.setAttribute(“name”，“c”)    //会话超时之前都存在，只要浏览器不关闭
application.setAttribute(“name”，“d”)    //服务器重启或关闭前都存在，只要服务器不关闭


```

**ModelAndView**

存储处理完后的结果数据，以及显示该数据的视图

```
ModelAndView构造方法可以指定返回的页面名称
还可以通过setViewName()方法跳转到指定的页面 

1.添加模型数据 :→用于输出到页面上
ModelAndView addObject(String attributeName, Object attributeValue)
ModelAndView addAllObject(Map<String, ?> modelMap)
@RequestMapping("/getModel")
    public String getModel(Model model){
        model.addAttribute("name","njl");
        model.addAttribute("sex","男");
        model.addAttribute("city","北京");
        return "/login/getModel";
    }

2.设置视图
void setView(View view)(自定义视图)
void setViewName(String viewName)
 @RequestMapping("/getModelAndView")
    public ModelAndView getModelAndView(){
      	ModelAndView modelAndView = new ModelAndView();
 //通过构造方法设置视图并跳转到success页面
      String viewName = "success";
ModelAndView mv = new ModelAndView(viewName );
// 通过setViewName()方法设置视图并跳转到指定的页面       modelAndView.setViewName("/login/getModelAndView");
        return modelAndView;
    }

```

## @ModelAttribute注解

```
1.在方法定义上使用 @ModelAttribute 注解：Spring MVC 在调用目标处理方法前，会先执行有 @ModelAttribute 注解的方法。

2.作用：在方法的入参前使用 @ModelAttribute 注解：可以从隐含对象中获取隐含的模型数据中获取对象，再将请求参数绑定到对象中，再传入入参
3.将方法入参对象添加到模型中
可以将请求参数添加到model模型中，做模型的初始化工作

//1. 由 @ModelAttribute 标记的方法, 会在每个目标方法执行之前被 SpringMVC 调用!
@RequestMapping("/testModelAttribute")
public String testModelAttribute(User user){
System.out.println("user="+user);                
return "success";
}

先执行该方法
@ModelAttribute
public void getUser(@RequestParam(value="id",required=false) Integer id,Map<String,Object> map){
if(id!=null){        
//模拟从数据库中获取到的user对象
User user = new User(1,"Tom","123456","tom@atguigu.com",12);
System.out.println("从数据库中查询的对象：user="+user );
//将数据库中的对象添加到模型map中
map.put("user", user);
}
}

```

## 视图和视图解析器

## restful CRUD

```
GET    用来获取资源
POST  用来新建资源
PUT    用来更新资源
DELETE  用来删除资源

```

## 自定义类型转换器

```
ConversionService 是 Spring 类型转换体系的核心接口。
可以利用 ConversionServiceFactoryBean 在 Spring 的 IOC 容器中定义一个
ConversionService. Spring 将自动识别出 IOC 容器中的 ConversionService，并在 Bean 属性配置及 Spring  MVC 处理方法入参绑定等场合使用它进行数据的转换
可通过 ConversionServiceFactoryBean 的 converters 属性注册自定义的类型转换器
1.自定义转换器类
2.将转换器类在springmvc.xml中用conversionService进行配置


在springmvc.xml中配置自定义类型转换器
<bean id="conversionService"  
class="org.springframework.context.support.ConversionServiceFactoryBean">
converters：用于设置转换器
<property name="converters">
<set>
<!-- 引用类型转换器 -->
<ref bean="stringToEmployeeConverter"/>
</set>                        
</property>
</bean>

```

## <mvc:annotation-driven />

```
<mvc:annotation-driven /> 作用
1.<mvc:annotation-driven /> 会自动注册：
RequestMappingHandlerMapping 、RequestMappingHandlerAdapter 与
ExceptionHandlerExceptionResolver  三个bean。
2.
支持使用 ConversionService 实例对表单参数进行类型转换
支持使用 @NumberFormat、@DateTimeFormat 注解完成数据类型的格式化
支持使用 @Valid 注解对 JavaBean 实例进行 JSR 303 验证
支持使用 @RequestBody 和 @ResponseBody 注解

```

## 数据格式化

```
对属性对象的输入/输出进行格式化，从其本质上讲依然属于 “类型转换” 的范畴。
1.日期格式化@DateTimeFormat 
@DateTimeFormat 注解可对 java.util.Date、java.util.Calendar、java.long.Long 时间类型进行标注：
pattern 属性：类型为字符串。指定解析/格式化字段数据的模式，如：”yyyy-MM-dd hh:mm:ss”
2.数值格式化@NumberFormat 
style：类型为 NumberFormat.Style。用于指定样式类型，包括三种：Style.NUMBER（正常数字类型）、 Style.CURRENCY（货币类型）、 Style.PERCENT（百分数类型）
pattern：类型为 String，自定义样式，如pattern="#,###"；



```

## JSR303 数据校验

```
1.添加<mvc:annotation-driven/> mvc注解驱动会注册LocalValidatorFactoryBean，
Spring 的 LocalValidatorFactroyBean作用：
实现了 Spring 的 Validator 接口，也实现了 JSR 303 的 Validator 接口
<mvc:annotation-driven/> 会默认装配好一个 LocalValidatorFactoryBean，

数据校验流程
1.导入依赖
2.开启注解驱动，注册hibernate的校验器
3.利用注解规则进行校验

<!-- 引入hibernate的校验器实现 -->
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-validator</artifactId>
    <version>5.1.1.Final</version>
</dependency>

<!--开启mvc注解驱动  -->
<mvc:annotation-driven validator="validator" />

<!-- 注册hibernate的校验器 LocalValidatorFactoryBean -->
<bean id = "validator" class = "org.springframework.validation.beanvalidation.LocalValidatorFactoryBean">
	<property name="providerClass" value ="org.hibernate.validator.HibernateValidator"></property>  		
</bean>

常用的校验注解
@NotNull 对象不为空
@AssertTrue Boolean 对象是否为 true
@AssertFalse 验证 Boolean 对象是否为 false
@Size(min =, max =) 长度是否在给定的范围
@Min(value=) 验证 Number 对象是否大等于指定的值
@Max(value=) 验证 Number 对象是否小等于指定的值
@Pattern 验证 String 对象是否符合正则表达式的规则
@Past @Future验证 Date 和 Calendar 时间对象是否在当前时间之前/之后
@Email 是否是正确的邮箱地址

```

## SpringMVC处理JSON请求

```java
1.页面向后台发送JSON数据
用@requestBody来绑定请求参数
2.后台想前端页面发送JSON数据
用responseBody来绑定发送数据
/**
 * 测试json的交互
 * @param item
 * @return
 */
@RequestMapping("/testJson.action")
// @ResponseBody //或者写在这里也可以
public @ResponseBody Items testJson(@RequestBody Items item) {
    return item;
}

```

## HttpEntity

## 拦截器

拦截器可以用于权限验证、解决乱码、操作日志记录、性能监控、异常处理等

```
自定义的拦截器必须实现HandlerInterceptor接口
通过实现实现HandlerInterceptor接口，并重写其preHandle()，postHandle()，afterCompletion()来实现拦截功能
preHandle()：这个方法在业务处理器处理请求之前被调用，在该方法中对用户请求 request 进行处理。
return true --放行该请求  return false---拦截该请求
若放行-return true 则后面方法会执行
postHandle()：这个方法在业务处理器处理完请求后，但是DispatcherServlet 向客户端返回响应前被调用，在该方法中对用户请求request进行处理。
afterCompletion()：这个方法在 DispatcherServlet 完全处理完请求后被调用，可以在该方法中进行一些资源清理的操作。

配置自定义拦截器
<mvc:interceptors>
<!-- 声明自定义拦截器 -->
<bean id="firstHandlerInterceptor"
      class="com.atguigu.springmvc.interceptors.FirstHandlerInterceptor"></bean>
</mvc:interceptors>

若配置了两个拦截器，其拦截器执行顺序 两个拦截器都放行 return true
执行顺序
com.atguigu.springmvc.interceptors.FirstHandlerInterceptor - preHandle
com.atguigu.springmvc.interceptors.SecondHandlerInterceptor - preHandle
************************************biz method*******************************
com.atguigu.springmvc.interceptors.SecondHandlerInterceptor - postHandle
com.atguigu.springmvc.interceptors.FirstHandlerInterceptor - postHandle
com.atguigu.springmvc.interceptors.SecondHandlerInterceptor - afterCompletion
com.atguigu.springmvc.interceptors.FirstHandlerInterceptor - afterCompletion

```

## 异常处理



