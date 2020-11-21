# JPQL语句查询

> Hibernate提供了异常强大的查询体系，而HQL则是Hibernate配备的查询语言，而JPQL则是JPA规范使用的查询语言，我们推荐使用JPQL来查询

## JPQL查询步骤

1. 获取JPA的EntityManager对象
2. 编写JPQL语句
3. 以JPQL语句作为参数，调用EntityManager的createQuery方法创建Query对象
4. 如果JPQL语句包含参数，则调用Query对象的setXXX方法为参数赋值
5. 调用Query对象的getResultList或getSingleResult方法返回查询结果

---

## from子句

- 语法: **from 持久化类类名**
- 作用: 从对应的持久化类中选出全部的实例
- 返回值: 返回的

---

## 关联和连接

- 语法
    - 内连接 - **from 持久化类名 inner join 持久化类中表示关联持久化类的属性名称 where 条件**
    - 左外连接 - **from 持久化类名 left outer  join 持久化类中表示关联持久化类的属性名称 where 条件**
    - 右外连接 - **from 持久化类名 right outer join 持久化类中表示关联持久化类的属性名称 where 条件**
    - 对于表示筛选条件的语句，还可以使用with关键字
        - **from 持久化类名 inner join 持久化类中表示关联持久化类的属性名称 with 筛选 where 其他条件** 
- 作用: 用于多表查询
- 返回值
- 注意: 对于集合属性，JPA默认采用的是延迟加载。如果需要改变，可以在ElementCollection注解中配置fetch属性来关闭延迟加载，或者在JPQL语句中使用join后加**fetch关键字**的方法来获取对象

---

## select子句

- 语法: select 选择的属性或持久化类 from 持久化类
- 作用: 选择返回指定的属性或直接选择某个实体
- 返回值

---

## 聚集函数

- 语法: select 聚集函数 from 持久化类
    - avg - 计算属性平均值
    - count - 统计选择对象数量
    - max - 统计属性值的最大值
    - min - 统计属性值的最小值
    - sum - 计算属性值的总和
- 作用: 在选出的属性上调用聚集函数，方便查询
- 返回值

---

## where子句

---

## 表达式

---

## order子句

---

## group by子句

---

## 子查询

---

## 分页函数