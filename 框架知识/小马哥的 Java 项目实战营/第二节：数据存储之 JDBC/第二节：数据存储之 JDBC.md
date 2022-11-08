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

      2. 获取 Driver 实现，下面三种方式可以同时做

         1. 通过 ClassLoader 加载 Drvier 实现（由用户/应用控制），数据库驱动Driver实现会显示的调用java.sql.DriverManager#registerDriver方法

            ```java
            Class.forName("com.mysql.jdbc.Driver");
            ```

         2. 通过 “jdbc.drivers” 系统属性

            通过读取“jdbc.drivers”系统属性后，再经过":"的分割，尝试获取多值，再通过ClassLoader加载对应的实现类。

            加载顺序和属性值的顺序有关系。

         3. 【推荐使用】通过 Java SPI ServiceLoader 获取 Driver 实现
         
            ServiceLoader会初始化Driver实现类（应用程序去主动配置的），包含Class加载。
            
            驱动的加载顺序与Class Path的顺序有关系。
            
            Java的SPI会一次性全部加载驱动，比较浪费资源。

      3. 额外的知识：

         * Reflection.getCallerClass()

         * Class.forName("com.mysql.jdbc.Driver"); 底层都做了些什么？

      4. 问题集合

         1. 当多个 Driver 同时被加载到 ClassLoader 后，到底用了哪个？

            getConnection 方法是通过 JDBC URL 判断的，通过迭代多次，返回第一个成功的 Connection 实例，参考java.sql.DriverManager#getConnection(java.lang.String, java.util.Properties, java.lang.Class<?>)方法。

            ```java
            for(DriverInfo aDriver : registeredDrivers) {
                // If the caller does not have permission to load the driver then
                // skip it.
                if(isDriverAllowed(aDriver.driver, callerCL)) {
                    try {
                        println("    trying " + aDriver.driver.getClass().getName());
                        Connection con = aDriver.driver.connect(url, info);
                        if (con != null) {
                            // Success!
                            println("getConnection returning " + aDriver.driver.getClass().getName());
                            return (con);
                        }
                    } catch (SQLException ex) {
                        if (reason == null) {
                            reason = ex;
                        }
                    }
            
                } else {
                    println("    skipping: " + aDriver.getClass().getName());
                }
            }
            ```

         2. java.sql.DriverManager#loadInitialDrivers 方法中 JavaSPI 空便利的意义在哪里？

            ```java
            try{
            	while(driversIterator.hasNext()) {
            	driversIterator.next();
            	}
            } catch(Throwable t) {
            	// Do nothing
            }
            ```

            ServiceLoader#next() 方法会主动触发 ClassLoader 加载。看这个方法：java.util.ServiceLoader.LazyIterator#nextService。

   4. 数据连接接口 - java.sql.Connection

      * 相近语义术语

        一个 JDBC Connection 相当于 MyBatis Session 或者Hibernate Session

      * 获取 Connection

        通过 ClassLoader 类加载数据库 JDBC Driver 实现类的方式，增加 java.sql.DriverManager#registeredDrivers 字段的元素，然后通过迭代的方式逐一 尝试 getConnection 方法参数的 JDBC URL 是否可用。

   5. SQL 命令接口 - java.sql.Statement

      1. 主要类型

         * 普通 SQL 命令 - java.sql.Statement
         * 预编译 SQL 命令 - java.sql.PreparedStatement
         * 存储过程 SQL 命令 - java.sql.CallableStatement

      2. DDL语句和DML语句

         * DML 语句 ：CRUD

           * R（读操作）：java.sql.Statement#executeQuery
           * CUD（增删改）：java.sql.Statement#executeUpdate(java.lang.String)

         * DDL 语句

           使用java.sql.Statement#execute(java.lang.String)

           * 成功的话：不需要返回值（返回值 false）
           * 失败的话：SQLException

      3. BeanInfo和PropertyDescriptor

      4. ORM精华

   6. SQL 执行结果接口 - java.sql.ResultSet

      * resultset.next()，如果存在且游标滚动（这里有两个语义）

   7. ResultSetMetaData

      列的个数、名称以及类型等

   8. SQLException

      * 基本特点
        1. 几乎所有的 JDBC API 操作都需要 try catch去捕获java.sql.SQLException
        2. java.sql.SQLException 属于检查类型异常，继承Exception

      1. org.springframework.jdbc.core.RowCallbackHandler，Spring是如何捕获SQLException的

      2. ClassUtils.wrapperToPrimitive，Spring将包装类型转换为原始类型

      3. Double ThrowableFunction，自己封住的将Function中的异常抛出来

      4. Java8 CompletableFuture 

         也就是说，他会把Exception吃掉，转成一个你的方法返回值对象进行返回

         ```java
         public CompletableFuture<T> exceptionally(
             Function<Throwable, ? extends T> fn) {
             return uniExceptionallyStage(fn);
         }
         
         
         A -> B -> C -> D -> E
                     -> Exception -> E
         ```

      5. 通用的日志处理Consumer

         ```java
         /**
         * 通用处理方式
         */
         private static Consumer<Throwable> COMMON_EXCEPTION_HANDLER = e -> logger.log(Level.SEVERE, e.getMessage());
         ```

5. 事务

   * 部分回滚

