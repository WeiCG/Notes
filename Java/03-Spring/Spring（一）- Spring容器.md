# Spring容器

> 依赖注入是Spring的核心机制之一，而Spring容器便是管理依赖的核心

## 依赖注入的概念

- 在Java程序的编写中，不论是桌面应用还是Web应用，都大量存在着A对象需要调用B对象的情形，这种情形被Spring成为依赖，即A对象的实现依赖于B对象
- 对于Java编写的所有应用中，他们总是由一些互相调用的对象构成，Spring把这种互相调用的关系称为依赖关系
- Spring容器作为一个超级大工厂，负责创建、管理所有的Java对象，同时也**使用“依赖注入”的方式来管理这些对象之间的依赖关系**

---

## 传统模式的做法

- 调用者主动创建被依赖对象，即使用"new"关键字
- 使用简单工厂模式，通过工厂来获取被依赖对象，但仍然是主动获取

---

## Spring的做法

- 调用者无需主动获取被依赖对象，只要被动接受Spring容器为调用者的成员变量赋值即可
- 需要注意的是，**依赖注入（DI）和控制反转（IoC）这两个术语的意义完全相同**，只不过是看待问题的角度不同
    - 控制反转是站在调用者的角度，不需要主动去获取被依赖对象，而将获取的权利交给Spring容器，只需被动接受
    - 依赖注入是站在Spring容器的角度，容器负责将被依赖对象赋值给调用者的成员变量，即为调用者注入依赖的对象实例
- Spring使用配置文件的方式来管理这些Java对象

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
               http://www.springframework.org/schema/beans/spring-beans.xsd">
  
</beans>
```

---

## 依赖的注入的方式

### 值注入

- IoC容器使用成员变量的set方法来注入被依赖对象
- 要求: 在调用者类中需要提供被依赖类对应成员变量的set方法
- 优点: 简单、直观
- 标签: 使用bean标签中的**property标签**

```xml
<bean id="调用者的唯一id" class="调用者的全路径">
    <property name="被依赖对象在调用者类中的名称" ref="被依赖类的唯一id" />
    <property name="调用者类中的其他基本属性" value="基本属性的值" />
</bean>
<bean id="被依赖类的唯一id" class="被依赖类的全路径" />
```

### 构造注入

- IoC容器使用构造方法来注入被依赖对象
- 要求: 在调用者类中需要提供带参的构造方法
- 优点: 所有依赖关系都在构造器中设定，没有set方法，不用担心后续代码对依赖关系的破坏
- 标签:bean标签下的constructor-arg标签

```xml
<bean id="调用者的唯一id" class="调用者的全路径">
    <constructor-arg ref="被依赖类的唯一id" type="被依赖的全路径" />
    <constructor-arg value="基本属性的值" type="基本属性的类型" />
</bean>
<bean id="被依赖类的唯一id" class="被依赖类的全路径" />
```

### 更多

- **详见Spring（二）中的配置依赖**

---

## Spring容器的基本要点

- 项目应面向接口编程，有利于项目后期的维护和扩展
- 各组件不再由程序主动创建，而是通过Spring容器负责产生并初始化
- 采用配置文件或注解来管理Bean的实现类、依赖关系

---

## 使用Spring容器

- Spring容器有两个核心接口: **BeanFactory和ApplicationContext**，其中ApplicationContext是BeanFactory的子接口

### 两个接口的区别

- ApplicationContext在创建时，会初始化容器中所有的单例对象，包括对象的依赖
- BeanFactory要等到程序需要Bean实例时才创建Bean

### BeanFactory接口

- BeanFactory负责配置、创建和管理Bean
- BeanFactory的常用实现类是DafaultListableBeanFactory

|    方法名    |            参数            |  返回值   | 说明                                     |
| :----------: | :------------------------: | :-------: | ---------------------------------------- |
| containsBean |        String name         |  boolean  | 判断容器中是否包含id为name的Bean实例     |
|   getBean    |       Class<T> type        |     T     | 获取容器中type类型的、唯一的Bean实例     |
|   getBean    |        String name         |  Object   | 获取容器中id为name的Bean实例             |
|   getBean    | String name, Class<T> type |     T     | 获取容器中id为name，类型为type的Bean实例 |
|   getType    |        String name         | Class <T> | 获取容器中id为name的Bean实例的类型       |

### ApplicationContext接口

- **一般情况下，都是使用ApplicationContext实例操作容器的**
- ApplicationContext作为BeanFactory的子接口，除了继承了所有BeanFactory已有的功能外，还提供了更加强大的功能

|      ApplicationContext实现类      | 说明                                               |
| :--------------------------------: | -------------------------------------------------- |
|   ClassPathXmlApplicationContext   | 加载类路径下的配置文件，要求配置文件必须在类路径下 |
|  FileSystemXmlApplicationContext   | 加载磁盘任意路径下的配置文件（必须要有访问权限）   |
| AnnotationConfigApplicationContext | 读取Java配置类创建容器，一般用的不多               |

### ApplicationContext的事件机制

- 事件机制包括事件源、事件和事件监听器组成

#### 事件源

- 在Spring中配置事件需要在Java类中实现ApplicationEvent接口

#### 事件监听器

- **在Spring中配置事件监听器需要实现ApplicationListener接口**
- 该接口实现onApplicationEvent方法，Spring容器会将发生的事件传入方法中

#### 事件

- 在Spring配置文件中配置监听器

```xml
<bean class="监听器的全路径" />
```

- **在调用中使用ApplicationContext对象的publishEvent方法触发事件**

#### 注意

- **在实际开发中，如果希望触发Bean实例所对应的的时间，必须首先先让Bean获得Spring容器，详见下一节**

### ApplicationContext的国际化支持

- ApplicationContext接口继承了MessageSource接口，具有国际化功能
- 当程序创建ApplicationContext容器时，会自动查找一个**名为"messageSource"的Bean**，就表明程序中指定了国际化资源，当调用MessageSource接口中的方法时，便会自动寻找类路径下的资源

- MessageSource中定义了两个getMessage方法来获取国际化资源，而ApplicationContext接口继承了该接口，所以使用ApplicationContext对象来调用该方法

```xml
<bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
	<property name="basename">
    	<list>
        	<!-- 如果有多个资源文件 -->
            <value>xxx</value>
        </list>
    </property>
</bean>
```

- **注意: 在使用ApplicationContext的国际化支持时，必须首先需要使用ApplicationContext对象调用getMessage方法，所以我们需要在Bean的内部来获取到ApplicationContext对象本身**

### 让Bean获取Spring容器

- **让Java类实现ApplicationContextAware接口**，Spring容器会检测容器中的所有Bean，如果发现某个Bean实现了该接口，则会自动调用Bean中的setApplicationContextAware方法将容器本身注入
- 使用ApplicationContextAware接口后，配置文件中不需要显示指定注入变量，Spring容器会自动将本身注入