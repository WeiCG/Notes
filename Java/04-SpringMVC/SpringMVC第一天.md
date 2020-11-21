# SpringMVC学习第一天

## 入门案例

### 搭建开发环境，即导包

* spring-context
* spirng-web
* spring-webmvc
* servlet-api
* jsp-api

### 编写入门程序

1. 首先需要在web.xml文件中配置Servlet，表示所有的请求都要通过SpringMVC
2. 创建SpringMVC的配置文件
3. 在配置文件中开启注解扫描
4. 在Controller类中写上对应的注解
    - Comtroller - 将控制器交给Spring容器管理
    - RequestMapping - 处理请求的方法
        - path属性: 设置请求路径，**注意要加上/**

5. 加载Spring配置文件 - 在web.xml的servlet标签中设置 &lt;init-param&gt;中标签设置读取xml配置文件

```XML
 <!-- web.xml文件 -->
 <!-- 配置UTF-8编码的过滤器 -->
 <filter>
     <filter-name>characterEncodingFilter</filter-name>
     <filter-class>
         org.springframework.web.filter.CharacterEncodingFilter
     </filter-class>
     <init-param>
         <param-name>encoding</param-name>
         <param-value>UTF-8</param-value>
     </init-param>
 </filter>
 <filter-mapping>
     <filter-name>characterEncodingFilter</filter-name>
     <url-pattern>/*</url-pattern>
 </filter-mapping>

 <!-- 配置Spring核心Servlet -->
 <servlet>
     <servlet-name>dispatcherServlet</servlet-name>
     <servlet-class>
         org.springframework.web.servlet.DispatcherServlet
     </servlet-class>
     <init-param>
         <param-name>contextConfigLocation</param-name>
         <param-value>classpath:springmvc.xml</param-value>
     </init-param>
     <load-on-startup>1</load-on-startup>
 </servlet>
 <servlet-mapping>
     <servlet-name>dispatcherServlet</servlet-name>
     <url-pattern>/</url-pattern>
 </servlet-mapping>
```

6. 在Spring的配置文件中配置视图解析器

```XML
<!-- springmvc配置文件 -->
<?xml version="1.0" encoding="UTF-8"?>
 <beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xsi:schemaLocation="
     http://www.springframework.org/schema/beans
     http://www.springframework.org/schema/beans/spring-beans.xsd
     http://www.springframework.org/schema/context
     http://www.springframework.org/schema/context/spring-context.xsd
     http://www.springframework.org/schema/mvc
     http://www.springframework.org/schema/mvc/spring-mvc.xsd">

     <!-- 开启注解扫描 -->
     <context:component-scan base-package="包路径" />
     <!--开启SpringMVC框架注解的支持-->
     <mvc:annotation-driven />

     <!--配置视图解析器-->
     <bean id="internalResourceViewResolver" class=
     "org.springframework.web.servlet.view.InternalResourceViewResolver">
         <!-- 配置页面前缀 -->
         <property name="prefix" value="/WEB-INF/pages/" />
         <!-- 配置页面后缀 -->
         <property name="suffix" value=".jsp" />
     </bean>
 </beans>
```

---

## Spring MVC组件介绍

- 前端控制器 - DispatcherServlet

    相当于MVC模式中的C，是流程控制的中心，由它调用其他组件处理用户的请求，降低了各组件之间的耦合度

- 处理器映射器 - HandlerMapping

    根据用户请求找到Handler

- 处理器 - Handler

    具体编写的业务控制器，使用前端控制器把用户请求转发到Handler。由Handler对用户的请求进行处理

- 处理器适配器 - HandlAdapter

- 视图解析器 - View Resolver

    将处理结果生成View视图

- View - 视图

---

## RequestMapping注解

- 将控制器与处理器映射器对应

- 属性
    - path - 指定映射路径
    - value - 同path
    - method - 当前方法可以接收什么样的请求方式，类型是个枚举类RequestMethod
    - params - 限制请求参数
      - params = {"username"} 表示请求当前方法必须包含一个username的参数
      - params = {"username=haha"} 表示请求当前方法时username参数的值必须是haha
    - headers - 限制请求消息头
      - headers = {"Accept"} 表示请求当前方法必须要包含Accept的请求头

---

## 请求参数的绑定

- 绑定机制

    只要请求参数的名字，即键值对中的k，与处理器方法的参数名称相同，则Spring容器会自动将数据注入到参数中去

- 支持的数据类型
    - 基本数据类型和String类型
    - 实体类型，即JavaBean
    - 集合类型

- 基本数据类型和String类型

    直接注入即可，注入时名称区分大小写

- 实体类型
    - 请求参数的名字需要与JavaBean中属性的名字相同
    - 如果该JavaBean类中还包含其他的引用类型，则请求参数的名称需要写成对象.属性

    -集合类型

    - 使用**list[0].对象中的属性名**来表示生成一个对象，将请求参数注入该对象的同名属性中，再将对象封装到一个list集合中去
    - Map和List同理，Map的[]中放入对应Map对应的key值

- **特殊情况 - 当数据类型转换出错后使用自定义数据转换**
    - 实现Spring提供的接口Converter
    - 在Spring配置文件中配置自定义类型转换器

```XML
<!--开启SpringMVC框架注解的支持-->
<mvc:annotation-driven conversion-service="conversionService" />

<!--配置Spring自定义类型转换器-->
<bean id="conversionService" class=
"org.springframework.context.support.ConversionServiceFactoryBean">
    <property name="converters">
        <set>
            <bean class="com.wei.utils.StringToDateConverter" />
        </set>
    </property>
</bean>
```

---

## 配置中文乱码的过滤器

- 在web.xml中配置Spring提供的一个过滤器，并在filter中配置初始化参数为UTF-8

```XML
<filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>
        org.springframework.web.filter.CharacterEncodingFilter
    </filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

---

## 在Spring MVC中拿到Servlet-API

- **直接在处理器方法参数中写入HttpServletRequest和HttpServletResponse对象即可**

```Java
// 原生的API获取
@RequestMapping("testServlet")
public String testServlet(HttpServletRequest request,HttpServletResponse response) {
        System.out.println("执行了");
        System.out.println(request);
        HttpSession session = request.getSession();
        System.out.println(session);
        return "success";
    }
```

---

## 常用注解

### RequestParam

* 作用: 把请求中指定名称的参数给处理器中的形参赋值
* 属性
  * value - 请求参数中的名称
  * required - 请求参数中是否必须提供此参数，默认为true

### RequestBody - 常用在异步过程中

* 作用: 用于获取请求体的内容，直接使用的到key=value&key=value...结构的数据
* 限制: get请求不适应，因为get请求没有请求体
* 属性: required - 是否必须有请求体，默认为true

### PathVariable

* 作用: 用于绑定url中的占位符
* 属性
  * value - 指定url中占位符的名称
  * required - 是否必须提供占位符，默认为true

### RequestHeader - 用的较少

* 作用: 获取请求消息头
* 属性
  * value - 请求头的名称
  * required - 是否必须有此请求头，默认为true

### CookieValue - 用的较少

* 作用: 把指定的cookie名称的值传入方法参数
* 属性
  * value - 指定cookie的名称
  * required - 是否必须有此cookie，默认为true

### ModelAttribute

* 作用
  * 出现在方法上，表示当前方法会在控制器的方法执行之前先执行
  * 出现在参数上，获取指定的数据给参数赋值
* 属性
  * value: 用于获取数据的key
* 应用场景: 当表单提交的数据不是完整的实体类数据时，保证没有提交数据的字段使用数据库对象原本的数据

### SessionAttribute(s)

* 作用: 用于多次执行控制器方法间的参数共享
* 属性
  * value - 用于指定存入的属性名称
  * type - 用于指定存入的数据类型

---

## HiddentHttpMethodFilter

- 作用: Spring提供的一个过滤器，用于测试HTTP协议中的DELETE、PUT等方法，可以将浏览器请求改为指定的请求方式发送到控制器中

- 使用方法
    1. 在web.xml中配置该过滤器
    2. 请求方式必须是post
    3. 按照要求**提供_method请求参数**的值，该参数的取值就是我们需要的请求方式