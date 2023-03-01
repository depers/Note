# MyBatis ORM框架的原理



## 1.Mybatis-config.xml文件解析

* org.apache.ibatis.session.SqlSessionFactoryBuilder#build(java.io.InputStream)

    * org.apache.ibatis.session.SqlSessionFactoryBuilder#build(java.io.InputStream, java.lang.String, java.util.Properties)

        * 执行

            ```
            protected final MapperRegistry mapperRegistry = new MapperRegistry(this);
            ```

        * 执行了`super(new Configuration());`

            ```
            typeAliasRegistry.registerAlias("POOLED", PooledDataSourceFactory.class);
            ```

        * org.apache.ibatis.session.Configuration：存储从mybatis-config.xml配置文件信息的对象

    * org.apache.ibatis.builder.xml.XMLConfigBuilder#parseConfiguration：解析数据库配置

        * org.apache.ibatis.builder.xml.XMLConfigBuilder#propertiesElement
            * 解析mybatis-config.xml文件中**properties**标签配置
        * org.apache.ibatis.builder.xml.XMLConfigBuilder#environmentsElement
            * 解析mybatis-config.xml文件中**environments**标签配置
            * org.apache.ibatis.builder.xml.XMLConfigBuilder#transactionManagerElement
                * org.apache.ibatis.transaction.jdbc.JdbcTransactionFactory，获取事物工厂
            * org.apache.ibatis.builder.xml.XMLConfigBuilder#dataSourceElement
                * org.apache.ibatis.datasource.pooled.PooledDataSourceFactory#PooledDataSourceFactory，获取数据库连接池的工厂
                * datasource是org.apache.ibatis.datasource.pooled.PooledDataSource

    * org.apache.ibatis.builder.xml.XMLConfigBuilder#mapperElement

        * 解析mapper包下标签，将生成mapper接口的代理工厂

        * org.apache.ibatis.binding.MapperRegistry#addMappers(java.lang.String, java.lang.Class<?>)

            * 将mapper的class获取

            * org.apache.ibatis.binding.MapperRegistry#addMapper

                * 生成mapper工厂

                    ````
                    knownMappers.put(type, new MapperProxyFactory<>(type));
                    ````

            * org.apache.ibatis.builder.annotation.MapperAnnotationBuilder#parse

                    * 解析mapper文件和注解

                 * org.apache.ibatis.builder.annotation.MapperAnnotationBuilder#loadXmlResource
                    * 解析mapper.xml文件
                    * org.apache.ibatis.builder.xml.XMLMapperBuilder#parse
                        * 解析mapper.xml文件
                        * org.apache.ibatis.builder.MapperBuilderAssistant#addMappedStatement(java.lang.String, org.apache.ibatis.mapping.SqlSource, org.apache.ibatis.mapping.StatementType, org.apache.ibatis.mapping.SqlCommandType, java.lang.Integer, java.lang.Integer, java.lang.String, java.lang.Class<?>, java.lang.String, java.lang.Class<?>, org.apache.ibatis.mapping.ResultSetType, boolean, boolean, boolean, org.apache.ibatis.executor.keygen.KeyGenerator, java.lang.String, java.lang.String, java.lang.String, org.apache.ibatis.scripting.LanguageDriver, java.lang.String)
                            * 将mapper.xml文件中的方法，添加到Configuration的mappedStatements属性中
                * org.apache.ibatis.builder.annotation.MapperAnnotationBuilder#parseStatement
                    * 猜测是解析注解写法的sql的逻辑

    * org.apache.ibatis.session.defaults.DefaultSqlSessionFactory

        * 通过Configuration构建
        * 是org.apache.ibatis.session.SqlSessionFactory的接口实现

## 2. 获取session

* org.apache.ibatis.session.defaults.DefaultSqlSessionFactory#openSessionFromDataSource
    * 从数据源中获取session
* 

## 3. 执行

* org.apache.ibatis.session.SqlSession#getMapper

    * 获取解析好的mapper
    * org.apache.ibatis.binding.MapperRegistry#getMapper
        * 最终拿到的就是org.apache.ibatis.binding.MapperRegistry#knownMappers属性里面之前已经生成好的MapperProxyFactory
        * 然后通过MapperProxyFactory生成org.apache.ibatis.binding.MapperProxy实现接口的代理类

* 执行mapper中的方法

    * org.apache.ibatis.binding.MapperProxy#invoke

        * 执行代理类的方法

        * org.apache.ibatis.binding.MapperProxy#cachedInvoker

            * 缓存执行的方法

                ```
                return new PlainMethodInvoker(new MapperMethod(mapperInterface, method, sqlSession.getConfiguration()));
                ```

            * org.apache.ibatis.binding.MapperMethod.SqlCommand#SqlCommand

                * org.apache.ibatis.binding.MapperMethod.SqlCommand#resolveMappedStatement
                    * 根据接口方法名获取之前在Configuration#mappedStatements的MappedStatement

            * org.apache.ibatis.binding.MapperMethod.MethodSignature

                * org.apache.ibatis.reflection.TypeParameterResolver#resolveType
                    * 解析返回值类型
                * org.apache.ibatis.reflection.ParamNameResolver#ParamNameResolver
                    * 解析@Param注解，获取参数map

            * org.apache.ibatis.binding.MapperProxy.PlainMethodInvoker#invoke

                * 



# 核心数据结构分析

1. org.apache.ibatis.session.Configuration
    * org.apache.ibatis.binding.MapperRegistry
        * org.apache.ibatis.binding.MapperRegistry#knownMappers
            * org.apache.ibatis.binding.MapperProxyFactory#MapperProxyFactory
    * org.apache.ibatis.session.Configuration#mappedStatements
        * org.apache.ibatis.mapping.MappedStatement
            * org.apache.ibatis.mapping.MappedStatement#sqlSource
    * org.apache.ibatis.mapping.Environment
        * org.apache.ibatis.transaction.TransactionFactory
        * javax.sql.DataSource
2. org.apache.ibatis.session.SqlSession
    * org.apache.ibatis.session.defaults.DefaultSqlSession
        * org.apache.ibatis.executor.Executor
3. org.apache.ibatis.binding.MapperProxy
    * org.apache.ibatis.binding.MapperProxy#sqlSession
    * org.apache.ibatis.binding.MapperProxy#mapperInterface
4. org.apache.ibatis.binding.MapperMethod
    * org.apache.ibatis.binding.MapperMethod.SqlCommand
    * org.apache.ibatis.binding.MapperMethod.MethodSignature