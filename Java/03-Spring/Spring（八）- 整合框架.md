# Spring整合框架

## 整合思路

### 配置思路

* 一般都是使用Spring整合表现层和持久层框架
* 配置方式为使用配置文件（复杂的配置，如数据池，AOP切面等） + 注解（简单配置，注册bean，注入参数等）

### 整合方向

* 首先需要保证每个框架都可以单独运行，即需要单独配置好各个框架的配置文件
* 一般的整合都是先表现层再持久层

---

## Spring整合SpringMVC

1. 目的: 在Controller中能成功调用Service中的方法
2. 步骤
   * 启动Tomcat服务器时，需要加载Spring的配置文件
     * 使用ServletContext监听器的初始化方法中加载Spring的配置文件
     * 当使用时可以从ServletContext域中取出Spring容器
3. 注意: 由于Spring自身提供的ContextLoaderListener监听类默认只会加载WEB-INF目录下的applicationContext.xml配置文件，所以在通常情况下我们会传入context参数使其加载类文件夹下的配置文件

```XML
<!-- 配置Spring的监听器 -->
<listener>
    <listener-class>
        org.springframework.web.context.ContextLoaderListener
    </listener-class>
</listener>
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:applicationContext.xml</param-value>
</context-param>
```

---

## Spring整合Hibernate

ha

## Spring整合SpringJDBC

6
