# SpringMVC第二天

## 响应数据和结果视图

### 返回值分类

* 返回字符串
  * 直接根据返回值通过视图解析器返回到对应的jsp页面
  * 通过Model对象将需要传递的对象添加到request作用域
* 返回void
  * 使用request的转发
  * 使用response的重定向
  * 直接使用response来响应
* 返回ModelAndView对象
  * 这是Spring框架提供的对象，可以用来调整具体的JSP视图
  * 直接在方法中创建ModelAndView对象
  * 使用addObject方法可以将属性存储到该对象中，之后传递给JSP页面
  * 同时还可以使用setViewName方法在对象中设置要跳转到哪个页面，该方法的跳转原理与返回字符串的情况相同
  
### 转发和重定向

* SpringMVC在返回值为字符串时，默认为请求转发
* 同时，SpringMVC提供了forward和redirect两个关键字，表示转发和重定向
* 当使用关键字时，必须写入实际的url，Spring不会调用视图解析器

```Java
 // 转发方式，Spring默认方法
return "forward:/WEB-INF/pages/xxx.jsp";
// 重定向，不需要加项目名，Spring会默认加上
return "redirect:/pages/xxx.jsp ";
```

### ResponseBody响应json数据

* 在Spring配置文件中设置静态文件不过滤，即不拦截关于js/css/image等文件的请求

```XML
<!--告诉前端控制器，哪些静态资源不拦截-->
<mvc:resources mapping="/js/**" location="/js/" />
<mvc:resources mapping="/css/**" location="/css/" />
<mvc:resources mapping="/images/**" location="/images/" />
```

* 首先使用RequestBody注解获取到请求体的内容，Spring可以直接注入对应的对象中
* 如果要返回json数据，而SpringMVC默认使用MappingJacksonHttpMessageConverter对json数据进行转换，所以需要加入jackson的jar包（**2.7以上，2.7及2.7以下无法使用**）
* 在返回值处使用ResponseBody注解，Spring会自动将返回对象转换为json对象

---

## SpringMVC实现文件上传

- **Spring默认提供的文件上传需要commons-fileupload和commons-io组件的支持**

### 单服务器文件上传

1. 在Spring配置文件中配置文件解析器

    ```XML
    <!--配置文件解析器-->
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <!--配置文件最大大小-->
        <property name="maxUploadSize" value="10485760" />
    </bean>
    ```

2. Spring的文件解析器会自动将浏览器传递过来的请求解析，从中分离出上传文件的部分
3. Spring会自动将分离出的文件部分注入处理方法的**MultipartFile参数**，参数名称必须与表单中上传文件的名称相同

### 跨服务器文件上传

- SUN公司提供了一组名为jersey-client和jersey-core的jar包，Spring实现跨服务器的文件上传需要这些jar包的支持

```Java
@RequestMapping("fileupload")
public String fileupload(MultipartFile upload) throws IOException {
    // 定义上传文件服务器的路径
    String path = "http://localhost:9090/fileupload/uploads/";  // 对应服务器的存储路径

    // 获取上传文件的名称
    String filename = upload.getOriginalFilename();
    String uuid = UUID.randomUUID().toString().replace("-", "");
    filename = uuid + "_" + filename;
    // 创建客户端对象
    Client client = Client.create(); // jersey包中的类
    // 与图片服务器进行连接
    WebResource webResource = client.resource(path + filename);
    // 使用二进制形式将对象传到文件服务器中
    webResource.put(upload.getBytes());
    return "success";
}
```

* 错误:
  * 409 - 对应的服务器存储路径错误
  * 403 - Tomcat服务器被设置为只读

---

## SpringMVC异常处理

### 处理思路

- 在实际开发中，都是Controller调用Service，Service调用Dao，异常全部向上抛出，最终所有的异常都是被前端控制器找异常处理器处理的

### SpringMVC的异常处理

#### 自定义异常类 - 做提示信息

```Java
/**
 * 自定义异常类
 */
public class SysException extends Exception {  // 继承Java的Exception
    // 存储提示信息
    private String message;
    public SysException(String message) {
        this.message = message;
    }

    message的Getter和Setter方法
}
```

#### 编写异常处理器 - 处理异常的业务逻辑

异常处理器需要实现Spring提供的HandlerExceptionResolver接口，该接口只有一个resolveException的方法

* 方法参数
  * HttpServletRequest - 请求
  * HttpServletResponse - 响应
  * Object - 当前处理的对象
  * Exception - 当前抛出的异常对象
* 返回值
  * ModelAndView

```Java
/**
 * 异常处理器对象
 */
public class SysExceptionResolver implements HandlerExceptionResolver {
    @Override
    public ModelAndView resolveException(HttpServletRequest request HttpServletResponse response, Object handler, Exception ex) {
        // 获取到异常对象
        SysException e = null;
        if(ex instanceof SysException) {
            e = (SysException)ex;
        } else {
            e = new SysException("系统正在维护...");
        }

        // 创建ModelAndView对象
        ModelAndView mv = new ModelAndView();
        mv.addObject("error", e.getMessage());
        mv.setViewName("error");  // 跳转到error.jsp页面
        return mv;
    }
}
```

#### 配置异常处理器 - 跳转到友好提示页面

```XML
<!--配置异常处理器-->
<!-- 只需配置一个单纯的bean即可 -->
<bean id="sysExceptionResolver" class="com.wei.exception.SysExceptionResolver" />
```

---

## SpringMVC拦截器

### 拦截器的作用

SpringMVC的拦截器类似于Servlet开发过程中的Filter，用于对处理器进行预处理和后处理

### 步骤

1. 编写拦截器类 - 需要实现HandlerInterceptor方法
   * preHandle - 预处理方法，控制器方法执行前
     * 参数
       * HttpServletRequest - 请求
       * HttpServletResponse - 响应
       * Object -
     * 返回值: boolean
       * true - 放行，执行下一个拦截器或执行控制器方法
       * false - 不放行，使用request或response跳到提示页面
   * postHandle - 后处理方法，控制器方法执行后，跳转页面前
     * 参数
       * HttpServletRequest - 请求
       * HttpServletResponse - 响应
       * Object -
       * ModelAndView - 方法返回的参数和视图
     * 返回值: void
   * afterCompletion - 跳转页面后
     * 参数
       * HttpServletRequest - 请求
       * HttpServletResponse - 响应
       * Object -
       * Exception - 方法抛出的异常
     * 返回值: void
2. 配置拦截器
   - 在Spring的配置文件中配置拦截器
   - 多个拦截器执行顺序为在<mvc:interceptors>标签中的配置顺序

```XML
<!--配置拦截器-->
<mvc:interceptors>
    <!--配置第一个拦截器-->
    <mvc:interceptor>
        <!--要拦截的具体方法-->
        <mvc:mapping path="拦截路径" />
        <!--配置拦截器对象-->
        <bean class="第一步写的拦截器类" />
    </mvc:interceptor>
    <!--配置第二个拦截器-->
    <mvc:interceptor>
        <mvc:mapping path="拦截路径" />
        <bean class="com.wei.interceptor.MyInterceptor2" />
    </mvc:interceptor>
</mvc:interceptors>
```
