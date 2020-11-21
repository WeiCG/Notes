# 开发环境和开发步骤

> JPA是Java EE规范之一，官方建议开发者应该使用JPA提供的标准API来进行持久层开发，因此在学习Hibernate开发时，推荐全部遵循JPA规范

## 配置开发环境

### 导包

- org.hibernate: hibernate-core: 核心类库
- org.hibernate: hibernate-entitymanager: Hibernate与JPA的实体类库
- log4j: log4j: 日志
- org.slf4j: slf4j-api: 日志
- org.slfj4: slf4j-log4j12: 日志
- mysql: mysql-connnector-java: MySQL驱动

### 创建持久化类 - 详见Hibernate（二）

- 创建一个实体类
- 为实体类添加对应的映射注解

```java
@Entity
@Table(name = "practice")
public class Practice {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "p_id")
    private Integer id;

    @Column(name = "p_name")
    private String name;

    @Column(name = "p_age")
    private Integer age;

    // get/set方法
}
```

### 创建JPA配置文件

- **在Resource目录下新建META-INF目录，新建配置文件persistence.xml，该目录固定**

- 关于JPA配置文件中更多关于Hibernate的内容**详见Hibernate补充知识（一）**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<persistence version="2.1"
             xmlns="http://xmlns.jcp.org/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence
             http://xmlns.jcp.org/xml/ns/persistence/persistence_2_1.xsd">
<!-- name - 指定JPA的名称，在接下来引入JPA配置文件时需要使用
	 transaction-type - 配置JPA事务处理策略
		RESOURCE_LOCAL: 默认值，只能针对一种数据库，不支持分布式事务
		JTA: 支持分布式事务 -->
<persistence-unit name="jpa" transaction-type="RESOURCE_LOCAL">
    	<!-- 配置JPA实现 -->
		<provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        <!-- 配置持久化类 -->
    	<class>com.wei.entity.Practice</class>
    
    	<!-- 配置各种属性 -->
        <properties>
            <!-- 配置数据库属性 -->
            <property name="javax.persistence.jdbc.driver" 
                      value="com.mysql.jdbc.Driver"/>
            <property name="javax.persistence.jdbc.url"
                      value="jdbc:mysql://localhost:3306/hibernate?useSSL=false"/>
            <property name="javax.persistence.jdbc.user" 
                      value="xxx"/>
            <property name="javax.persistence.jdbc.password" 
                      value="xxx"/>
            
            <!-- 配置Hibernate的各种属性 -->
            <property name="hibernate.hbm2ddl.auto" value="update"/>
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.MySQL57Dialect"/>
        </properties>
    </persistence-unit>
</persistence>
```

---

## JPA的开发步骤

```java
public class NewsManager {
    @Test
    public void Test() {
        // 导入JPA的配置文件，这里的jpa是配置文件中persistence-unit的name属性
        // 创建EntityManagerFactory对象，相当于Hibernate中的SessionFactory对象
        EntityManagerFactory emf = 
            Persistence.createEntityManagerFactory("jpa"); 
        // 创建EntityManager对象，相当于Hibernate中的Session对象
        EntityManager em = emf.createEntityManager();
		
        // 获取EntityTransaction对象，管理事务
        EntityTransaction transaction = em.getTransaction();
        // 开启事务
        transaction.begin();

        Practice p = new Practice();
        p.setName("呵呵");
        p.setAge(20);

        // 保存
        em.persist(p);
		
        // 提交事务
        transaction.commit();
		// 关闭各种资源
        em.close();
        emf.close();
    }
}
```

