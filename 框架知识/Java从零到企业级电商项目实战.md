# Java 从零到企业级电商项目实战

## 开发环境的安装

### vsftpd

* yum install vsftpd

* 添加ftp用户：useradd ftpuser -d /usr/local/webapp/ftpfile -s /sbin/nologin

* 授权：chown -R ftpuser.ftpuser /usr/local/webapp/ftpfile

* 设置密码：passwd ftpuser

* 查找ftp安装的位置：whereis vsftpd

*  vim /etc/vsftpd/vsftpd.conf

* 配置

  ```
  # 下面这些是加的
  local_root=/usr/local/webapp/ftpfile
  # anno_root=/usr/local/webapp/ftpfile 匿名访问的路径
  use_localtime=yes
  
  pasv_min_port=61001
  pasv_max_port=62000
  
  # 下面这些是改的
  ftpd_banner=Welcome to mall FTP service.
  chroot_local_user=NO
  chroot_list_enable=YES
  chroot_list_file=/etc/vsftpd/chroot_list
  anonymous_enable=NO
  ```

* 在/etc/vsftpd/下面创建chroot_list文件

  ```
  ftpuser
  ```

* 防火墙配置：

  centos6

  ```
  # vsftpd
  -A INPUT -p TCP --dport 61001:62000 -i ACCEPT
  -A INPUT -p TCP --dport 61001:62000 -i ACCEPT
  
  -A INPUT -p TCP --dport 20 -i ACCEPT
  -A INPUT -p TCP --dport 21 -i ACCEPT
  -A OUTPUT -p TCP --dport 20 -i ACCEPT
  -A OUTPUT -p TCP --dport 21 -i ACCEPT
  ```

  centos7

  我关闭了防火墙，直接在阿里云上打开的访问端口

* 启动ftp，输入`systemctl start vsftpd`

* 检测

  * 打开浏览器检测ftp://39.106.115.25/
  * 使用命令行检测`ftp 39.106.115.25`

* 修改`vim /etc/selinux/config`

  ```
  SELINUX=disabled
  ```

* 然后命令行输入`setenforce 0`

* 然后重启ftp服务器`systemctl restart vsftpd`

### Nginx的配置

#### 1.指向端口（learning.happymall.com.conf）

```nginx
server { 
	listen 80; 
	autoindex on; 
	server_name learning.happymmall.com; 
	access_log /usr/local/nginx/logs/access.log combined; 
	index index.html index.htm index.jsp index.php; 
	#error_page 404 /404.html; 
	if ( $query_string ~* ".*[\;'\<\>].*" ){ 
		return 404; 
	} 

	location / { 
		proxy_pass http://127.0.0.1:81/learning/;
		add_header Access-Control-Allow-Origin *; 
	} 
}
```

#### 2.指向目录（img.happymall.com/conf）

```nginx
server { 
	listen 80; 
	autoindex off; 
	server_name img.happymmall.com; 
	access_log /usr/local/nginx/logs/access.log combined; 
	index index.html index.htm index.jsp index.php; 
	#error_page 404 /404.html; 
	if ( $query_string ~* ".*[\;'\<\>].*" ){ 
		return 404; 
	} 
	
	location ~ /(mmall_fe|mmall_admin_fe)/dist/view/* { 
		deny all; 
	} 

	location / { 
		root /product/ftpfile/img/; 
		add_header Access-Control-Allow-Origin *; 
	} 
}
```


## Java SSH项目初始化

### Navicat 初始化项目数据库

### 安装IDEA

### maven创建web项目
#### 1. 配置JDK
#### 2. 配置maven

#### 3.配置快捷键设置
#### 4.通过maven的archetype创建web空白项目

* maven-atchetype-webapp的项目原型

#### 5.初始化项目文件夹结构

* 在main下创建Java-->mark Java文件为Source Root
* 在src下创建test-->mark test文件为Test Sources Root

#### 6.配置Tomcat
* 在Run/Debug Configurations下配置tomcat
* 其中在Deployment界面下，添加Artfact，选择War包
#### 7.发布验证

* 访问localhost://8080

### 创建Git仓库及初始化
#### 1.安装git及ssh的配置
#### 2.创建和使用git仓库
#### 3.git初始化

* 创建README文件，`touch README.md`

#### 4.gitignore文件配置

* 创建.gitignore文件，`touch .gitignore`

#### 5.添加更新文件

* `git init`
* `git add .`
* `git status`

#### 6.推送至远程git仓库
* `git commit -am 'first commit init project'`
* `git remote add origin https://...`
* `git push -u origin master`
* `git push -u -f origin master` 强制推送

#### 7.创建及切换分支（分支开发，主干发布）

* `git branch ` 查看本机分支

* `git branch -r ` 查看远程分支
* `git checkout -b v1.0 origin/master` 在master的基础上创建v1.0的分支
* `git push origin HEAD -u`  将分支推送

### maven之POM初始化
#### 1.认识maven的pom文件
#### 2.支付宝SDK的编译发布

```xml
<plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                    <compilerArguments>
                        <extdirs>${project.basedir}/src/main/webapp/WEB-INF/lib</extdirs>
                    </compilerArguments>
                </configuration>
            </plugin>
```



### 项目文件的包结构

```
|./src/main/java/
|--com/fmmall/
|--|--common
|--|--controller
|--|--dao
|--|--pojo
|--|--service
|--|--util
|--|--vo
```



### Mybatis三剑客
#### 1.Mybatis-generator
> 自动化生成数据库交互代码

* 引入jar包

  ```xml
  <plugin>
      <groupId>org.mybatis.generator</groupId>
      <artifactId>mybatis-generator-maven-plugin</artifactId>
      <version>1.3.2</version>
      <configuration>
      <verbose>true</verbose>
      <overwrite>true</overwrite>
      </configuration>
  </plugin>
  ```

* generatorConfig.xml配置

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE generatorConfiguration
          PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
          "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
  
  <generatorConfiguration>
      <!--导入属性配置-->
      <properties resource="datasource.properties"></properties>
  F
      <!--指定特定数据库的jdbc驱动jar包的位置-->
      <classPathEntry location="D:\Program Files (x86)\maven\repository\mysql\mysql-connector-java\5.1.6\mysql-connector-java-5.1.6.jar"/>
  
      <context id="default" targetRuntime="MyBatis3">
  
          <!-- optional，旨在创建class时，对注释进行控制 -->
          <commentGenerator>
              <property name="suppressDate" value="true"/>
              <property name="suppressAllComments" value="true"/>
          </commentGenerator>
  
          <!--jdbc的数据库连接 -->
          <jdbcConnection
                  driverClass="${db.driverClassName}"
                  connectionURL="${db.url}"
                  userId="${db.username}"
                  password="${db.password}">
          </jdbcConnection>
  
  
          <!-- 非必需，类型处理器，在数据库类型和java类型之间的转换控制-->
          <javaTypeResolver>
              <property name="forceBigDecimals" value="false"/>
          </javaTypeResolver>
  
  
          <!-- Model模型生成器,用来生成含有主键key的类，记录类 以及查询Example类
              targetPackage     指定生成的model生成所在的包名
              targetProject     指定在该项目下所在的路径
          -->
          <!--<javaModelGenerator targetPackage="com.mmall.pojo" targetProject=".\src\main\java">-->
          <javaModelGenerator targetPackage="com.fmall.pojo" targetProject="./src/main/java">
              <!-- 是否允许子包，即targetPackage.schemaName.tableName -->
              <property name="enableSubPackages" value="false"/>
              <!-- 是否对model添加 构造函数 -->
              <property name="constructorBased" value="true"/>
              <!-- 是否对类CHAR类型的列的数据进行trim操作 -->
              <property name="trimStrings" value="true"/>
              <!-- 建立的Model对象是否 不可改变  即生成的Model对象不会有 setter方法，只有构造方法 -->
              <property name="immutable" value="false"/>
          </javaModelGenerator>
  
          <!--mapper映射文件生成所在的目录 为每一个数据库的表生成对应的SqlMap文件 -->
          <!--<sqlMapGenerator targetPackage="mappers" targetProject=".\src\main\resources">-->
          <sqlMapGenerator targetPackage="mappers" targetProject="./src/main/resources">
              <property name="enableSubPackages" value="false"/>
          </sqlMapGenerator>
  
          <!-- 客户端代码，生成易于使用的针对Model对象和XML配置文件 的代码
                  type="ANNOTATEDMAPPER",生成Java Model 和基于注解的Mapper对象
                  type="MIXEDMAPPER",生成基于注解的Java Model 和相应的Mapper对象
                  type="XMLMAPPER",生成SQLMap XML文件和独立的Mapper接口
          -->
  
          <!-- targetPackage：mapper接口dao生成的位置 -->
          <!--<javaClientGenerator type="XMLMAPPER" targetPackage="com.mmall.dao" targetProject=".\src\main\java">-->
          <javaClientGenerator type="XMLMAPPER" targetPackage="com.fmall.dao" targetProject="./src/main/java">
              <!-- enableSubPackages:是否让schema作为包的后缀 -->
              <property name="enableSubPackages" value="false" />
          </javaClientGenerator>
  
  
          <table tableName="mmall_shipping" domainObjectName="Shipping" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>
          <table tableName="mmall_cart" domainObjectName="Cart" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>
          <table tableName="mmall_cart_item" domainObjectName="CartItem" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>
          <table tableName="mmall_category" domainObjectName="Category" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>
          <table tableName="mmall_order" domainObjectName="Order" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>
          <table tableName="mmall_order_item" domainObjectName="OrderItem" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>
          <table tableName="mmall_pay_info" domainObjectName="PayInfo" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>
          <table tableName="mmall_product" domainObjectName="Product" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false">
              <columnOverride column="detail" jdbcType="VARCHAR" />
              <columnOverride column="sub_images" jdbcType="VARCHAR" />
          </table>
          <table tableName="mmall_user" domainObjectName="User" enableCountByExample="false" enableUpdateByExample="false" enableDeleteByExample="false" enableSelectByExample="false" selectByExampleQueryId="false"></table>
  
  
          <!-- geelynote mybatis插件的搭建 -->
      </context>
  </generatorConfiguration>
  ```

* datasource.properties

  ```properties
  db.driverLocation=D:\\Program Files (x86)\\maven\\repository\\mysql\\mysql-connector-java\\5.1.6\\mysql-connector-java-5.1.6.jar
  db.driverClassName=com.mysql.jdbc.Driver
  
  #db.url=jdbc:mysql://192.1.1.1:3306/mmall?characterEncoding=utf-8
  db.url=jdbc:mysql://127.0.0.1:3306/learning_mmall?characterEncoding=utf-8
  db.username=root
  db.password=fx1212
  ```

* 生成文件

  在IDEA右侧的Maven Project的Plugins中点击mybatis-generator下的mybatis-generator.generate

#### 2.Mybatis-plugin

> IDEA插件，实现了mybatis接口文件和实现xml自动跳转，验证争取行，在xml中智能提示等功能。

#### 3.Mybatis-pagehelper
> Mybatis非常好用的分页组件。

* 添加依赖

### Spring配置初始化
#### 1.Spring相关的项目参考
> 配置文件都是从官方demo中学习配置的

[Spring-mvc-showcase](https://github.com/spring-projects/spring-mvc-showcase)

[Spring-petclinic]()

[greenhouse]()

[Spring-boot]()
#### 2.web.xml的配置初始化

* 过滤器的配置，转码用的

  ```xml
  <filter>
          <filter-name>characterEncodingFilter</filter-name>
          <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
          <init-param>
              <param-name>encoding</param-name>
              <param-value>UTF-8</param-value>
          </init-param>
          <init-param>
              <param-name>forceEncoding</param-name>
              <param-value>true</param-value>
          </init-param>
      </filter>
      <filter-mapping>
          <filter-name>characterEncodingFilter</filter-name>
          <url-pattern>/*</url-pattern>  <!-- 拦截所有请求-->
      </filter-mapping>
  ```

* 监视web容器启动和关闭的监听器

  ```xml
   <listener>
          <listener-class>org.springframework.web.context.request.RequestContextListener</listener-class>
      </listener>
  ```

* 将Spring容器和web容器进行整合的监听器

  ```xml
  <listener>
          <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
      </listener>
  ```

* 通过ContextLoaderListener加载Spring的配置文件

  ```xml
  <context-param>
          <param-name>contextConfigLocation</param-name>
          <param-value>
              classpath:applicationContext.xml
          </param-value>
      </context-param>
  ```

* Spring mvc的拦截器的配置

  ```xml
      <servlet>
          <servlet-name>dispatcher</servlet-name>
          <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
          <!--默认的配置的话是不需要写下面的这一段的，如果要修该文件名称则就要在这里声明。-->
          <init-param>
              <param-name>contextConfigLocation</param-name>
              <param-value>WEB-INF/dispatcher-servlet.xml</param-value>
          </init-param>
          <load-on-startup>1</load-on-startup> <!--当大于0是，在web容器在启动时会初始化这个Servlet-->
      </servlet>
  
      
      <servlet-mapping>
          <servlet-name>dispatcher</servlet-name>
          <url-pattern>*.do</url-pattern> <!--会拦截所有的.do结尾的请求-->
      </servlet-mapping>
  ```
#### 3.Spring容器配置文件applicationContext.xml、applicationContext-datasource.xml、mmall.properties配置

* applicationContext.xml配置

  ```xml
      <!--添加注解扫描的包-->
      <context:component-scan base-package="com.fmall" annotation-config="true"/>
  
      <!--AOP的配置-->
      <aop:aspectj-autoproxy/>
  
      <!--将Spring配置文件进行分离-->
      <import resource="applicationContext-datasource.xml"/>
  ```

* applicationContext-datasource.xml配置

  * 添加注解扫描

    ```xml
    <!--添加注解扫描-->
        <context:component-scan base-package="com.fmall" annotation-config="true"/>
    ```

  * 配置数据库

    ```xml
        <!--将Spring配置中的常量进行分离-->
        <bean id="propertyConfigurer"
              class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
            <property name="order" value="2"/>
            <property name="ignoreUnresolvablePlaceholders" value="true"/>
            <property name="locations">
                <list>
                    <value>classpath:datasource.properties</value>
                </list>
            </property>
            <property name="fileEncoding" value="utf-8"/>
        </bean>
    
    
        <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
            <property name="driverClassName" value="${db.driverClassName}"/>
            <property name="url" value="${db.url}"/>
            <property name="username" value="${db.username}"/>
            <property name="password" value="${db.password}"/>
            <!-- 连接池启动时的初始值 -->
            <property name="initialSize" value="${db.initialSize}"/>
            <!-- 连接池的最大值 -->
            <property name="maxActive" value="${db.maxActive}"/>
            <!-- 最大空闲值.当经过一个高峰时间后，连接池可以慢慢将已经用不到的连接慢慢释放一部分，一直减少到maxIdle为止 -->
            <property name="maxIdle" value="${db.maxIdle}"/>
            <!-- 最小空闲值.当空闲的连接数少于阀值时，连接池就会预申请去一些连接，以免洪峰来时来不及申请 -->
            <property name="minIdle" value="${db.minIdle}"/>
            <!-- 最大建立连接等待时间。如果超过此时间将接到异常。设为－1表示无限制 -->
            <property name="maxWait" value="${db.maxWait}"/>
            <!--#给出一条简单的sql语句进行验证 -->
             <!--<property name="validationQuery" value="select getdate()" />-->
            <property name="defaultAutoCommit" value="${db.defaultAutoCommit}"/>
            <!-- 回收被遗弃的（一般是忘了释放的）数据库连接到连接池中 -->
             <!--<property name="removeAbandoned" value="true" />-->
            <!-- 数据库连接过多长时间不用将被视为被遗弃而收回连接池中 -->
             <!--<property name="removeAbandonedTimeout" value="120" />-->
            <!-- #连接的超时时间，默认为半小时。 -->
            <property name="minEvictableIdleTimeMillis" value="${db.minEvictableIdleTimeMillis}"/>
    
            <!--# 失效检查线程运行时间间隔，要小于MySQL默认-->
            <property name="timeBetweenEvictionRunsMillis" value="40000"/>
            <!--# 检查连接是否有效-->
            <property name="testWhileIdle" value="true"/>
            <!--# 检查连接有效性的SQL语句-->
            <property name="validationQuery" value="SELECT 1 FROM dual"/>
        </bean>
    ```

    ​	

  * MyBatis配置

    ```xml
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
            <!--将上面id为dataSource的bean注入到SqlSessionFactoryBean-->
            <property name="dataSource" ref="dataSource"/>
            <!--指定Mybatis SQL实现的文件位置-->
            <property name="mapperLocations" value="classpath*:mappers/*Mapper.xml"></property>
    
            <!-- 分页插件 -->
            <property name="plugins">
                <array>
                    <bean class="com.github.pagehelper.PageHelper">
                        <property name="properties">
                            <value>
                                <!--配置方言-->
                                dialect=mysql
                            </value>
                        </property>
                    </bean>
                </array>
            </property>
    
        </bean>
    
        <!--配置Mybatis dao层接口-->
        <bean name="mapperScannerConfigurer" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
            <property name="basePackage" value="com.fmall.dao"/>
        </bean>
    ```

  * 事务管理

    ```xml
      <!-- 使用@Transactional进行声明式事务管理需要声明下面这行 -->
        <tx:annotation-driven transaction-manager="transactionManager" proxy-target-class="true" />
        <!-- 事务管理 -->
        <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
            <property name="dataSource" ref="dataSource"/>
            <!--当提交失败时是否进行回滚-->
            <property name="rollbackOnCommitFailure" value="true"/>
        </bean>
    ```

#### 4.SpringMVC配置文件dispatcher-servlet.xml配置

* 添加注解扫描

  ```xml
  <!--包扫描-->
      <context:component-scan base-package="com.fmall" annotation-config="true"/>
  ```

* 编码和反序列化配置

  ``` xml
  <mvc:annotation-driven>
          <mvc:message-converters>
              <!--编码配置-->
              <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                  <property name="supportedMediaTypes">
                      <list>
                          <value>text/plain;charset=UTF-8</value>
                          <value>text/html;charset=UTF-8</value>
                      </list>
                  </property>
              </bean>
              <!--Spring mvc自动反序列化的配置-->
              <bean class="org.springframework.http.converter.json.MappingJacksonHttpMessageConverter">
                  <property name="supportedMediaTypes">
                      <list>
                          <value>application/json;charset=UTF-8</value>
                      </list>
                  </property>
              </bean>
          </mvc:message-converters>
      </mvc:annotation-driven>
  ```

* 上传文件配置

  ```xml
  <!-- 文件上传 -->
      <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
          <property name="maxUploadSize" value="10485760"/> <!-- 上传的最大字节数10m -->
          <property name="maxInMemorySize" value="4096" /> <!-- 上传文件所用内存块的最大字节数 -->
          <property name="defaultEncoding" value="UTF-8"></property> <!--编码 -->
      </bean>
  ```

### Logback初始化
#### 1.日志管理logback的初始化及配置
#### 2.dao层的日志输出

* Mybatis dao层显示

  ```xml
  <!-- geelynote mybatis log 日志 -->
  
      <logger name="com.fmall.dao" level="DEBUG"/>
  ```

### FTP服务器配置
#### 1.FTP服务器的配置详解

* fmmall.properties

  ```properties
  ftp.server.ip=192.168.179.131
  ftp.user=ftpuser
  ftp.pass=123456
  ftp.server.http.prefix= http://192.168.179.131/
  ```

### IDEA注入和实时编译的配置
#### 1.IDEA使用Mybatis及Spring scan时，autowired注入时报错处理

* 打开settings-->inspections-->Spring Core-->Autowriting for Bean Class-->将severity设置为Warning

#### 2.开启Problem窗口，实时编译的配置及作用

* 打开settings-->compiler-->勾选Build Project automatically

### 两个小工具
#### 1.Restlet client
> 和postman同类软件，模拟http请求

#### 2.FE助手
> 主要用于里面的JSON格式化功能
