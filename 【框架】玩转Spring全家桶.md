# 玩转Spring全家桶——极客时间

## 第一章：初识Spring (4讲)

### 1.Spring课程介绍

课程主要分为4部分：

* 初识Spring
  * Spring家族的主要成员
  * 跟着Spring了解技术趋势
  * 编写第一个Spring应用程序
* 数据库操作
  * JDBC必知必会
  * O/R Mapping实践
  * NoSQL实践
  * 数据库访问进阶
* Web开发
  * Spring MVC
  * Web开发进阶
  * 访问Web资源
* Spring Boot
  * 自动化配置原理及实现
  * 起步依赖原理及定制
  * 配置文件加载机制
  * 获取运行状态
  * 配置运行容器
  * 可执行Jar背后的秘密
* Spring Cloud
  * 云原生和微服务
  * 服务注册、发现、熔断与配置
  * Spring Cloud Stream
  * 服务链路追踪

### 2.一起认识Spring家族的主要成员

![1](E:\markdown笔记\笔记图片\13\1.png)

![2](E:\markdown笔记\笔记图片\13\2.png)

![3](E:\markdown笔记\笔记图片\13\3.png)

![4](E:\markdown笔记\笔记图片\13\4.png)

### 3.跟着Spring了解技术趋势

![5](E:\markdown笔记\笔记图片\13\5.png)

![6](E:\markdown笔记\笔记图片\13\6.png)

### 4.编写你的第一个Spring程序

![7](E:\markdown笔记\笔记图片\13\7.png)

* Spring Initializr

  打开<https://start.spring.io/>，然后在Search dependencies to add添加spring-boot-starter-actuator，spring-boot-starter-web，接着下载程序包。

* 启动程序，输入下面的命令：

  ```
  E:\demo\Java\idea\hello-spring>curl http://localhost:8080/hello
  hello Spring
  E:\demo\Java\idea\hello-spring>curl http://localhost:8080/actuator/health
  {"status":"UP"}
  ```

* 问题：如果我们在Maven中不使用spring-boot-starter-parent作为parent，而使用自己的parent该怎么配置？

  * 删除pom文件原来通过Spring Initializr中生成的parent标签

    ```xml
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.5.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    ```

  * 生明一个dependencyManagement：

    ```xml
    <dependencyManagement>
    		<dependencies>
    			<dependency>
    				<groupId>org.springframework.boot</groupId>
    				<artifactId>spring-boot-dependencies</artifactId>
    				<version>2.1.5.RELEASE</version>
    				<type>pom</type>
    				<scope>import</scope>
    			</dependency>
    		</dependencies>
    	</dependencyManagement>
    ```

  * 修改build，添加executions：

    ```xml 
    <build>
    		<plugins>
    			<plugin>
    				<groupId>org.springframework.boot</groupId>
    				<artifactId>spring-boot-maven-plugin</artifactId>
    				<executions>
    					<execution>
    						<goals>
    							<goal>repackage</goal>
    						</goals>
    					</execution>
    				</executions>
    			</plugin>
    		</plugins>
    	</build>
    ```

## 第二部分：Spring中的数据操作

## 第二章 JDBC必知必会

### 5.如何配置单数据源

* Spring Boot的配置演示，项目地址：<https://github.com/depers/geektime-spring-code/tree/master/5-1/datasource-demo-1>

  ![9](E:\markdown笔记\笔记图片\13\9.png)

  ![8](E:\markdown笔记\笔记图片\13\8.png)

  在Spring Initializr中初始化构建后，直接下载。在DatasourceDemoApplication中编写如下代码：

  ```java
  @SpringBootApplication
  @Slf4j
  public class DatasourceDemoApplication implements CommandLineRunner {
  
  	@Autowired
  	private DataSource dataSource;
  
  	public static void main(String[] args) {
  		SpringApplication.run(DatasourceDemoApplication.class, args);
  	}
  
  
  	@Override
  	public void run(String... args) throws Exception {
  		showConnection();
  	}
  
  	private void showConnection() throws SQLException{
          // 打印数据库源信息
  		log.info(dataSource.toString());
  		Connection conn = dataSource.getConnection();
          // 打印数据库连接信息
  		log.info(conn.toString());
  		conn.close();
  	}
  
  }
  ```

  运行结果如下图：

  ![14](E:\markdown笔记\笔记图片\13\14.png)

* 手动直接配置所需的Bean。不使用Spring Boot的自动配置数据源，自己手动配置。项目地址：<https://github.com/depers/geektime-spring-code/tree/master/5-1/pure-spring-datasource-demo>

  ![10](E:\markdown笔记\笔记图片\13\10.png)

  ![11](E:\markdown笔记\笔记图片\13\15.png)

  

如上图所示，这个项目不使用Spring Boot，引入了运行时的h2驱动、dbcp2的数据库连接池、Spring jdbc。然后编写了如下程序：

![11](E:\markdown笔记\笔记图片\13\11.png)

除了通过代码配置，还可以在applicationContext.xml中进行配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="geektime.spring.data" />
   
    <bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource"
          destroy-method="close">
        <property name="driverClassName" value="org.h2.Driver"/>
        <property name="url" value="jdbc:h2:mem:testdb"/>
        <property name="username" value="SA"/>
        <property name="password" value=""/>
    </bean>
    
</beans>
```

![12](E:\markdown笔记\笔记图片\13\12.png)

* 数据源相关的配置，项目地址：<https://github.com/depers/geektime-spring-code/tree/master/5-1/datasource-demo-2>

  ![13](E:\markdown笔记\笔记图片\13\13.png)

  在这个项目中，我们在application.properties中配置了数据源的相关信息，新建了数据库并对数据库进行了相关的操作。

### 6.如何配置多数据源

![16](E:\markdown笔记\笔记图片\13\16.png)

![17](E:\markdown笔记\笔记图片\13\17.png)

从上图中可以看到多数据源的配置有两种办法：1)手工配置两组DataSource及相关内容（不使用Spring Boot），2)与Spring Boot协同工作。

而在使用Spring Boot配置多数据源中又有两种办法：1)配置@Primary类型的Bean，2)排除Spring Boot的自动配置，在接下来内容中介绍了第2中排除Spring Boot的自动配置的方案。项目地址：<https://github.com/depers/geektime-spring-code/tree/master/6/multi-datasource-demo>

![18](E:\markdown笔记\笔记图片\13\18.png)

### 7.那些好用的连接池们

![19](E:\markdown笔记\笔记图片\13\19.png)

![20](E:\markdown笔记\笔记图片\13\20.png)

![21](E:\markdown笔记\笔记图片\13\21.png)

![22](E:\markdown笔记\笔记图片\13\22.png)

HikariCP的官网：<https://github.com/brettwooldridge/HikariCP>

其中Spring Boot2中默认使用HikariCP，我们可以在DataSourceConfiguration.java中看到相关配置：

```java
/**
	 * Hikari DataSource configuration.
	 */
	@ConditionalOnClass(HikariDataSource.class)
	@ConditionalOnMissingBean(DataSource.class)
	@ConditionalOnProperty(name = "spring.datasource.type", havingValue = "com.zaxxer.hikari.HikariDataSource", matchIfMissing = true)
	static class Hikari {

		@Bean
		@ConfigurationProperties(prefix = "spring.datasource.hikari")
		public HikariDataSource dataSource(DataSourceProperties properties) {
			HikariDataSource dataSource = createDataSource(properties,
					HikariDataSource.class);
			if (StringUtils.hasText(properties.getName())) {
				dataSource.setPoolName(properties.getName());
			}
			return dataSource;
		}

	}
```

