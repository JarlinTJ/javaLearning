[TOC]

# 一、SpringMVC简介

### 1、什么是MVC

MVC是一种软件架构的思想，将软件按照模型、视图、控制器来划分

M：Model，模型层，指工程中的JavaBean，作用是处理数据

JavaBean分为两类：

- 一类称为实体类Bean：专门存储业务数据的，如 Student、User 等
- 一类称为业务处理 Bean：指 Service 或 Dao 对象，专门用于处理业务逻辑和数据访问。

V：View，视图层，指工程中的html或jsp等页面，作用是与用户进行交互，展示数据

C：Controller，控制层，指工程中的servlet，作用是接收请求和响应浏览器

MVC的工作流程：
用户通过视图层发送请求到服务器，在服务器中请求被Controller接收，Controller调用相应的Model层处理请求，处理完毕将结果返回到Controller，Controller再根据请求处理的结果找到相应的View视图，渲染数据后最终响应给浏览器

### 2、什么是SpringMVC

SpringMVC是Spring的一个后续产品，是Spring的一个子项目

SpringMVC 是 Spring 为表述层开发提供的一整套完备的解决方案。在表述层框架历经 Strust、WebWork、Strust2 等诸多产品的历代更迭之后，目前业界普遍选择了 SpringMVC 作为 Java EE 项目表述层开发的**首选方案**。

> 注：三层架构分为表述层（或表示层）、业务逻辑层、数据访问层，表述层表示前台页面和后台servlet

### 3、SpringMVC的特点

- **Spring 家族原生产品**，与 IOC 容器等基础设施无缝对接
- **基于原生的Servlet**，通过了功能强大的**前端控制器DispatcherServlet**，对请求和响应进行统一处理
- 表述层各细分领域需要解决的问题**全方位覆盖**，提供**全面解决方案**
- **代码清新简洁**，大幅度提升开发效率
- 内部组件化程度高，可插拔式组件**即插即用**，想要什么功能配置相应组件即可
- **性能卓著**，尤其适合现代大型、超大型互联网项目要求

# 二、HelloWorld

### 1、开发环境

IDE：idea 2019.2

构建工具：maven3.5.4

服务器：tomcat7

Spring版本：5.3.1

### 2、创建maven工程

##### a>添加web模块

##### b>打包方式：war

##### c>引入依赖

```xml
<dependencies>
    <!-- SpringMVC -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.3.1</version>
    </dependency>

    <!-- 日志 -->
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.2.3</version>
    </dependency>

    <!-- ServletAPI -->
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.1.0</version>
        <scope>provided</scope>
    </dependency>

    <!-- Spring5和Thymeleaf整合包 -->
    <dependency>
        <groupId>org.thymeleaf</groupId>
        <artifactId>thymeleaf-spring5</artifactId>
        <version>3.0.12.RELEASE</version>
    </dependency>
</dependencies>
```

注：由于 Maven 的传递性，我们不必将所有需要的包全部配置依赖，而是配置最顶端的依赖，其他靠传递性导入。

![images](img\img001.png)

新建模块总流程：配打包方式-->配依赖-->配web.xml-->创建请求控制器-->配springMVC.xml

### 3、配置web.xml

注册SpringMVC的前端控制器DispatcherServlet

为什么要注册？——因为浏览器不能直接访问一个类，需要通过设置匹配路径来访问servlet，请求就会被servlet处理

##### a>默认配置方式（不采用）

此配置作用下，SpringMVC的配置文件默认位于WEB-INF下，默认名称为\<servlet-name>-servlet.xml，例如，以下配置所对应SpringMVC的配置文件位于WEB-INF下，文件名为springMVC-servlet.xml

```xml
<!-- 配置SpringMVC的前端控制器，对浏览器发送的请求统一进行处理 -->
<servlet>
    <!--servlet-name要与servlet-mapping保持一致-->
    <servlet-name>springMVC</servlet-name>
    <!--servlet的全类名-->
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>springMVC</servlet-name>
    <!--
        设置springMVC的核心控制器所能处理的请求的请求路径
        /所匹配的请求可以是/login或.html或.js或.css方式的请求路径
        但是/不能匹配.jsp请求路径的请求
    -->
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

##### b>扩展配置方式（采用）

可通过init-param标签自定义设置SpringMVC配置文件的**位置和名称**，通过load-on-startup标签设置SpringMVC前端控制器DispatcherServlet的初始化时间

```xml
<!-- 配置SpringMVC的前端控制器，对浏览器发送的请求统一进行处理 -->
<servlet>
    <servlet-name>springMVC</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!-- 通过初始化参数指定SpringMVC配置文件的位置和名称 -->
    <init-param>
        <!-- contextConfigLocation为固定值 -->
        <param-name>contextConfigLocation</param-name>
        <!-- 使用classpath:表示从类路径查找配置文件，例如maven工程中的src/main/resources -->
        <param-value>classpath:springMVC.xml</param-value>
    </init-param>
    <!-- 
 		作为框架的核心组件，在启动过程中有大量的初始化操作要做
		而这些操作放在第一次请求时才执行会严重影响访问速度
		因此需要通过此标签将启动控制DispatcherServlet（前端控制器）的初始化时间提前到服务器启动时
	-->
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>springMVC</servlet-name>
    <!--
        设置springMVC的核心控制器所能处理的请求的请求路径
        /所匹配的请求可以是/login或.html或.js或.css方式的请求路径
        但是/不能匹配.jsp请求路径的请求
    -->
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

> 注：
>
> \<url-pattern>标签中使用/和/*的区别：
>
> /所匹配的请求可以是/login或.html或.js或.css方式的请求路径，但是/不能匹配.jsp请求路径的请求
>
> 因此就可以避免在访问jsp页面时，该请求被DispatcherServlet处理，从而找不到相应的页面
>
> /*则能够匹配所有请求，例如在使用过滤器时，若需要对所有请求进行过滤，就需要使用/\*的写法

##### c>通用方式

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <!--配置编码过滤器-->
    <filter>
        <filter-name>CharacterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceResponseEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>CharacterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!--配置处理请求方式put和delete的HiddenHttpMethodFilter-->
    <filter>
        <filter-name>HiddenHttpMethodFilter</filter-name>
        <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>HiddenHttpMethodFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!--配置springMVC的前端控制器DispatcherServlet-->
    <servlet>
        <servlet-name>DispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <!--设置springmvc配置文件自定义的位置和名称-->
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springMVC.xml</param-value>
        </init-param>
        <!--将前端控制器的初始化时间提前到服务器启动时，因为springmvc在初始化时要做大量工作-->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>DispatcherServlet</servlet-name>
        <!--
            设置springMVC的核心控制器所能处理的请求的请求路径
            /所匹配的请求可以是/login或.html或.js或.css方式的请求路径
            但是/不能匹配.jsp请求路径的请求
        -->
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```



### 4、创建请求控制器

由于前端控制器对浏览器发送的请求进行了统一的处理，但是具体的请求有不同的处理过程，因此需要创建处理具体请求的**类，即请求控制器**

请求控制器中每一个处理请求的方法成为控制器方法

因为SpringMVC的控制器由一个POJO（普通的Java类）担任，因此需要通过@Controller注解将其标识为一个控制层组件，交给Spring的IoC容器管理，此时SpringMVC才能够识别控制器的存在

```java
@Controller
public class HelloController {
    
}
```

### 5、创建springMVC的配置文件

![image-20210917120202199](C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20210917120202199.png)

这个图标说明已经将bean注入IOC容器中了

```xml
<!-- 自动扫描包，扫描+注解就能找到类了 -->
<context:component-scan base-package="com.atguigu.mvc.controller"/>

<!-- 配置Thymeleaf视图解析器 -->
<bean id="viewResolver" class="org.thymeleaf.spring5.view.ThymeleafViewResolver">
    <!-- 优先级。有多个视图解析器时有用 -->
    <property name="order" value="1"/>
    <property name="characterEncoding" value="UTF-8"/>
    <property name="templateEngine">
        <!-- 内部bean对templateEngine属性赋值 -->
        <bean class="org.thymeleaf.spring5.SpringTemplateEngine">
            <property name="templateResolver">
                <bean class="org.thymeleaf.spring5.templateresolver.SpringResourceTemplateResolver">
    
                    <!-- 视图前缀 -->
                    <property name="prefix" value="/WEB-INF/templates/"/>
    
                    <!-- 视图后缀 -->
                    <property name="suffix" value=".html"/>
                    <property name="templateMode" value="HTML5"/>
                    <property name="characterEncoding" value="UTF-8" />
                </bean>
            </property>
        </bean>
    </property>
</bean>

<!-- 
   处理静态资源，例如html、js、css、jpg
  若只设置该标签，则只能访问静态资源，其他请求则无法访问
  此时必须设置<mvc:annotation-driven/>解决问题
 -->
<mvc:default-servlet-handler/>

<!-- 开启mvc注解驱动 -->
<mvc:annotation-driven>
    <mvc:message-converters>
        <!-- 处理响应中文内容乱码 -->
        <bean class="org.springframework.http.converter.StringHttpMessageConverter">
            <property name="defaultCharset" value="UTF-8" />
            <property name="supportedMediaTypes">
                <list>
                    <value>text/html</value>
                    <value>application/json</value>
                </list>
            </property>
        </bean>
    </mvc:message-converters>
</mvc:annotation-driven>
```

### 6、测试HelloWorld

##### a>实现对首页的访问

在请求控制器中创建处理请求的方法

```java
// @RequestMapping注解：处理请求和控制器方法之间的映射关系
// @RequestMapping注解的value属性可以通过请求地址匹配请求，/表示的当前工程的上下文路径
// localhost:8080/springMVC/
@RequestMapping("/")
public String index() {
    //设置视图名称
    return "index";
}
```

##### b>通过超链接跳转到指定页面

```html
<a href="/target">访问目标页面target.html</a>

/ 访问的是localhost:8080后的路径，而不是上下文路径，需加上上下文路径(绝对方式，如下)

<a href="/springMVC/target">访问目标页面target.html</a>
```

上下文路径：

<img src="C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20210918162242766.png" alt="image-20210918162242766" style="zoom: 67%;" />

在主页index.html中设置超链接

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>首页</title>
</head>
<body>
    <h1>首页</h1>
    <!-- 使用thymeleaf方式th:href解析路径：当@{}内检测到绝对路径，就会自动加上上下文路径解析 -->
    <a th:href="@{/hello}">HelloWorld</a><br/>
</body>
</html>
```

在请求控制器中创建处理请求的方法（控制器里的方法是真正处理请求的方法）

```java
@RequestMapping("/hello")//value可省略不写，与方法名无关
public String HelloWorld() {
    return "target";
}
```

### 7、总结

浏览器发送请求，若请求地址符合前端控制器的url-pattern，该请求就会被前端控制器DispatcherServlet处理。前端控制器会读取SpringMVC的核心配置文件，通过扫描组件找到控制器，将请求地址和控制器中@RequestMapping注解的**value属性值**进行匹配，若匹配成功，该注解所标识的控制器方法就是**处理请求的方法**。处理请求的方法需要返回一个字符串类型的**视图名称**，该视图名称会被视图解析器解析，加上前缀和后缀组成**视图的路径**，通过Thymeleaf对视图进行渲染，最终转发到视图所对应**页面**

浏览器访问-》转到分发器-》读取配置文件-》组件扫描到对应控制器-》RequestManpping映射

# 三、@RequestMapping注解

### 1、@RequestMapping注解的功能

从注解名称上我们可以看到，@RequestMapping注解的作用就是将**请求**和处理请求的**控制器方法**关联起来，建立映射关系。

SpringMVC 接收到指定的请求，就会来找到在映射关系中对应的控制器方法来处理这个请求。

路径必须唯一，否则编译不通过。多个Controller中若有相同value值，可在类上加@RequestMapping注解区分路径。

### 2、@RequestMapping注解的位置

@RequestMapping标识一个类：设置映射请求的请求路径的**初始**信息

@RequestMapping标识一个方法：设置映射请求请求路径的**具体**信息

类+方法 两个注解是叠加关系，当项目中模块较多时使用

```java
@Controller
@RequestMapping("/test")
public class RequestMappingController {

	//此时请求映射所映射的请求的请求路径为：/test/testRequestMapping
    @RequestMapping("/testRequestMapping")
    public String testRequestMapping(){
        return "success";
    }

}
```

### 3、@RequestMapping注解的value属性

<img src="C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20210919122037150.png" alt="image-20210919122037150" style="zoom:67%;" />

@RequestMapping注解的value属性通过请求的请求地址匹配请求映射

@RequestMapping注解的value属性是一个**字符串类型的数组**，表示该请求映射能够匹配**多个**请求地址所对应的请求(如下2)

@RequestMapping注解的value属性**必须设置**，**至少通过请求地址匹配**请求映射

```html
<a th:href="@{/testRequestMapping}">测试@RequestMapping的value属性-->/testRequestMapping</a><br>
<a th:href="@{/test}">测试@RequestMapping的value属性-->/test</a><br>
```

```java
@RequestMapping(
        value = {"/testRequestMapping", "/test"}
)
public String testRequestMapping(){
    return "success";
}
```

请求不到**报错404**

### 4、@RequestMapping注解的method属性

@RequestMapping注解的method属性通过请求的请求方式（**get或post**）匹配请求映射，**超链接是get请求**

@RequestMapping注解的method属性是一个RequestMethod类型的数组，表示该请求映射能够匹配多种请求方式的请求

若没有method参数，则不限制请求方式，都可以

若当前请求的请求地址满足请求映射的value属性，但是请求方式不满足method属性，则浏览器**报错405**：Request method 'POST' not supported

```html
<a th:href="@{/test}">测试@RequestMapping的value属性-->/test</a><br>
<form th:action="@{/test}" method="post">
    <input type="submit">
</form>
```

```java
@RequestMapping(
        value = {"/testRequestMapping", "/test"},
        method = {RequestMethod.GET, RequestMethod.POST}
)
public String testRequestMapping(){
    return "success";
}
```

> 注：
>
> 1、对于处理指定请求方式的控制器方法，SpringMVC中提供了@RequestMapping的派生注解，用以下4种方式可以不用再设置method属性
>
> 处理get请求的映射-->@GetMapping
>
> 处理post请求的映射-->@PostMapping
>
> 处理put请求的映射-->@PutMapping
>
> 处理delete请求的映射-->@DeleteMapping
>
> 2、常用的请求方式有get，post，put，delete
>
> 对于相同的请求地址，实现对数据的不同操作功能使用这4种方式
>
> 但是目前浏览器**只支持get**（查询）和**post**（添加），若在form表单提交时，为method**设置了其他请求方式的字符串**（put或delete），则按照**默认**的请求方式**get**处理
>
> 若要发送put（修改）和delete（删除）请求，则需要通过spring提供的过滤器HiddenHttpMethodFilter，在RESTful部分会讲到
>
> 3、GET、POST区别
>
> **get**请求参数会拼接在地址后面，以？进行拼接“请求参数名1=请求参数值1&请求参数名2=请求参数值2”；
>
> 不安全，传输速度快；
>
> 传输量有限；
>
> **post**请求参数放在请求体中；
>
> 安全，传输速度慢；
>
> 传输量大；
>
> 上传文件只能用post

### 5、@RequestMapping注解的params属性（了解）

@RequestMapping注解的params属性通过请求的请求参数匹配请求映射

@RequestMapping注解的params属性是一个字符串类型的数组（多个需同时满足，不同于value、method），可以通过四种表达式设置请求参数和请求映射的匹配关系

"param"：要求请求映射所匹配的请求**必须携带**param请求参数

"!param"：要求请求映射所匹配的请求必须**不能携带**param请求参数

"param=value"：要求请求映射所匹配的请求必须携带param请求参数，且param=value

"param!=value"：要求请求映射所匹配的请求必须携带param请求参数，但是param!=value

```html
<a th:href="@{/test(username='admin',password=123456)">测试@RequestMapping的params属性-->/test</a><br>
```

```java
@RequestMapping(
        value = {"/testRequestMapping", "/test"}
        ,method = {RequestMethod.GET, RequestMethod.POST}
        ,params = {"username","password!=123456"}
)
public String testRequestMapping(){
    return "success";
}
```

> 注：
>
> 若当前请求满足@RequestMapping注解的value和method属性，但是不满足params属性，此时页面回**报错400**：Parameter conditions "username, password!=123456" not met for actual request parameters: username={admin}, password={123456}

### 6、@RequestMapping注解的headers属性（了解）

<img src="C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20210920185948025.png" alt="image-20210920185948025" style="zoom: 80%;" />

@RequestMapping注解的headers属性通过请求的请求头信息匹配请求映射

@RequestMapping注解的headers属性是一个字符串类型的数组，可以通过四种表达式设置请求头信息和请求映射的匹配关系

"header"：要求请求映射所匹配的请求必须携带header请求头信息

"!header"：要求请求映射所匹配的请求必须不能携带header请求头信息

"header=value"：要求请求映射所匹配的请求必须携带header请求头信息且header=value

"header!=value"：要求请求映射所匹配的请求必须携带header请求头信息且header!=value

若当前请求满足@RequestMapping注解的value和method属性，但是不满足headers属性，此时页面显示**404错误**，即资源未找到

### 7、SpringMVC支持ant风格的路径

@RequestMapping的value中设置以下3种字符实现模糊匹配：

？：表示任意的单个字符（不能为空，？和 / 不行）

*：表示任意的0个或多个字符

\**：表示任意的一层或多层目录

**注意**：在使用\**时，只能使用/**/xxx的方式。不能在\*\*前后加东西，如/a\*\*a/xxx，如使用这种方式，表示为两个单独的\*

### 8、SpringMVC支持路径中的占位符（重点）

原始方式：/deleteUser?id=1

rest方式：/deleteUser/1

将当前传输到服务器的请求参数全部以请求路径的方式，拼接在路径中。

SpringMVC路径中的占位符常用于RESTful风格中，当**请求路径中将某些数据通过路径的方式传输到服务器中**，就可以在相应的@RequestMapping注解的**value属性**中通过占位符{xxx}表示传输的数据，在通过**@PathVariable**注解，将**占位符所表示的数据**赋值(**绑定**)给控制器方法的**形参**

```html
<a th:href="@{/testRest/1/admin}">测试路径中的占位符-->/testRest</a><br>
```

```java
@RequestMapping("/testRest/{id}/{username}")
public String testRest(@PathVariable("id") String id, @PathVariable("username") String username){
    System.out.println("id:"+id+",username:"+username);
    return "success";
}
//最终输出的内容为-->id:1,username:admin
```

**注：**有占位符时，在http请求路径中必须要有这一层，不能为空，否则**报错404**，如上不能只写http://localhost:8080/springMVC/testRest/

<img src="C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20210920222243181.png" alt="image-20210920222243181" style="zoom:67%;" />

# 四、SpringMVC获取请求参数

### 1、通过ServletAPI获取（一般不用）

将HttpServletRequest作为控制器方法的形参，此时HttpServletRequest类型的参数表示封装了当前请求的请求报文的对象

```java
@RequestMapping("/testParam")
public String testParam(HttpServletRequest request){
    String username = request.getParameter("username");
    String password = request.getParameter("password");
    System.out.println("username:"+username+",password:"+password);
    return "success";
}

//获取同名的多个参数如复选框，使用request.getParameterValues(String)返回String[]
```

使用getParameter这种方式不能通过占位符来获取值，因为路径中只有value，没有key，getParameter需要传入key

### 2、通过控制器方法的形参获取请求参数

在控制器方法的形参位置，设置**和请求参数同名的形参**，当浏览器发送请求，匹配到请求映射时，在DispatcherServlet中就会将请求参数赋值给相应的形参

若请求参数名和形参名不一致，则无法接收，为null

```html
<a th:href="@{/testParam(username='admin',password=123456)}">测试获取请求参数-->/testParam</a><br>
```

```java
@RequestMapping("/testParam")
public String testParam(String username, String password){
    System.out.println("username:"+username+",password:"+password);
    return "success";
}

@RequestMapping("/testParam")
//public String testParam(String username,String password,String hobby){
public String testParam(String username,String password,String[] hobby){
    System.out.println("username:"+username+",password:"+password+",hobby:"+hobby);
    System.out.println("username:"+username+",password:"+password+",hobby:"+ Arrays.toString(hobby));
    return "success";
}
```

> 注：
>
> 若请求所传输的请求参数中有**多个同名的请求参数**，此时可以在控制器方法的形参中设置**字符串数组**或者**字符串**类型的形参接收此请求参数
>
> 若使用**字符串数组**类型的形参，此参数的数组中包含了每一个数据
>
> 若使用**字符串**类型的形参，此参数的值为每个数据中间使用逗号拼接的结果

### 3、@RequestParam

```java
@RequestMapping("/testParam")
public String testParam(
        @RequestParam(value = "user_name",defaultValue = "hehe") String username,
        String password,
        String[] hobby){
    System.out.println("username:"+username+",password:"+password+",hobby:"+ Arrays.toString(hobby));
    return "success";
}
```

@RequestParam是将**请求参数和控制器方法的形参创建映射关系**，解决两个参数不同名无法接收的问题

@RequestParam注解一共有三个属性：

<img src="C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20210921152816196.png" alt="image-20210921152816196" style="zoom: 80%;" />

value/name：指定为形参赋值的请求参数的参数名

required：设置是否**必须传输此请求参数**，默认值为true。如上必须传入user_name，否则报错400(没有defaultValue情况)

若设置为true时，则当前请求必须传输value所指定的请求参数，若没有传输该请求参数，且没有设置defaultValue属性，则页面**报错400**：Required String parameter 'xxx' is not present；若设置为false，则当前请求不是必须传输value所指定的请求参数，若没有传输，则注解所标识的形参的值为null

**defaultValue**(常用)：不管required属性值为true或false，当value所指定的请求参数**没有传输**或传输的值为**""**时，则使用默认值为形参赋值

### 4、@RequestHeader

@RequestHeader是将**请求头信息和控制器方法的形参创建映射关系**

@RequestHeader注解一共有三个属性：value、required、defaultValue，用法同@RequestParam

与@RequestParam不同的是，若想获取请求头中的信息，必须用@RequestHeader；而获取请求参数可以不用@RequestParam

```java
@RequestMapping("/testParam")
public String testParam(
        @RequestParam(value = "user_name",defaultValue = "hehe") String username,
        String password,
        String[] hobby,
    //将请求头中Host对应的值赋给形参host
        @RequestHeader("Host") String host{
    System.out.println("username:"+username+",password:"+password+",hobby:"+ Arrays.toString(hobby));
    System.out.println("host:"+host);
    return "success";
}
```

### 5、@CookieValue

@CookieValue是将**cookie数据和控制器方法的形参创建映射关系**

@CookieValue注解一共有三个属性：value、required、defaultValue，用法同@RequestParam

```java
@RequestMapping("/testParam")
public String testParam(
        @RequestParam(value = "user_name",defaultValue = "hehe") String username,
        String password,
        String[] hobby,
        @RequestHeader("Host") String host,
        @CookieValue("JSESSIONID")String JSESSIONID){
    System.out.println("username:"+username+",password:"+password+",hobby:"+ Arrays.toString(hobby));
    System.out.println("host:"+host);
    System.out.println("JSESSIONID:"+JSESSIONID);//存储在服务器的（某个浏览器的）sessionID空间地址，当浏览器再次请求该站点资源时，就知道从哪个空间中取出数据
    return "success";
}
```

### 6、通过POJO获取请求参数

可以在控制器方法的形参位置设置一个实体类类型的形参，此时若浏览器传输的**请求参数的参数名**和**实体类中的属性名**一致，那么请求参数就会为此属性赋值

解决通过控制器方法形参获取参数过多繁琐的问题

```html
<form th:action="@{/testpojo}" method="post">
    用户名：<input type="text" name="username"><br>
    密码：<input type="password" name="password"><br>
    性别：<input type="radio" name="sex" value="男">男<input type="radio" name="sex" value="女">女<br>
    年龄：<input type="text" name="age"><br>
    邮箱：<input type="text" name="email"><br>
    <input type="submit">
</form>
```

```java
@RequestMapping("/testpojo")
public String testPOJO(User user){
    System.out.println(user);
    return "success";
}
//最终结果-->User{id=null, username='张三', password='123', age=23, sex='男', email='123@qq.com'}
```

### 7、解决获取请求参数的乱码问题

解决获取请求参数的乱码问题，可以使用SpringMVC提供的编码过滤器CharacterEncodingFilter，但是必须在web.xml中进行注册

```xml
<!--配置springMVC的编码过滤器-->
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <!--请求-->
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
    <!--响应-->
    <init-param>
        <param-name>forceResponseEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

> 注：
>
> SpringMVC中处理编码的过滤器一定要配置到其他过滤器之前，否则无效

# 五、域对象共享数据

域对象种类：

request——一次请求，如保存错误信息

session——一次会话，浏览器开启到关闭的过程(默认30min失效)，如保存用户登录状态

servletContext(application)——整个应用范围，服务器开启到关闭的范围（一般不用）

能共享数据的原因是因为在各个过程内建立的对应的域对象是同一个。



在success.html中需要配置属性显示：

```xml
<p th:text="${testScope}"></p>
```

${}内是属性的键，对应下面的设置键名

### 1、使用ServletAPI向request域对象共享数据（不用，使用后面4种方式）

```java
@RequestMapping("/testServletAPI")
public String testServletAPI(HttpServletRequest request){
    request.setAttribute("testScope", "hello,servletAPI");//键，值
    return "success";
}
```

### 2、使用ModelAndView向request域对象共享数据（建议）

因为在SpringMVC中，无论采用5个中的哪种方式，最终都会将模型数据和视图信息**封装为一个ModelAndView**中。

可以在doDispatch方法中打断点调试，发现所有方法均返回一个ModelAndView对象。

```java
@RequestMapping("/testModelAndView")
public ModelAndView testModelAndView(){
    /**
     * ModelAndView有Model和View的功能
     * Model主要用于向请求域共享数据
     * View主要用于设置视图，实现页面跳转
     * 不管用5种方式的哪一种，都要包装成ModelAndView
     */
    ModelAndView mav = new ModelAndView();
    //处理模型数据，即向请求域共享数据
    mav.addObject("testScope", "hello,ModelAndView");//键，值
    //设置视图，实现页面跳转
    mav.setViewName("success");
    return mav;
}
```

### 3、使用Model向request域对象共享数据

```java
//需要传入Model形参，不能在方法中new Model；使用上一种方法ModelAndView也可以放在形参中传进来。
@RequestMapping("/testModel")
public String testModel(Model model){
    model.addAttribute("testScope", "hello,Model");
    return "success";
}
```

### 4、使用map向request域对象共享数据

```java
@RequestMapping("/testMap")
public String testMap(Map<String, Object> map){
    map.put("testScope", "hello,Map");
    return "success";
}
```

### 5、使用ModelMap向request域对象共享数据

```java
@RequestMapping("/testModelMap")
public String testModelMap(ModelMap modelMap){
    modelMap.addAttribute("testScope", "hello,ModelMap");
    return "success";
}
```

### 6、Model、ModelMap、Map的关系

```java
//打印加载类
System.out.println(model.getClass().getName());
System.out.println(map.getClass().getName());
System.out.println(modelMap.getClass().getName());
```

Model、ModelMap、Map类型的参数其实本质上都是 BindingAwareModelMap 类型的

![image-20210928175528807](C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20210928175528807.png)

继承树：

<img src="C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20210928175725367.png" alt="image-20210928175725367" style="zoom: 80%;" />

```java
public interface Model{}
public class ModelMap extends LinkedHashMap<String, Object> {}
public class ExtendedModelMap extends ModelMap implements Model {}
public class BindingAwareModelMap extends ExtendedModelMap {}
```

### 7、向session域共享数据（原生API，建议，简单）

```java
@RequestMapping("/testSession")
public String testSession(HttpSession session){
    session.setAttribute("testSessionScope", "hello,session");
    return "success";
}
```

```html
<p th:text="${session.testSessionScope}"></p>
```

```html
<a th:href="@{/testSession}">通过servletAPI向session域对象共享数据</a><br>
```

### 8、向application域共享数据(servletContext)

随服务器启动而产生，整个运行过程中独一份。需要在整个工程中共享的数据才放在这里

```java
@RequestMapping("/testApplication")
public String testApplication(HttpSession session){
	ServletContext application = session.getServletContext();
    application.setAttribute("testApplicationScope", "hello,application");
    return "success";
}
```

```html
<p th:text="${application.testApplicationScope}"></p>
```

```html
<a th:href="@{/testApplication}">通过servletAPI向application域对象共享数据</a><br>
```

# 六、SpringMVC的视图

SpringMVC中的视图是View接口，视图的作用渲染数据，将模型Model中的数据展示给用户

SpringMVC视图的种类很多，默认有转发视图InternalResourceView和重定向视图RedirectView

当工程引入jstl的依赖（用java代码控制jsp，十分繁琐），转发视图会自动转换为JstlView

若使用的视图技术为Thymeleaf，在SpringMVC的配置文件中配置了Thymeleaf的视图解析器，由此视图解析器解析之后所得到的是ThymeleafView（不是所有情况获取的都是ThymeleafView，视图名称无任何前缀才行，前缀forward：、redirect：就不行）

注：若无视图解析器，SpringMVC默认使用转发视图InternalResourceView解析

### 1、ThymeleafView

当控制器方法中所设置的视图名称没有任何前缀时，此时的视图名称会被SpringMVC配置文件中所配置的视图解析器解析，视图名称拼接视图前缀和视图后缀所得到的最终路径，会通过转发的方式实现跳转

```java
@RequestMapping("/testHello")
public String testHello(){
    return "hello";
}
```

![](img/img002.png)

源码：

①执行转发结果：对mv（modelandview）进行处理

![image-20211012213826987](C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20211012213826987.png)

②处理执行渲染：处理mv、请求对象、响应对象

![image-20211012213927683](C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20211012213927683.png)

③local：本地化环境zh-cn

![image-20211012214221567](C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20211012214221567.png)

④先获取视图名称，再进行解析视图名称获取视图对象。**即返回什么视图对象类型只和视图名称有关**。**没有前缀就是thymeleaf，前缀是forward:就是转发视图，前缀是redirect:是重定向视图**

getModelInternal就是返回了个模型

![image-20211012214344655](C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20211012214344655.png)

### 2、转发视图（基本不用，因为Thymeleaf本身就是转发访问）

**WEB_INF下页面必须通过服务器转发访问，不能直接在return中写具体的html页面名称**

SpringMVC中默认的转发视图是InternalResourceView——内部资源视图

SpringMVC中创建转发视图的情况：

当控制器方法中所设置的视图名称以"forward:"为前缀时，创建InternalResourceView视图，此时的视图名称不会被SpringMVC配置文件中所配置的视图解析器解析，而是会将前缀"forward:"去掉，剩余部分作为最终路径通过转发的方式实现跳转

例如"forward:/"重定向到首页，"forward:/employee"

```java
@RequestMapping("/testForward")
public String testForward(){
    return "forward:/testHello";
}
```

![image-20210706201316593](img/img003.png)

### 3、重定向视图

SpringMVC中默认的重定向视图是RedirectView

当控制器方法中所设置的视图名称以"redirect:"为前缀时，创建RedirectView视图，此时的视图名称不会被SpringMVC配置文件中所配置的视图解析器解析，而是会将前缀"redirect:"去掉，剩余部分作为最终路径通过重定向的方式实现跳转

例如"redirect:/"，"redirect:/employee"

**注：重定向到的是请求，而不是具体页面**

```java
@RequestMapping("/testRedirect")
public String testRedirect(){
    return "redirect:/testHello";
}
```

![image-20210706201602267](img/img004.png)

> 注：
>
> 重定向视图在解析时，会先将redirect:前缀去掉，然后会判断剩余部分是否以/开头，若是则会自动拼接上下文路径

可重定向到其他页面，如百度

```java
    @RequestMapping("/testRedirect")
    public String testRedirect(){
        return "redirect:https://www.baidu.com";
//        return "redirect:/testThymeleafView";
    }
```



### 4、视图控制器view-controller

使用场景：当请求映射对应的控制器方法中，**没有其它请求过程的处理**，只需设置视图名称时，使用view-controller；等价于

当控制器方法中，仅仅用来实现页面跳转，即只需要设置视图名称时，可以将处理器方法使用view-controller标签进行表示

```xml
<!--
在spirngMVC.xml中配置
	path：设置处理的请求地址
	view-name：设置请求地址所对应的视图名称，即网页文件名，再加上视图前缀和视图后缀就能找到对应页面文件
-->
<mvc:view-controller path="/testView" view-name="success"></mvc:view-controller>
```
使用view-controller可代替(**可以少写一个方法甚至类**)：

```xml
<mvc:view-controller path="/" view-name="index"></mvc:view-controller>
```

```Java
@Controller
public class TestController {
	//用了视图控制器可以代替下面，可理解为注解和xml配置的区别
//    @RequestMapping("/")
//    public String index(){
//        return "index";
//    }

    @RequestMapping("/test_view")
    public String testView(){
        return "test_view";
    }
}
```



> 注：
>
> 当SpringMVC中设置任何一个view-controller时，其他控制器中的请求映射将全部失效报404，此时需要在SpringMVC的核心配置文件中设置开启mvc注解驱动的标签：
>
> 开启MVC的注解驱动（功能很多，很多情况下都会写，但功能不同）
>
> <mvc:annotation-driven />
>
> 如①view-controller
>
> ②resuful对js、css静态资源访问，需要servlet标签处理静态资源，此时控制器请求映射失效
>
> ③java对象转json对象时

# 七、RESTful

### 1、RESTful简介

REST：**Re**presentational **S**tate **T**ransfer，表现层/展现层的资源状态转移。一种风格。

总结：请求地址URL不变，根据请求方式的不同，对操作资源的方式进行区分。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210414160840547.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ppYTUxMTk5Nw==,size_16,color_FFFFFF,t_70)

##### a>资源（内容）

资源是一种看待服务器的方式，即，将服务器看作是由很多离散的资源组成。每个资源是服务器上一个可命名的抽象概念。因为资源是一个抽象的概念，所以它不仅仅能代表服务器文件系统中的一个文件、数据库中的一张表等等具体的东西，可以将资源设计的要多抽象有多抽象，只要想象力允许而且客户端应用开发者能够理解。与面向对象设计类似，资源是以名词为核心来组织的，首先关注的是名词。一个资源可以由一个或多个URI来标识。URI既是资源的名称，也是资源在Web上的地址。对某个资源感兴趣的客户端应用，可以通过资源的URI与其进行交互。

##### b>资源的表述（资源状态，啥内容？）

资源的表述是一段对于资源在某个特定时刻的状态的描述。可以在客户端-服务器端之间转移（交换）。资源的表述可以有多种格式，例如HTML/XML/JSON/纯文本/图片/视频/音频等等。资源的表述格式可以通过协商机制来确定。请求-响应方向的表述通常使用不同的格式。

##### c>状态转移（要对内容干啥？）

状态转移说的是：在客户端和服务器端之间转移（transfer）代表资源状态的表述。通过转移和操作资源的表述，来间接实现操作资源的目的。

### 2、RESTful的实现

具体说，就是 HTTP 协议里面，四个表示操作方式的动词：GET、POST、PUT、DELETE。

它们分别对应四种基本操作：GET 用来获取资源，POST 用来新建资源，PUT 用来更新资源，DELETE 用来删除资源。

REST 风格提倡 URL 地址使用**统一的风格设计**，从前到后各个单词使用**斜杠**分开，不使用**问号键值对**方式携带请求参数，而是将要发送给服务器的数据作为 URL 地址的一部分，以保证整体风格的一致性。

| 操作     | 传统方式         | REST风格                |
| -------- | ---------------- | ----------------------- |
| 查询操作 | getUserById?id=1 | user/1-->get请求方式    |
| 保存操作 | saveUser         | user-->post请求方式     |
| 删除操作 | deleteUser?id=1  | user/1-->delete请求方式 |
| 更新操作 | updateUser       | user-->put请求方式      |

### 3、HiddenHttpMethodFilter

由于浏览器只支持发送get和post方式的请求，那么该如何发送put和delete请求呢？

SpringMVC 提供了 **HiddenHttpMethodFilter** 帮助我们**将 POST 请求转换为 DELETE 或 PUT 请求**

**HiddenHttpMethodFilter** 处理put和delete请求的条件：

a>当前请求的请求方式必须为post

b>当前请求必须传输请求参数_method
```html
<form th:action="@{/user}" method="post">  
    <input type="hidden" name="_method" value="PUT">
    用户名：<input type="text" name="username"><br>
    密码：<input type="password" name="password"><br>
    <input type="submit" value="修改"><br>
</form>
```
满足以上条件，**HiddenHttpMethodFilter** 过滤器就会将当前请求的请求方式转换为请求参数_method的值，因此请求参数\_method的值才是最终的请求方式

注：本质还是POST，只是为了业务需求作区分，分发到不同的@RequestMapping

在web.xml中注册**HiddenHttpMethodFilter** 

```xml
<filter>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

> 注：
>
> 目前为止，SpringMVC中提供了两个过滤器：CharacterEncodingFilter和HiddenHttpMethodFilter
>
> 在web.xml中注册时，必须先注册CharacterEncodingFilter，再注册HiddenHttpMethodFilter
>
> 原因：
>
> - 在 CharacterEncodingFilter 中通过 request.setCharacterEncoding(encoding) 方法设置字符集的
>
> - request.setCharacterEncoding(encoding) 方法要求前面不能有任何获取请求参数的操作
>
> - 而 HiddenHttpMethodFilter 恰恰有一个获取请求方式的操作：
>
> - ```
>   String paramValue = request.getParameter(this.methodParam);
>   ```





# 八、RESTful案例

### 1、准备工作

和传统 CRUD 一样，实现对员工信息的增删改查。

- 搭建环境

- 准备实体类

  ```java
  package com.atguigu.mvc.bean;
  
  public class Employee {
  
     private Integer id;
     private String lastName;
  
     private String email;
     //1 male, 0 female
     private Integer gender;
     
     public Integer getId() {
        return id;
     }
  
     public void setId(Integer id) {
        this.id = id;
     }
  
     public String getLastName() {
        return lastName;
     }
  
     public void setLastName(String lastName) {
        this.lastName = lastName;
     }
  
     public String getEmail() {
        return email;
     }
  
     public void setEmail(String email) {
        this.email = email;
     }
  
     public Integer getGender() {
        return gender;
     }
  
     public void setGender(Integer gender) {
        this.gender = gender;
     }
  
     public Employee(Integer id, String lastName, String email, Integer gender) {
        super();
        this.id = id;
        this.lastName = lastName;
        this.email = email;
        this.gender = gender;
     }
  
     public Employee() {
     }
  }
  ```

- 准备dao模拟数据

  ```java
  package com.atguigu.mvc.dao;
  
  import java.util.Collection;
  import java.util.HashMap;
  import java.util.Map;
  
  import com.atguigu.mvc.bean.Employee;
  import org.springframework.stereotype.Repository;
  
  
  @Repository
  public class EmployeeDao {
  
     private static Map<Integer, Employee> employees = null;
     
     static{
        employees = new HashMap<Integer, Employee>();
  
        employees.put(1001, new Employee(1001, "E-AA", "aa@163.com", 1));
        employees.put(1002, new Employee(1002, "E-BB", "bb@163.com", 1));
        employees.put(1003, new Employee(1003, "E-CC", "cc@163.com", 0));
        employees.put(1004, new Employee(1004, "E-DD", "dd@163.com", 0));
        employees.put(1005, new Employee(1005, "E-EE", "ee@163.com", 1));
     }
     
     private static Integer initId = 1006;
     
     public void save(Employee employee){
        if(employee.getId() == null){
           employee.setId(initId++);
        }
        employees.put(employee.getId(), employee);
     }
     
     public Collection<Employee> getAll(){
        return employees.values();
     }
     
     public Employee get(Integer id){
        return employees.get(id);
     }
     
     public void delete(Integer id){
        employees.remove(id);
     }
  }
  ```

### 2、功能清单

| 功能                | URL 地址    | 请求方式 |
| ------------------- | ----------- | -------- |
| 访问首页√           | /           | GET      |
| 查询全部数据√       | /employee   | GET      |
| 删除√               | /employee/2 | DELETE   |
| 跳转到添加数据页面√ | /toAdd      | GET      |
| 执行保存√           | /employee   | POST     |
| 跳转到更新数据页面√ | /employee/2 | GET      |
| 执行更新√           | /employee   | PUT      |

### 3、具体功能：访问首页

##### a>配置view-controller

```xml
<mvc:view-controller path="/" view-name="index"/>
```

##### b>创建页面

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8" >
    <title>Title</title>
</head>
<body>
<h1>首页</h1>
<a th:href="@{/employee}">访问员工信息</a>
</body>
</html>
```

### 4、具体功能：查询所有员工数据

##### a>控制器方法

```java
@RequestMapping(value = "/employee", method = RequestMethod.GET)
public String getEmployeeList(Model model){
    Collection<Employee> employeeList = employeeDao.getAll();
    model.addAttribute("employeeList", employeeList);
    return "employee_list";
}
```

##### b>创建employee_list.html

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Employee Info</title>
    <script type="text/javascript" th:src="@{/static/js/vue.js}"></script>
</head>
<body>

    <!--table表格-->
    <table border="1" cellpadding="0" cellspacing="0" style="text-align: center;" id="dataTable">
        <!--tr一行-->
        <tr>
            <!--th表头，自动加粗并居中,colspan单元格可横跨的列数-->
            <th colspan="5">Employee Info</th>
        </tr>
        <tr>
            <th>id</th>
            <th>lastName</th>
            <th>email</th>
            <th>gender</th>
            <th>options(<a th:href="@{/toAdd}">add</a>)</th>
        </tr>
        <!--th:each循环一行即：tr th:each。类似增强for循环，一个对象:${请求域中的集合}-->
        <tr th:each="employee : ${employeeList}">
            <!--td单元格-->
            <td th:text="${employee.id}"></td>
            <td th:text="${employee.lastName}"></td>
            <td th:text="${employee.email}"></td>
            <td th:text="${employee.gender}"></td>
            <td>
                <a class="deleteA" @click="deleteEmployee" th:href="@{'/employee/'+${employee.id}}">delete</a>
                <a th:href="@{'/employee/'+${employee.id}}">update</a>
            </td>
        </tr>
    </table>
</body>
</html>
```
关闭Thymeleaf错误检查

![image-20211110174750319](C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20211110174750319.png)

### 5、具体功能：删除

##### a>创建处理delete请求方式的表单

```html
<!-- 作用：通过超链接控制表单的提交，将post请求转换为delete请求 -->
<form id="delete_form" method="post">
    <!-- HiddenHttpMethodFilter要求：必须传输_method请求参数，并且值为最终的请求方式 -->
    <input type="hidden" name="_method" value="delete"/>
</form>
```

##### b>删除超链接绑定点击事件

引入vue.js

```html
<script type="text/javascript" th:src="@{/static/js/vue.js}"></script>
```

删除超链接（超链接路径为@{'/employee/'+${employee.id}}，+前为路径，+后为数据。路径与请求数据的拼接）

```html
<a class="deleteA" @click="deleteEmployee" th:href="@{'/employee/'+${employee.id}}">delete</a>
```

通过vue处理点击事件

```html
<script type="text/javascript">
    var vue = new Vue({
        el:"#dataTable",
        methods:{
            //event表示当前事件
            deleteEmployee:function (event) {
                //通过id获取表单标签
                var delete_form = document.getElementById("delete_form");
                //将触发点击事件的超链接的href属性为表单的action属性赋值
                delete_form.action = event.target.href;
                //提交表单
                delete_form.submit();
                //阻止超链接的默认跳转行为
                event.preventDefault();
            }
        }
    });
</script>
```

##### c>控制器方法

```java
@RequestMapping(value = "/employee/{id}", method = RequestMethod.DELETE)
public String deleteEmployee(@PathVariable("id") Integer id){
    employeeDao.delete(id);
    return "redirect:/employee";
}
```
##### d>请求405/404，无法解析vue.js

在springMVC.xml中

```xml
	<!--开放对静态资源的访问-->
    <mvc:default-servlet-handler></mvc:default-servlet-handler>
```

处理流程：

静态资源被访问时，先被springMVC即前端控制器处理。如果在控制器中找不到相对应的请求映射，就会交给默认的servlet处理。如果默认的servlet能找到相对应的资源，就访问资源；若找不到，返回404。

### 6、具体功能：跳转到添加数据页面

##### a>配置view-controller

```xml
<mvc:view-controller path="/toAdd" view-name="employee_add"></mvc:view-controller>
```

##### b>创建employee_add.html

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Add Employee</title>
</head>
<body>

<form th:action="@{/employee}" method="post">
    lastName:<input type="text" name="lastName"><br>
    email:<input type="text" name="email"><br>
    gender:<input type="radio" name="gender" value="1">male
    <input type="radio" name="gender" value="0">female<br>
    <input type="submit" value="add"><br>
</form>

</body>
</html>
```

### 7、具体功能：执行保存

##### a>控制器方法

```java
@RequestMapping(value = "/employee", method = RequestMethod.POST)
public String addEmployee(Employee employee){
    employeeDao.save(employee);
    return "redirect:/employee";
}
```

### 8、具体功能：跳转到更新数据页面

##### a>修改超链接

超链接方式为GET请求，寻找value值对应的控制器方法

```html
<a th:href="@{'/employee/'+${employee.id}}">update</a>
```

##### b>控制器方法

```java
@RequestMapping(value = "/employee/{id}", method = RequestMethod.GET)
public String getEmployeeById(@PathVariable("id") Integer id, Model model){
    Employee employee = employeeDao.get(id);
    model.addAttribute("employee", employee);
    return "employee_update";
}
```

##### c>创建employee_update.html

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Update Employee</title>
</head>
<body>

<form th:action="@{/employee}" method="post">
    <!--
		修改表单submit真正的请求方式为put，点击submit后可调用控制器方法
		value = "/employee",method = RequestMethod.PUT
	-->
    <input type="hidden" name="_method" value="put">
    <input type="hidden" name="id" th:value="${employee.id}">
    lastName:<input type="text" name="lastName" th:value="${employee.lastName}"><br>
    email:<input type="text" name="email" th:value="${employee.email}"><br>
    <!--
        th:field="${employee.gender}"可用于单选框或复选框的回显
        若单选框的value和employee.gender的值一致，则添加checked="checked"属性
    -->
    gender:<input type="radio" name="gender" value="1" th:field="${employee.gender}">male
    <input type="radio" name="gender" value="0" th:field="${employee.gender}">female<br>
    <input type="submit" value="update"><br>
</form>

</body>
</html>
```

### 9、具体功能：执行更新

##### a>控制器方法

```java
@RequestMapping(value = "/employee", method = RequestMethod.PUT)
public String updateEmployee(Employee employee){
    employeeDao.save(employee);
    return "redirect:/employee";
}
```
### 10、处理静态资源的过程

tomcat安装目录/config/web.xml和项目工程中的web.xml是全局和局部的关系，有冲突时就近原则。

##### a>默认的servlet

服务器启动时初始化

![image-20211111180211823](C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20211111180211823.png)

##### b>处理对jsp的访问

![image-20211111180329813](C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20211111180329813.png)

##### c>映射信息

默认的servlet的url-pattern是/，处理所有请求。与springMVC中dispatcherservlet配置的一样，冲突。

而dispatcherservlet会将请求地址拿去控制器中找对应的请求映射，但是控制器中没有写访问静态资源的请求，所以404。

![image-20211111180458427](C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20211111180458427.png)

因此需要在springMVC.xml中配置。即：若浏览器发送的请求dispatcherservlet处理不了时，交给default-servlet处理。

```xml
	<!--开放对静态资源的访问-->
    <mvc:default-servlet-handler></mvc:default-servlet-handler>
```
并且使用时需配合以下配置，否则所有请求都会由default-servlet处理，此时只有静态资源能访问，其它请求映射不能访问。
```xml
	<!--开启mvc注解驱动-->
    <mvc:annotation-driven></mvc:annotation-driven>
```


# 八、HttpMessageConverter

HttpMessageConverter，报文信息转换器，将请求报文转换为Java对象，或将Java对象转换为响应报文

HttpMessageConverter提供了两个注解和两个类型：@RequestBody，@ResponseBody，RequestEntity，

ResponseEntity

其中，两个响应用得多。因为请求信息获取很简单。

### 1、@RequestBody

@RequestBody可以获取请求体，需要在控制器方法设置一个形参，使用@RequestBody进行标识，当前请求的请求体就会为当前注解所标识的形参赋值

```html
<form th:action="@{/testRequestBody}" method="post">
    用户名：<input type="text" name="username"><br>
    密码：<input type="password" name="password"><br>
    <input type="submit">
</form>
```

```java
@RequestMapping("/testRequestBody")
public String testRequestBody(@RequestBody String requestBody){
    System.out.println("requestBody:"+requestBody);
    return "success";
}
```

输出结果：

requestBody:username=admin&password=123456

### 2、RequestEntity

RequestEntity封装请求报文的一种类型，需要在控制器方法的形参中设置该类型的形参，当前请求的请求报文就会赋值给该形参，可以通过getHeaders()获取请求头信息，通过getBody()获取请求体信息

```java
@RequestMapping("/testRequestEntity")
public String testRequestEntity(RequestEntity<String> requestEntity){
    System.out.println("requestHeader:"+requestEntity.getHeaders());//可以继续.getXxx
    System.out.println("requestBody:"+requestEntity.getBody());
    return "success";
}
```

输出结果：
requestHeader:[host:"localhost:8080", connection:"keep-alive", content-length:"27", cache-control:"max-age=0", sec-ch-ua:"" Not A;Brand";v="99", "Chromium";v="90", "Google Chrome";v="90"", sec-ch-ua-mobile:"?0", upgrade-insecure-requests:"1", origin:"http://localhost:8080", user-agent:"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/90.0.4430.93 Safari/537.36"]
requestBody:username=admin&password=123

### 3、@ResponseBody

@ResponseBody用于标识一个控制器方法，可以将该方法的返回值直接作为响应报文的**响应体**响应到浏览器

```java
@RequestMapping("/testResponseBody")
@ResponseBody
public String testResponseBody(){
    return "success";
}
```

结果：浏览器页面显示success

**注：与不加注解时的区别**

```html
<body>
<h1>success</h1>
</body>
```

不加注解时返回值代表：作为视图名称被视图控制器解析，加视图前后缀之后，匹配要跳转的**页面**；

<img src="C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20211117175132809.png" alt="image-20211117175132809" style="zoom:80%;" />

加注解返回值代表：响应到浏览器的响应体，即只设置了显示**文本**；

<img src="C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20211117175223001.png" alt="image-20211117175223001" style="zoom:80%;" />

```java
@RequestMapping("/testResponseBody")
//@ResponseBody
public String testResponseBody(){
    return "success";
}
```



@ResponseBody可返回Object(需将对象转换为json，否则报500错误)

```java
	@RequestMapping("/testResponseUser")
	@ResponseBody
	public User testResponseUser(){
        return new User(1001,"admin","123456",24,"男");
    }
```

见下

### 4、SpringMVC处理json

@ResponseBody处理json的步骤：

a>导入jackson的依赖(fastjson不行)

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.12.1</version>
</dependency>
```

b>在SpringMVC的核心配置文件中开启mvc的注解驱动，此时在HandlerAdaptor中会自动装配一个**消息转换器**：MappingJackson2HttpMessageConverter，可以将响应到浏览器的**Java对象**转换为**Json格式的字符串**(非对象)

```
<mvc:annotation-driven />
```

c>在处理器方法上使用@ResponseBody注解进行标识

d>将Java对象直接作为控制器方法的返回值返回，就会自动转换为Json格式的字符串

```java
@RequestMapping("/testResponseUser")
@ResponseBody
public User testResponseUser(){
    return new User(1001,"admin","123456",23,"男");
}
```

浏览器的页面中展示的结果：

{"id":1001,"username":"admin","password":"123456","age":23,"sex":"男"}



### 4.1	json回顾

大括号表示对象，中括号表示数组。区分后方便获取json中数据，get键/for循环。

实体类、map转换为json是json对象；list转换为json是json数组

微服务与微服务之间的交互就是HTTP+json，以后每一个控制器方法都要包含@ResponseBody，可直接在类上替换为@RestController(@ResponseBody+@Controller)

### 5、SpringMVC处理ajax

a>请求超链接：

```html
<div id="app">
	<a th:href="@{/testAjax}" @click="testAjax">testAjax</a><br>
</div>
```

b>通过vue和axios处理点击事件：

```html
<script type="text/javascript" th:src="@{/static/js/vue.js}"></script>
<script type="text/javascript" th:src="@{/static/js/axios.min.js}"></script>
<script type="text/javascript">
    var vue = new Vue({
        el:"#app",
        methods:{
            testAjax:function (event) {
                axios({
                    method:"post",
                    url:event.target.href,
                    params:{
                        username:"admin",
                        password:"123456"
                    }
                }).then(function (response) {
                    alert(response.data);
                });
                event.preventDefault();
            }
        }
    });
</script>
```

c>控制器方法：

```java
@RequestMapping("/testAjax")
@ResponseBody
public String testAjax(String username, String password){
    System.out.println("username:"+username+",password:"+password);
    return "hello,ajax";
}
```

### 6、@RestController注解

@RestController注解是springMVC提供的一个复合注解，标识在控制器的类上，就相当于为类添加了@Controller注解，并且为其中的每个方法添加了@ResponseBody注解

### 7、ResponseEntity

ResponseEntity用于控制器方法的返回值类型，该控制器方法的返回值就是响应到浏览器的响应报文，即**自定义响应报文**（包含响应状态码、响应头、响应体）。

主要用于实现文件上传下载功能。

# 九、文件上传和下载

### 1、文件下载

使用ResponseEntity实现下载文件的功能

```java
@RequestMapping("/testDown")
public ResponseEntity<byte[]> testResponseEntity(HttpSession session) throws IOException {
    //获取ServletContext对象
    ServletContext servletContext = session.getServletContext();
    //获取服务器中文件的真实路径，即在服务器上的部署路径。在target/模块名下
    String realPath = servletContext.getRealPath("/static/img/1.jpg");
    //创建输入流
    InputStream is = new FileInputStream(realPath);
    //创建字节数组，available()获取当前输入流的所有字节数
    byte[] bytes = new byte[is.available()];
    //将流读到字节数组中，bytes就是文件全部内容
    is.read(bytes);
    //创建HttpHeaders对象设置响应头信息，本质还是map
    MultiValueMap<String, String> headers = new HttpHeaders();
    //设置要下载方式（attachment以附件形式）以及下载文件的名字，add(键，值)
    headers.add("Content-Disposition", "attachment;filename=1.jpg");
    //设置响应状态码
    HttpStatus statusCode = HttpStatus.OK;
    //创建ResponseEntity对象
    ResponseEntity<byte[]> responseEntity = new ResponseEntity<>(bytes, headers, statusCode);
    //关闭输入流
    is.close();
    return responseEntity;
}
```

### 2、文件上传

文件上传要求form表单的请求方式必须为post，并且添加属性enctype="multipart/form-data"。

默认为enctype="application/x-www-form-urlencoded"。改后form表单中的数据不会通过name=value方式上传，而通过二进制上传。

SpringMVC中将上传的文件封装到MultipartFile对象中，通过此对象可以获取文件相关信息

上传步骤：

a>添加依赖：

```xml
<!-- https://mvnrepository.com/artifact/commons-fileupload/commons-fileupload -->
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.1</version>
</dependency>
```

b>在SpringMVC的配置文件中添加配置**文件上传解析器**：

将通过表单上传的文件转换成MultipartFile对象

```xml
<!--必须通过文件解析器的解析才能将文件转换为MultipartFile对象-->
<bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver"></bean>
```

c>控制器方法：

<img src="C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20211118113119436.png" alt="image-20211118113119436" style="zoom:80%;" />

```java
@RequestMapping("/testUp")
//形参名需要与页面中的上传文件名一致，如photo，并封装为MultipartFile对象
public String testUp(MultipartFile photo, HttpSession session) throws IOException {
    //获取上传的文件的文件名
    String fileName = photo.getOriginalFilename();
    //处理文件重名问题，没有会覆盖流
    //1.获取上传文件的后缀名.jpg，lastIndexOf最后一个.分隔的
    String hzName = fileName.substring(fileName.lastIndexOf("."));
    //2.将UUID作为文件名，如果不想要uuid中的-，可以.replaceAll("-","")
    fileName = UUID.randomUUID().toString() + hzName;
    //获取服务器中photo目录的绝对路径，如F:\Jarlin\Java\workspace0913\springmvc\springMVC-converter\target\springMVC-converter-1.0-SNAPSHOT\photo
    ServletContext servletContext = session.getServletContext();
    String photoPath = servletContext.getRealPath("photo");
    File file = new File(photoPath);
    //目录不存在先创建
    if(!file.exists()){
        file.mkdir();
    }
    //File.separator是文件分隔符，不知道是\还是/时候用这个
    String finalPath = photoPath + File.separator + fileName;
    //实现上传功能，传入一个含保存路径的File对象
    photo.transferTo(new File(finalPath));
    return "success";
}
```

# 十、拦截器

### 1、拦截器的配置

SpringMVC中的拦截器用于拦截**控制器方法**(controller中的方法)的执行

SpringMVC中的拦截器需要实现HandlerInterceptor

SpringMVC的拦截器必须在SpringMVC的配置文件中进行配置：

```xml
<!-- bean表示某个类型的对象就是一个拦截器 -->
<bean class="com.atguigu.interceptor.FirstInterceptor"></bean>
<!-- ref表示引用当前IOC容器中某个拦截器bean的id -->
<ref bean="firstInterceptor"></ref>
<!-- 以上两种配置方式都是对DispatcherServlet所处理的！！所有的！！请求进行拦截 -->

<mvc:interceptor>
    <!-- /**表示拦截多级目录，/*表示只拦截一层目录 -->
    <mvc:mapping path="/**"/>
    <mvc:exclude-mapping path="/testRequestEntity"/>
    <ref bean="firstInterceptor"></ref>
</mvc:interceptor>
<!-- 
	以上配置方式可以指定拦截规则。可以通过ref或bean标签设置拦截器，通过mvc:mapping设置需要拦截的请求路径，通过mvc:exclude-mapping设置需要排除的请求路径，即不需要拦截的请求，包含-排除=最后的
-->
```
### 1.1 拦截器的创建

```java
@Component//将当前类注册到IOC中，便于ref方式引用
public class FirstInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        return HandlerInterceptor.super.preHandle(request, response, handler);//返回false是拦截，true放行
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        HandlerInterceptor.super.postHandle(request, response, handler, modelAndView);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        HandlerInterceptor.super.afterCompletion(request, response, handler, ex);
    }
}
```



### 2、拦截器的三个抽象方法

SpringMVC中的拦截器有三个抽象方法：

在DispatcherServlet.java中

```java
				//控制器方法执行之前拦截
				if (!mappedHandler.applyPreHandle(processedRequest, response)) {
					return;
				}

				// Actually invoke the handler.控制器方法执行
				mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

				if (asyncManager.isConcurrentHandlingStarted()) {
					return;
				}

				applyDefaultViewName(processedRequest, mv);
				//在控制器方法执行之后拦截
				mappedHandler.applyPostHandle(processedRequest, response, mv);
			}
			catch (Exception ex) {
				dispatchException = ex;
			}
			catch (Throwable err) {
				// As of 4.3, we're processing Errors thrown from handler methods as well,
				// making them available for @ExceptionHandler methods and other scenarios.
				dispatchException = new NestedServletException("Handler dispatch failed", err);
			}
			//后续处理，处理模型数据并渲染视图
			processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
		}
```

1.preHandle：控制器方法执行之前执行preHandle()，其boolean类型的返回值表示是否拦截或放行，返回true为放行，即调用控制器方法；返回false表示拦截，即不调用控制器方法

2.postHandle：控制器方法执行之后执行postHandle()

在processDispatchResult方法中

```java
		if (mv != null && !mv.wasCleared()) {
            //渲染视图
			render(mv, request, response);
			if (errorView) {
				WebUtils.clearErrorRequestAttributes(request);
			}
		}
		else {
			if (logger.isTraceEnabled()) {
				logger.trace("No view rendering, null ModelAndView returned.");
			}
		}

		if (WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
			// Concurrent handling started during a forward
			return;
		}		

		if (mappedHandler != null) {
			// Exception (if any) is already handled..
            //	在视图渲染完毕之后拦截	
			mappedHandler.triggerAfterCompletion(request, response, null);
		}
```

3.afterCompletion：处理完视图和模型数据，渲染视图完毕之后执行afterCompletion()

### 2.1	过滤器与拦截器区别

过滤器在DispatcherServlet前端控制器之前，过滤请求/*；

拦截器在控制器方法执行前后拦截

<img src="C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20211118205445192.png" alt="image-20211118205445192" style="zoom: 33%;" />

### 3、多个拦截器的执行顺序

a>若每个拦截器的preHandle()都返回true

此时多个拦截器的执行顺序和拦截器在SpringMVC的配置文件的**配置顺序**有关：

preHandle()会按照配置的**顺序**执行，而postHandle()和afterCompletion()会按照配置的**反序**执行

b>若某个拦截器的preHandle()返回了false

preHandle()返回false和它之前的拦截器的preHandle()都会执行，postHandle()都**不执行**，返回false的拦截器之前的拦截器的afterCompletion()会执行（倒序）

源码：

```java
				//mappedHandler是控制器执行链，包含控制器方法及处理当前控制器方法的自定义n个拦截器+1个springMVC创建的拦截器
				if (!mappedHandler.applyPreHandle(processedRequest, response)) {
					return;
				}

				// Actually invoke the handler.
				mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

				if (asyncManager.isConcurrentHandlingStarted()) {
					return;
				}

				applyDefaultViewName(processedRequest, mv);
				mappedHandler.applyPostHandle(processedRequest, response, mv);
			}
```

```java
	boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) throws Exception {
		//遍历拦截器列表
        for (int i = 0; i < this.interceptorList.size(); i++) {
			HandlerInterceptor interceptor = this.interceptorList.get(i);
			if (!interceptor.preHandle(request, response, this.handler)) {
				triggerAfterCompletion(request, response, null);
				return false;
			}
            //mappedHandler的一个成员，记录当前的指向拦截器
			this.interceptorIndex = i;
		}
		return true;
	}
```

```java
	void applyPostHandle(HttpServletRequest request, HttpServletResponse response, @Nullable ModelAndView mv)
			throws Exception {
		//遍历拦截器列表，i--，从后往前
		for (int i = this.interceptorList.size() - 1; i >= 0; i--) {
			HandlerInterceptor interceptor = this.interceptorList.get(i);
			interceptor.postHandle(request, response, this.handler, mv);
		}
	}
```

```java
	void triggerAfterCompletion(HttpServletRequest request, HttpServletResponse response, @Nullable Exception ex) {
        //遍历拦截器放行的列表，这里的interceptorIndex就是之前最后通过的拦截器！
		for (int i = this.interceptorIndex; i >= 0; i--) {
			HandlerInterceptor interceptor = this.interceptorList.get(i);
			try {
				interceptor.afterCompletion(request, response, this.handler, ex);
			}
			catch (Throwable ex2) {
				logger.error("HandlerInterceptor.afterCompletion threw exception", ex2);
			}
		}
	}
```



# 十一、异常处理器

### 1、基于配置的异常处理

SpringMVC提供了一个处理控制器方法执行过程中所出现的异常的接口：HandlerExceptionResolver

HandlerExceptionResolver接口的实现类有：

DefaultHandlerExceptionResolver：默认

SimpleMappingExceptionResolver：自定义

<img src="C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20211119105009900.png" alt="image-20211119105009900" style="zoom:80%;" />

```java
//默认DefaultHandlerExceptionResolver
protected ModelAndView doResolveException(
	HttpServletRequest request, HttpServletResponse response, @Nullable Object handler, Exception ex) {
    //捕获异常
}

//可自定义SimpleMappingExceptionResolver
protected ModelAndView doResolveException(
	HttpServletRequest request, HttpServletResponse response, @Nullable Object handler, Exception ex) {

		// Expose ModelAndView for chosen error view.
		String viewName = determineViewName(ex, request);
		if (viewName != null) {
			// Apply HTTP status code for error views, if specified.
			// Only apply it if we're processing a top-level request.
			Integer statusCode = determineStatusCode(request, viewName);
			if (statusCode != null) {
				applyStatusCodeIfPossible(request, response, statusCode);
			}
			return getModelAndView(viewName, ex, request);
		}
		else {
			return null;
		}
	}
```

都是返回ModelAndView，一个新的视图页面。可自定义功能，如缺少指定页面不会跳转500而跳转自定义的页面。

SpringMVC提供了自定义的异常处理器SimpleMappingExceptionResolver，使用方式1.基于配置 2.基于注解：

```xml
<bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
    <property name="exceptionMappings">
        <props>
        	<!--
        		properties的键表示处理器方法执行过程中出现的异常(全类名)，如下是数学运算异常
        		properties的值表示若出现指定异常时，设置一个新的视图名称，跳转到指定页面
        	-->
            <prop key="java.lang.ArithmeticException">error</prop>
        </props>
    </property>
    <!--
    	exceptionAttribute属性设置一个属性名，将出现的异常信息在请求域中进行共享。
		value是键，错误信息是值，可在对应异常页面获取此异常信息。如下所示
    -->
    <property name="exceptionAttribute" value="ex"></property>
</bean>
```
```html
出现错误
<p th:text="${ex}"></p>
```

<img src="C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20211119111854731.png" alt="image-20211119111854731" style="zoom:80%;" />

### 2、基于注解的异常处理

```java
//@ControllerAdvice将当前类标识为异常处理的组件，包含@Controller的功能
@ControllerAdvice
public class ExceptionController {

    //@ExceptionHandler用于设置所标识方法处理的异常，value中写可能出现的异常类。
    //即当出现异常时，调用此控制器方法
    @ExceptionHandler(value = {ArithmeticException.class,NullPointerException.class})
    //ex表示当前请求处理中出现的异常对象
    public String handleArithmeticException(Exception ex, Model model){
        //此处的"ex"为键名，要与页面中对应键名相同
        model.addAttribute("ex", ex);
        return "error";
    }

}
```

# 十二、注解配置SpringMVC

使用配置类和注解**代替web.xml**和**SpringMVC配置文件**的功能

### 1.0 前置初始化

1）建工程

2）添加依赖(见二、2)+创建webapp/WEB-INF目录

3）配置tomcat上下文路径

<img src="C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20211119172102543.png" alt="image-20211119172102543" style="zoom: 50%;" />

<img src="C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20211119172126721.png" alt="image-20211119172126721" style="zoom:50%;" />

### 1、创建初始化类，代替web.xml

在Servlet3.0环境中，容器会在类路径(src、resource等)中查找实现javax.servlet.ServletContainerInitializer接口的类，如果找到的话就用它来配置Servlet容器(tomcat服务器)。
Spring提供了这个接口的实现，名为SpringServletContainerInitializer，这个类反过来又会查找实现WebApplicationInitializer的类并将配置的任务交给它们来完成。Spring3.2引入了一个便利的WebApplicationInitializer基础实现，名为AbstractAnnotationConfigDispatcherServletInitializer（抽象注解配置前端控制初始化器），当我们的类扩展了AbstractAnnotationConfigDispatcherServletInitializer并将其部署到Servlet3.0容器的时候，容器会自动发现它，并用它来配置Servlet上下文。

```java
public class WebInit extends AbstractAnnotationConfigDispatcherServletInitializer {

    /**
     * 指定spring的配置类
     * @return
     */
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{SpringConfig.class};
    }

    /**
     * 指定SpringMVC的配置类
     * @return
     */
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{WebConfig.class};
    }

    /**
     * 指定DispatcherServlet的映射规则，即web.xml中servlet-mapping下的url-pattern
     * @return
     */
    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }

    /**
     * 添加过滤器，把需要的过滤器创建出来放在数组里返回
     * @return
     */
    @Override
    protected Filter[] getServletFilters() {
        //编码过滤器
        CharacterEncodingFilter encodingFilter = new CharacterEncodingFilter();
        encodingFilter.setEncoding("UTF-8");
        encodingFilter.setForceRequestEncoding(true);
        //处理PUT、DELETE的HiddenHttpMethodFilter
        HiddenHttpMethodFilter hiddenHttpMethodFilter = new HiddenHttpMethodFilter();
        return new Filter[]{encodingFilter, hiddenHttpMethodFilter};
    }
}
```

### 2、创建SpringConfig配置类，代替spring的配置文件

```java
@Configuration
public class SpringConfig {
	//ssm整合之后，spring的配置信息写在此类中
}
```

### 3、创建WebConfig配置类，代替SpringMVC的配置文件

@Bean功能：作用在方法上，将方法的返回值作为一个Bean注册到IOC容器中

所有springMVC.xml中配置为<bean>的都可以用这个注解代替

```java
/**
 * 代替SpringMVC的配置文件
 * 1.扫描组件    2.视图解析器 3.静态资源访问default-servlet-handler(需实现WebMvcConfigurer接口)
 * 4.mvc注解驱动    5.拦截器   6.异常处理   7.视图控制器view-controller   8.文件上传解析器
 */
@Configuration							//将当前类标识为配置类
@ComponentScan("com.atguigu.mvc.controller")//1.扫描组件
@EnableWebMvc							//4.开启MVC注解驱动
public class WebConfig implements WebMvcConfigurer {

    //3.使用默认的servlet处理静态资源，即默认servlet可用
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    //8.配置文件上传解析器
    @Bean
    public CommonsMultipartResolver multipartResolver(){
        return new CommonsMultipartResolver();
    }

    //5.配置拦截器
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        FirstInterceptor firstInterceptor = new FirstInterceptor();
        registry.addInterceptor(firstInterceptor).addPathPatterns("/**");//这里可用对拦截器进行规则设置，可拦截多个也可排除多个，通过.方法
    }
    
    //7.配置视图控制器view-controller
    
    /*@Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/").setViewName("index");
    }*/
    
    //6.配置异常映射，对照xml文件中逐层配置，一共2个属性exceptionMappings异常映射、exceptionAttribute共享异常信息
    /*@Override
    public void configureHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
        SimpleMappingExceptionResolver exceptionResolver = new SimpleMappingExceptionResolver();
        
        Properties prop = new Properties();
        prop.setProperty("java.lang.ArithmeticException", "error");//此处不能用put，因为put放入Object类型，setProperty只能放String
        //设置异常映射
        exceptionResolver.setExceptionMappings(prop);
        //设置共享异常信息的键，在error.html中可以获取"ex"作为键，从请求域中获取异常信息
        exceptionResolver.setExceptionAttribute("ex");
        
        //将自定义的异常映射放入配置中
        resolvers.add(exceptionResolver);
    }*/

    //2.以下3个Bean合起来为Thymeleaf视图解析器
    //配置生成模板解析器
    @Bean
    public ITemplateResolver templateResolver() {
        WebApplicationContext webApplicationContext = ContextLoader.getCurrentWebApplicationContext();
        // ServletContextTemplateResolver需要一个ServletContext作为构造参数，可通过WebApplicationContext 的方法获得
        ServletContextTemplateResolver templateResolver = new ServletContextTemplateResolver(
                webApplicationContext.getServletContext());
        templateResolver.setPrefix("/WEB-INF/templates/");
        templateResolver.setSuffix(".html");
        templateResolver.setCharacterEncoding("UTF-8");
        templateResolver.setTemplateMode(TemplateMode.HTML);
        return templateResolver;
    }

    //生成模板引擎并为模板引擎注入模板解析器
    //ITemplateResolver参数通过自动装配，根据参数类型在IOC容器中找到对应的bean
    @Bean
    public SpringTemplateEngine templateEngine(ITemplateResolver templateResolver) {
        SpringTemplateEngine templateEngine = new SpringTemplateEngine();
        templateEngine.setTemplateResolver(templateResolver);
        return templateEngine;
    }

    //生成视图解析器并为解析器注入模板引擎
    @Bean
    public ViewResolver viewResolver(SpringTemplateEngine templateEngine) {
        ThymeleafViewResolver viewResolver = new ThymeleafViewResolver();
        viewResolver.setCharacterEncoding("UTF-8");
        viewResolver.setTemplateEngine(templateEngine);
        return viewResolver;
    }


}
```

### 4、测试功能

```java
@RequestMapping("/")
public String index(){
    return "index";
}
```

# 十三、SpringMVC执行流程

### 1、SpringMVC常用组件

- DispatcherServlet：**前端控制器**，不需要工程师开发，由框架提供

作用：统一处理请求和响应，整个流程控制的中心，由它调用其它组件处理用户的请求，以下5个组件都由前端控制器调用

- HandlerMapping：**处理器映射器**，不需要工程师开发，由框架提供，Mapping对应@RequestMapping

作用：根据请求的url、method等信息**查找**Handler（即Controller），即控制器方法

- Handler（即Controller）：**处理器**，需要工程师开发

作用：在DispatcherServlet的控制下Handler对具体的用户请求进行处理

- HandlerAdapter：**处理器适配器**，不需要工程师开发，由框架提供

作用：通过HandlerAdapter对处理器（控制器方法）进行执行，即**调用执行**对应的控制器方法

- ViewResolver：**视图解析器**，不需要工程师开发，由框架提供

作用：进行视图解析，得到相应的视图，例如：ThymeleafView、InternalResourceView、RedirectView

- View：**视图**

作用：将模型数据通过页面展示给用户

### 2、DispatcherServlet初始化过程

DispatcherServlet 本质上是一个 Servlet，所以天然的遵循 Servlet 的生命周期。所以宏观上是 Servlet 生命周期来进行调度。

其中，DispatcherServlet的9大组件在此过程中初始化。

先找Servlet中的init()方法，逐级向子类找看哪个往里写东西了。

![image-20211119224445822](C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20211119224445822.png)

从右到左找：

![images](img/img005.png)

##### a>初始化WebApplicationContext

即对springMVC的IOC容器进行初始化

所在类：org.springframework.web.servlet.FrameworkServlet，直接看33行

```java
protected WebApplicationContext initWebApplicationContext() {
    WebApplicationContext rootContext =
        WebApplicationContextUtils.getWebApplicationContext(getServletContext());
    WebApplicationContext wac = null;

    if (this.webApplicationContext != null) {
        // A context instance was injected at construction time -> use it
        wac = this.webApplicationContext;
        if (wac instanceof ConfigurableWebApplicationContext) {
            ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) wac;
            if (!cwac.isActive()) {
                // The context has not yet been refreshed -> provide services such as
                // setting the parent context, setting the application context id, etc
                if (cwac.getParent() == null) {
                    // The context instance was injected without an explicit parent -> set
                    // the root application context (if any; may be null) as the parent
                    cwac.setParent(rootContext);
                }
                configureAndRefreshWebApplicationContext(cwac);
            }
        }
    }
    if (wac == null) {
        // No context instance was injected at construction time -> see if one
        // has been registered in the servlet context. If one exists, it is assumed
        // that the parent context (if any) has already been set and that the
        // user has performed any initialization such as setting the context id
        wac = findWebApplicationContext();
    }
    if (wac == null) {
        // No context instance is defined for this servlet -> create a local one
        // 创建WebApplicationContext，springMVC的IOC容器，重点，见b>
        wac = createWebApplicationContext(rootContext);
    }

    if (!this.refreshEventReceived) {
        // Either the context is not a ConfigurableApplicationContext with refresh
        // support or the context injected at construction time had already been
        // refreshed -> trigger initial onRefresh manually here.
        synchronized (this.onRefreshMonitor) {
            // 触发刷新事件，刷新WebApplicationContext，见c>
            onRefresh(wac);
        }
    }

    if (this.publishContext) {
        // Publish the context as a servlet context attribute.
        // 获取ServletContext的属性名；获取ServletContext对象，将IOC容器在应用域共享。即可以获取springMVC的IOC容器！
        String attrName = getServletContextAttributeName();//全类名+".CONTEXT."+"DispatcherServlet",作为键
        getServletContext().setAttribute(attrName, wac);
    }

    return wac;
}
```
```java
	protected void onRefresh(ApplicationContext context) {
		// For subclasses: do nothing by default.去c>中找实现
	}
```

**总结**：先创建wac（springMVC的IOC容器），再刷新（调用子类初始化策略，初始化各个组件），最后将IOC容器共享到应用域中。

##### b>创建WebApplicationContext

会经过一次参数子类WebApplicationContext强转为父类ApplicationContext（只能调用父类方法，若子类重写过，调用子类实现）

```java
	protected WebApplicationContext createWebApplicationContext(@Nullable WebApplicationContext parent) {
		return createWebApplicationContext((ApplicationContext) parent);
	}
```

所在类：org.springframework.web.servlet.FrameworkServlet

```java
protected WebApplicationContext createWebApplicationContext(@Nullable ApplicationContext parent) {
    Class<?> contextClass = getContextClass();
    if (!ConfigurableWebApplicationContext.class.isAssignableFrom(contextClass)) {
        throw new ApplicationContextException(
            "Fatal initialization error in servlet with name '" + getServletName() +
            "': custom WebApplicationContext class [" + contextClass.getName() +
            "] is not of type ConfigurableWebApplicationContext");
    }
    // 通过反射创建 IOC 容器对象
    ConfigurableWebApplicationContext wac =
        (ConfigurableWebApplicationContext) BeanUtils.instantiateClass(contextClass);

    wac.setEnvironment(getEnvironment());
    // 设置父容器，spring和springMVC配置各管各的，分别加载自己的IOC容器。(不然维护很麻烦，配置地狱)
    // 设置完成后spring是父容器，springMVC是子容器
    // （弹幕）利用contextloaderListener，在tomcat工程启动，servletcontext初始化完后，先加载初始化spring的配置。然后dipatcherServlet初始化时，在这里从servletcontext获取父容器，设置给springmvc的容器。后续获取bean，在getBean方法中，会现在自己容器中找bean。找不到会到父容器中找。分成2个容器，子容器可以获取父容器的组件但是父容器不能拿子容器的组件
    wac.setParent(parent);
    String configLocation = getContextConfigLocation();
    if (configLocation != null) {
        wac.setConfigLocation(configLocation);
    }
    configureAndRefreshWebApplicationContext(wac);

    return wac;
    //继续看a>
}
```

##### c>DispatcherServlet初始化策略

```java
	//DispatcherServlet.java中重写的方法
	protected void onRefresh(ApplicationContext context) {
		initStrategies(context);
	}
```

FrameworkServlet创建WebApplicationContext后，刷新容器，调用onRefresh(wac)，此方法在DispatcherServlet中进行了重写，调用了initStrategies(context)方法，**初始化策略**，即初始化DispatcherServlet的各个组件

所在类：org.springframework.web.servlet.DispatcherServlet

```java
protected void initStrategies(ApplicationContext context) {
   initMultipartResolver(context);	//初始化文件上传解析器
   initLocaleResolver(context);
   initThemeResolver(context);		
   initHandlerMappings(context);	//初始化处理器映射器，将请求与控制器方法创建映射关系【找方法】
   initHandlerAdapters(context);	//初始化处理器适配器，可用来调用执行对应的控制器方法【调方法】
   initHandlerExceptionResolvers(context);//初始化异常处理器
   initRequestToViewNameTranslator(context);
   initViewResolvers(context);		//初始化视图解析器
   initFlashMapManager(context);
}
//看完在a>后看总结
```

### 3、DispatcherServlet调用组件处理请求

在2、中初始化了很多组件，下面是调用他们的过程。

源头是service()方法。![image-20211119224445822](C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20211119224445822.png)

从右到左找：

```java
	//FrameworkServlet中的service()
	protected void service(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {

		HttpMethod httpMethod = HttpMethod.resolve(request.getMethod());
        //请求方式
		if (httpMethod == HttpMethod.PATCH || httpMethod == null) {
			processRequest(request, response);
		}
		else {
            //调HttpServlet中的service，因为直接父类HttpServletBean没重写这个方法。
            //里面判断doXxx,调对应的方法(如果FrameworkServlet中重写过，调子类doXxx，否则调父类HttpServlet中doXxx。实际重写了，并且4个常用的doXxx方法里只有processRequest方法，所以只用看第8行即可，即a>)
			super.service(request, response);
		}
	}
```

##### a>processRequest()

FrameworkServlet重写HttpServlet中的service()和doXxx()，这些方法中调用了processRequest(request, response)

所在类：org.springframework.web.servlet.FrameworkServlet

```java
protected final void processRequest(HttpServletRequest request, HttpServletResponse response)
    throws ServletException, IOException {

    long startTime = System.currentTimeMillis();
    Throwable failureCause = null;

    LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
    LocaleContext localeContext = buildLocaleContext(request);

    RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
    ServletRequestAttributes requestAttributes = buildRequestAttributes(request, response, previousAttributes);

    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
    asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new RequestBindingInterceptor());

    initContextHolders(request, localeContext, requestAttributes);

    try {
		// 执行服务，doService()是一个抽象方法，在DispatcherServlet中进行了重写。用来处理请求和响应的
        // 为抽象方法，调用子类实现，见b>
        doService(request, response);
    }
    catch (ServletException | IOException ex) {
        failureCause = ex;
        throw ex;
    }
    catch (Throwable ex) {
        failureCause = ex;
        throw new NestedServletException("Request processing failed", ex);
    }

    finally {
        resetContextHolders(request, previousLocaleContext, previousAttributes);
        if (requestAttributes != null) {
            requestAttributes.requestCompleted();
        }
        logResult(request, response, failureCause, asyncManager);
        publishRequestHandledEvent(request, response, startTime, failureCause);
    }
}
```

##### b>doService()

所在类：org.springframework.web.servlet.DispatcherServlet

```java
@Override
protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
    logRequest(request);

    // Keep a snapshot of the request attributes in case of an include,
    // to be able to restore the original attributes after the include.
    Map<String, Object> attributesSnapshot = null;
    if (WebUtils.isIncludeRequest(request)) {
        attributesSnapshot = new HashMap<>();
        Enumeration<?> attrNames = request.getAttributeNames();
        while (attrNames.hasMoreElements()) {
            String attrName = (String) attrNames.nextElement();
            if (this.cleanupAfterInclude || attrName.startsWith(DEFAULT_STRATEGIES_PREFIX)) {
                attributesSnapshot.put(attrName, request.getAttribute(attrName));
            }
        }
    }

    // Make framework objects available to handlers and view objects.
    request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
    request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
    request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
    request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());

    if (this.flashMapManager != null) {
        FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
        if (inputFlashMap != null) {
            request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
        }
        request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
        request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);
    }

    RequestPath requestPath = null;
    if (this.parseRequestPath && !ServletRequestPathUtils.hasParsedRequestPath(request)) {
        requestPath = ServletRequestPathUtils.parseAndCache(request);
    }

    try {
        // 处理请求和响应，重点！见c>
        doDispatch(request, response);
    }
    finally {
        if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
            // Restore the original attribute snapshot, in case of an include.
            if (attributesSnapshot != null) {
                restoreAttributesAfterInclude(request, attributesSnapshot);
            }
        }
        if (requestPath != null) {
            ServletRequestPathUtils.clearParsedRequestPath(request);
        }
    }
}
```

##### c>doDispatch()（入口）

对当前请求进行统一处理

所在类：org.springframework.web.servlet.DispatcherServlet

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HttpServletRequest processedRequest = request;
    HandlerExecutionChain mappedHandler = null;//见下注释说明
    boolean multipartRequestParsed = false;

    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

    try {
        ModelAndView mv = null;
        Exception dispatchException = null;

        try {
            processedRequest = checkMultipart(request);
            multipartRequestParsed = (processedRequest != request);

            // Determine handler for the current request.
            /*重点
            	mappedHandler：调用链
                包含handler、interceptorList、interceptorIndex
            	handler：浏览器发送的请求所匹配的控制器方法
            	interceptorList：处理控制器方法的所有拦截器集合
            	interceptorIndex：拦截器索引，控制拦截器afterCompletion()的执行
            */
            mappedHandler = getHandler(processedRequest);
            if (mappedHandler == null) {
                noHandlerFound(processedRequest, response);
                return;
            }

            // Determine handler adapter for the current request.
           	// 通过控制器方法创建相应的处理器适配器，可用来调用所对应的控制器方法，此处不调用
            HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

            // Process last-modified header, if supported by the handler.
            String method = request.getMethod();
            boolean isGet = "GET".equals(method);
            if (isGet || "HEAD".equals(method)) {
                long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
                if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
                    return;
                }
            }
			
            // 调用拦截器的preHandle()
            if (!mappedHandler.applyPreHandle(processedRequest, response)) {
                return;
            }

            // Actually invoke the handler.
            // 由处理器适配器调用具体的控制器方法，最终获得ModelAndView对象
            // 此过程干了很多事情，如参数数据类型的转换，收集请求参数为形参赋值，形参能使用servletAPI为其赋值
            mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

            if (asyncManager.isConcurrentHandlingStarted()) {
                return;
            }

            applyDefaultViewName(processedRequest, mv);
            // 调用拦截器的postHandle()，反序执行，用size-1，不影响index
            mappedHandler.applyPostHandle(processedRequest, response, mv);
        }
        catch (Exception ex) {
            dispatchException = ex;
        }
        catch (Throwable err) {
            // As of 4.3, we're processing Errors thrown from handler methods as well,
            // making them available for @ExceptionHandler methods and other scenarios.
            dispatchException = new NestedServletException("Handler dispatch failed", err);
        }
        // 后续处理：处理模型数据和渲染视图，见d>
        processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
    }
    catch (Exception ex) {
        triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
    }
    catch (Throwable err) {
        triggerAfterCompletion(processedRequest, response, mappedHandler,
                               new NestedServletException("Handler processing failed", err));
    }
    finally {
        if (asyncManager.isConcurrentHandlingStarted()) {
            // Instead of postHandle and afterCompletion
            if (mappedHandler != null) {
                mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
            }
        }
        else {
            // Clean up any resources used by a multipart request.
            if (multipartRequestParsed) {
                cleanupMultipart(processedRequest);
            }
        }
    }
}
```

##### d>processDispatchResult()

```java
private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
                                   @Nullable HandlerExecutionChain mappedHandler, @Nullable ModelAndView mv,
                                   @Nullable Exception exception) throws Exception {

    boolean errorView = false;

    //此处处理上一层中最近的try-catch内的异常
    if (exception != null) {
        if (exception instanceof ModelAndViewDefiningException) {
            logger.debug("ModelAndViewDefiningException encountered", exception);
            mv = ((ModelAndViewDefiningException) exception).getModelAndView();
        }
        else {
            Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);
            mv = processHandlerException(request, response, handler, exception);
            errorView = (mv != null);
        }
    }

    // Did the handler return a view to render?
    if (mv != null && !mv.wasCleared()) {
        // 处理模型数据和渲染视图，如把模型数据放在请求域中，根据视图名称创建对应的视图
        render(mv, request, response);
        if (errorView) {
            WebUtils.clearErrorRequestAttributes(request);
        }
    }
    else {
        if (logger.isTraceEnabled()) {
            logger.trace("No view rendering, null ModelAndView returned.");
        }
    }

    if (WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
        // Concurrent handling started during a forward
        return;
    }

    if (mappedHandler != null) {
        // Exception (if any) is already handled..
        // 调用拦截器的afterCompletion()，反序执行，用index
        mappedHandler.triggerAfterCompletion(request, response, null);
    }
}
```
### 4、SpringMVC的执行流程

1) 用户向服务器发送请求，请求被SpringMVC 前端控制器 DispatcherServlet捕获。

2) DispatcherServlet对请求URL进行解析，得到请求资源标识符（URI），判断请求URI对应的映射：

##### a) 不存在

i. 再判断是否配置了mvc:default-servlet-handler

ii. 如果没配置，则控制台报映射查找不到，客户端展示404错误

![image-20210709214911404](img/img006.png)

![image-20210709214947432](img/img007.png)

iii. 如果有配置，则访问目标资源（一般为静态资源，如：JS,CSS,HTML），找不到客户端也会展示404错误

（区别一个是DispatcherServlet，一个是DefaultServletHttpRequestHandler）

![image-20210709215255693](img/img008.png)

![image-20210709215336097](img/img009.png)

##### b) 存在则执行下面的流程

3) 根据该URI，调用HandlerMapping获得该Handler配置的所有相关的对象（包括Handler对象以及Handler对象对应的拦截器，及拦截器索引），最后以HandlerExecutionChain执行链对象的形式返回。

```java
	mappedHandler = getHandler(processedRequest);
```

4) DispatcherServlet 根据获得的Handler，选择一个合适的HandlerAdapter(如RequestMappingHandlerAdapter)。

```java
	HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
```

5) 如果成功获得HandlerAdapter，此时将开始执行拦截器的preHandler(…)方法【正向】

```java
if (!mappedHandler.applyPreHandle(processedRequest, response)) {
	return;
}
```

6) **提取Request中的模型数据，填充Handler(Controller)入参**，开始执行Handler（Controller)方法，处理请求。在填充Handler的入参过程中，根据你的配置，Spring将帮你做一些额外的工作：

a) HttpMessageConveter： 将请求消息（如Json、xml等数据）转换成一个对象，将对象转换为指定的响应信息

b) 数据转换：对请求消息进行数据转换。如String转换成Integer、Double等

c) 数据格式化：对请求消息进行数据格式化。 如将字符串转换成格式化数字或格式化日期等

d) 数据验证： 验证数据的有效性（长度、格式等），验证结果存储到BindingResult或Error中

7) Handler执行完成后，向DispatcherServlet 返回一个ModelAndView对象。

```java
// Actually invoke the handler.
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
```

8) 此时将开始执行拦截器的postHandle(...)方法【逆向】。

```java
mappedHandler.applyPostHandle(processedRequest, response, mv);
```

9) 根据返回的ModelAndView（此时会判断是否存在异常：如果存在异常，则执行**HandlerExceptionResolver**进行异常处理，即WebConfig.java中的自定义异常处理器）选择一个适合的ViewResolver进行视图解析，根据Model和View，来渲染视图。

```java
		//处理异常		
		if (exception != null) {
			if (exception instanceof ModelAndViewDefiningException) {
				logger.debug("ModelAndViewDefiningException encountered", exception);
				mv = ((ModelAndViewDefiningException) exception).getModelAndView();
			}
			else {
                //mappedHandler是控制器执行链，包含控制器方法及处理当前控制器方法的自定义n个拦截器+1个springMVC创建的拦截器
				Object handler = (mappedHandler != null ? mappedHandler.getHandler() : null);
				mv = processHandlerException(request, response, handler, exception);
				errorView = (mv != null);
			}
		}

		// Did the handler return a view to render?
		if (mv != null && !mv.wasCleared()) {
            //处理模型数据和渲染视图，如把模型数据放在请求域中，根据视图名称创建对应的视图
			render(mv, request, response);
			if (errorView) {
				WebUtils.clearErrorRequestAttributes(request);
			}
		}
		else {
			if (logger.isTraceEnabled()) {
				logger.trace("No view rendering, null ModelAndView returned.");
			}
		}
```

10) 渲染视图完毕执行拦截器的afterCompletion(…)方法【逆向】。

```java
		if (mappedHandler != null) {
			// Exception (if any) is already handled..
			mappedHandler.triggerAfterCompletion(request, response, null);
		}
```

11) 将渲染结果返回给客户端。



# 十四、IDEA快捷键

###  1.Alt+7查看类的属性

<img src="C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20210919122037150.png" alt="image-20210919122037150" style="zoom: 80%;" />

判断方法是重写还是重载：看方法旁有没有向上箭头。有箭头的是重写，没箭头的是重载。

<img src="C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20211119202614051.png" alt="image-20211119202614051" style="zoom: 80%;" />

### 2.F9跳过断点

<img src="C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20210928182918004.png" alt="image-20210928182918004" style="zoom:67%;" />

### 3.Alt+Insert生成构造器、get、set等

<img src="C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20211117175745835.png" alt="image-20211117175745835" style="zoom: 80%;" />

### 4.Ctrl+O重写方法

<img src="C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20211118211215559.png" alt="image-20211118211215559" style="zoom: 67%;" />

### 5.Ctrl+H查看继承实现关系

<img src="C:\Users\Jarlin\AppData\Roaming\Typora\typora-user-images\image-20211119104856982.png" alt="image-20211119104856982" style="zoom: 80%;" />