# Hibernate第四天

## 对象导航查询 - 用于一对多查询

根据id查询出某个一，再查询这个一拥有的所有多

```Java
// 查询cid = 1的客户
Customer customer = session.get(Customer.class, 1);
// 查询这个客户里面所有的联系人
Set<LinkMan> linkMan = customer.getLinkManSet();
System.out.println(linkMan.size());
```

---

## OID查询 - 用于单表查询

根据id查询某一条记录，返回对象

```Java
// 查询cid = 1的客户
Customer customer = session.get(Customer.class, 1);
System.out.println(customer);
```

---

## HQL查询 - 用于单表查询

* HQL: Hibernate Query Language是Hibernate提供的一种查询语言，和SQL语言很相似
  * 区别: 普通SQL操作数据库表和字段，HQL操作实体类和属性
* 使用HQL语句操作实体类时，使用**Query对象**

### HQL使用步骤

1. 创建Query对象，向其中传入HQL语句
2. 调用Query对象中的方法得到结果

### HQL相关操作

#### HQL查询所有

* 语法: from 实体类名称

```Java
// 创建Query对象
Query query = session.createQuery("from Customer");
List<Customer> list = query.list();

for (Customer customer : list) {
    System.out.println(customer);
}
```

#### HQL条件查询

* 普通查询: **from 实体类名称 where 实体类属性名称 = ? and 实体类属性名称 = ? ...**

```Java
// 创建Query对象
Query query = session.createQuery("from Customer where cid = ? and cName = ?");
// 设置条件值
query.setParameter(0, 1);
query.setParameter(1, "百度");

List<Customer> list = query.list();
for (Customer customer : list) {
    System.out.println(customer);
}
```

* 模糊查询: **from 实体类名称 where 实体类属性名称 = ? like ...**

```Java
// 创建Query对象
Query query = session.createQuery("from Customer where cName like ?");
query.setParameter(0, "%浪%");

// 调用方法
List<Customer> list = query.list();
for (Customer customer : list) {
    System.out.println(customer);
}
```

#### HQL排序查询

* 语法: **from 实体类名称 order by 实体类属性名称 asc/desc**
  * asc - 默认值，升序
  * desc - 降序
* 使用场景: 显示新品

```Java
// 创建Query对象
Query query = session.createQuery("from Customer order by cid desc");

// 调用方法
List<Customer> list = query.list();
for (Customer customer : list) {
    System.out.println(customer);
}
```

#### HQL分页查询

* MySQL实现分页: limit关键字
* hibernate实现分页
  * 在HQL语句中，首先查询出所有的结果对象
  * hibernate的Query对象封装了两个方法实现分页操作
    * setFirstResult - 设置分页数据开始的位置
    * setMaxResult - 设置每页的记录数
* 开始页面位置公式: (当前页数 - 1) * 每页记录数

```Java
// 创建Query对象，首先查询所有
Query query = session.createQuery("from Customer");

// 设置分页数据的开始位置
query.setFirstResult(2);
// 设置每页记录数
query.setMaxResults(3);

// 调用方法
List<Customer> list = query.list();
for (Customer customer : list) {
    System.out.println(customer);
}
```

#### HQL投影查询

* 查询表中部分字段的值
* 语法: **select 实体类属性名称1, 实体类属性名称2, ... from 实体类名称**
* 注意: select后面不能写*号，*号请使用查询所有

```Java
// 创建Query对象，首先查询所有
Query query = session.createQuery("select cName from Customer");

// 调用方法，因为是部分字段，所以List中是Object
List<Object> list = query.list();
for (Object customer : list) {
    System.out.println(customer);
}
```

#### HQL聚合函数使用

* 常用的聚合函数 - **注意: 这些函数返回的都是Long类型**
  * count - 统计个数
  * sum - 和
  * avg - 平均数
  * max - 最大值
  * min - 最小值
* 语法: **select count(*) from 实体类名称**

```Java
// 创建Query对象
Query query = session.createQuery("select count(*) from Customer");
// 返回唯一的结果的值
Object obj = query.uniqueResult();
System.out.println(obj);
```

---

## HQL查询 - 用于多表查询

### 内连接

* **显示双方都有联系的数据**
* MySQL的内连接查询
  * select * from t_customer c, t_linkman l where c.cid = l.cid;
  * select * from t_customer c inner join t_linkman l on c.cid = l.cid;
* Hibernate的内连接查询 - **建议使用迫近内连接**
  * 语句: **from 实体类名称 别名 inner join 实体类中表示对应关系的Set集合名称**
  * 该操作返回的List中每部分都是一个二维数组形式
    * 第一个数组存储左表的所有属性，包括Set集合
    * 第二个数组存储右表的所有属性
* 代码: 略

### 左外连接

* **显示左表的所有数据，并显示右表中与左表有联系的所有数据**
* MySQL的左外连接查询
  * select * from t_customer c **left outer join** t_linkman l on c.cid = l.cid;
* Hibernate的左外连接查询 - **建议使用迫近左外连接**
  * 语句: **from 实体类名称 别名 left outer join 实体类中表示对应关系的Set集合名称**
  * 该操作返回的List中每部分都是一个二维数组形式
    * 第一个数组存储左表的所有属性，包括Set集合
    * 第二个数组存储右表的所有属性
* 代码: 略

### 右外连接

* **显示右表的所有数据，并显示左表中与右表有联系的所有数据**
* MySQL的右外连接查询
  * select * from t_customer c **right outer join** t_linkman l **on** c.cid = l.cid;
* Hibernate的右外连接查询
  * 语句: **from 实体类名称 别名 right outer join 实体类中表示对应关系的Set集合名称**
  * 该操作返回的List中每部分都是一个二维数组形式
    * 第一个数组存储左表的所有属性，包括Set集合
    * 第二个数组存储右表的所有属性

### 迫切内连接 - HQL独有

* 迫切内连接与内连接底层实现是一样的
* 区别: 使用内连接返回的List中每部分是二维数组，迫切内连接返回的List每部分是左表对象
* HQL语句: **from 实体类名称 别名 inner join fetch 实体类中表示对应关系的Set集合名称**
* 代码: 略

### 迫切左外连接 - HQL独有

* 迫切左外连接与左外连接底层实现是一样的
* 区别: 使用左外连接返回的List中每部分是二维数组，迫切左外连接返回的List每部分是左表对象
* HQL语句: **from 实体类名称 别名 left outer join fetch 实体类中表示对应关系的Set集合名称**
* 代码: 略

---

## QBC查询 - 用于单表查询

* QBC查询不需要写任何语句，可以直接使用方法实现各种功能
* 使用QBC时，仍然操作实体类和属性
* QBC查询使用的是: Criteria对象

### QBC操作步骤

1. 创建Criteria对象
2. 调用方法得到结果

### QBC相关操作

#### QBC查询所有

```Java
// 创建Criteria对象
Criteria criteria = session.createCriteria(Customer.class);
List<Customer> list = criteria.list();

for (Customer c : list) {
    System.out.println(c.getCid() + ": " + c.getcName());
}
```

#### QBC条件查询

* QBC的条件查询需要借助Restrictions类来实现，该类中定义了非常多的条件语句方法
  * eq - 等于
  * gt/ge - 大于/大于等于
  * lt/le - 小于/小于等于
  * like - 模糊查询

* 普通查询

```Java
// 创建Criteria对象
Criteria criteria = session.createCriteria(Customer.class);
// 在add方法中设置条件值，类似于cid = 1
criteria.add(Restrictions.eq("cid", 1));
criteria.add(Restrictions.eq("cName", "百度"));

List<Customer> list = criteria.list();

for (Customer c : list) {
    System.out.println(c.getCid() + ": " + c.getcName());
}
```

* 模糊查询

```Java
// 创建Criteria对象
Criteria criteria = session.createCriteria(Customer.class);

// 在add方法中设置条件值，使用like方法设置模糊查询条件
criteria.add(Restrictions.like("cName", "%百%"));
List<Customer> list = criteria.list();

for (Customer c : list) {
    System.out.println(c.getCid() + ": " + c.getcName());
}
```

#### QBC排序查询

* QBC的排序查询需要借助Order类来实现，该类中定义了两个方法
  * asc - 升序方法，一般不用，因为默认就是升序
  * desc - 降序方法

```Java
// 创建Criteria对象
Criteria criteria = session.createCriteria(Customer.class);
// 设置对哪个属性进行排序，设置升/降序
criteria.addOrder(Order.desc("cid"));

List<Customer> list = criteria.list();
for (Customer c : list) {
    System.out.println(c.getCid() + ": " + c.getcName());
}
```

#### QBC分页查询

* 思路基本和HQL相同
* Criteria对象中同样有setFirstResult和setMaxResults两个方法
  * setFirstResult - 设置分页数据开始的位置
  * setMacResult - 设置每页的记录数
* 开始页面位置公式: (当前页数 - 1) * 每页记录数
  
```Java
// 创建Criteria对象
Criteria criteria = session.createCriteria(Customer.class);
// 设置分页开始位置
criteria.setFirstResult(0);
// 设置每页记录数
criteria.setMaxResults(3);

List<Customer> list = criteria.list();
for (Customer c : list) {
    System.out.println(c.getCid() + ": " + c.getcName());
}
```

#### QBC统计查询

* QBC的排序查询需要借助Projections类来实现，该类中定义了多个统计方法
  * rowcount - 统计表中有多少行

```Java
// 创建Criteria对象
Criteria criteria = session.createCriteria(Customer.class);
// 设置操作
criteria.setProjection(Projections.rowCount());

Object obj = criteria.uniqueResult();
System.out.println(obj);
```

#### 离线查询

在多层调用模型中，如果使用QBC来进行查询操作，但同时又不应该在DAO层做各种查询设置，如条件设置、排序设置等，可以在Service层或Web层使用离线Criteria的方式完成设置，在DAO层使用DetachedCriteria对象，直接生成Criteria对象，进行查询操作

* **离线即直接创建Criteria对象，而不用Session对象来创建，在只在最终执行时才使用Session对象**

```Java
// 创建离线Criteria对象
DetachedCriteria detachedCriteria = DetachedCriteria.forClass(Customer.class);
// 最终执行
Criteria criteria = detachedCriteria.getExecutableCriteria(session);
List<Customer> list = criteria.list();

for (Customer c : list) {
    System.out.println(c.getCid() + ": " + c.getcName());
}
```

---

## 本地SQL查询

使用SQLQuery对象，可以使用SQL语句实现查询操作

---

## Hibernate检索策略

### 检索策略的概念

#### hibernate检索策略的分类

* 立即查询: 根据id查询，调用方法马上发送语句查询数据库，如get方法
* 延迟查询: 根据id查询，调用方法不会马上发送语句查询数据，如load方法，只有**得到对象里面的值时(除id外的值**)才会发送语句查询数据库

#### 延迟查询的分类

* 类级别延迟: 根据id查询返回实体类对象
* 关联级别延迟: **默认开启**
  * 查询出了某个一，想查询出该一所拥有的的所有多。查询多的过程是否需要延迟便是关联级别延迟

#### 关联延迟级别的操作

1. 在一对应的映射文件中进行配置
2. 在set标签上进行配置
   * fetch属性 - 值select即可
   * lazy属性
     * true - 关联延迟，默认
     * false - 不关联延迟
     * extra - 极其关联延迟，要什么给什么，不需要的不会给

#### 批量检索

底层是调用in语句实现的

* 需求
  1. 查询所有的一，返回了List集合
  2. 遍历List集合，得到每个一所拥有的所有多

* 原来的操作，功能上可以实现，但多次查询数据库，效率不高

```Java
// 查询所有账户
Criteria criteria = session.createCriteria(Customer.class);
List<Customer> list = criteria.list();

for(Customer c : list) {
    System.out.println(c.getCid() + ": " + c.getcName());
    // 每个客户里面所有的联系人
    Set<LinkMan> linkMan = c.getLinkManSet();
    for (LinkMan l : linkMan) {
        System.out.println("  " + l.getLkm_id() + ": " + l.getLkm_name());
    }
}
```

* 批量检索方式
  1. 在一的映射文件中进行配置
  2. 在set标签上进行配置
      * batch-size属性 - 整数值，值越大，效率越高，性能越强
