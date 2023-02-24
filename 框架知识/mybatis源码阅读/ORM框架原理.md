MyBatis ORM框架的原理



## session

* org.apache.ibatis.session.SqlSessionFactoryBuilder#build(java.io.InputStream)
    * org.apache.ibatis.session.SqlSessionFactoryBuilder#build(java.io.InputStream, java.lang.String, java.util.Properties)
        * 执行了`super(new Configuration());`
        * org.apache.ibatis.session.Configuration：存储从mybatis-config.xml配置文件信息的对象
    * org.apache.ibatis.builder.xml.XMLConfigBuilder#propertiesElement
        * 解mybatis-config.xml.xml文件中**properties**标签配置



* org.apache.ibatis.mapping.MappedStatement：存储了Mapper方法相关参数的对象

* org.apache.ibatis.session.defaults.DefaultSqlSessionFactory
    * 通过Configuration构建
    * 是org.apache.ibatis.session.SqlSessionFactory的接口实现
* org.apache.ibatis.session.defaults.DefaultSqlSessionFactory#openSessionFromDataSource
    * 从数据源中获取session
    * 