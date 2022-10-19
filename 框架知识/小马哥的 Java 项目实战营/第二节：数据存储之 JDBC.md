# 第二节 数据存储之 JDBC

1. 数据库方言指的是什么？

2. 多数据源

3. Native SQL，指的是什么？

4. JDBC核心API

   1. javax.sql.DataSource

      1. 主流实现
         * Apache DBCP
           * https://commons.apache.org/
           * 间接依赖：apache commons Pool
             * 对象池，“池”化，“肉少狼多”
         * C3P0
         * HairCP
         * Alibaba Druid（字节码优化）
      2. 获取方式
         1. 普通对象初始化
            * Spring Bean
            * API 实现
         2. JNDI依赖查找

   2. 驱动接口：java.sql.Driver

   3. 驱动管理器接口：java.sql.DriverManager
   
      1. 管理器的角色
   
      2. 获取 Driver 实现，数据库驱动Driver实现会显示的调用java.sql.DriverManager#registerDriver方法，下面三种方式可以同时做
   
         1. 通过 ClassLoader 加载 Drvier 实现（由用户/应用控制）
   
            ```
            Class.forName("com.mysql.jdbc.Driver");
            ```

         2. 通过 “jdbc.drivers” 系统属性

            通过读取“jdbc.drivers”系统属性后，再经过":"的分割，尝试获取多值，再通过ClassLoader加载对应的实现类

         3. 【推荐使用】通过 Java SPI ServiceLoader 获取 Driver 实现

            ServiceLoader会初始化Driver实现类（应用主动配置），包含Class加载

      额外的知识：

      * Reflection.getCallerClass()
      * Class.forName("com.mysql.jdbc.Driver"); 底层都做了些什么？
   
   4. Connection
   
      * 获取 Connection
   
        通过 ClassLoader 类加载数据库 JDBC Driver 实现类的方式，增加 java.sql.DriverManager#registeredDrivers 字段的元素，然后通过迭代的方式逐一 尝试 getConnection 方法参数的 JDBC URL 是否可用。
   
   5. Statement
   
      1. BeanInfo和PropertyDescriptor
      2. ORM精华
   
   6. ResultSet
   
   7. ResultSetMetaData
   
   8. SQLException
   
      1. org.springframework.jdbc.core.RowCallbackHandler
   
      2. ClassUtils.wrapperToPrimitive
   
      3. Double ThrowableFunction
   
      4. Java8 CompletionFuture 
   
         也就是说，他会把Exception吃掉转成一个对象进行返回
   
         ```java
         public CompletableFuture<T> exceptionally(
             Function<Throwable, ? extends T> fn) {
             return uniExceptionallyStage(fn);
         }
         
         
         A -> B -> C -> D -> E
                     -> Exception -> E
         ```
   
         
   
      

