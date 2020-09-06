# 玩转Spring全家桶-上卷——极客时间

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

从上图中可以看到多数据源的配置有两种办法：

1)手工配置两组DataSource及相关内容（不使用Spring Boot），2)与Spring Boot协同工作。

而在使用Spring Boot配置多数据源中又有两种办法：1)配置@Primary类型的Bean，2)排除Spring Boot的自动配置，在接下来内容中介绍了第2种排除Spring Boot的自动配置的方案。项目地址：<https://github.com/depers/geektime-spring-code/tree/master/6/multi-datasource-demo>

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

@Entity：标识实体类是JPA实体，告诉JPA在程序运行时生成实体类对应表

@MappedSuperclass：通过这个注解，我们可以将该实体类当成基类实体，它不会映射到数据库表，但继承它的子类实体在映射时会自动扫描该基类实体的映射属性，添加到子类实体的对应数据库表中。

@Table：设置实体类在数据库所对应的表名。如果没有特别的标注Table注解的话，类名就是表名。

@Id：标识类里所在变量为主键

​	@GeneratedVaule：设置主键生成策略，此方式依赖于具体的数据库

​	@SequenceGenerator：用来定义一个生成主键的序列，与@GeneratedValue联合使用才有效。具体使用可以参考<https://blog.csdn.net/lvxinzhi/article/details/43484923>

![68](E:\markdown笔记\笔记图片\13\68.png)

![69](E:\markdown笔记\笔记图片\13\69.png)

@Column：表示属性所对应字段名进行个性化设置

@JoinTable：

@JoinColumn：

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

* joda-money：用于表示货币金额的Java库
* userType：解决joda-money类的映射

![75](E:\markdown笔记\笔记图片\13\75.png)

![76](E:\markdown笔记\笔记图片\13\76.png)

我们在生成表结构的时候jpa会为我们创建三张表和一个序列，如果我们使用@GeneratedValue(strategy = GenerationType.IDENTITY)标明主键生成的策略：数据库会有自增主键。这样后，jpa就不会为我们生成自增主键了。

以上实体的定义请参照：<https://github.com/depers/geektime-spring-code/tree/master/17/jpa-demo>

![77](E:\markdown笔记\笔记图片\13\77.png)

上面介绍了@MappedSuperclass的使用方法，将子类都有的属性转移到父类中。

![78](E:\markdown笔记\笔记图片\13\78.png)

这里注意的是@ToString(callSuper = true)，他会将父类中的属性也打印出来。

![79](E:\markdown笔记\笔记图片\13\79.png)

以上实体的定义请参照：<https://github.com/depers/geektime-spring-code/tree/master/17/jpa-complex-demo>

在上面的代码中，我们可以看到枚举的使用。我们定义了OrderState枚举，然后在CoffeeOrder中使用了他，在数据库生成的时候，在表中jpa为我们生成了一个integer类型。如下图所示：

![81](E:\markdown笔记\笔记图片\13\81.png)

### 18.Spring Data Jpa操作数据库

![82](E:\markdown笔记\笔记图片\13\82.png)

这里我们需要注意的是：

* @EnableJpaRepositories,我们只需在geektime.spring.springbucks.jpademo.JpaDemoApplication类上添加该注解，Spring会自动发现我们实现Repository接口的类。
* @CrudRepository接口：基本的增删改查。
* @PagingAndSortingRepository接口：分页和排序。
* @JpaRespository接口：JpaRepository接口同时拥有了基本CRUD功能以及分页功能。
* 其中泛型T表示操作的Bean，ID则为主键id的类型。

![83](E:\markdown笔记\笔记图片\13\83.png)

![84](E:\markdown笔记\笔记图片\13\84.png)

![85](E:\markdown笔记\笔记图片\13\85.png)

![86](E:\markdown笔记\笔记图片\13\86.png)

上面两张图片的代码在：<https://github.com/depers/geektime-spring-code/tree/master/17/jpa-demo>

![87](E:\markdown笔记\笔记图片\13\87.png)

在这张图片中定义了一个**通用的Repository**。其中@NoRepositoryBean表明Spring不需要为我创建一个Repository的Bean。

![88](E:\markdown笔记\笔记图片\13\88.png)

上图中开启事务用的是@Transactional注解。可以在方法执行的时候开启一个事务。

### 19.Spring Data JPA的Repository是怎么从接口变成Bean的

![89](E:\markdown笔记\笔记图片\13\89.png)

其中：

* JpaRepositoriesRegister：org.springframework.data.jpa.repository.config.JpaRepositoriesRegistrar
* JpaRepositoryConfigExtension：org.springframework.data.jpa.repository.config.JpaRepositoryConfigExtension
* RepositoryBeanDefinitionRegistrarSupport：org.springframework.data.repository.config.RepositoryBeanDefinitionRegistrarSupport#registerBeanDefinitions
* RepositoryConfigurationExtensionSupport：org.springframework.data.repository.config.RepositoryConfigurationExtensionSupport#getRepositoryConfiguration
* JpaRepositoryFactory：org.springframework.data.jpa.repository.support.JpaRepositoryFactory#getTargetRepository

![90](E:\markdown笔记\笔记图片\13\90.png)

其中：

* org.springframework.data.repository.core.support.RepositoryFactorySupport#getRepository
* org.springframework.data.projection.DefaultMethodInvokingMethodInterceptor
* org.springframework.data.repository.core.support.RepositoryFactorySupport.QueryExecutorMethodInterceptor
* org.springframework.data.repository.core.support.RepositoryFactorySupport.QueryExecutorMethodInterceptor#doInvoke
* org.springframework.data.repository.query.parser.Part

### 20.通过MyBatis操作数据库

![91](E:\markdown笔记\笔记图片\13\91.png)

![92](E:\markdown笔记\笔记图片\13\92.png)

其中：

* mybatis.mapper-locations = classpath*:mapper/**/\*.xml：指定mapper文件的位置。
* mybatis.type-aliases-package：指定Bean的报名，这样在xml的namespace中就可以直接写类名了。
* mabatis.type-handlers-package：指定类型转换类的前缀。
* myabtis.configuration.map-underscore-to-camel-case = true：将数据库中的下划线的字段名转换为驼峰规则。

![93](E:\markdown笔记\笔记图片\13\93.png)

MyBatis也支持注解，在定义映射时，可以使用xml或是注解，或是两者混用。

![94](E:\markdown笔记\笔记图片\13\94.png)

上面介绍了mybatis基于注解的使用方法，该实例项目的源代码在<https://github.com/depers/geektime-spring-code/tree/master/20/mybatis-demo>

* 使用springboot jdbc初始化数据库

  在该项目的resource目录下面有一个schema.sql文件。 spring.datasource下有两个属性  schema、data，其中schema为表初始化语句，data为数据初始化，默认加载schema.sql与data.sql。脚本位置可以通过spring.datasource.schema  与spring.datasource.data 来改变。

  spring.datasource.initialization-mode  初始化模式（springboot2.0），其中有三个值，always为始终执行初始化，embedded只初始化内存数据库（默认值）,如h2等，never为不执行初始化。

  在pom文件中引入的h2数据库：

  ```
  <dependency>
     <groupId>com.h2database</groupId>
     <artifactId>h2</artifactId>
     <scope>runtime</scope>
  </dependency>
  ```

### 21.让MyBatis更好用的那些工具：MyBatis Generator

![96](E:\markdown笔记\笔记图片\13\96.png)

![97](E:\markdown笔记\笔记图片\13\97.png)

其中：

* 值得注意的是MyBatis Generator配置的节点是**有顺序的**。
* jdbcConnection：数据库连接配置
* javaModelGenerator：model生成配置
* sqlMapGenerator：mapper生成配置
* javaClientGenerator：mapper生成策略。ANNOTATEDMAPPER：采用注解，XMLMAPPER：使用xml文件；MIXEDMAPPER：注解和xml混合
* table：处理表与对象之间的关系。

![98](E:\markdown笔记\笔记图片\13\98.png)

其中：

* FluentBuilderMethodsPlugin：生成带有fluent风格的model代码，with...
* ToStringPlugin：为生成的Java模型创建一个toString方法
* SerializablePlugin：为生成的Java模型类添加序列化接口，并生成serialVersionUID字段
* RowBoundsPlugin：这个插件可以生成一个新的selectByExample方法，这个方法可以接受一个RowBounds参数，主要用来实现分页。

![99](E:\markdown笔记\笔记图片\13\99.png)

![100](E:\markdown笔记\笔记图片\13\100.png)

注意：如果我们第二次生成代码时，为了避免覆盖自己修改的代码。自己应实现一套与生成代码不同的代码，我们因采取两套model，两套mapper和两套xml。

具体代码实例地址：<https://github.com/depers/geektime-spring-code/tree/master/21/mybatis-generator-demo>

### 22.让MyBatis更好用的那些工具：MyBatis PageHelper

![101](E:\markdown笔记\笔记图片\13\101.png)

* 引入POM

  ``` 
  <dependency>
     <groupId>org.mybatis.spring.boot</groupId>
     <artifactId>mybatis-spring-boot-starter</artifactId>
     <version>1.3.2</version>
  </dependency>
  <dependency>
     <groupId>com.github.pagehelper</groupId>
     <artifactId>pagehelper-spring-boot-starter</artifactId>
     <version>1.2.10</version>
  </dependency>
  ```

* application.properties的配置

  ```
  pagehelper.offset-as-page-num=true  #讲RowBounds作为页码来使用
  pagehelper.reasonable=true			#如果页码小于0...
  pagehelper.page-size-zero=true		#若请求的页数为0则将所有的数据返回
  pagehelper.support-methods-arguments=true	#方法参数的支持
  ```

具体代码的实例地址：<https://github.com/depers/geektime-spring-code/tree/master/22/mybatis-pagehelper-demo>

### 23.SpringBucks实战项目进度小结

![102](E:\markdown笔记\笔记图片\13\102.png)

![103](E:\markdown笔记\笔记图片\13\103.png)

值得注意的是：

* 在设置了spring.jpa.hibernate.ddl-auto=none，不让hibernate帮我们做ddl的变更。
* 接口继承：JpaRepository继承了PagingAndSortingRepository继承了CrudRepository。
* QueryByExampleExecutor、ExampleMatcher和Example的用法：geektime.spring.springbucks.service.CoffeeService#findOneCoffee

## 第四章：NoSQL实践

### 24.通过 Docker 辅助开发

![104](E:\markdown笔记\笔记图片\13\104.png)

![105](E:\markdown笔记\笔记图片\13\105.png)

![106](E:\markdown笔记\笔记图片\13\106.png)

![107](E:\markdown笔记\笔记图片\13\107.png)

![108](E:\markdown笔记\笔记图片\13\108.png)

![109](E:\markdown笔记\笔记图片\13\109.png)

![110](E:\markdown笔记\笔记图片\13\110.png)

运行MongoDB:（该命令需要在linux控制台输入）

```
docker run --name mongo -p 27017:27017 -v  ~/docker-data/mongo:/data/db -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=admin -d mongo

dcoker ps -a

docker ps

docker start mongo

dcoker exec -it mongo mongo -u admin -p admin
```

![111](E:\markdown笔记\笔记图片\13\111.png)

### 25.在 Spring 中访问 MongoDB

![112](E:\markdown笔记\笔记图片\13\112.png)

![113](E:\markdown笔记\笔记图片\13\113.png)

![114](E:\markdown笔记\笔记图片\13\114.png)

创建用户：

```
# 切换到springbucks这个库上
use springbucks
# 创建用户
db.createUser(
	{
		user: "springbucks",
		pwd: "springbucks",
		roles: [
			{
				role: "readWrite",
				db: "springbucks"
			}
		]
	}
)
```

* 操作mongo的相关命令：

  ```
  show dbs；
  
  show users；
  
  show collections;
  
  db.coffee.find();
  
  db.coffee.remove({"name":"espresso"})
  
  db.coffee.count()
  ```

* 该课程项目的代码请访问：

  MongoTemplate项目地址：[https://github.com/depers/geektime-spring-code/tree/master/Chapter%204/mongo-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 4/mongo-demo)

  Repository项目地址：[https://github.com/depers/geektime-spring-code/tree/master/Chapter%204/mongo-repository-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 4/mongo-repository-demo)

### 26.在 Spring 中访问 Redis

![116](E:\markdown笔记\笔记图片\13\116.png)

![117](E:\markdown笔记\笔记图片\13\117.png)

![118](E:\markdown笔记\笔记图片\13\118.png)

![119](E:\markdown笔记\笔记图片\13\119.png)

```
docker pull redis

docker run --name redis -d -p 6379:6379 redis

// 重启服务器后启动redis镜像
docker start redis

docker exec -it redis redis-cli
```

### 27.Redis 的哨兵与集群模式

![120](E:\markdown笔记\笔记图片\13\120.png)

代码分析：redis.clients.jedis.JedisSentinelPool

![121](E:\markdown笔记\笔记图片\13\121.png)

源码分析：redis.clients.jedis.JedisCluster

### 28.了解 Spring 的缓存抽象

![122](E:\markdown笔记\笔记图片\13\122.png)

* 缓存的使用场景

  * 那些缓存应该放在JVM内部：

    长久不变的信息，或是信息我们可以接受他有一定的延时。比如一天里面不会变的信息我们可以放到JVM内部。因为访问分布式缓存会有网络上面的开销。

  * 那些缓存应该使用像Redis这样的分布式缓存：

    在整个集群里面都有一个缓存且访问的时候具有一定的一致性的。

  * 那些其实根本不需要用缓存：

    数据的读写比非常的不好。比如写一次读一次的这种数据。像写一次读十次的这种数据我们就可以考虑使用缓存；如果数据的读写比非常大的话应该要使用缓存来减轻后端的压力。

Spring缓存抽象的相关注解：

![123](E:\markdown笔记\笔记图片\13\123.png)

代码中演示了使用Spring的缓存抽象在JVM中做缓存，相关代码：[https://github.com/depers/geektime-spring-code/tree/master/Chapter%204/cache-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 4/cache-demo)

![124](E:\markdown笔记\笔记图片\13\124.png)

代码中演示了使用Spring的缓存抽象在Redis中做缓存，相关代码：[https://github.com/depers/geektime-spring-code/tree/master/Chapter%204/cache-with-redis-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 4/cache-with-redis-demo)

### 29.Redis 在 Spring 中的其他用法

* 配置连接工厂：在新的Spring-Data-Redis中，spring已经使用Lettuce代替了Jedis。

  ![125](E:\markdown笔记\笔记图片\13\125.png)

  * RedisStandaloneConfiguration：单节点
  * RedisSentisnelConfiguration：哨兵
  * RedisClusterConfiguration：集群
  * spring.redis.cluster.max-redirects=3，默认值为3还是恰到好处的。
    * org.springframework.boot.autoconfigure.data.redis.RedisProperties

* 读写分离

  JedisCluster不支持读写分离。

  ![126](E:\markdown笔记\笔记图片\13\126.png)

  

* RedisTemplate

  ![127](E:\markdown笔记\笔记图片\13\127.png)

  * StringRedisTemplate：如果K与V都是String的话可以使用这个工具类。

  * 在redis-demo项目中的Pom文件中我们引入了：

    ```XML
    <dependency>
    	<groupId>org.springframework.boot</groupId>
    	<artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    
    <dependency>
    	<groupId>org.apache.commons</groupId>
    	<artifactId>commons-pool2</artifactId>
    </dependency>
    ```

  * 项目代码：[https://github.com/depers/geektime-spring-code/tree/master/Chapter%204/redis-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 4/redis-demo)

* Redis Repository

  ![128](E:\markdown笔记\笔记图片\13\128.png)

  ![129](E:\markdown笔记\笔记图片\13\129.png)

  * 项目地址：[https://github.com/depers/geektime-spring-code/tree/master/Chapter%204/redis-repository-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 4/redis-repository-demo)

## 第五章：数据访问进阶

### 31.Project Reactor 介绍（上）

官网：<https://projectreactor.io/>

![132](E:\markdown笔记\笔记图片\13\132.png)

打个比方，在程序中我会会编写a=b+c的赋值，执行完a=b+c之后a的值就确定了下来，若b和c再发生变化，那么a的值就不会在发生变化。而响应式编程中，就像excel表格中的运算一样，如果b和c发生变化，那么a也会变化。

![133](E:\markdown笔记\笔记图片\13\133.png)

![134](E:\markdown笔记\笔记图片\13\134.png)

  上面的图片是事例演示。

### 32.Project Reactor 介绍（下）

![135](E:\markdown笔记\笔记图片\13\135.png)

![136](E:\markdown笔记\笔记图片\13\136.png)

项目代码中我做了详细注释，如果不理解再看下视频：[https://github.com/depers/geektime-spring-code/tree/master/Chapter%205/simple-reactor-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 5/simple-reactor-demo)

### 33.通过 Reactive 的方式访问 Redis

![137](E:\markdown笔记\笔记图片\13\137.png)

本节的代码地址：[https://github.com/depers/geektime-spring-code/tree/master/Chapter%205/redis-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 5/redis-demo)

### 34.通过 Reactive 的方式访问 MongoDB

![138](E:\markdown笔记\笔记图片\13\138.png)

本节代码的地址：[https://github.com/depers/geektime-spring-code/tree/master/Chapter%205/mongodb-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 5/mongodb-demo)

### 35.通过 Reactive 的方式访问 RDBMS

Spring Data R2DBC，用于针对关系型数据库进行反应式编程。

![139](E:\markdown笔记\笔记图片\13\139.png)

* 通过Spring Data R2DBC

  参考项目：[https://github.com/depers/geektime-spring-code/tree/master/Chapter%205/simple-r2dbc-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 5/simple-r2dbc-demo)

![140](E:\markdown笔记\笔记图片\13\140.png)

* R2DBC Repository支持

  参考项目：[https://github.com/depers/geektime-spring-code/tree/master/Chapter%205/r2dbc-repository-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 5/r2dbc-repository-demo)

![141](E:\markdown笔记\笔记图片\13\141.png)

### 36.通过 AOP 打印数据访问层的摘要（上）

![142](E:\markdown笔记\笔记图片\13\142.png)

![143](E:\markdown笔记\笔记图片\13\143.png)

* @EnableAspectJAutoProxy：表示开启*AOP*代理自动配置，如果配@EnableAspectJAutoProxy表示使用*cglib*进行代理对象的生成；设置*@EnableAspectJAutoProxy(exposeProxy=true)*表示通过*aop*框架暴露该代理对象，*aopContext*能够访问。
* @aspect：定义切面
* @pointcut：定义切点
* @before：标注Before Advice定义所在的方法
* @afterreturning：标注After Returning Advice定义所在的方法
* @afterthrowing：标注After Throwing Advice定义所在的方法
* @after 标注：After(Finally) Advice定义所在的方法
* @around：标注Around Advice定义所在的方法

### 37.通过 AOP 打印数据访问层的摘要（下）

Spring 相关文档：<https://docs.spring.io/spring/docs/5.2.0.RELEASE/spring-framework-reference/core.html#aop-ataspectj>

![144](E:\markdown笔记\笔记图片\13\144.png)

### 38.SpringBucks 实战项目进度小结

![145](E:\markdown笔记\笔记图片\13\145.png)

项目地址：[https://github.com/depers/geektime-spring-code/tree/master/Chapter%205/reactive-springbucks](https://github.com/depers/geektime-spring-code/tree/master/Chapter 5/reactive-springbucks)

## 第六章：Spring MVC实践

### 39.编写第一个 Spring MVC Controller

![146](E:\markdown笔记\笔记图片\13\146.png)

![147](E:\markdown笔记\笔记图片\13\147.png)

其中@RestController结合了@Controller和@ResponseBody两个注解的功能。

该项目的地址是：[https://github.com/depers/geektime-spring-code/tree/master/Chapter%206/simple-controller-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 6/simple-controller-demo)

在该项目中我们使用了spring-boot-starter-cache，其中涉及的注解有：

* @EnableCaching：开启缓存
* @Cacheable：@Cacheable可以标记在一个方法上，也可以标记在一个类上。当标记在一个方法上时表示该方法是支持缓存的，当标记在一个类上时则表示该类所有的方法都是支持缓存的。@Cacheable 的作用 主要针对方法配置，能够根据方法的请求参数对其结果进行缓存
* @CacheConfig(cacheNames = "CoffeeCache")：所有的@Cacheable（）里面都有一个value＝“xxx”的属性，这显然如果方法多了，写起来也是挺累的，如果可以一次性声明完 那就省事了， 
  所以，有了@CacheConfig这个配置，@CacheConfig is a class-level annotation that allows to share the cache names

具体注解的使用请参考文档：<https://docs.spring.io/spring/docs/5.2.0.RELEASE/spring-framework-reference/integration.html#cache-annotations>

### 40.理解 Spring 的应用上下文

![148](E:\markdown笔记\笔记图片\13\148.png)

![149](E:\markdown笔记\笔记图片\13\149.png)

![150](E:\markdown笔记\笔记图片\13\150.png)

![151](E:\markdown笔记\笔记图片\13\151.png)

上图中是Root webApplicationContext、Servlet webApplicationContext在web.xml和Java基于注解的配置方式。

* 在web.xml中Root webApplicationContext是通过ContextLoaderListener配置的；而Servlet webApplicationContext是在DispatcherServlet中加载的。
* 在Java中我们是通过RootConfig配置的Root webApplicationContext，App1Config配置Servlet webApplicationContext。

项目地址：[https://github.com/depers/geektime-spring-code/tree/master/Chapter%206/context-hierarchy-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 6/context-hierarchy-demo)

代码演示效果：

* 第一种情况（只有父得到了增强）：

  ```
  hello foo
  after hello()
  
  =============
  hello Bar
  
  hello foo
  after hello()
  ```

  其中fooContext得到了增强，barContext没有得到增强，通过barContext去访问parent Context中的bean的testBeanY也得到了增强。

* 第二种情况（只有子得到了增强）：

  * 先做修改

    * 修改geektime.spring.web.foo.FooConfig

      ```
      //    @Bean
      //    public FooAspect fooAspect() {
      //        return new FooAspect();
      //    }
      ```

    * 修改applicationContext.xml

      ```
      <aop:aspectj-autoproxy/>
      
      <bean id="fooAspect" class="geektime.spring.web.foo.FooAspect" />
      ```

  * 结果为：

    ```
    hello foo
    
    =============
    hello Bar
    after hello()
    
    hello foo
    ```

    其中fooContext没有得到了增强，barContext得到增强，通过barContext去访问parent Context中的bean的testBeanY也没有得到了增强。

* 第三种情况（父子都得到了增强）：

  * 先做修改，在第二步的基础上

    * 修改geektime.spring.web.foo.FooConfig

      ```
      @Bean
      public FooAspect fooAspect() {
      	return new FooAspect();
      }
      ```

    * 修改applicationContext.xml

      ```
      <!--    <bean id="fooAspect" class="geektime.spring.web.foo.FooAspect" />-->
      ```

### 41.理解请求的处理机制

![152](E:\markdown笔记\笔记图片\13\152.png)

![153](E:\markdown笔记\笔记图片\13\153.png)

项目代码：

在该节内容中我们进行了org.springframework.web.servlet.DispatcherServlet#doDispatch方法的代码bebug：

```
mappedHandler = getHandler(processedRequest);
```

### 42.如何定义处理方法（上）

![154](E:\markdown笔记\笔记图片\13\154.png)

![155](E:\markdown笔记\笔记图片\13\156.png)

<https://docs.spring.io/spring/docs/5.2.0.RELEASE/spring-framework-reference/web.html#mvc-ann-methods>

<https://docs.spring.io/spring/docs/5.2.0.RELEASE/spring-framework-reference/web.html#mvc-ann-return-types>

![157](E:\markdown笔记\笔记图片\13\157.png)

* consumes：可以根据请求的 `Content-Type` 缩小请求映射
* produces：可以根据请求的`Accept` 缩小请求映射

![158](E:\markdown笔记\笔记图片\13\158.png)

上面这段代码的效果是一样的，都是取请求路径的参数。

项目代码：[https://github.com/depers/geektime-spring-code/tree/master/Chapter%206/complex-controller-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 6/complex-controller-demo)

### 43.如何定义处理方法（下）

![159](E:\markdown笔记\笔记图片\13\159.png)

![160](E:\markdown笔记\笔记图片\13\160.png)

![161](E:\markdown笔记\笔记图片\13\161.png)

#### default关键字的使用

* 在`switch`语句的时候使用`default`

  ```java
  int day = 8;
  String dayString;
  switch (day) {
      case 1:	dayString = "Monday";
      break;
      case 2: dayString = "Tuesday";
      break;
      case 3: dayString = "Wednesday";
      break;
      case 4: dayString = "Thursday";
      break;
      case 5: dayString = "Friday";
      break;
      case 6: dayString = "Saturday";
      break;
      //如果case没有匹配的值，那么肯定是星期日
      default: dayString = "Sunday";
      break;
  }
  System.out.println(dayString);
  ```

  **总结**：使用比较简单，就是当case里的值与switch里的key没有匹配的时候，执行default里的方法。在这里的例子中就是key为8，所以key与所有的case的值都不匹配，所以输出星期天Sunday.

* 在定义接口的时候使用`default`来修饰具体的方法

  接口的定义IntefercaeDemo，定义一个接口，里面有两个具体的方法，和一个抽象方法

  **IntefercaeDemo .java**

  ```java
  public interface IntefercaeDemo {
      //具体方法
      default void showDefault(){
          System.out.println("this is showDefault method");
      }
      static void showStatic(){
          System.out.println("this is showStatic method");
      }
  
      //没有实现的抽象方法
      void sayHi();
  }
  ```
  LearnDefault 实现IntefercaeDemo 接口。

  **LearnDefault .java**

  ```java
  public class LearnDefault implements IntefercaeDemo{
  	//实现抽象方法
  	@Override
  	public void sayHi() {
  		System.out.println("this is sayHi mehtod");
  	}
  	
  	public static void main(String[] args) {
  		//接口中被static所修饰的具体方法
  		IntefercaeDemo.showStatic();
  		
  		//将实现了IntefercaeDemo的类实例化
  		LearnDefault learnDefault = new LearnDefault();
  		//被Default所修饰的具体方法可以通过引用变量来调用
  		learnDefault.showDefault();
  
  	}
  }
  ```

  * **说明：**

    JDK1.8中为了加强接口的能力，使得接口可以存在具体的方法，前提是方法需要被default或static关键字所修饰。

  * **总结：**

    default修饰的目的是让接口可以拥有具体的方法，让接口内部包含了一些默认的方法实现。
    被default修饰的方法是接口的默认方法。既只要实现该接口的类，都具有这么一个默认方法，默认方法也可以被重写。
    我们可以想象这么一个场景，既实现某个接口的类都具有某个同样的功能，如果像Java8以前的版本，那么每个实现类都需要写一段重复的代码去实现那个功能，显得没有必要。这就是存在的意义。
  
  * **注意：**
  
    另外这里既然提到了接口的修饰符default，那么就要注意一点，如果一个类实现了两个接口（可以看做是“多继承”），这两个接口又同时都包含了一个名字相同的default方法，那么会发生什么情况？ 在这样的情况下，编译器是会报错，得到一个编译器错误，因为编译器不知道应该在两个同名的default方法中选择哪一个，因此产生了二义性。

  * **补充：容易混淆的地方**
  
    这里的关键字default不要跟平时我们在类中定义方法时，没有加任何访问修饰符时的(default)相混淆，它们的意义是不一样的。
  
    ```
    public class Demo{
    	//没有访问修饰符修饰，所以默认为(default)
    	String name;
    	void show(){}
    }
    ```
  
    这里的(default)指的是一种场景，既类中成员没有被访问修饰符修饰，所以属于(default)的情况，效果是(default)情况的成员只能在Demo类所在的包内被访问。
    而本篇博文所说的default关键字是一个实实在在存在的关键字，是需要显式声明的，目前只有所说的两种用法。与(default)场景毫无关系。
  
* 代码分析：

  - org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration.WebMvcAutoConfigurationAdapter#addFormatters
  - org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration
    - org.springframework.boot.autoconfigure.web.servlet.MultipartProperties

* 项目代码

  * 地址：[https://github.com/depers/geektime-spring-code/tree/master/Chapter%206/more-complex-controller-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 6/more-complex-controller-demo)

  * 注意的点：

    * 自定义Formatter

      geektime.spring.springbucks.waiter.support.MoneyFormatter

    * 修改pom

      ```
      <dependency>
      	<groupId>org.apache.commons</groupId>
      	<artifactId>commons-lang3</artifactId>
      </dependency>
      ```

      因为在geektime.spring.springbucks.waiter.support.MoneyFormatter中使用了NumberUtils、StringUtils工具类。

    * @Valid的演示

      `public Coffee addCoffee(@Valid NewCoffeeRequest newCoffee, BindingResult result) `

    * 文件上传MultipartFile的演示

      `public List<Coffee> batchAddCoffee(@RequestParam("file") MultipartFile file)`

    * 处理@Valid的结果，将结果绑定到BindingResult进行处理

      ```
      public Coffee addCoffee(@Valid NewCoffeeRequest newCoffee,
                                  BindingResult result) {
              if (result.hasErrors()) {
                  // 这里先简单处理一下，后续讲到异常处理时会改
                  log.warn("Binding Errors: {}", result);
                  return null;
              }
              return coffeeService.saveCoffee(newCoffee.getName(),newCoffee.getPrice());
          }
      ```


### 44.Spring MVC 中的视图解析机制（上）-- TODO

![162](E:\markdown笔记\笔记图片\13\162.png)

![163](E:\markdown笔记\笔记图片\13\163.png)

### 45.Spring MVC 中的视图解析机制（下）-- TODO

![164](E:\markdown笔记\笔记图片\13\164.png)

![165](E:\markdown笔记\笔记图片\13\165.png)

### 46.Spring MVC 中的常用视图（上）

![166](E:\markdown笔记\笔记图片\13\166.png)

![168](E:\markdown笔记\笔记图片\13\168.png)

<https://docs.spring.io/spring/docs/5.2.0.RELEASE/spring-framework-reference/web.html#mvc-view>

项目代码：

* 地址：[https://github.com/depers/geektime-spring-code/tree/master/Chapter%206/json-view-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 6/json-view-demo)

* 修改的地方

  * pom文件

    ```xml
    <!-- 增加Jackson的Hibernate类型支持 -->
    <dependency>
    	<groupId>com.fasterxml.jackson.datatype</groupId>
    	<artifactId>jackson-datatype-hibernate5</artifactId>
    	<version>2.9.8</version>	
    </dependency>
    <!-- 增加Jackson XML支持 -->
    <dependency>
    	<groupId>com.fasterxml.jackson.dataformat</groupId>
    	<artifactId>jackson-dataformat-xml</artifactId>
        <version>2.9.0</version>
    </dependency>
    ```

  * 在geektime.spring.springbucks.waiter.model.BaseEntity

    ```
    // 增加了jackson-datatype-hibernate5就不需要这个Ignore了
    //@JsonIgnoreProperties(value = {"hibernateLazyInitializer"})
    ```

  * Json的序列化与反序列化器

    geektime.spring.springbucks.waiter.support.MoneySerializer

    geektime.spring.springbucks.waiter.support.MoneyDeserializer

    有了`@JsonComponent`这个注解Spring Boot就会把序列化和反序列化器注册到Jackson的类上去，对特殊类型的做序列化处理。

### 47.Spring MVC 中的常用视图（下）

![169](E:\markdown笔记\笔记图片\13\169.png)

![170](E:\markdown笔记\笔记图片\13\170.png)

![171](E:\markdown笔记\笔记图片\13\171.png)

项目代码：

* 地址：[https://github.com/depers/geektime-spring-code/tree/master/Chapter%206/thymeleaf-view-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 6/thymeleaf-view-demo)

* 注意点：

  * pom的修改

    ```xml
    <dependency>
    	<groupId>org.springframework.boot</groupId>
    	<artifactId>spring-boot-starter-thymeleaf</artifactId>
    </dependency>
    ```

  * @ModelAttribute

    这个注解应用的有返回参数的方法上面，在请求具体的URL时首先会执行添加了@ModelAttribute注解的方法，这个方法会将方法名为key，返回值为value的map添加到ModelMap中去。

    ```java
    @ModelAttribute
    public List<Coffee> coffeeList() {
    	return coffeeService.getAllCoffee();
    }
    ```

  * redirect的使用，前端302重定向

    ```
    return "redirect:/order/" + order.getId();
    ```

### 48.静态资源与缓存

![172](E:\markdown笔记\笔记图片\13\172.png)

![173](E:\markdown笔记\笔记图片\13\173.png)

![174](E:\markdown笔记\笔记图片\13\174.png)

![177](E:\markdown笔记\笔记图片\13\177.png)

项目代码：

* 地址：[https://github.com/depers/geektime-spring-code/tree/master/Chapter%206/cache-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 6/cache-demo)
* 注意点：
  * 源码解析
    * org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration.WebMvcAutoConfigurationAdapter#addResourceHandlers
    * org.springframework.http.CacheControl
  
  * 静态图片的缓存配置，在application.properties中
  
    * spring.resources.cache.cachecontrol.max-age=20s
    
      缓存的最长时间是20秒
    
    * 如果我们在请求的头部加上If-Modified-Since：
    
      **If-Modified-Since** 是一个条件式请求首部，服务器只在所请求的资源在给定的日期时间之后对内容进行过修改的情况下才会将资源返回，状态码为 [`200`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status/200)  。如果请求的资源从那时起未经修改，那么返回一个不带有消息主体的  [`304`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Status/304)  响应
    
    * spring.resources.cache.cachecontrol.no-cache=true
    
      这样的话在response的头部信息中Cache-Control就会是no-cache
    
    * Controller中手动设置缓存
    
      geektime.spring.springbucks.waiter.controller.CoffeeController#getById

### 49.Spring MVC 中的异常处理机制

![175](E:\markdown笔记\笔记图片\13\175.png)

![176](E:\markdown笔记\笔记图片\13\176.png)

上面图片中值得注意的是：我们在@ControllerAdvice书写的标有@ExceptionHandler的方法的优先级是低于在@Controller中书写的标有@ExceptionHandler的方法的优先级。

项目代码：

* 地址：[https://github.com/depers/geektime-spring-code/tree/master/Chapter%206/exception-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 6/exception-demo)

* 源码分析

  * org.springframework.web.servlet.DispatcherServlet#processDispatchResult
  * org.springframework.web.servlet.DispatcherServlet#processHandlerException

* 怎么书写一个异常处理方法：

  * geektime.spring.springbucks.waiter.controller.GlobalControllerAdvice#validationExceptionHandler

  * 官方文档：

    <https://docs.spring.io/spring/docs/5.2.0.RELEASE/spring-framework-reference/web.html#mvc-ann-exceptionhandler>

### 50.了解 Spring MVC 的切入点

![178](E:\markdown笔记\笔记图片\13\178.png)

上图中是同步请求的拦截接口，这部分的源码可以参考org.springframework.web.servlet.DispatcherServlet#doDispatch中的：

* org.springframework.web.servlet.HandlerExecutionChain#applyPreHandle

  这里会调用interceptor的preHandle方法，该方法的作用是请求处理之前执行。这里可以做权限的验证，如果有权限的话就返回true，没有就终止操作返回false

* org.springframework.web.servlet.HandlerExecutionChain#applyPostHandle

  这里会调用Interceptor的postHandle方法，该方法的作用是请求处理之后执行。

* org.springframework.web.servlet.DispatcherServlet#triggerAfterCompletion

  这里会调用Interceptor的afterCompletion方法，该方法的作用是postHandle方法(报异常也会执行)之后会执行。

![179](E:\markdown笔记\笔记图片\13\179.png)

上图中针对AsyncHandlerInterceptor，在org.springframework.web.servlet.DispatcherServlet#doDispatch中会执行下列代码，他不会执行HandlerInterceptor的postHandle和afterCompletion方法。

``` java
if (asyncManager.isConcurrentHandlingStarted()) {
	return;
}
```

项目代码：

* 地址：[https://github.com/depers/geektime-spring-code/tree/master/Chapter%206/springbucks](https://github.com/depers/geektime-spring-code/tree/master/Chapter 6/springbucks)

* 注意点：

  * 添加Interceptor(geektime.spring.springbucks.waiter.WaiterServiceApplication)

    ``` java
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
    	registry.addInterceptor(new PerformanceInteceptor())
    		.addPathPatterns("/coffee/**").addPathPatterns("/order/**");
    }
    ```

  * json格式化输出和时区设置(geektime.spring.springbucks.waiter.WaiterServiceApplication)

    json输出的时间上就会显示`+0800`

    ```
    @Bean
    public Jackson2ObjectMapperBuilderCustomizer jacksonBuilderCustomizer() {
    	return builder -> {
    		builder.indentOutput(true);
    		builder.timeZone(TimeZone.getTimeZone("Asia/Shanghai"));
    	};
    }
    ```

  * Interceptor的编写方法：

    geektime.spring.springbucks.waiter.controller.PerformanceInteceptor

### 51.SpringBucks 实战项目进度小结（waiter-service）

![181](E:\markdown笔记\笔记图片\13\181.png)

![182](E:\markdown笔记\笔记图片\13\182.png)

项目springbucks集成了本章的所有知识点，生成了服务waiter-service.

### 52.课程答疑

![183](E:\markdown笔记\笔记图片\13\183.png)

* 使用MySQL数据库代替H2

  在之前的项目中一直使用的是H2数据库却没有在配置文件中做任何的配置，是因为Spring boot对内内嵌的数据库有自动的配置功能。

  项目地址：[https://github.com/depers/geektime-spring-code/tree/master/Chapter%206/jpa-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 6/jpa-demo)

  ![184](E:\markdown笔记\笔记图片\13\184.png)

  ![185](E:\markdown笔记\笔记图片\13\185.png)

* Java语言特性说明

  项目地址：[https://github.com/depers/geektime-spring-code/tree/master/Chapter%206/stream-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 6/stream-demo)

  ![186](E:\markdown笔记\笔记图片\13\186.png)

  ![187](E:\markdown笔记\笔记图片\13\187.png)

  map和flatMap的区别：

  ```
  Arrays.stream(new String[] { "s1", "s2", "s3" })
          // map是将str放到了list中
          .map(s -> Arrays.asList(s))
          // flat则是将list中的元素取出然后拍平
          .flatMap(l -> l.stream())
          .forEach(System.out::println);
  ```

* MyBatis Generator生成代码的一些说明

  <https://mp.baomidou.com/>

## 第七章：访问Web资源

### 53.通过 RestTemplate 访问 Web 资源

![189](E:\markdown笔记\笔记图片\13\189.png)

![190](E:\markdown笔记\笔记图片\13\190.png)

![191](E:\markdown笔记\笔记图片\13\191.png)

![192](E:\markdown笔记\笔记图片\13\192.png)

项目代码：

* 地址：[https://github.com/depers/geektime-spring-code/tree/master/Chapter%207/simple-resttemplate-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 7/simple-resttemplate-demo)

* 注意点：

  运行该项目首先要启动上一章中的customer-service项目。

  * 修改pom文件:

    ```XML
    <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    ```
  
  * 启动项目
  
    一种“平滑”的API调用方式来启动工程，即使用SpringApplicationBuilder将多个方法调用串起来，来创建多层次的ApplicationContext。

    ```java
  public static void main(String[] args) {
        new SpringApplicationBuilder()
          .sources(CustomerServiceApplication.class)
            .bannerMode(Banner.Mode.OFF)
          // web这里设为none的话就不会为我们启动一个tomcat来运行了
            .web(WebApplicationType.NONE)
            .run(args);
    }
    ```
  
  * RestTemplate的配置
  
    查看geektime.spring.springbucks.customer.CustomerServiceApplication#restTemplate
  
    Spring Boot虽然没有为我们自动配置一个RestTemplate，但是提供了一RestTemplateAutoConfiguration(org.springframework.boot.autoconfigure.web.client.RestTemplateAutoConfiguration)
  
  * RestTemplate的具体使用
  
    geektime.spring.springbucks.customer.CustomerServiceApplication#run

### 54.RestTemplate 的高阶用法

![193](E:\markdown笔记\笔记图片\13\193.png)

![194](E:\markdown笔记\笔记图片\13\194.png)

![195](E:\markdown笔记\笔记图片\13\195.png)

![196](E:\markdown笔记\笔记图片\13\196.png)

项目代码：

* 地址：[https://github.com/depers/geektime-spring-code/tree/master/Chapter%207/complex-resttemplate-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 7/complex-resttemplate-demo)

* 注意点：

  * @JsonComponent的使用

  * 几个关键的API

    * org.springframework.http.RequestEntity
    * org.springframework.http.ResponseEntity
    * org.springframework.http.RequestEntity.HeadersBuilder#accept
    * org.springframework.web.client.RestTemplate#exchange
    * `.price(Money.of(CurrencyUnit.of("CNY"), 25.00))`
    * org.springframework.core.ParameterizedTypeReference

  * 为什么非要用ParameterizedTypeReference来解析泛型呢？

    ```java
    List<Coffee> list1 = new ArrayList<>();
    list1 = restTemplate.getForObject(coffeeUri, list1.getClass());
    list1.forEach(c -> log.info("c：{}", c.getClass()));
    ```

    运行上面的代码报错： java.util.LinkedHashMap cannot be cast to geektime.spring.springbucks.customer.model.Coffee

### 55.简单定制 RestTemplate

![197](E:\markdown笔记\笔记图片\13\197.png)

![198](E:\markdown笔记\笔记图片\13\198.png)

![199](E:\markdown笔记\笔记图片\13\199.png)

![200](E:\markdown笔记\笔记图片\13\200.png)

项目代码：

* 地址：[https://github.com/depers/geektime-spring-code/tree/master/Chapter%207/advanced-resttemplate-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 7/advanced-resttemplate-demo)

* 注意点：

  * Apache HttpComponents官网：<http://hc.apache.org/>

  * pom文件修改：

    ```
    <dependency>
    	<groupId>org.apache.commons</groupId>
    	<artifactId>commons-lang3</artifactId>
    </dependency>
    
    <dependency>
    	<groupId>org.apache.httpcomponents</groupId>
    	<artifactId>httpclient</artifactId>
    	<version>4.5.7</version>
    </dependency>
    ```

  * 编写了geektime.spring.springbucks.customer.support.CustomConnectionKeepAliveStrategy

  * 编写了geektime.spring.springbucks.customer.CustomerServiceApplication#requestFactory

  * 修改了geektime.spring.springbucks.customer.CustomerServiceApplication#restTemplate

### 56.通过 WebClient 访问 Web 资源

![201](E:\markdown笔记\笔记图片\13\201.png)

![202](E:\markdown笔记\笔记图片\13\202.png)

项目代码：

* 地址： [https://github.com/depers/geektime-spring-code/tree/master/Chapter%207/webclient-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 7/webclient-demo) 

* 注意点：

  * pom文件修改

    ```xml 
    <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>
    ```

  * 代码分析：

    * geektime.spring.reactor.webclient.WebclientDemoApplication#webClient
    * geektime.spring.reactor.webclient.WebclientDemoApplication#run

### 57.SpringBucks 实战项目进度小结（cumtomer-service）

![204](E:\markdown笔记\笔记图片\13\204.png)

![205](E:\markdown笔记\笔记图片\13\205.png)

项目代码：

* 地址：[https://github.com/depers/geektime-spring-code/tree/master/Chapter%207/customer-service](https://github.com/depers/geektime-spring-code/tree/master/Chapter 7/customer-service)
* 注意点：
  * 该项目保留了本章所有的知识点，创建了cumtomer-service项目
  * 编写了geektime.spring.springbucks.customer.CustomerRunner

## 第八章： Web开发进阶

### 58.设计好的 RESTful Web Service（上）

![206](E:\markdown笔记\笔记图片\13\206.png)

该篇论文：[架构风格与基于网络应用软件的架构设计（中文修订版）](<https://www.infoq.cn/article/web-based-apps-archit-design>)

![207](E:\markdown笔记\笔记图片\13\207.png)

![208](E:\markdown笔记\笔记图片\13\208.png)

![209](E:\markdown笔记\笔记图片\13\209.png)

![210](E:\markdown笔记\笔记图片\13\210.png)

### 59.设计好的 RESTful Web Service（下）

![211](E:\markdown笔记\笔记图片\13\211.png)

![212](E:\markdown笔记\笔记图片\13\212.png)

注意：

* 安全是指：请求不会改变资源的内容。
* 幂等是指：无论我请求多少次，他的结果都是一样的。

![213](E:\markdown笔记\笔记图片\13\213.png)

![214](E:\markdown笔记\笔记图片\13\214.png)

![215](E:\markdown笔记\笔记图片\13\215.png)

### 60.什么是 HATEOAS

![216](E:\markdown笔记\笔记图片\13\216.png)

![217](E:\markdown笔记\笔记图片\13\217.png)

![218](E:\markdown笔记\笔记图片\13\218.png)

我理解的HATEOAS就是通过link来定义操作。

![219](E:\markdown笔记\笔记图片\13\219.png)

### 61.使用 Spring Data REST 实现简单的超媒体服务（上）

![220](E:\markdown笔记\笔记图片\13\220.png)

![221](E:\markdown笔记\笔记图片\13\221.png)

项目代码：

* 地址：[https://github.com/depers/geektime-spring-code/tree/master/Chapter%208/hateoas-waiter-service](https://github.com/depers/geektime-spring-code/tree/master/Chapter 8/hateoas-waiter-service)

* 注意点：

  * pom文件

    ```xml
    <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-rest</artifactId>
    </dependency>
    ```

  * 编写了geektime.spring.springbucks.waiter.repository.CoffeeOrderRepository

  * 编写了geektime.spring.springbucks.waiter.repository.CoffeeRepository

    ```java
    @RepositoryRestResource(path = "/coffee")
    public interface CoffeeRepository extends JpaRepository<Coffee, Long> {
        List<Coffee> findByNameInOrderById(List<String> list);
        Coffee findByName(String name);
    }
    ```

  * 无需编写任何controller，启动项目后，分别访问：

    * <http://localhost:8080/>

    * <http://localhost:8080/coffee>

    * <http://localhost:8080/coffee?page=0&size=3&sort=id,desc>

    * <http://localhost:8080/coffee/search>

      该接口就是geektime.spring.springbucks.waiter.repository.CoffeeRepository中定义的

      * <http://localhost:8080/coffee/search/findByName?name=mocha>
      * <http://localhost:8080/coffee/search/findByNameInOrderById?list=mocha,espresso>

### 62.使用 Spring Data REST 实现简单的超媒体服务(下)

![222](E:\markdown笔记\笔记图片\13\222.png)

项目代代码：

* 地址：[https://github.com/depers/geektime-spring-code/tree/master/Chapter%208/hateoas-customer-service](https://github.com/depers/geektime-spring-code/tree/master/Chapter 8/hateoas-customer-service)

* 注意点：

  在运行该项目之前，先要运行上一节中的hateoas-waiter-service，然后再运行**hateoas-customer-service**

  * 添加jackson对HAL的支持（geektime.spring.springbucks.customer.CustomerServiceApplication）

    ```java
    // jackson就可以支持HAL
    @Bean
    public Jackson2HalModule jackson2HalModule() {
       return new Jackson2HalModule();
    }
    ```

  * 编写请求逻辑

    geektime.spring.springbucks.customer.CustomerRunner#run

  * 使用postMan测试hateoas-waiter-service(参照[Spring全家桶.postman_collection.json]()中的HATEOAS)

    * 创建咖啡：http://localhost:8080/coffee
    * 创建订单：http://localhost:8080/coffeeOrders/
    * 查询订单：http://localhost:8080/coffeeOrders/1
    * 向订单添加咖啡：http://localhost:8080/coffeeOrders/1/items
    * 再查询订单：http://localhost:8080/coffeeOrders/1，就会发现订单里面又多了一杯咖啡

  * 在程序geektime.spring.springbucks.customer.CustomerRunner#run中演示了上述postMan中进行的测试。

### 63.分布式环境中如何解决 Session 的问题

![223](E:\markdown笔记\笔记图片\13\223.png)

![224](E:\markdown笔记\笔记图片\13\224.png)

Spring session其实就是集中式会话的一个实现；

![225](E:\markdown笔记\笔记图片\13\225.png)

![226](E:\markdown笔记\笔记图片\13\226.png)

项目代码：

* 地址：[https://github.com/depers/geektime-spring-code/tree/master/Chapter%208/session-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 8/session-demo)

* 注意点：

  * 源码分析

    * org.springframework.session.web.http.SessionRepositoryFilter#doFilterInternal
    * org.springframework.session.web.http.SessionRepositoryFilter.SessionRepositoryRequestWrapper#getSession(boolean)
    * org.springframework.session.data.redis.RedisOperationsSessionRepository#createSession
    * org.springframework.session.web.http.SessionRepositoryFilter.SessionRepositoryRequestWrapper#commitSession
    * org.springframework.session.data.redis.RedisOperationsSessionRepository#save

  * 修改pom

    ``` xml
    <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
    <dependency>
       <groupId>org.springframework.session</groupId>
       <artifactId>spring-session-core</artifactId>
    </dependency>
    <dependency>
       <groupId>org.springframework.session</groupId>
       <artifactId>spring-session-data-redis</artifactId>
    </dependency>
    ```

  * 配置application.properties

    ```properties
    spring.redis.host=localhost
    ```

  * 在geektime.spring.web.session.SessionDemoApplication添加注解@EnableRedisHttpSession

  * 在浏览器中输入http://localhost:8080/hello?name=spring

  * 然后在redis中查看` hgetall spring:session:sessions:aec8b286-1f8b-4194-8069-97af645d9677`

### 64.使用 WebFlux 代替 Spring MVC（上）

![228](E:\markdown笔记\笔记图片\13\228.png)

![229](E:\markdown笔记\笔记图片\13\229.png)

![230](E:\markdown笔记\笔记图片\13\230.png)

![231](E:\markdown笔记\笔记图片\13\231.png)

其中基于注解的控制器与Spring MVC的方式是十分相似的。

![232](E:\markdown笔记\笔记图片\13\232.png)

### 65.使用 WebFlux 代替 Spring MVC（下）

项目代码：

* 地址：[https://github.com/depers/geektime-spring-code/tree/master/Chapter%208/webflux-waiter-service](https://github.com/depers/geektime-spring-code/tree/master/Chapter 8/webflux-waiter-service)

* 注意点：

  * 修改pom文件

    ``` xml
    <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>
    
    <dependency>
       <groupId>org.springframework.data</groupId>
       <artifactId>spring-data-r2dbc</artifactId>
       <version>1.0.0.M1</version>
    </dependency>
    
    <dependency>
       <groupId>io.r2dbc</groupId>
       <artifactId>r2dbc-h2</artifactId>
       <version>1.0.0.M6</version>
    </dependency>
    ```

  * 编写了geektime.spring.springbucks.waiter.repository.CoffeeOrderRepository

  * 编写了geektime.spring.springbucks.waiter.repository.CoffeeRepository

  * 其他的基本上与前面项目中写法的差不多

### 66.SpringBucks 实战项目进度小结

![233](E:\markdown笔记\笔记图片\13\233.png)

![234](E:\markdown笔记\笔记图片\13\234.png)

## 第九章：重新认识Spring Boot

![235](E:\markdown笔记\笔记图片\13\235.png)

![236](E:\markdown笔记\笔记图片\13\236.png)

![237](E:\markdown笔记\笔记图片\13\237.png)

![238](E:\markdown笔记\笔记图片\13\238.png)

Spring Boot的文档：<https://docs.spring.io/spring-boot/docs/2.1.9.RELEASE/reference/html/>

### 68.了解自动配置的实现原理

![239](E:\markdown笔记\笔记图片\13\239.png)

其中在注解@SpringBootApplication中包含了@EnableAutoConfiguration。

![240](E:\markdown笔记\笔记图片\13\240.png)

![241](E:\markdown笔记\笔记图片\13\241.png)

* @Conditional：它的作用是按照一定的条件进行判断，满足条件给容器注册bean。
* @ConditionalOnClass：判断当前classpath下是否存在指定类，若存在则将当前的配置装载入spring容器。
* @ConditionalOnBean：就是在Spring容器里面存在某个特定名称的bean的时候该怎么做。
* @ConditionalOnMissingBean：就是我没有某个bean的时候该怎么做。
* @ConditionalOnProperty：当配置了特定属性会怎么样。
* @Import注解可以普通类导入到 IoC容器中。
* @Configuration标注在类上，相当于把该类作为spring的xml配置文件中的`<beans>`，作用为：配置spring容器(应用上下文)
* @EnableConfigurationProperties：在配置类上使用@EnableConfigurationProperties注解去指定这个类，这个时候就会让该类上的@ConfigurationProperties生效，然后作为bean添加进spring容器中

![242](E:\markdown笔记\笔记图片\13\242.png)

在项目启动参数中添加`--debug`，然后再启动项目。就可以看到Spring自动配置的类有哪些，计算报告的生成是由org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLoggingListener来做的。

* org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportMessage

![243](E:\markdown笔记\笔记图片\13\243.png)

项目代码：

* 地址： [https://github.com/depers/geektime-spring-code/tree/master/Chapter%209/autoconfigure-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 9/autoconfigure-demo) 

* 注意点：

  * 源码分析

    * @EnbaleAutoConfiguration

      * org.springframework.boot.autoconfigure.SpringBootApplication#exclude
      * org.springframework.boot.autoconfigure.AutoConfigurationImportSelector
        * org.springframework.boot.autoconfigure.AutoConfigurationImportSelector#selectImports
        * org.springframework.boot.autoconfigure.AutoConfigurationImportSelector#getAutoConfigurationEntry
        * org.springframework.boot.autoconfigure.AutoConfigurationImportSelector#getCandidateConfigurations
      * spring-boot-autoconfigure-2.1.3.RELEASE.jar!/META-INF/spring.factories：org.springframework.boot.autoconfigure.EnableAutoConfiguration

    * 条件注解

      * org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration

        ```java 
        @Configuration
        // 当满足EmbeddedDatabaseCondition的条件时
        @Conditional(EmbeddedDatabaseCondition.class)
        // 当Spring容器中不包含DataSource.class, XADataSource.class
        @ConditionalOnMissingBean({ DataSource.class, XADataSource.class })
        // 将EmbeddedDataSourceConfiguration注入到IOC容器中
        @Import(EmbeddedDataSourceConfiguration.class)
        protected static class EmbeddedDatabaseConfiguration {
        
        }
        ```
### 69.动手实现自己的自动配置

![244](E:\markdown笔记\笔记图片\13\244.png)

![245](E:\markdown笔记\笔记图片\13\245.png)

![246](E:\markdown笔记\笔记图片\13\246.png)

![247](E:\markdown笔记\笔记图片\13\247.png)

![248](E:\markdown笔记\笔记图片\13\248.png)

在IDEA中先通过File>import导入autoconfigure-demo，然后通过File>New>Module from existing source...分别导入geektime-spring-boot-autoconfig，greeting项目。

下面我将介绍三个小项目来说明本节的知识点：

* greeting

  1. 编写了geektime.spring.hello.greeting.GreetingApplicationRunner

* geektime-spring-boot-autoconfig

  1. pom文件

     ```xml
     <dependencies>
     		<dependency>
     			<groupId>org.springframework.boot</groupId>
     			<artifactId>spring-boot-autoconfigure</artifactId>
     		</dependency>
     
     		<dependency>
     			<groupId>geektime.spring.hello</groupId>
     			<artifactId>greeting</artifactId>
     			<version>0.0.1-SNAPSHOT</version>
     			<scope>provided</scope>
     		</dependency>
     </dependencies>
     ```

  2. 编写了geektime.spring.hello.greeting.GreetingAutoConfiguration

     ```java
     @Configuration
     // classPath存在GreetingApplicationRunner时才会生效
     @ConditionalOnClass(GreetingApplicationRunner.class)
     public class GreetingAutoConfiguration {
         @Bean
         // 只有在Spring上下文中不存在这个bean的时候才会生效
         @ConditionalOnMissingBean(GreetingApplicationRunner.class)
         // 只有greeting.enabled属性为true时才会生效，如果没有该属性默认为true
         @ConditionalOnProperty(name = "greeting.enabled", havingValue = "true", matchIfMissing = true)
         public GreetingApplicationRunner greetingApplicationRunner() {
             return new GreetingApplicationRunner();
         }
     }
     ```

  3. 编写了META-INF/spring.factories

     ```properties
     org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
     geektime.spring.hello.greeting.GreetingAutoConfiguration
     ```

* autoconfigure-demo

  1. pom文件

     ```xml
     <!-- 以下是基于Spring Boot的自动配置 -->
     <dependency>
        <groupId>geektime.spring.hello</groupId>
        <artifactId>geektime-spring-boot-autoconfigure</artifactId>
        <version>0.0.1-SNAPSHOT</version>
     </dependency>
     
     <dependency>
        <groupId>geektime.spring.hello</groupId>
        <artifactId>greeting</artifactId>
        <version>0.0.1-SNAPSHOT</version>
     </dependency>
     ```

  2. 启动项目
  
     1. 在application.properties中配置**greeting.enabled=true**：
  
        日志：
  
        2019-10-14 15:45:38.487  INFO 17952 --- [           main] g.s.h.g.GreetingApplicationRunner        : Initializing GreetingApplicationRunner for **GeekTime**.
        2019-10-14 15:45:38.602  INFO 17952 --- [           main] g.s.hello.AutoconfigureDemoApplication   : Started AutoconfigureDemoApplication in 1.735 seconds (JVM running for 3.66)
        2019-10-14 15:45:38.602  INFO 17952 --- [           main] g.s.h.g.GreetingApplicationRunner        : Hello everyone! We all like **Spring**！
  
     2. 在geektime.spring.hello.AutoconfigureDemoApplication添加
  
        ```java
        @Bean
        public GreetingApplicationRunner greetingApplicationRunner(){
           return new GreetingApplicationRunner("Spring");
        }
        ```
  
        日志：
  
        2019-10-14 15:51:46.724  INFO 10176 --- [           main] g.s.h.g.GreetingApplicationRunner        : Initializing GreetingApplicationRunner for **Spring**.
        2019-10-14 15:51:46.811  INFO 10176 --- [           main] g.s.hello.AutoconfigureDemoApplication   : Started AutoconfigureDemoApplication in 0.86 seconds (JVM running for 2.024)
        2019-10-14 15:51:46.811  INFO 10176 --- [           main] g.s.h.g.GreetingApplicationRunner        : Hello everyone! We all like **Spring**! 
  
     3. 在application.properties将greeting.enabled修改为**false**:
  
        日志：
        2019-10-14 16:24:11.813  INFO 7316 --- [           main] g.s.hello.AutoconfigureDemoApplication   : Started AutoconfigureDemoApplication in 0.747 seconds (JVM running for 1.883)

下面介绍一下本节的第二个知识点：Spring Boot中错误的分析机制：

* 包：org.springframework.boot.diagnostics.analyzer
* 源码分析：
  * org.springframework.boot.diagnostics.FailureAnalysis
  * org.springframework.boot.diagnostics.AbstractFailureAnalyzer
  * org.springframework.boot.autoconfigure.jdbc.DataSourceBeanCreationFailureAnalyzer

### 70.如何在低版本 Spring 中快速实现类似自动配置的功能

![249](E:\markdown笔记\笔记图片\13\249.png)

![250](E:\markdown笔记\笔记图片\13\250.png)

![251](E:\markdown笔记\笔记图片\13\251.png)

![252](E:\markdown笔记\笔记图片\13\252.png)

![253](E:\markdown笔记\笔记图片\13\253.png)

* 源码解析
  * org.springframework.beans.factory.support.AbstractBeanFactory#getBean
  * org.springframework.beans.factory.support.AbstractBeanFactory#doGetBean
  * org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#createBean
  * org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#doCreateBean
  * org.springframework.beans.factory.support.AbstractBeanFactory#registerDisposableBeanIfNecessary
  * org.springframework.beans.factory.support.AbstractBeanFactory#requiresDestruction
  * org.springframework.beans.factory.support.DisposableBeanAdapter#hasDestroyMethod
  
* 自己实现类似自动装配的功能

  在这个功能实现中，我们和上一节一样也有三个项目通过Module from existing source...添加，这三个项目跟分别是autoconfigure-demo，geektime-spring-boot-backport，greeting。

  * geektime-spring-boot-backport

    1. 编辑pom文件(可与上一节中geektime-spring-boot-autoconfig的pom比较一下)

       ```xml
       <dependencies>
          <dependency>
             <groupId>org.springframework</groupId>
             <artifactId>spring-context</artifactId>
          </dependency>
       
          <dependency>
             <groupId>org.projectlombok</groupId>
             <artifactId>lombok</artifactId>
          </dependency>
       
          <dependency>
             <groupId>geektime.spring.hello</groupId>
             <artifactId>greeting</artifactId>
             <version>0.0.1-SNAPSHOT</version>
             <scope>provided</scope>
          </dependency>
       </dependencies>
       ```

    2. 编写了geektime.spring.hello.greeting.GreetingAutoConfiguration

    3. 编写了geektime.spring.hello.greeting.GreetingBeanFactoryPostProcessor

  * autoconfigure-demo

    1. 修改pom文件

       ```xml
       <!-- 以下是手工实现的自动配置 -->
       <dependency>
       <groupId>geektime.spring.hello</groupId>
       <artifactId>geektime-autoconfigure-backport</artifactId>
       <version>0.0.1-SNAPSHOT</version>
       </dependency>
       
       <!-- 以下是基于Spring Boot的自动配置 -->
       <!--<dependency>-->
          <!--<groupId>geektime.spring.hello</groupId>-->
          <!--<artifactId>geektime-spring-boot-autoconfigure</artifactId>-->
          <!--<version>0.0.1-SNAPSHOT</version>-->
       <!--</dependency>-->
       ```

    2. pom文件右键单击>maven>reImport

    3. 正常启动geektime.spring.hello.AutoconfigureDemoApplication，就可以实现自动配置的功能了

  * greeting

    不做修改

  * 在autoconfigure-demo中为什么没有显示的引入配置类(通过component-scan或xml)

    这是因为在项目autoconfigure-demo中AutoconfigureDemoApplication的package是**geektime.spring.hello**，而在项目geektime-spring-boot-backport中GreetingAutoConfiguration的package是**geektime.spring.hello.greeting**。

    对于Spring Boot而言他会寻找带有@SpringBootApplication注解类的packeage以下这些package中的所有的配置。

### 71.了解起步依赖及其实现原理

起步依赖的作用就是简化maven的依赖配置。

![](E:\markdown笔记\笔记图片\13\254.png)

![255](E:\markdown笔记\笔记图片\13\255.png)

在Spring Boot项目中，我们查看了他的子module spring-boot-parent，该项目里面只有一个pom文件，在module spring-boot-dependencies中的pom文件中则声明了大量的版本控制信息。这样做的意思就是，我通过一个只包含pom文件的starter来声明项目中具体的依赖的版本控制信息。

如果我们想在自己的项目中对Spring Boot的某个依赖的version要做修改，升级或降级版本。我们只需要在自己的pom文件中定义与spring-boot-dependencies pom中声明的相同的version号就可以覆盖spring boot的version号。例如：

```
<lettuce.version>5.2.0.RELEASE</lettuce.version>
```

![256](E:\markdown笔记\笔记图片\13\256.png)

Spring Boot为我们提供了很多的starter，详情可以阅读相关的文档： https://docs.spring.io/spring-boot/docs/2.1.9.RELEASE/reference/html/using-boot-build-systems.html#using-boot-starter 

在 https://github.com/spring-projects/spring-boot/blob/master/spring-boot-project/spring-boot-starters/spring-boot-starter-jdbc/pom.xml 这个pom文件中我们就可以看到，当我们引入 spring-boot-starter-jdbc 时，他还会为我们引入 spring-boot-starter 、 HikariCP 和 spring-jdbc 这三个依赖。

### 72.定制自己的起步依赖

![](E:\markdown笔记\笔记图片\13\257.png)

![258](E:\markdown笔记\笔记图片\13\258.png)

在本节的操作中我们引入了四个项目分别是autoconfigure-demo，geektime-spring-boot-starter，geektime-spring-boot-backport，greeting，如下图所示：

![](E:\markdown笔记\笔记图片\13\260.png)

其中geektime-spring-boot-backport，greeting这两个项目没有变化。这里我来分别介绍下另外两个项目的变化：

* autoconfigure-demo项目

  1. pom文件的修改，修改后的pom文件的骨架结构如下图所示：

     ![](E:\markdown笔记\笔记图片\13\259.png)

     添加了：

     ```
     <dependency>
     	<groupId>geektime.spring.hello</groupId>
     	<artifactId>geektime-spring-boot-starter</artifactId>
     	<version>0.0.1-SNAPSHOT</version>
     </dependency>
     ```

* geektime-spring-boot-starter项目

  该项目的变化只是编辑了他的pom文件。项目地址： [https://github.com/depers/geektime-spring-code/tree/master/Chapter%209/geektime-spring-boot-starter](https://github.com/depers/geektime-spring-code/tree/master/Chapter 9/geektime-spring-boot-starter) 

通过上面的设置，我们如果要在项目中使用geektime-spring-boot-backport，greeting这两个项目，只需引入geektime-spring-boot-starter项目即可。我们可以运行geektime.spring.hello.AutoconfigureDemoApplication，这个项目的运行结果应该和之前的运行效果是一致的。

### 73.深挖 Spring Boot 的配置加载机制

#### 1.外化配置的加载顺序

![](E:\markdown笔记\笔记图片\13\261.png)

* 其中标红的命令行参数，例如使用**--server.port=9000**这个配置就会覆盖默认的8080端口启动。

![262](E:\markdown笔记\笔记图片\13\262.png)

* System.getProperties()处理的就是java -D参数指定的属性。
* 操作系统环境变量：比如linux系统中环境变量就可以加载进来的作为系统的配置。

![](E:\markdown笔记\笔记图片\13\263.png)

![264](E:\markdown笔记\笔记图片\13\264.png)

#### 2.application.properties的存放位置

![](E:\markdown笔记\笔记图片\13\265.png)

![](E:\markdown笔记\笔记图片\13\266.png)

#### 3.Relaxed Binding

![](E:\markdown笔记\笔记图片\13\267.png)

外置配置的相关参考文档： https://docs.spring.io/spring-boot/docs/2.1.9.RELEASE/reference/html/boot-features-external-config.html 

### 74.理解配置背后的 PropertySource 抽象

#### 1.基本知识

![](E:\markdown笔记\笔记图片\13\268.png)

![269](E:\markdown笔记\笔记图片\13\269.png)

![270](E:\markdown笔记\笔记图片\13\270.png)

#### 2.源码分析

* org.springframework.boot.autoconfigure.jdbc.JdbcProperties

  * @ConfigurationProperties(prefix = "spring.jdbc")

    说明所有以*spring.jdbc*打头的属性都会绑定到JdbcProperties这个类上。

* org.springframework.boot.env.RandomValuePropertySource

#### 2.实操

该项目实现了propertySource的添加

* 地址： [https://github.com/depers/geektime-spring-code/tree/master/Chapter%209/property-source-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 9/property-source-demo) 
* 实现步骤：
  1. 编写yapf.properties；
  2. 编写geektime.spring.hello.YapfEnvironmentPostProcessor；
  3. 在META-INF/spring.factories添加该类的声明，在spring boot启动时会实例化spring.factories中声明的类；
  4. 然后启动geektime.spring.hello.PropertySourceDemoApplication，读取yapf.properties中的属性；

#### 4.扩展知识点

我编写了一个simple-property-source-demo，地址：

在该项目中我们实现了只用@PropertySource注解就可以实现非application.properties的.properties文件内容的读取。

值得注意的是：application.properties中的内容是默认加载的，而且如果除application.properties的.properties也声明了相同的配置属性，则application.properties中声明的配置属性会覆盖其他配置的属性。

* application.properties中配置

  ```
  geektime.greeting=hi
  ```

* yapf.properties中配置

  ```
  geektime.greeting=hello
  ```

则通过@Value("${geektime.greeting}")取到的属性是hi。

