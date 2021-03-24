# JPA注解配置表映射

> 在Hibernate的发展中，Hibernate抛弃了原先使用XML的形式配置映射文件，全面转向支持JPA注解，下面的注解全部都在javax.persistence包中

## 实体类映射

### Entity

- 作用: 声明该类是一个Hibernate的持久化类
- 属性: **name - 设置实体类名称，默认为该类类名**

### Table

- 作用: 声明实体类映射的表
- 属性: **name - 设置表名**

---

## 普通属性的映射

### Column

- 作用: 指定实体类属性在表中的一些特性

属性 | 说明 | 默认值
:--: | :--: | :--:
name | 设置字段名 | ""
nullable | 是否允许为null | true
unique | 是否具有唯一约束 | false
table | 该字段所属表名，用于多张表保存同一个实体时 | ""
insertable | 是否包含在Hibernate生成的insert语句的列列表中 | true
updatable | 是否包含在Hibernate生成的update语句的列列表中 | true

### Transient

- 作用: 若持久化时不想保存某些属性，可以通过该注解修饰
- 属性: 无

---

## 标识属性（主键）的映射

### Id

- 作用: 用于指定该类的标识属性
- 属性: 无

### GeneratedValue

- 作用: 指定主键的生成策略，常与Id注解连用

属性 | 说明 | 默认值
:--: | :--: | :--:
strategy | 主键生成策略，该值是GenerationType枚举类（见下表） | AUTO
generator | 当使用SEQUENCE、TABLE主键生成策略时，用来引用生成器的名称 | "" 

GenerationType取值 | 说明
:--: | :-:
AUTO | Hibernate自动选择最适合底层数据库的主键生成策略
IDENTITY | 自动增长，一般用于MySQL/SQL Server 
SEQUENCE | 基于序列，一般用于Oracle，与SequenceGenerator注解一起使用 
TABLE | 基于辅助表，一般多用于继承映射，详见**Hibernate（五）**，与TableGenerator注解一起使用 

- **由SEQUENCE注解基本不使用，所以SequenceGenerator注解不再介绍**

| TableGenerator属性 |       说明       | 默认值 |
| :----------------: | :--------------: | :----: |
|        name        | 主键生成器的名称 |   无   |
|       table        |   辅助表的名称   |   ""   |
|    pkColumnName    | 存放主键名的列名 |   ""   |
|   pkColumnValue    |    指定主键名    |   ""   |
|  valueColumnName   | 存放主键值的列名 |   ""   |

### 联合主键

- **如果需要使用多个标识属性时，可以直接在多个属性上以Id注解标识。但同时，持久化类必须根据联合主键所对应的属性重写equals和hashCode方法**

---

## 属性为枚举类型的映射

### Enumerated

- 作用: 修饰枚举类型的属性

属性 | 说明 | 默认值
:--: | :--: | :--:
value | 保存的是枚举值名称还是枚举值序号，该值是EnumType枚举类。若为**STRING**，则保存名称；若为**ORDINAL**，保存序号 | ORDINAL

---

## 属性为图片等大数据类型的映射

### Lob

- 作用: 修饰大数据类型的属性，一般为图片、大段文章等
- 属性: 无
- 若属性为byte[]、Byte[]等二进制形式时，Hibernate底层会将其映射为Blob类型
- 若属性为char[]、Character[]、String等类型时，Hibernate会将其映射为Clob类型

### Basic

- 作用: 与Lob注解连用，由于大数据类型的读取对于系统开支较大，所以使用该注解修饰大数据类型的属性，显示指定获取时是否要延迟加载

属性 | 说明 | 默认值
:--: | :--: | :--:
fetch | 是否延迟加载该属性，该值是FetchType枚举类。若为**EAGER**，则立即加载；若为**LAZY**，则延迟加载 | EAGER

---

## 属性为时间类型的映射

### Temporal

- 作用: 用于时间类型的属性上，指定在数据库中保存为何种时间类型

属性 | 说明 | 默认值
:--: | :--: | :--:
value | 指定要转换为哪种数据库时间类型，值为TemporalType枚举类（见下表） | 无

TemporalType取值 | 说明
:--: | :--
DATE | 保存为date类型的值，即保存日期，不保存时间
TIME | 保存为time类型的值，即保存时间，不保存日期
TIMESTAMP | 保存为timestamp类型的值，即保存时间也保存日期

---

## 属性为集合类型的映射

- 对于实体类的集合属性，其集合属性的值是保存在另外一张数据表中的，两者之间的关系通过外键来联系。
- 一旦包含有集合属性的实体类持久化后，其对应的集合属性的值全部自动持久化；持久化对象被删除时，其对应的所有集合属性全部被删除

### ElementCollection

- 作用: 映射集合属性，包括List / Set / Map / Array等

属性 | 说明 | 默认值
:--: | :--: | :--:
fetch | 设置抓取属性，即当初始化该实体时，是否要从数据库中抓取该实体的集合属性中所有的值，值为FetchType枚举类（[见Basic注解](###Basic)） | LAZY
targetClass | 指定集合属性中元素的类型 | void.class

### CollectionTable

- 作用: 映射保存集合数据的表

属性 | 说明 | 默认值
:--: | :--: | :--:
name | 设置表名 | ""
joinColumns | 值是一个JoinColumn注解的数组，每个JoinColumn注解映射一个外键 | {}

### JoinColumns/JoinColumn

- 作用: 在映射表中定义外键

属性 | 说明 | 默认值
:--: | :--: | :--:
name | 设置列名 | ""
nullable | 是否允许为空 | true
unique | 是否增加唯一约束 | false
referencedColumnName | 指定该外键所参照的主键 | ""

### OrderColumn/MapKeyColumn

- 作用: 由于除Set集合外其他所有集合都有着显示的索引值，所以在集合元素所在的数据表中指定一个索引列，用于保存数组索引、List索引和Map集合的key索引
- 属性 - **该属性与普通的Column注解属性相同**

### MapKeyClass

- 作用: 显示指定Map Key的类型

| 属性  |        说明        |   默认值   |
| :---: | :----------------: | :--------: |
| value | 指定Map的Key的类型 | void.class |

---

## 属性为组件类的映射

- **这里的组件类指的是单纯的类类型，在持久化过程中，该类型只被当做值类型来看待，并不是另一个持久化实体，关于持久化类型的情况请看Hibernate（四）- 关联映射**

### Embeddable

- 作用: 写在组件类上方，表明该类是一个单纯的值类型，不是一个持久化类
- **在组件类中需要使用Column注解，表明该类的属性储存在表中的指定字段**
- 属性: 无

### Embedded

- 作用: 类似于Embeddable注解，写在持久化类的组件类属性上，表明该属性的类型是一个组件类类型。
- **注意: 该注解与Embeddable注解只能同时使用一个**
- 属性: 无

### AttributeOverrides/AttributeOverride

- 作用: 与Embedded注解连用，同于指定组件类中属性的对应字段
- 由于Embeddable注解与Embedded注解只能同时使用一个，所以使用Embedded注解后，无法在组件类中规定组件类属性的存储字段，所以必要要使用该注解指定

属性 | 说明 | 默认值
:--: | :--: | :--:
name | 指定对组件类中的哪个属性进行配置 | 无
column | 指定该属性映射的数据表字段，**该属性的值为Column注解** | 无

### EmbeddedId

- 作用: 如果需要组件类属性作为标识属性时，使用该注解修饰主键
- 该注解同样需要搭配AttributeOverrides注解
- 如果出现这种情况，则组件类中必须
  - 有无参的构造函数
  - 实现Serializable接口
  - 重写equals和hashCode方法
- 属性: 无
- 注意: 此时不需要使用Embeddable注解配置组件类

### 特别注意

- **如果组件类中的存在集合属性，则必须使用Embeddable注解来配置组件类，其余配置与集合类型映射相同**
- **如果集合中的元素为组件类对象时，则必须使用Embeddable注解来配置组件类，其余配置与常见集合类型相同，但需要在集合属性上去掉Column注解**
- **如果组件类对象要作为Map中的Key值时，则必须使用Embeddable注解来配置组件类，并且重写equals和hashCode方法**
