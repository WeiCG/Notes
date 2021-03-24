# 批量处理命令

> 使用Hibernate来解决批量插入、批量更新和批量删除三方面

## 批量插入

- 批量插入中，由于Hibernate默认开启的一级缓存，因此如果同时插入过多的数据，会抛出OutOfMemoryException异常，因此需要我们在每隔一段时间便显示刷新一级缓存

```java
public class NewsManager {
    @Test
    public void Test() {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpa");
        EntityManager em = emf.createEntityManager();
        EntityTransaction transaction = em.getTransaction();
        transaction.begin();

        for (int i = 0; i < 10000; i++) {
            Practice p = new Practice();
            p.setName("哈哈" + i);
            p.setAge(i);
			
           	// 保存数据
            em.persist(p);
            // 每隔一段时间刷新Hibernate一级缓存
            if (i % 20 == 0) {
                em.flush();
                em.clear();
            }
        }

        transaction.commit();
        em.close();
        emf.close();
    }
}
```

## 批量更新和删除

- 批量更新和批量删除一般都是使用JPQL语句来完成的，通过调用Query对象的executeUpdate方法来执行批量更新与删除操作
- 更新语句: **update 实体类名称 别名 set 要更改的字段**
- 删除语句: **delete 实体类名称**

