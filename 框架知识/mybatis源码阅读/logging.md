# Mybatis 3.5.6- logging

## 1. org.apache.ibatis.logging.LogFactory

#### 1. 基本内容

* **采用简单工厂**
* Thread类`start`方法和`run`方法的区别，以及`new Runnable()`的Java 8写法，具体可以参考：org.apache.ibatis.logging.example.LogFactoryExample
* 反射调用构造方法的使用，参见：org.apache.ibatis.logging.example.ConstructorExample

  *  getConstructor()和getDeclaredConstructor()区别：
    1. `getDeclaredConstructor()`这个方法会返回制定参数类型的所有构造器，包括public的和非public的，当然也包括private的。
    2. `getConstructor()`这个方法返回的是上面那个方法返回结果的子集，**只返回制定参数类型访问权限是public的**构造器。
* 泛型的使用，参见：org.apache.ibatis.rcode.example.logging.Generic

#### 2. 拓展知识

* Slf4j和log4j，logback，java.util.logging等日志框架的区别，可以参考博文：https://www.cnblogs.com/hanszhao/p/9754419.html
* Slf4j有一个基本要求：**嵌入式组件(如库或框架)不应该声明对任何SLF4J绑定的依赖，而应该仅依赖于SLF4J -api。**正因如此你可以看到mybatis中的第三方绑定日志组件的maven依赖都是`<optional>true</optional>`。
* log4j和log4j2的区别，他两是否都用slf4j-log412.jar来进行适配？

#### 3. mybatis项目pom依赖中的日志组件

* log4j

  ```xml
  <dependency>
      <groupId>log4j</groupId>
      <artifactId>log4j</artifactId>
      <version>1.2.17</version>
      <optional>true</optional>
  </dependency>
  ```

  这里引入了log4j 1.x版本，目前1.2.17版本是1.x版本的最后一个版本。目前看这个包是在org.apache.ibatis.logging.log4j这个里面用到的它的api。

* log4j-core

  ```
  
  ```

  

## 2. slf4j包

#### 1. org.apache.ibatis.logging.slf4j.Slf4jImpl

这个类我一直在考虑它用的是什么设计模式，目前锁定的三种设计模式，分别是代理模式、适配器模式和装修器模式。根据反复分析这三种设计模式，我认为都不是这三种模式，而是**合成复用原则**。





