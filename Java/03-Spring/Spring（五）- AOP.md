# 面向切面编程 --- AOP

> AOP与OOP互为补充，提供了强大的动态编程能力

## AOP编程的基本概念

|      概念名词      | 说明                                                   |
| :----------------: | ------------------------------------------------------ |
|   Aspect - 切面    | 用于组织多个Advice，Advice放入切面中定义               |
| Joinpoint - 连接点 | 程序执行过程中明确调用的点                             |
| Advice - 增强处理  | AOP框架在特定的切入点执行的增强处理                    |
| Pointcut - 切入点  | 可以插入增强处理的连接点                               |
|   Waving - 织入    | 将增强处理添加到目标对象中，并创建一个被增强对象的过程 |

---

## AOP编程步骤

1. 定义普通业务组件，将普通Bean转换为切面Bean
2. 编写增强处理方法
3. 定义切入点

---

## AOP代理公式

- **AOP代理的方法 = 增强处理 + 目标对象的方法**

---

## 切面类注解注解配置

### 启用AOP注解支持

- 首先需要确保Spring开启了AOP注解支持

- 在配置文件的beans标签中插入

    ```
    xmlns:aop = "http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/aop
    		http://www.springframework.org/schema/aop/spring-aop.xsd"
    ```

- 之后开启AOP注解扫描

    `<aop:aspectj-autoproxy />`

### 定义普通业务组件

- 定义一个简单的Java对象，在将其配置为Bean对象后，在类上添加注解Aspect
- Aspect注解会将普通的Spring Bean对象转化为切面Bean
- 在切面Bean中编写增强处理方法
- **一定要将Java类变为Bean对象才能被Spring容器所管理，因此首先需要将Java对象变为Bean对象，再将普通Bean对象转为切面Bean对象**

### 定义增强处理

- **增强处理的执行流程是: Around - Before - 切入点方法 - Around - After - AfterReturning/AfterThrowing**

#### Before 

- 在切入点执行之前织入增强

#### AfterReturning

- 切入点正常执行之后被织入
- returning属性 - 在增强方法中可定义与此同名的形参，用于访问切入点的返回值
- 虽然可以访问切入点方法的返回值，但不可以改变

#### AfterThrowing

- 切入点抛出异常之后被织入
- throwing属性 - 在增强方法中可定义与此同名的形参，用于访问切入点抛出的异常

#### After 

- 不论切入点是否正常结束都会被织入，类似于finally代码块
- 通常用于释放资源
- **该注解存在BUG，After作为最后的注解，如果与AfterReturning/AfterThrowing注解连用时，会在这两个注解前执行。一旦AOP管理事务，则释放资源会在提交和回滚前执行，会报错**

#### Around

- 近似等于Before和AfterReturning两种增强的综合。Around可以决定目标方法在什么时候执行，如何执行，甚至完全可以不执行
- Around增强处理可以改变切入点的参数值，也可以改变切入点的返回值
- 当定义一个Around方法时，第一个参数必须是ProceedingsJoinPoint类型，并在方法内部调用该参数的proceed方法才会执行目标方法，并传入一个Object[]对象作为参数

#### 获取切入点方法的相关信息

- 将增强方法的第一个形参定义为JoinPoint类型，便可以获取切入点的相关信息

- ProceedingJoinPoint方法是该方法的子类

    |           方法           | 说明                         |
    | :----------------------: | ---------------------------- |
    |    Object[] getArgs()    | 返回切入点方法的参数         |
    | Signature getSignature() | 返回切入点方法的相关信息     |
    |    Object getTarget()    | 返回被织入增强处理的目标对象 |
    |     Object getThis()     | 返回AOP框架生成的AOP代理对象 |

### 定义切入点

- 使用**Pointcut注解**可以定义一个切入点表达式，该注解修饰一个空方法

- 上述的增强处理的五种注解都有value属性，在value属性中可以指定一个切入点表达式，该表达式用于指定该增强处理要处理哪个切入点。**value属性的值是Pointcut注解修饰方法的方法名，要加()**

- Pointcut注解的value属性中要写切入点表达式，格式如下

    `execution(modifiers-pattern? ret-type-pattern declaring-type-pattern? name-pattern(param-pattern) throws-pattern?)`

|      表达式各部分      | 说明                                                         |
| :--------------------: | ------------------------------------------------------------ |
|   modifiers-pattern    | 可选。指定方法的修饰符                                       |
|    ret-type-pattern    | 指定方法的返回值类型，可用"*"来表示所有返回值                |
| declaring-type-pattern | 可选。指定方法所属的类                                       |
|      name-pattern      | 指定方法名，可用"*"表示所有的方法                            |
|     param-pattern      | 指定方法中的形参列表，使用"*"表示任意一个，使用"..."表示一个或多个任意类型 |
|     throws-pattern     | 可选。指定方法抛出的异常                                     |

### 多个切面类的执行顺序

- 当出现多个切面类同时增强同一个切入点方法时，可以在切面类上使用**Order注解**来确定切面类的执行顺序。如果不使用，则随机执行
- 传入一个int类型的整数，数字越小，优先级越高，越先执行

---

## 切面类XML文件配置

- 使用XML文件配置，所有关于切面的配置全部都写在**aop:config标签**中
- **一个aop:config标签包含pointcut、advisor、aspect三个元素，且这三个元素必须按照此顺序来定义**

### aop:pointcut

- 该标签配置切入点表达式，详见[定义切入点](###定义切入点)

### aop:advisor

### aop:aspect

- 定义切面，实质是将普通Spring Bean对象转化为切面Bean对象
- id属性 - 定义切面的id值
- ref属性 - 将ref属性所引用的普通Bean转化为切面Bean
- order属性 - 指定该切面Bean的优先级