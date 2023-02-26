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
    
    * org.apache.ibatis.builder.xml.XMLConfigBuilder#parseConfiguration：解析配置
    
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
    
        * 解析mapper标签，将生成mapper接口的代理工厂
        * org.apache.ibatis.binding.MapperRegistry#addMappers(java.lang.String, java.lang.Class<?>)
            * 将mapper的class获取
            * org.apache.ibatis.binding.MapperRegistry#addMapper
                * 生成mapper工厂
                * org.apache.ibatis.builder.annotation.MapperAnnotationBuilder#parse
                    * 解析mapper文件
                * org.apache.ibatis.builder.annotation.MapperAnnotationBuilder#loadXmlResource
                    * 解析mapper.xml文件
                * 

* org.apache.ibatis.mapping.MappedStatement：存储了Mapper方法相关参数的对象

* org.apache.ibatis.session.defaults.DefaultSqlSessionFactory
    * 通过Configuration构建
    * 是org.apache.ibatis.session.SqlSessionFactory的接口实现
* org.apache.ibatis.session.defaults.DefaultSqlSessionFactory#openSessionFromDataSource
    * 从数据源中获取session
    * 