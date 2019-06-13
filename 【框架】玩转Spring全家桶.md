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

### 7.那些好用的连接池们：HikariCP

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
### 8.那些好用的连接池们：Alibaba Druid

![23](E:\markdown笔记\笔记图片\13\23.png)

![24](E:\markdown笔记\笔记图片\13\24.png)

![25](E:\markdown笔记\笔记图片\13\25.png)

使用druid的两种方法：

![26](E:\markdown笔记\笔记图片\13\26.png)

![27](E:\markdown笔记\笔记图片\13\27.png)

![28](E:\markdown笔记\笔记图片\13\28.png)

![29](E:\markdown笔记\笔记图片\13\29.png)

* 在Spring Boot中使用Druid时应该注意的点

  在pom文件之中，我们应在spring-boot-starter-jdbc中去除HikariCP，引入druid-spring-boot-starter。

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-jdbc</artifactId>
          <exclusions>
              <exclusion>
                  <artifactId>HikariCP</artifactId>
                  <groupId>com.zaxxer</groupId>
          	</exclusion>
      	</exclusions>
  </dependency>
  <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid-spring-boot-starter</artifactId>
      <version>1.1.10</version>
  </dependency>
  ```

* 详情访问<https://github.com/depers/geektime-spring-code/tree/master/8/druid-demo>，在这个例子中我们在每次连接数据库的前后都会做相应的操作。

![30](E:\markdown笔记\笔记图片\13\30.png)

### 9.如何通过Spring JDBC访问数据库

![31](E:\markdown笔记\笔记图片\13\31.png)

![32](E:\markdown笔记\笔记图片\13\32.png)

![33](E:\markdown笔记\笔记图片\13\33.png)

![34](E:\markdown笔记\笔记图片\13\34.png)

![35](E:\markdown笔记\笔记图片\13\35.png)

该部分的代码详情请参考：<https://github.com/depers/geektime-spring-code/tree/master/9/simple-jdbc-demo>

### 10.什么是Spring的事务抽象（上）

![36](E:\markdown笔记\笔记图片\13\36.png)

![37](E:\markdown笔记\笔记图片\13\37.png)

![38](E:\markdown笔记\笔记图片\13\38.png)

![39](E:\markdown笔记\笔记图片\13\39.png)

![40](E:\markdown笔记\笔记图片\13\40.png)
### 11.什么是Spring的事务抽象（下）
![41](E:\markdown笔记\笔记图片\13\41.png)

编程式事务的两种办法，本视频中着重演示了第一个方法Transaction Tempate，具体参见：<https://github.com/depers/geektime-spring-code/tree/master/11/programmatic-transaction-demo>

![42](E:\markdown笔记\笔记图片\13\42.png)

![43](E:\markdown笔记\笔记图片\13\43.png)

声明式事务中主要演示了三个方法：

![44](E:\markdown笔记\笔记图片\13\44.png)

可以看到第一个和第二个方法都加入@Transactional注解，都是支持事务的。而第三个方法他调用了第二个带有事务的方法。但是通过调用这三个方法发现第三个方法他是不支持事务的。这是因为Spring其实为你的类做了一个代理，你必须去调用代理类才能执行那些被代理曾强的方法。如果方法是在类的内部调用的话，并没有走到那些代理方法，虽然invikeInsertThenRollback调用了带有@Transactional注解的方法，但是因为它本身是没有事务注解的，所以他调用insertThenRollback也不会有事务的支持。

### 12.了解Spring的JDBC异常抽象

![45](E:\markdown笔记\笔记图片\13\45.png)

其中错误码的定义有两个地方：一个是Spring的ErrorCode，其次我们还可以自己定义ErrorCode在Classpath下的sql-error-codes.xml中，我们通过定制可以覆盖官方的一些配置。

![46](E:\markdown笔记\笔记图片\13\46.png)

![47](E:\markdown笔记\笔记图片\13\47.png)

上述代码的演示项目请参见：<https://github.com/depers/geektime-spring-code/tree/master/12/errorcode-demo>

### 13.课程答疑（上）

![48](E:\markdown笔记\笔记图片\13\48.png)

* 开发环境

  ![49](E:\markdown笔记\笔记图片\13\49.png)

* 一些注解

  ![49](E:\markdown笔记\笔记图片\13\50.png)

  * @Configuration：标明当前的这个类是一个配置类。

  * @ImportResource：Spring Boot里面没有Spring的配置文件，我们自己编写的配置文件，也不能自动识别；想让Spring的配置文件生效，加载进来；@ImportResource标注在一个配置类上。

  * @ComponentScan 组件扫描，可自动发现和装配一些Bean。

  * @Bean： 注解在方法上，声明当前方法返回一个Bean。

  * @ConfigurationProperties：通过 @ConfigurationProperties 来获取配置文件的信息。

  ![51](E:\markdown笔记\笔记图片\13\51.png)
  
  * @Component：注解主要用于标记一个类作为spring的组件，spring扫描的时候扫描的是这个注解。所有Java的Bean都可以通过@Component来定义。
  * @Repository：标识该类是一个数据库访问层的Repository
  * @Service：服务层的一个Bean
  * @Controller：Web层的Bean
  * @RestController：Web层的RESTful Bean
  * @RequestMapping：url映射
  * @ Autowired：在上下文当中按照类型查Bean找并注入进来
  * @Qualifier：如果我在上下中有多个同类型的Bean只使用@Autowired可能会有些歧义，我们可以使用@注解配合类的名字进行注入
  * @Resource：根据名字进行注入
  * @Value：通过 @value 将 配置文件中的信息对应的赋值给需要的属性
  
* 关于Actuator Endpoint访问不到的说明

  ![52](E:\markdown笔记\笔记图片\13\52.png)

  ![53](E:\markdown笔记\笔记图片\13\53.png)
  
* 读数据源、分库分表、读写分离的关系

  ![54](E:\markdown笔记\笔记图片\13\54.png)

  * 系统需要访问几个完全不同的数据库，就像上图中的左图
  * 系统访问同一个库的主库和备库：主库用来做读写操作，备库用来做读操作。

  ![55](E:\markdown笔记\笔记图片\13\55.png)

  * 系统需要访问一组做了分库分表的数据库可以使用数据库中间件进行操作。

### 14.课程答疑（下）

* 内部方法调用与事务的课后问题

  ![56](E:\markdown笔记\笔记图片\13\56.png)

  这里回答了第11节的课后问题：即该节最后一幅图中第三个方法为什么没有做事务处理？根据上图问题的解法，我们修改源代码，这样第三个方法就得到了事务的支持：

  ![57](E:\markdown笔记\笔记图片\13\57.png)

* REQUIRES_NEW与NESTED事务传播特性的说明

  ![57](E:\markdown笔记\笔记图片\13\58.png)

  这一部分没有听懂，而且代码运行也与视频中不一样。这个项目的代码是：<https://github.com/depers/geektime-spring-code/tree/master/14/transaction-propagation-demo>

* Alibaba Druid的一些展开说明

  ![57](E:\markdown笔记\笔记图片\13\59.png)

  代码详情参考<https://github.com/depers/geektime-spring-code/tree/master/14/druid-demo>，设置以上慢SQL的设置后，我们启动项目可以在控制台看到如下的显示：

  ![61](E:\markdown笔记\笔记图片\13\61.png)

  ![60](E:\markdown笔记\笔记图片\13\60.png)
## 第三章：O R Mapping实践

 ### 15.认识Spring Data JPA

![62](E:\markdown笔记\笔记图片\13\62.png)

![63](E:\markdown笔记\笔记图片\13\63.png)

![64](E:\markdown笔记\笔记图片\13\64.png)

![65](E:\markdown笔记\笔记图片\13\65.png)

![66](E:\markdown笔记\笔记图片\13\66.png)

### 16.定义JPA的实体对象

![67](E:\markdown笔记\笔记图片\13\67.png)

@Entity：

@MappedSuperclass：

@Table：

@Id：

@GeneratedVaule：

@SequenceGenerator：

![68](E:\markdown笔记\笔记图片\13\68.png)

![69](E:\markdown笔记\笔记图片\13\69.png)

@Column：

@JoinTable：

@OneToOne：

@OneToMany：

@ManyToOne：

@MangToMany：

@OrderBy：

![70](E:\markdown笔记\笔记图片\13\70.png)

@Getter：

@Setter：

@ToString：

@NoArgsConstructor：

@RequireArgsConstructor：

@AllArgsConstructor：

@Data：

@Builder：

@Slf4j：

@CommonSLog：

@Log4j2：

### 17.开始我们的线上咖啡馆实战项目：SpringBucks

![71](E:\markdown笔记\笔记图片\13\71.png)

![72](E:\markdown笔记\笔记图片\13\72.png)

![73](E:\markdown笔记\笔记图片\13\73.png)

![74](E:\markdown笔记\笔记图片\13\74.png)

![75](E:\markdown笔记\笔记图片\13\75.png)

![76](E:\markdown笔记\笔记图片\13\76.png)

以上实体的定义请参照：<https://github.com/depers/geektime-spring-code/tree/master/17/jpa-demo>

![77](E:\markdown笔记\笔记图片\13\77.png)

![78](E:\markdown笔记\笔记图片\13\78.png)

![79](E:\markdown笔记\笔记图片\13\79.png)

以上实体的定义请参照：<https://github.com/depers/geektime-spring-code/tree/master/17/jpa-complex-demo>