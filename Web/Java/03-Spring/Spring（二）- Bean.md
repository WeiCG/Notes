# Bean对象

> Spring的核心功能便是负责创建、管理所有的Java对象，这些Java对象被称为Bean对象

## IoC的本质

- **根据配置文件创建Bean实例，并调用Bean实例的方法完成“依赖注入”**
- IoC容器本质上是一个Map集合

---

## beans标签

- beans标签是Spring配置文件的根元素

|            属性             | 说明                                               |
| :-------------------------: | -------------------------------------------------- |
|      default-lazy-init      | 指定所有Bean对象是否需要延迟初始化                 |
|        default-merge        | 指定所有Bean对象默认的merge行为                    |
|      default-autowire       | 配置所有Bean对象的自动装配行为，**默认值为no**     |
| default-autowire-candidates | 配置Bean对象是否作为自动装配的候选Bean             |
|     default-init-method     | 配置所有Bean对象的默认初始化方法，**生命周期属性** |
|   default-destroy-method    | 配置所有Bean对象的默认回收方法，**生命周期属性**   |

- **PS - 上述所有属性，在bean标签中都可以使用（将属性名称中的default去掉即可），那么这些属性就只会对特定的Bean对象起作用**

---

## bean标签

- bean标签是beans标签的子元素，是Spring配置文件中最重要的标签之一
- 每个bean标签定义一个Bean对象，对应Spring容器中的一个Java实例

| 属性  | 说明                                         |
| :---: | -------------------------------------------- |
|  id   | 确定该bean标签的唯一标识                     |
| class | 指定该bean标签的具体实现类，且必须指向实现类 |
| scope | 为bean标签指定作用域                         |

---

## 容器中Bean对象的作用域

- **在bean标签中使用scope属性指定**

| 作用域        | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| singleton     | 单例模式，在整个Spring容器中只会生成一个Bean实例，**默认值** |
| prototype     | 多例模式，每次获取一个Bean时都将产生一个新的Bean实例         |
| request       | 在同一个HTTP请求中，Spring容器只会生成一个Bean实例           |
| session       | 在同一个Session会话中，Spring容器只会生成一个Bean实例        |
| globalSession | 在整个Web应用中，Spring容器只会生成一个Bean实例，一般用在集群环境下 |

- **singleton对象是由Spring容器负责创建和销毁的。而对于prototype对象，Spring容器只负责创建，销毁是由Java的GC来负责的**

---

## 配置依赖

- 构造注入和值注入是Spring所使用的的两种配置依赖的方法，前面已经写过（**详见Spring（一）中的依赖注入的方式**），现在具体说明对于不同的参数类型如何注入依赖
- **Spring只推荐在配置文件中管理组件与组件之间的依赖注入，对于普通属性值的注入仍然应该在代码中设置**
- 如果想为Bean对象注入一个类变量，有三种方式
    1. [手动设置其他Bean](###手动设置其他Bean)
    2. [自动装配其他Bean](###自动设置其他Bean)
    3. [嵌套注入Bean](###注入嵌套Bean)

### 设置普通属性值

- 普通属性包括8种基本数据类型和String类型
- **为Bean对象注入普通属性的值使用property和constructor-arg标签的value属性，value属性的值默认是字符串类型，但Spring可以将其转换成对应的基本数据类型**

### 手动设置其他Bean

- **为Bean设置的属性值是容器中的另一个Bean实例，则应该使用property和constructor-arg标签中的ref属性，引用容器中其他Bean实例的id属性值**

### 自动装配其他Bean

- 自动装配Bean会减少配置文件的工作量，但降低了依赖关系的透明度和清晰度
- 自动装配一般用于注解配置依赖比较多
- **在bean标签中使用autowire属性，表明该Bean对象使用自动装配来注入其他Bean对象。或直接在beans标签中使用default-autowire属性，开启全局的自动装配功能**

| autowire属性值 | 说明                                                         |
| :------------: | ------------------------------------------------------------ |
|       no       | 不使用自动装配，**默认值**                                   |
|     byName     | 根据set方法名来自动装配，查找其他Bean对象的id来匹配set方法   |
|     byType     | 根据set方法的形参类型来自动装配。**如果找到多个匹配，便会抛出异常** |
|  constructor   | 与byType类似，自动匹配构造方法的参数，**如果找到零个或多个，便会抛出异常** |
|   autodetect   | 如果Bean对象中有无参构造方法，则为byType；如果没有，则为constructor |

- 对于大型项目，不建议使用自动装配
- **如果希望将一些特定的Bean排除到自动装配之外，可以使用bean标签中的autowire-candidates属性值为"false"；或者使用beans标签中的default-autowire-candidates属性指定**

### 注入嵌套Bean

- 有一些Java实例不想被Spring容器直接访问，所以可以使用嵌套Bean的方式来注入
- **在property和constructor-arg标签中使用子标签bean，该子标签只需指定class路径即可，不需要指定id，其他与bean标签无任何区别**

### 注入集合属性

- **使用property和constructor-arg标签的子标签list/set/map/props/array来分别为List/Set/Map/Properties/Array来赋值**

#### 注意

- list/array/set标签可以接受value/ref/bean和集合属性标签

- Map是键值对型的集合元素，在map标签中使用entry子标签为Map集合赋值，共有四个属性key key-ref value value-ref
- 如果Map集合中有存储了集合属性，建议使用如下写法

```xml
<property name="map">
    <map>
        <entry>
            <key>
                <value>学科</value>
            </key>
            <list>
				...
            </list>
        </entry>
    </map>
</property>
```

- Properties集合是特殊的Map，键值对只能是String类型，所以直接使用props标签的子标签prop。prop的key属性指定key值，prop的内容指定value值

---

## 创建Bean的3种方式

### 调用构造器

- 最常见的创建方法
- 此时bean标签的class属性将指向Bean实例的实现类

### 调用静态工厂方法

- 使用这种方法时，**bean标签的class属性要指向静态工厂类，而factory-method属性指向创建Bean实例的静态工厂方法**
- 如果工厂方法有参数的话，可以使用bean标签的子标签constructor-arg来配置参数
- 使用这种方法创建Bean实例后，Spring容器仍可以对该Bean实例进行值注入和管理生命周期

### 调用实例工厂方法

- 调用实例工厂方法需要首先创建实例工厂，因此我们需要先配置工厂bean标签
- 使用bean标签的factory-bean属性指向工厂Bean实例，factory-method属性指向创建Bean实例的实例工厂方法
- 如果工厂方法有参数的话，可以使用bean标签的子标签constructor-arg来配置参数
- 使用这种方法创建Bean实例后，Spring容器仍可以对该Bean实例进行值注入和管理生命周期