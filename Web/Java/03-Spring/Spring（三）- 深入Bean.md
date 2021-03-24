# 深入理解Bean对象

> 除了（二）中介绍的简单使用外，Spring还为Bean的管理增添了更多的功能

## Bean之间的继承关系

### 抽象Bean

- 在大型项目的开发中，可能会出现多个bean标签中存在大致相同的配置，这些bean标签只有少量信息不同，这些导致了配合文件中出现了大量冗余
- 因此，可以将多个bean标签中的相同信息抽取出来，集中为一个模板，即抽象Bean
- **抽象Bean不是真正的Bean，因此不需要Spring容器去创建，所以需要指定bean标签的abstract属性值为true**
- **抽象Bean的作用仅仅是被继承**，同时作为配置信息的模板，抽象Bean可以不指定class属性

### 子Bean

- 子Bean可以从父Bean中，继承实现类、构造器参数、属性值等配置信息
- 同时子Bean也可以定义新的配置信息，或覆盖父Bean的配置信息
- **通过bean标签的parent属性指定该Bean是一个子Bean，值为父Bean的id**

---

## 工厂Bean

---

## 获得Bean本身的id

- 通过**实现BeanNameAware接口**，该接口需要实现一个setBeanName的方法，Spring容器会在创建该Bean之后，自动调用它的setBeanName方法，将Bean的id传入
- 该set方法不需要在配置文件中显示指定，Spring会主动调用

```java
import org.springframework.beans.factory.BeanNameAware;

public class ExampleBean implements BeanNameAware {
    private String name;

    @Override
    public void setBeanName(String name) {
        this.name = name;
    }
    
    @Override
    public String toString() {
        return "Name=" + name;
    }
}
```

---

## 强制初始化Bean

- **使用bean标签中的depends-on属性，显示指定必须在初始化该Bean对象之前初始化该属性对应的Bean对象**

---

## Bean的生命周期

- 对于singleton作用域的Bean生命周期，Spring容器负责创建、初始化和销毁。而对于prototype作用域的Bean，Spring仅仅负责创建和初始化，销毁由Java的GC负责
- **Bean生命周期的管理有两个时间点**
    1. **注入依赖关系之后**
    2. **销毁Bean之前**
- 如果很多Bean实例都需要指定生命周期行为，则建议使用beans标签的default-init-method和default-destroy-method属性

### 注入依赖关系之后

- **使用bean标签的init-method属性，指定某个方法在Bean全部依赖关系设置结束后自动执行；或让Bean类实现InitializingBean接口，在完成依赖关系注入后，调用接口的afterProperitesSet方法**
- 由于实现接口会影响原本的Java类，因此不建议使用该方法

### 销毁Bean之前

- **使用bean标签的destroy-method属性，指定某个方法在Bean实例销毁之前自动执行；或让Bean类实现DisposableBean接口，在Bean实例销毁之前，调用接口的destroy方法**
- 由于实现接口会影响原本的Java类，因此不建议使用该方法

---

## 作用域不同步问题

- 问题: 如果一个singleton作用域对象依赖于一个prototype作用域对象时，由于singleton对象只会依赖一次，因此会一直维护着所依赖的prototype对象，导致prototype对象不会被GC回收，等价为了一个singleton对象。当使用singleton对象访问prototype对象时，就会发现始终都是同一个对象，就会产生不同步的问题
- 解决方法: 利用方法注入
    1. 将调用者类定义为抽象类，并定义一个抽象方法来获取被依赖的prototype Bean
    2. 在bean标签中使用lookup-method子标签，即让Spring去实现指定的抽象方法
        - name属性 - 指定需要Spring实现的方法
        - bean属性 - 指定Spring实现方法的返回值，也就是对应的prototype Bean

---

## 高级依赖关系配置

- Spring配置依赖关系更多的是使用set方法，除了这些外，Spring同样还可以调用更多的代码

### 获取其他Bean的成员属性值

- 通过将bean标签的实现类指向PropertyPathFactoryBean，可以获取其他Bean的属性值，**实质是调用其他Bean对象的get方法**
- 使用时需要指明两个信息
    1. 调用哪个Bean对象 - targetBeanName
    2. 调用Bean对象的哪个get方法 - propertyPath
- 调用getBean方法后，会直接定义成容器中的Bean实例

```xml
<bean id="person" class="com.entity.Person">
	<property name="age" value="30" />
    <property name="son">
    	<bean class="com.entity.Son">
        	<property name="age" value="11" />
        </bean>
    </property>
</bean>
<!-- 下面配置在程序中调用getBean("son")会返回Son对象 -->
<bean id="son" class="org.springframework.beans.factory.config.PropertyPathFactoryBean">
	<property name="targetBeanName" value="person" />
    <property name="propertyPath" value="son" />
</bean>
```

- 还可以注入另一个Bean中使用，使用"."运算符来简便获取属性中的值

```xml
<bean id="son" class="">
    <property name="son">
        <!-- 使用.运算符快速访问 -->
    	<bean id="person.son.age" class="org.springframework.beans.factory.config.PropertyPathFactoryBean" />
    </property>
</bean> 
```

### 获取类的静态属性值

- 通过将bean标签的实现类指向FieldRetrievingFactoryBean，可以访问类的静态Field值
- 使用时需要指明两个信息
    1. 调用哪个类 - targetClass
    2. 访问类中的哪个static属性 - targetField
- 调用getBean方法后，会返回对应的静态属性值

```xml
<!-- 下面配置在程序中调用getBean("son")会返回Son对象 -->
<bean id="transactioon" class="org.springframework.beans.factory.config.FieldRetrievingFactoryBean">
	<property name="targetClass" value="java.sql.Connection" />
    <!-- 访问静态属性 -->
    <property name="targetField" value="TRANSACTION_SERIALIZALBE" />
</bean>
```

- 也可以注入另一个Bean中使用，使用.运算符来获取静态属性值

```xml
<bean id="son" class="">
    <property name="son">
        <!-- 使用.运算符快速访问 -->
    	<bean id="java.sql.Connection.TRANSACTION_SERIALIZALBE" 
              class="org.springframework.beans.factory.config.FieldRetrievingFactoryBean" />
    </property>
</bean>
```

### 获取方法的返回值

- 通过将bean标签的实现类指向MethodInvokingFactoryBean，可以调用任意类的类方法，也可以调用任意对象的实例方法。
- 如果调用的方法有返回值，既可以定义为容器中的Bean，也可以将其注入给其他Bean
- 使用时需要指明三个信息
    1. 调用哪个类或对象 - targetClass或targetObject
    2. 调用哪个方法 - targetMethod
    3. 调用方法的参数 - arguments（PS: 无参数可省略），传入一个数组

```xml
<bean class="org.springframework.beans.factory.config.MethodInvokingFactoryBean">
	<property name="targetClass" ref="目标类的id" />
    <property name="targetMethod" value="类中的方法" />
    <property name="arguments">
    	<array>
        	<value>xxx</value>
			<bean class="xxx" />
            <ref bean="xxx" />
        </array>
    </property>
</bean>
```

---

## 简化配置方式

- 基于DTD的配置文件在大型项目会导致Spring配置文件中配置过于繁琐。因此，Spring官方提出了基于XML Schema的配置方式

```xml
<!-- 需要在配置文件导入 -->
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns="http://www.springframework.org/schema/beans"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:c="http://www.springframework.org/schema/c"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
		   http://www.springframework.org/schema/util
           http://www.springframework.org/schema/util/spring-util.xsd">

</beans>
```

### 使用p:命名空间简化值注入

- 在之前如果需要为Bean对象的属性进行值注入时，需要使用property标签。而使用p:命名空间后，可以直接在bean标签中使用属性来驱动set方法
- 使用**p:属性名**可以直接在bean标签中为Bean对象的属性赋值
- 如果要引用其他Bean，则使用**p:属性名-ref**即可
- 使用p:名称空间方法只能简化简单的注入，对于复杂的注入（集合注入、嵌套注入等）需要与util:命名空间或SpEL配合使用

```java
// Person类
public class Person {
    private Subject subject;
    private int age;

    public Person() { // 无参
    }

    public Person(Subject subject, int age) {  // 有参
        this.subject = subject;
        this.age = age;
    }
    
    // 属性的get和set方法
    ...
}
```

```xml
<bean id="person" class="Person类的路径"
      p:age="20" p:subject-ref="subject" />
<bean id="subject" class="Subject类的路径" />
```

### 使用c:命名空间简化构造注入

- 与使用p:命名空间简化值注入的配置基本相同，只不过是调用构造方法来创建新的Bean对象

```xml
<bean id="person" class="Person类的路径"
      c:age="20" c:subject-ref="subject" />
<bean id="subject" class="Subject类的路径" />
```

### 使用util:名称空间简化高级依赖配置

- 该命名空间提供了如下元素

| 元素          | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| constant      | 获取指定类的静态属性。FieldRetrievingFactoryBean的简化配置   |
| property-path | 获取指定对象的get方法的返回值。PropertyPathFactoryBean的简化配置 |
| list          | 定义一个List Bean，支持value/ref/bean标签                    |
| map           | 定义一个Map Bean，支持entry标签来定义key-value对             |
| set           | 定义一个Set Bean，支持value/ref/bean标签                     |
| properties    | 加载一份资源文件                                             |

- 其中，list/map/set/properties四个元素各提供了三个属性
    - id属性 - 定义一个唯一的Bean实例
    - list-class/map-class/set-class/location属性 - 前三个分别指定使用哪个List/Map/Set实现类来创建Bean，后一个则是资源文件所在的位置
    - scope属性 - 定义作用域 

```xml
<!-- 普通Bean注入集合属性 -->
<bean id="person" class="com.wei.entity.Person"
          p:subject-ref="subject"
          p:age-ref="person.age"
          p:subjects-ref="person.subjects"
          p:schools-ref="person.schools"
          p:scores-ref="person.scores" />   
<!-- 普通Bean -->
<bean id="subject" class="com.wei.entity.Subject">
    <property name="subject">
        <array>
            <value>数学</value>
            <value>语文</value>
            <value>英语</value>
        </array>
    </property>
</bean>

<!-- 获取类的静态属性 -->
<util:constant id="person.age"
               static-field="java.sql.Connection.TRANSACTION_SERIALIZABLE" />
<!-- 定义一个List Bean -->
<util:list id="person.schools" list-class="java.util.LinkedList">
    <value>小学</value>
    <value>中学</value>
    <value>大学</value>
</util:list>
<!-- 定义一个Set Bean -->
<util:set id="person.subjects" set-class="java.util.HashSet">
    <value>字符串</value>
    <bean class="com.wei.entity.ExampleBean" />
    <ref bean="subject" />
</util:set>
<!-- 定义一个Map Bean -->
<util:map id="person.scores" map-class="java.util.TreeMap">
    <entry key="数学" value="80" />
    <entry key="语文" value="80" />
    <entry key="英语" value="80" />
</util:map>
```

---

## SpEL

- **SpEL的作用便是搭配上述的简化配置方式，更大程度的扩展Spring容器的功能**
- **SpEL表达式以#开头，内容放在{}中**

### 常用语法

- 在表达式中使用Java语言支持的直接量，包括字符串、日期、数值、boolean值和null

