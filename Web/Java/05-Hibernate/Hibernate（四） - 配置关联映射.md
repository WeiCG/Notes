# 配置关联映射

> 世间万物都有联系，关联映射就是配置这些联系

## 1 - 1 关系

- 1 - 1 关系一般都使用**外键**相关联，需要为两端的持久化类中都添加代表关联实体的成员变量，使用OneToOne注解修饰
- 1 - 1 关系中外键放在哪个表中都可以，**但一般都放在逻辑上的次表中**

### OneToOne

属性 | 说明 | 默认值
:--: | :--: |:--:
cascade | 指定级联策略，值为[CascadeType枚举类](###CascadeType的枚举值) | {}
fetch | 指定抓取规则，值为[FetchType枚举类](###FetchType的枚举值) | EAGER
mappedBy | 指定关联实体中对应的引用属性 | ""
orphanRemoval | 当关联的主表不存在时，是否删除对应记录 | false
optional | 指定关联联系是否可选 | true
targetEntity | 指定关联实体的类名 | void.class

### 使用JoinColumn注解来配置 1 - 1 关系的外键

- 详情见 **Hiberante入门 - JPA注解配置表映射 中的JoinColumn注解**

---

## 1 - N 关系

- 1 - N 关系一般都使用外键相关联
  - 在 N 的一端添加代表关联实体的属性，并使用ManyToOne注解来修饰
  - 在 1 的一端添加代表关联实体的Set属性，并使用OneToMany注解修饰

### OneToMany

属性 | 说明 | 默认值
:--: | :--: |:--:
cascade | 指定级联策略，值为[CascadeType枚举类](###CascadeType的枚举值) | {}
fetch | 指定抓取规则，值为[FetchType枚举类](###FetchType的枚举值) | EAGER
mappedBy | 指定关联实体中对应的引用属性 | ""
orphanRemoval | 当关联的主表不存在时，是否删除对应记录 | false
targetEntity | 指定关联实体的类名 | void.class

- **为提高Hibernate的性能，通常情况下会不允许 1 的一端控制关联关系，应该在OneToMany注解中指定mappedBy属性，表明 1 的一端放弃控制关系**

### ManyToOne

属性 | 说明 | 默认值
:--: | :--: |:--:
cascade | 指定级联策略，值为[CascadeType枚举类](###CascadeType的枚举值) | {}
fetch | 指定抓取规则，值为[FetchType枚举类](###FetchType的枚举值) | EAGER
optional | 指定关联联系是否可选 | true
targetEntity | 指定关联实体的类名 | void.class

- **注意: 虽然ManyToOne注解中有cascade属性，但非特殊情况不要使用**

### 使用JoinColumn注解来配置 1 - N 关系的外键

- 详情见 **Hiberante入门 - JPA注解配置表映射 中的JoinColumn注解**

---

## N - N 关系

- N - N 关系一般都使用连接表相关联，需要为两端的持久化类中都添加代表关联实体的Set集合属性，使用ManyToMany注解修饰，并在两端都使用JoinTable注解映射连接表

### ManyToMany

属性 | 说明 | 默认值
:--: | :--: |:--:
cascade | 指定级联策略，值为[CascadeType枚举类](###CascadeType的枚举值) | {}
fetch | 指定抓取规则，值为[FetchType枚举类](###FetchType的枚举值) | EAGER
mappedBy | 指定关联实体中对应的引用属性 | ""
targetEntity | 指定关联实体的类名 | void.class

- **如果在程序中希望某一端不再控制关联关系，则可以在ManyToMany注解中显示指定mappedBy的属性，表明该端放弃控制关系，则不需要使用JoinTable注解映射连接表**

### JoinTable

属性 | 说明 | 默认值
:--: | :--: |:--:
name | 指定连接表的表名 | ""
joinColumns | 配置连接表中外键列的字段信息，参照当前实体对应表的主键 | {}
inverseJoinColumns | 配置连接表中外键列的字段信息，参照关联实体对应表的主键 | {}

---

## 组件类和关联关系

- **对于主从表的约束关系，Hibernate使用了两种不同的映射**
  - 将从表映射为组件类
  - 将从表映射为持久化类

### 组件类

- 当映射为组件类时，Hibernate会默认开启级联操作。当主表类对象保存时，组件同样保存；当主表类对象被删除时，从表对象也全部删除
- 组件类不存在自己的生命周期，它依附于主表而存在
- 组件类的注解请看 **Hibernate入门 - JPA注解配置表映射**

### 持久化类

- 当映射为持久化类时，Hibernate不会开启级联操作，当主表类对象保存时，关联实体不会保存；当主表类对象被删除时，从表对象也不会被删除
- 持久化类有着自己的生命周期，它是独立存在的个体，不依附于主表存在
- 可以在OneToOne、OneToMany和ManyToMany注解中，通过cascade属性来指定级联操作

---

## 组件类包含关联实体

---

## 附录及特别说明

- **对指定了mappedBy属性的OneToOne、ManyToMany、OneToMany注解，都不能与JoinColumn或JoinTable注解一起使用**

### CascadeType的枚举值

枚举值 | 说明
:--: | :--:
ALL | 所有持久化操作都级联到关联实体
MERGE | 将merge操作级联到关联实体
PERSIST | 将persist操作级联到关联实体
REFRESH | 将refresh操作级联到关联实体
REMOVE | 将remove操作级联到关联实体

### FetchType的枚举值

枚举值 | 说明
:--: | :--:
EAGER | 立即抓取
LAZY | 延迟抓取
