# Hibernate第二天

## Hibernate的一级缓存

### Hibernate缓存

hibernate框架中提供了很多优化方式，缓存就是一种优化方式

#### 缓存特点

* 一级缓存
  * hibernate的一级缓存默认打开
  * 一级缓存的使用范围是hibernate的Session对象的范围
    * 即从Session对象创建到Session对象关闭的范围
  * hibernate的一级缓存中，存储的数据必须是持久态数据

* 二级缓存 - 已经不在使用，现在都在使用redis
  * 二级缓存默认不是打开的，需要配置
  * 二级缓存的使用范围是hibernate的SessionFactory对象的范围

### 验证一级缓存

1. 首先根据uid = 1查询，返回对象
2. 其次再根据uid = 1查询，返回对象

```Java
@Test
public void testDelete() {
    // 同添加操作的前三行代码
    ...
    // 1.根据id进行查询
    User user1 = session.get(User.class, 4);
    System.out.println(user1);

    // 2.再根据id进行查询
    User user2 = session.get(User.class, 4);
    System.out.println(user2);
    // 同添加操作的后三行代码
    ...
}
```

![查询结果](./picture/day02_01.png)

可以明显看出，虽然查询了两次，但只访问了一次数据库

同时需要注意的是，两次查询后的结果实际指向的是同一个对象

### 一级缓存特性

* **获取持久态对象会自动更新数据库**

```Java
@Test
public void testDelete() {
    // 同添加操作的前三行代码
    ...
    // 1.根据id进行查询
    User user = session.get(User.class, 4);
    System.out.println(user);

    // 2.设置返回对象值
    user.setUsername("韩梅梅");
    // session.update(user);
    // 同添加操作的后三行代码
    ...
}
```

![执行结果](picture/day02_02.png)

可以看到我们并没有执行update方法，执行程序之后仍然执行了更新操作，则说明持久态对象会自动更新数据库

#### 原理

* 在使用查询获取持久态对象时，对象除了在一级缓存中会备份外，还会在一个叫**快照区**的数据域进行备份，对持久态对象的每一次操作都会同时作用到一级缓存中
* 当执行事务的commit操作时，**hibernate会将快照区和一级缓存中的数据进行比对**，当不相同时，hibernate会自动执行update操作，将一级缓存中的数据更新到数据库中

---

## Hibernate的事务操作 - 重要

### 事务的概念

1. 什么是事务？
2. 事务的特性？
3. 不考虑隔离性产生的问题？
   * 脏读
   * 不可重复度
   * 虚读
4. MySQL默认隔离级别: repeatable read

### 事务代码的规范写法

#### 代码格式

```Java
@Test
public void testTx() {
    SessionFactory sessionFactory = null;
    Session session = null;
    Transaction tx = null;
    try {
        sessionFactory = HibernateUtils.getSessionFactory();
        session = sessionFactory.openSession();
        // 开启事务
        tx = session.beginTransaction();
        // 执行CRUD操作
        ...
        // 提交事务
        tx.commit();
    } catch (Exception e) {
        // 回滚事务
        tx.rollback();
    } finally {
        // 关闭资源
        session.close();
        sessionFactory.close();
    }
}
```

#### 获取与本地线程绑定的Session对象

hibernate已经实现了Session对象与本地线程的绑定，其中底层仍然是使用**ThreadLocal**实现的

1. 在hibernate核心配置文件中配置

    ```XML
    <!-- 配置与本地线程绑定的Session，要配置在hibernate信息中 -->
    <property name="hibernate.current_session_context_class">thread</property>
    ```

2. 调用SessionFactory里面的getCurrentSession方法，即可得到
3. 获取与本地线程绑定的Session对象后，关闭Session时会报错，因为Session与线程相关联，所以当线程结束后，Session会自动关闭，**不需要显示关闭Session**
