# Spring的注解配置

> 现如今，越来越多的项目都是注解为主，XML为辅

## 开启注解

- 使用Spring的注解配置时，需要现在Spring中开启注入扫描

```xml
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
		   http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context.xsd">
	<!-- 开启扫描 --> 
    <context:component-scan base-package="包路径" />
</beans>
```

- context:component-scan包含有两个子标签

    - include-filter - 强制Spring将Java类转换为Bean对象，即使这些类没有使用注解修饰

    - exclude-filter - 强制将某些注解修饰的Bean对象排除在扫描范围外

        |    属性    |           说明           |
        | :--------: | :----------------------: |
        |    type    |      指定过滤器类型      |
        | expression | 指定过滤器所需要的表达式 |

        | type属性可选值 | 说明                                             |
        | :------------: | ------------------------------------------------ |
        |   annotation   | 注解过滤器，expression中需要指定一个注解名       |
        |   assignable   | 类名过滤器，expression中直接指定一个Java类       |
        |     regex      | 正则表达式过滤器，expression中指定一个正则表达式 |
        |    aspectj     | AspectJ过滤器                                    |

---

## Bean对象的注解

### 设置Bean对象

- Spring需要显示指定搜索哪些路径下的Java类，并把其中合适的类注册为Bean对象

- 常用注解

    - **Component** - 标注一个普通的Spring Bean类

    - **Controller** - 标注一个控制器组件类

    - **Service** - 标注一个业务逻辑组件类

    - **Repository** - 标注一个DAO组件类

        | 属性  |      说明      | 默认值 |
        | :---: | :------------: | :----: |
        | value | 指定Bean的id值 |   ""   |

        - 如果注解中不显示指定id，则默认使用类名首字母小写后的字符串作为id

- 其中，Controller、Service、Repository三个注解携带着更多的语义。在Java EE开发中，应尽量使用这三种注解

### Lookup注解

- Spring提供了解决作用域不同步问题的Lookup注解，与lookup-method标签的作用完全相同
- 由于Lookup注解直接修饰需要执行lookup方法注入的方法，因此不再需要指定name属性

| 属性  |                       说明                       | 默认值 |
| :---: | :----------------------------------------------: | :----: |
| value | 指定实现方法的返回值，也就是对应的prototype Bean |   ""   |

### 指定Bean的作用域

- 在类上方使用Scope注解来指定Bean的作用域，只需要在Scope中提供作用域名称即可

### 生命周期注解

- PostConstruct和PreDestroy两个注解来自javax.annotation包下

#### PostConstruct

- 等价于bean标签中的init-method属性
- 修饰方法，在依赖关系注入完成之后调用该方法

#### PreDestroy

- 等价于bean标签中的destroy-method属性
- 修饰方法，在Spring容器销毁该Bean之前调用该方法

### 初始化行为

#### DependsOn

- 写在类上方，用于强制初始化其他Bean
- 该注解的值是一个字符数组，每个数组元素对应一个强制初始化的Bean对象

#### Lazy

- 写在类上方，用于指定该Bean是否需要预初始化
- 该注解的值是一个boolean类型。当值为true时，表明该Bean对象不需要预初始化

---

## 配置依赖

### Autowired - 自动装配

- 作用: 指定其他Bean对象的自动装配。该注解无法注入普通属性，只能注入Bean对象
- 该注解可以修饰实例属性、普通方法、set方法和构造器。但一般修饰实例属性居多，使用该注解修饰后，实例对象可以不再提供set方法
- **默认采用的是byType自动装配策略**
- 如果实例对象是集合属性的话，该注解修饰后会将容器中所有符合条件的Bean对象作为集合元素来创建集合。这里的List和Set集合一定要使用泛型，Map集合的key值是Bean对象的id值
- 自动装配时，如果恰好找到一个符合的Bean对象，则会直接注入；如果找到多个或没有找到，则都会抛出异常
- 如果找到多个对应的Bean对象而报错，则建议使用Resource注解
- 如果没有找到对应的Bean对象而报错，则是因为注解的required属性，该属性默认为true，即该注解修饰的实例对象必须要被依赖注入，否则就会报错；或者使用[Nullable注解](###Nullable)修饰变量

### Resource - 自动装配 + 精准装配

- 作用: 基本与Autowired相同，但该注解除了可以进行自动装配外，还可以进行精准装配

- **默认采用的是byName自动装配策略，因此该注解不建议修饰集合属性**

- 该注解位于javax.annotation包下，是Java EE本身的注解

    | 属性 |         说明         |    默认值    |
    | :--: | :------------------: | :----------: |
    | name |  指定所依赖Bean的id  |      ""      |
    | type | 指定所依赖Bean的类型 | Object.class |

### Qualifier

- 作用: 一般多用于修饰方法的形参，为方法的形参注入指定的Bean对象
- 使用value属性引入被注入的Bean对象的id

### Value

- 作用: 为Bean对象的普通属性配置属性值
- 该注解只有一个value属性，值是要注入的属性值。**可以直接使用SpEL**

---

## Spring 5新增注解

### Nullable

- 作用: 修饰参数、返回值和属性，表明允许为空

### NonNull

- 作用:修饰参数、返回值和属性，表明不允许为空

### Required

- 作用: 检查注入，作用不大
- 如果开发者既没有在配置文件中配置依赖，也没有使用注解配置依赖，就会报错