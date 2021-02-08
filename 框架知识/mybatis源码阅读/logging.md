# Mybatis 3.5.6- logging

## 1. org.apache.ibatis.logging.LogFactory

* **采用简单工厂**
* Thread类`start`方法和`run`方法的区别，以及`new Runnable()`的Java 8写法，具体可以参考：org.apache.ibatis.logging.example.LogFactoryExample
* 反射调用构造方法的使用，参见：org.apache.ibatis.logging.example.ConstructorExample

  *  getConstructor()和getDeclaredConstructor()区别：
    1. `getDeclaredConstructor()`这个方法会返回制定参数类型的所有构造器，包括public的和非public的，当然也包括private的。
    2. `getConstructor()`这个方法返回的是上面那个方法返回结果的子集，**只返回制定参数类型访问权限是public的**构造器。
* 泛型的使用，参见：org.apache.ibatis.rcode.example.logging.Generic

## 2. slf4j包

#### 1. org.apache.ibatis.logging.slf4j.Slf4jImpl

这个类我一直在考虑它用的是什么设计模式，目前锁定的三种设计模式，分别是代理模式、适配器模式和装修器模式。根据反复分析这三种设计模式，我认为都不是这三种模式，而是**合成复用原则**。





