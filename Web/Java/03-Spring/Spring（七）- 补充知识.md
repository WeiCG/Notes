# Spring的补充知识点

> 两种后处理器、资源访问、缓存

## Spring的两种后处理器

### 后处理器概述

- Spring提供了两个常用的后处理器
    - Bean后处理器 - 对容器中的Bean进行后处理，对Bean进行额外增强
    - 容器后处理器 - 对IoC容器进行后处理，增强容器的功能
- 同时，如果Bean对象转化为切面对象后，该对象便不会在被Bean后处理器做增强处理了

### Bean后处理器

- 这是一种特殊的Bean，不对外提供服务，唯一的作用就是对容器中的其他Bean执行后处理。因此，该Bean不应该提供id属性
- 当Bean实例创建成功后，该处理器会对Bean实例进行进一步增强处理
- 作为Bean后处理器的类需要实现BeanPostProcessor接口，该接口包含如下两个方法
    - Object postProcessBeforeInitialization(Object bean, String name) - 该方法执行于Bean初始化之前，Object是系统即将后处理的Bean实例，String是后处理的Bean实例的配置id
    - Object postProcessAfterInitialization(Object bean, String name) - 该方法执行于Bean初始化之后，Object是系统即将后处理的Bean实例，String是后处理的Bean实例的配置id
    - **注意：执行完上述两个方法后，需要将处理后的Bean实例在返回给Spring容器**

### 容器后处理器

- Bean后处理器负责处理容器中所有Bean实例，而容器后处理器则负责处理容器本身
- 作为容器后处理器的类需要实现BeanFactoryPostProcessor接口，该接口只包含一个方法
    - postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) - 对Spring容器进行自定义扩展，传入的参数就是Spring容器本身

---

## 资源访问

---

## Spring的缓存机制