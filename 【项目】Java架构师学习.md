# 阶段一：单体电商项目架构，开发与上线

## 1.万丈高楼，地基首要

### 第1章 学习指南

#### 1.3 大型网站架构演变历程

1. web1.0：静态网站，用户只能访问服务器，用户和服务器之间的交互是单向的
2. web2.0：加入数据库，用户和服务器之间的交互式双向的
3. 单体项目：一台服务器由应用程序、文件服务器和数据库组成。应用程序是一个war包，
4. 模块分离：使用三台服务器分开部署程序、文件服务和数据库，分别是应用服务器、文件服务器和数据库服务器
5. 优化数据库查询：在模块分离的基础上再数据库前加一层缓存中间件
6. 集群化：引入负载均衡、tomcat应用集群、文件服务器集群、缓存集群，数据库不变
7. 数据库的读写分离（数据库主从分离架构）：采用数据库主库写入、从库读取的模式
8. 数据库主从分离集群化：采用主数据库集群写入、从数据库集群读取的模式（其中涉及分布式主键）
9. 引入搜索引擎技术：使用提供海量数据的搜索，并对数据库进行一定的保护
10. 微服务改造：应用拆分为商品服务集群、订单服务集群等，数据库拆分为商品数据库、订单数据库...
11. 引入异步通信机制：抽离公共服务资源，服务通过异步队列MQ与公共服务资源进行通信（其中涉及分布式锁、分布式事务、分布式会话）
12. 系统调优：JVM调优、Tomcat调优、数据库调优

#### 1.4 架构师需要的能力

1. 技术全面，有广度
2. 关注前沿技术
3. 全局观、预判
4. 把控团队、忙而不乱
5. 系统分解与模块拆分
6. 指导和培训
7. 沟通与协调能力
8. 抽象、举例、画图
9. 软技能

#### 1.5 关于异构系统的讨论

![](.\笔记图片\20\1\1\2-彩蛋： 关于异构系统的讨论.png)

###  第2章 单体架构设计与准备工作

#### 2.1 单体架构阶段概述与项目演示

单体项目Demo的访问网址： http://shop.t.mukewang.com/ 

#### 2.2 前后端技术选型讲解

技术选型需要考虑的问题

1. 切合业务
2. 社区是否活跃
3. 团队技术水平
4. 版本更新迭代周期
5. 试错精神
6. 安全性
7. 成功案例
8. 开源精神

#### 2.3 前后端分离开发模式

1. 早期传统Javaweb开发：用户在浏览器中请求的页面都是通过url进行跳转的
2. 前后端单页面交互，mvvm开发模式

#### 2.4项目分层设计

1. 项目拆分与聚合
2. Maven聚合项目

#### 2.5 构建聚合工程-1

1. 聚合工程里可分为顶级项目（顶级工程、父工程）与子工程，这两者的关系其实就是父子继承的关系，子工程在maven里称为模块（module），模块之间是平级的，是可以相互依赖的
2. 子模块可以使用顶级工程里所有的资源（依赖），子模块之间如果要使用资源，必须构建依赖（构建关系）
3.  一个顶级工程可以有多个不同的子工程共同组合而成。

#### 2.6 构建聚合工程-2

1. 搭建mall项目聚合结构，mall项目分别包含mall-common、mall-pojo、mall-mapper、mall-service、mall-api五个module。
2. github提交commit： https://github.com/depers/mall/commit/bec41aeb38a4ce37284ebdd39744c49405d8b2f3 

#### 2.7 PDMan数据库建模工具使用

1. 介绍了PDMan工具的使用方法
2. 本项目的设计文件： [https://github.com/depers/mall/blob/master/%E9%A1%B9%E7%9B%AE%E8%AE%BE%E8%AE%A1%E6%96%87%E4%BB%B6/mall.pdman.json](https://github.com/depers/mall/blob/master/项目设计文件/mall.pdman.json) 

#### 2.8 生产环境增量与全量脚本迭代讲解

在PDMan中的模型版本设置中，有一个选项是同步配置，在这里有两个选项一个是全量脚本迭代(重建数据表)，另一个是增量脚本迭代（字段增量）。推荐使用增量脚本迭代，因为全量脚本迭代耗时过多。

<img src="E:\markdown笔记\笔记图片\20\1\4.png" style="zoom:70%;" />

#### 2.9 数据库物理外键移除原因讲解

采用数据库外键带来的问题

1. 性能影响：影响数据库性能
2. 热更新：导致项目无法做热更新
3. 降低耦合度：若不使用物理外键会极大的降低数据库表与表之间的耦合度，保留逻辑外键
4. 数据库分库分表：难以实现数据库的分库分表

#### 2.10 聚合工程整合SpringBoot

1. 引入依赖parent

   ```xml
   <parent>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-parent</artifactId>
       <version>2.1.5.RELEASE</version>
       <relativePath />
   </parent>
   ```

2. 设置资源属性

   ```xml
   <properties>
       <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
       <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
       <java.version>1.8</java.version>
   </properties>
   ```

3. 引入依赖dependency

   ```xml
   <dependencies>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter</artifactId>
           <!--排除spring日志模块-->
           <exclusions>
               <exclusion>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-starter-logging</artifactId>
               </exclusion>
           </exclusions>
       </dependency>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-web</artifactId>
       </dependency>
       <!--spring默认使用yml中的配置，但有时候要用传统的xml或properties配置，就需要使用spring-boot-configuration-processor-->
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-configuration-processor</artifactId>
           <optional>true</optional>
       </dependency>
   </dependencies>
   ```

4. github提交commit： https://github.com/depers/mall/commit/cb41e10ba698d6b17a9321ce16e827e5022c3781 

#### 2.12 SpringBoot自动装配简述

探究Spring Boot自动装配背后的秘密

1. @SpringBootApplication 
2. org.springframework.boot.autoconfigure.EnableAutoConfiguration
3. org.springframework.boot.autoconfigure.AutoConfigurationImportSelector
   1. getAutoConfigurationEntry
   2. getCandidateConfigurations
      1. \META-INF\spring.factories
         1. org.springframework.boot.autoconfigure.web.embedded.EmbeddedWebServerFactoryCustomizerAutoConfiguration
         2. org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration
         3. org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration

#### 2.13 HikariCP数据源简述

* HikariCP Github:  https://github.com/brettwooldridge/HikariCP 

* HikariCP为什么那么快？

  *  https://github.com/brettwooldridge/HikariCP/wiki/Down-the-Rabbit-Hole 
* 参见《Spring全家桶-上卷》第二章第七节。

#### 2.14 数据层HikariCP与MyBatis整合

1. 在模块mall-api中，在pom文件中引入数据源驱动与Mybatis依赖

   ```xml
   <!-- mysql驱动 -->
   		<dependency>
   			<groupId>mysql</groupId>
   			<artifactId>mysql-connector-java</artifactId>
   			<version>5.1.41</version>
   		</dependency>
   		<!-- mybatis -->
   		<dependency>
   			<groupId>org.mybatis.spring.boot</groupId>
   			<artifactId>mybatis-spring-boot-starter</artifactId>
   			<version>2.1.0</version>
   		</dependency>
   ```

2. 在模块mall-api中，在yml中配置数据源和mybatis

   ```yaml
   ############################################################
   #
   # 配置数据源信息
   #
   ############################################################
   spring:
     profiles:
       active: dev
     datasource:                                           # 数据源的相关配置
       type: com.zaxxer.hikari.HikariDataSource          # 数据源类型：HikariCP
       driver-class-name: com.mysql.jdbc.Driver          # mysql驱动
       url: jdbc:mysql://localhost:3306/mall-dev?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true
       username: root
       password: root
       hikari:
         connection-timeout: 30000       # 等待连接池分配连接的最大时长（毫秒），超过这个时长还没可用的连接则发生SQLException， 默认:30秒
         minimum-idle: 5                 # 最小连接数
         maximum-pool-size: 20           # 最大连接数
         auto-commit: true               # 自动提交
         idle-timeout: 600000            # 连接超时的最大时长（毫秒），超时则被释放（retired），默认:10分钟
         pool-name: DateSourceHikariCP     # 连接池名字
         max-lifetime: 1800000           # 连接的生命时长（毫秒），超时而且没被使用则被释放（retired），默认:30分钟 1800000ms
         connection-test-query: SELECT 1
     servlet:
       multipart:
         max-file-size: 512000     # 文件上传大小限制为500kb
         max-request-size: 512000  # 请求大小限制为500kb
   
   ############################################################
   #
   # mybatis 配置
   #
   ############################################################
   mybatis:
     type-aliases-package: cn.bravedawn.pojo        # 所有POJO类所在包路径
     mapper-locations: classpath:mapper/*.xml      # mapper映射文件
   ```

3. 在模块mall-api中，进行内置Tomcat的配置

   ```yaml
   ############################################################
   #
   # web访问端口号  约定：8088
   #
   ############################################################
   server:
     port: 8088
     tomcat:
       uri-encoding: UTF-8
     max-http-header-size: 80KB
   ```

4. github提交commit： https://github.com/depers/mall/commit/1bf357a0723a0580125120a588205284aff27ecb 

#### 2.15 数据源连接数详解

在模块mall-api中的application.yml文件中的配置如下：

```yaml
minimum-idle: 5                 # 最小连接数
maximum-pool-size: 20           # 最大连接数
```

1. 在HikariCP中默认最大连接数为10，没有配置最小连接数，此时最小连接数等于最大连接数为10，固定的连接数性能较好。
2. 最大连接数是要根据自己服务器的具体配置而定的，四核处理器建议配置为10，八核的话建议配置为20。
3. 这里最小的连接数是为了防止闲置的连接数过多而配置的，建议配置为5-10。
4. 在自己的项目中使用的话，建议配置：
   1. 将最大最小连接数配置为相同值，具体大小根据服务器性能而定；
   2. 最小连接数配置为5，最大连接数根据服务器性能而定；

#### 2.15 MyBatis 数据库逆向生成工具

MyBatis数据库你像生成工具——MyBatis-Generator（内置MyMapper插件），可以帮助开发人员生成相关文件：

* pojo
* *mapper.xml
* *mapper.java

1. 在mall项目的pom中引入通用mapper工具

   ```xml
   <!-- 通用mapper逆向工具 -->
   <dependency>
       <groupId>tk.mybatis</groupId>
       <artifactId>mapper-spring-boot-starter</artifactId>
       <version>2.1.5</version>
   </dependency>
   ```

2. 在mall-api项目的yml中引入通用mapper配置

   ```yaml
   ############################################################
   #
   # mybatis mapper 配置
   #
   ############################################################
   # 通用 Mapper 配置
   mapper:
     mappers: cn.bravedawn.my.mapper.MyMapper
     not-empty: false    # 在进行数据库操作的的时候，判断表达式 username != null, 是否追加 username != ''
     identity: MYSQL
   ```

3. 在mall-mapper项目中引入MyMapper接口类

   ```java
   import tk.mybatis.mapper.common.Mapper;
   import tk.mybatis.mapper.common.MySqlMapper;
   
   /**
    * 继承自己的MyMapper
    */
   public interface MyMapper<T> extends Mapper<T>, MySqlMapper<T> {
   }
   ```

4. github提交commit： https://github.com/depers/mall/commit/79d55b03c185ada8c6fd98f1b4f98856e6986c95 

#### 2.17 通用Mapper接口所封装的常用方法

![](E:\markdown笔记\笔记图片\20\1\5.jpg)

#### 2.18 关于Restful webservice的那些事儿

* Restful Web Service

  1. 它是一种通信方式，例如rpc通信
  2. 它是系统与系统之间进行消息传递的一种方式，使用JSON作为介质
  3. 所有的请求都是无状态的
  4. 使用Restful进行交互的系统，他们各自是独立的

* Rest设计规范

  1. get（/order/{id} -> /getOrder?id=1）：获取资源
  2. post（/order -> /saveOrder）：保存
  3. put（/order/{id} -> /modifyOrder?id=1）：更新
  4. delete（/order/{id} -> /deleteOrder?id=1）：删除

  注意：严格规范下url中不要出现动词，但是弱规范下其实可以使用更加有意义的动词去描述接口的功能

#### 2.19  基于通用Mapper基于Rest编写api接口

1. 添加mybatis通用mapper的包扫描，在mall-api的application.java文件中

   ```java
   // 扫描mybatis通用mapper所在的包
   @MapperScan("cn.bravedawn.mapper")
   ```

   这里的@MapperScan切记是**import tk.mybatis.spring.annotation.MapperScan;**下的。
   
2. 注入Mapper实例时IDEA红色下划线的错误，解决方法：

   ![](E:\markdown笔记\笔记图片\20\1\6.png)

   去掉图中红色框中的对勾。

#### 2.20 详解事务的传播-Propagation.REQUIRED

对于Spring事务这部分的知识，也可以参考《玩转Spring全家桶-上卷》2.10节。

1. 在mall-api项目中的pom文件中引入依赖：

   ```xml
   <!--引入junit单元测试-->
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-test</artifactId>
       <scope>test</scope>
   </dependency>
   ```

2. **事务测试1**-没有任何事务

   1. （mall-service）此时的cn.bravedawn.service.Impl.StuServiceImpl文件：

      ```java
      public class StuServiceImpl implements StuService {
          @Autowired
          private StuMapper stuMapper;
      
          public void saveParent() {
              Stu stu = new Stu();
              stu.setName("parent");
              stu.setAge(19);
              stuMapper.insert(stu);
          }
          
          // 子方法
          public void saveChildren() {
              saveChild1();
              int a = 1 / 0;  //------------------------------报除0异常
              saveChild2();
          }
      
          public void saveChild1() {
              Stu stu1 = new Stu();
              stu1.setName("child-1");
              stu1.setAge(11);
              stuMapper.insert(stu1);
          }
          public void saveChild2() {
              Stu stu2 = new Stu();
              stu2.setName("child-2");
              stu2.setAge(22);
              stuMapper.insert(stu2);
          }
      }
      ```

   2. （mall-service）此时的cn.bravedawn.service.Impl.TestTransServiceImpl文件：

      ```java
      @Service
      public class TestTransServiceImpl implements TestTransService {
      
          @Autowired
          private StuService stuService;
          
          @Override
          public void testPropagationTrans() {
              // 1. 在没有任何事务下
              stuService.saveParent();
              stuService.saveChildren();
          }
      }
      
      ```

   3. （mall-api）此时的cn.bravedawn.TransTest文件：

      ```java
      @RunWith(SpringRunner.class)
      @SpringBootTest(classes = Application.class)
      public class TransTest {
      
          @Autowired
          private StuService stuService;
      
          @Autowired
          private TestTransService testTransService;
      
      	@Test
          public void myTest() {
              testTransService.testPropagationTrans();
          }
      
      }
      ```

   4. 数据库的保存效果，select * from stu：

      ![](E:\markdown笔记\笔记图片\20\1\5.PNG)

      如上图所示，测试案例只插入了parent和child-1，但是没有插入child-2。因为程序cn.bravedawn.service.Impl.StuServiceImpl中的saveChildren()执行出现了除0错误。

3. **测试案例2**-父方法添加事务传播Propagation.REQUIRED

   1. （mall-service）此时的cn.bravedawn.service.Impl.TestTransServiceImpl文件：

      ```java
      @Transactional(propagation = Propagation.REQUIRED)
      @Override
      // 父方法
      public void testPropagationTrans() {
          // 2. 在事务传播特性Propagation.REQUIRED下
          stuService.saveParent();
          stuService.saveChildren();
      }
      ```

   2. 数据库的保存效果，select * from stu：没有存入任何数据。**父方法上添加了事务，父方法中调用的子方法也具备事务性，一并回滚了**。

4. **测试案例2**-子方法添加事务传播Propagation.REQUIRED，父方法则没有声明事务

   1. （mall-service）此时的cn.bravedawn.service.Impl.TestTransServiceImpl文件：

      ```java
      @Override
      // 父方法
      public void testPropagationTrans() {
          // 2. 在事务传播特性Propagation.REQUIRED下
          stuService.saveParent();
          stuService.saveChildren();
      }
      ```

      去掉该父方法的事务声明。

   2. （mall-service）此时的cn.bravedawn.service.Impl.StuServiceImpl文件:

      ```java
      @Transactional(propagation = Propagation.REQUIRED)
      // 子方法
      public void saveChildren() {
      	saveChild1();
      	int a = 1 / 0;  //------------------------------报除0异常
      	saveChild2();
      }
      ```

      给子方法添加事务声明。

   3. 数据库的保存效果，select * from stu：

      ![](E:\markdown笔记\笔记图片\20\1\7.png)

      从图中我们可以看出，子方法声明了事务，父方法未声明事务，则子方法有事务特性，而父方法没有。

5. **测试案例3**-父子方法都添加事务传播Propagation.REQUIRED

   1. （mall-service）此时的cn.bravedawn.service.Impl.TestTransServiceImpl文件：

      ```java
      @Transactional(propagation = Propagation.REQUIRED)
      @Override
      // 父方法
      public void testPropagationTrans() {
          // 2. 在事务传播特性Propagation.REQUIRED下
          stuService.saveParent();
          stuService.saveChildren();
      }
      ```

      去掉该父方法的事务声明。

   2. （mall-service）此时的cn.bravedawn.service.Impl.StuServiceImpl文件:

      ```java
      @Transactional(propagation = Propagation.REQUIRED)
      // 子方法
      public void saveChildren() {
      	saveChild1();
      	int a = 1 / 0;  //------------------------------报除0异常
      	saveChild2();
      }
      ```

      给子方法添加事务声明。

   3. 数据库的保存效果，select * from stu：父子方法都遵从事务特性，数据库中没有数据。

6. 总结：

   1. **对子方法而言：当前有事务就用当前的，当前没有事务则自己新建一个事务，子方法是必须运行在一个事务中的；**。
   2. 父级有事务，则子级也有事务。此时父子共用一个事务；
   3. 子级声明事务，则父级没有声明事务，则子有父没有；
   4. 父子都声明事务，其实共用的是一个事务；
   5. 常用于增、删、改操作；

#### 2.21 详解事务的传播-Propagation.SUPPORTS

1. **测试案例1**-给子方法添加Propagation.SUPPORTS声明

   1. （mall-service）此时的cn.bravedawn.service.Impl.TestTransServiceImpl文件：

      ```java
      @Override
      // 父方法
      public void testPropagationTrans() {
          stuService.saveParent();
          stuService.saveChildren();
      }
      ```

      去掉该父方法的事务声明。

   2. （mall-service）此时的cn.bravedawn.service.Impl.StuServiceImpl文件:

      ```java
      @Transactional(propagation = Propagation.SUPPORTS)
      // 子方法
      public void saveChildren() {
      	saveChild1();
      	int a = 1 / 0;  //------------------------------报除0异常
      	saveChild2();
      }
      ```

      给子方法添加事务声明。

   3. 数据库的保存效果，select * from stu：

      ![](E:\markdown笔记\笔记图片\20\1\8.png)

      可以看到数据库中有两条数据，和父子都没有添加任何事务的效果一致。

2. **测试案例2**-给子方法添加Propagation.SUPPORTS声明，给父方法添加Propagation.REQUIRED

   1. （mall-service）此时的cn.bravedawn.service.Impl.TestTransServiceImpl文件：

      ```java
      @Transactional(propagation = Propagation.REQUIRED)
      @Override
      // 父方法
      public void testPropagationTrans() {
          stuService.saveParent();
          stuService.saveChildren();
      }
      ```

      去掉该父方法的事务声明。

   2. （mall-service）此时的cn.bravedawn.service.Impl.StuServiceImpl文件:

      ```java
      @Transactional(propagation = Propagation.SUPPORTS)
      // 子方法
      public void saveChildren() {
      	saveChild1();
      	int a = 1 / 0;  //------------------------------报除0异常
      	saveChild2();
      }
      ```

      给子方法添加事务声明。

   3. 数据库的保存效果，select * from stu：没有插入任何数据，父方法的事务覆盖了子方法的事务。

3. 总结：

   1. 事务可有可无，不是必须的
   2. 对子方法而言：当前有事务，则使用事务；若当前没有事务，则不使用事务

#### 2.22 详解事务的传播-Propagation.MANDATORY

1. **测试案例1**-给子方法添加Propagation.MANDATORY声明

   1. （mall-service）此时的cn.bravedawn.service.Impl.TestTransServiceImpl文件：

      ```java
      @Override
      // 父方法
      public void testPropagationTrans() {
          stuService.saveParent();
          stuService.saveChildren();
      }
      ```

      去掉该父方法的事务声明。

   2. （mall-service）此时的cn.bravedawn.service.Impl.StuServiceImpl文件:

      ```java
      @Transactional(propagation = Propagation.MANDATORY)
      // 子方法
      public void saveChildren() {
      	saveChild1();
      	int a = 1 / 0;  //------------------------------报除0异常
      	saveChild2();
      }
      ```

      给子方法添加事务声明。

   3. 抛出**org.springframework.transaction.IllegalTransactionStateException: No existing transaction found for transaction marked with propagation 'mandatory'**异常。

      若子方法声明mandatory事务，而父方法没有声明任何事务，则会抛出异常。

   4. 数据库的保存效果，select * from stu：

      ![](E:\markdown笔记\笔记图片\20\1\7.png)

      只执行了saveParent()，并未执行声明了mandatory事务的saveChildren()方法。

2. **测试案例2**-给子方法添加Propagation.MANDATORY声明，给父方法添加Propagation.REQUIRED声明

   1. （mall-service）此时的cn.bravedawn.service.Impl.TestTransServiceImpl文件：

      ```java
      @Transactional(propagation = Propagation.REQUIRED)
      @Override
      // 父方法
      public void testPropagationTrans() {
          stuService.saveParent();
          stuService.saveChildren();
      }
      ```

      去掉该父方法的事务声明。

   2. （mall-service）此时的cn.bravedawn.service.Impl.StuServiceImpl文件:

      ```java
      @Transactional(propagation = Propagation.MANDATORY)
      // 子方法
      public void saveChildren() {
      	saveChild1();
      	int a = 1 / 0;  //------------------------------报除0异常
      	saveChild2();
      }
      ```

      给子方法添加事务声明。

   3. 数据库的保存效果，select * from stu：没有插入任何数据，并没有报测试案例2的错误。

3. 总结：子方法声明该事物，该传播属性强制父方法必须存在一个事务，如果不存在，则抛出异常

#### 2.23 详解事务的传播-Propagation.REQUIRES_NEW

1. **测试案例1**-给子方法添加Propagation.REQUIRES_NEW声明

   1. （mall-service）此时的cn.bravedawn.service.Impl.TestTransServiceImpl文件：

      ```java
      @Override
      // 父方法
      public void testPropagationTrans() {
          stuService.saveParent();
          stuService.saveChildren();
      }
      ```

      去掉该父方法的事务声明。

   2. （mall-service）此时的cn.bravedawn.service.Impl.StuServiceImpl文件:

      ```java
      @Transactional(propagation = Propagation.REQUIRES_NEW)
      // 子方法
      public void saveChildren() {
      	saveChild1();
      	int a = 1 / 0;  //------------------------------报除0异常
      	saveChild2();
      }
      ```

      给子方法添加事务声明。

   3. 数据库的保存效果，select * from stu：

      ![](E:\markdown笔记\笔记图片\20\1\7.png)

      此时，子方法有事务，而父方法没有事务。

2. **测试案例2**-给子方法添加Propagation.REQUIRES_NEW声明，给父方法添加Propagation.REQUIRES声明：

   1. （mall-service）此时的cn.bravedawn.service.Impl.TestTransServiceImpl文件：

      ```java
      @Transactional(propagation = Propagation.REQUIRED)
      @Override
      // 父方法
      public void testPropagationTrans() {
          stuService.saveParent();
          stuService.saveChildren();
      }
      ```

      去掉该父方法的事务声明。

   2. （mall-service）此时的cn.bravedawn.service.Impl.StuServiceImpl文件:

      ```java
      @Transactional(propagation = Propagation.REQUIRES_NEW)
      // 子方法
      public void saveChildren() {
      	saveChild1();
      	int a = 1 / 0;  //------------------------------报除0异常
      	saveChild2();
      }
      ```

      给子方法添加事务声明。

   3. 数据库的保存效果，select * from stu：没有插入任何数据。

      此时，父方法和子方法各有一个事务。

3. **测试案例3**-给子方法添加Propagation.REQUIRES_NEW声明，给父方法添加Propagation.REQUIRES声明，并修改父子方法：

   1. （mall-service）此时的cn.bravedawn.service.Impl.TestTransServiceImpl文件：

      ```java
      @Transactional(propagation = Propagation.REQUIRED)
      @Override
      // 父方法
      public void testPropagationTrans() {
          stuService.saveParent();
          stuService.saveChildren();
          int a = 1 / 0; //----------------------------除0异常
      }
      ```

      去掉该父方法的事务声明。

   2. （mall-service）此时的cn.bravedawn.service.Impl.StuServiceImpl文件:

      ```java
      @Transactional(propagation = Propagation.REQUIRES_NEW)
      // 子方法
      public void saveChildren() {
      	saveChild1();
      	saveChild2();
      }
      ```

      给子方法添加事务声明。

   3. 数据库的保存效果，select * from stu：

      ![](E:\markdown笔记\笔记图片\20\1\9.png)

      这下就有意思了，子方法会顺利执行，而父方法抛出除0异常，导致saveParent()方法回滚，却没有影响到saveChildren()子方法。

      这说明子方法和父方法是各持有一个事务，互不干扰。

      是否有异常也不影响自己。

4. 总结：

   1. 如果当前有事务，则挂起该事务，并且自己创建一个新的事务给自己使用；
   2. 如果当前没有事务，则同 REQUIRED
   3. 无论是否有事务都起个新事务

#### 2.24 详解事务的传播-Propagation.NOT_SUPPORTED

1. **测试案例1**-给子方法添加Propagation.NOT_SUPPORTED声明

   1. （mall-service）此时的cn.bravedawn.service.Impl.TestTransServiceImpl文件：

      ```java
      @Override
      // 父方法
      public void testPropagationTrans() {
          stuService.saveParent();
          stuService.saveChildren();
      }
      ```

      去掉该父方法的事务声明。

   2. （mall-service）此时的cn.bravedawn.service.Impl.StuServiceImpl文件:

      ```java
      @Transactional(propagation = Propagation.NOT_SUPPORTED)
      // 子方法
      public void saveChildren() {
      	saveChild1();
      	int a = 1 / 0;  //------------------------------报除0异常
      	saveChild2();
      }
      ```

      给子方法添加事务声明。

   3. 数据库的保存效果，select * from stu：

      ![](E:\markdown笔记\笔记图片\20\1\8.png)

      和没有添加任何事务的情况是相同的。

2. **测试案例2**-给子方法添加Propagation.NOT_SUPPORTED声明，给父方法添加Propagation.REQUIRED声明

   1. （mall-service）此时的cn.bravedawn.service.Impl.TestTransServiceImpl文件：

      ```java
      @Transactional(propagation = Propagation.REQUIRED)
      @Override
      // 父方法
      public void testPropagationTrans() {
          stuService.saveParent();
          stuService.saveChildren();
      }
      ```

   2. （mall-service）此时的cn.bravedawn.service.Impl.StuServiceImpl文件:

      ```java
   @Transactional(propagation = Propagation.NOT_SUPPORTED)
      // 子方法
      public void saveChildren() {
      	saveChild1();
      	int a = 1 / 0;  //------------------------------报除0异常
      	saveChild2();
      }
      ```
   
      给子方法添加事务声明。

   3. 数据库的保存效果，select * from stu：

      ![](E:\markdown笔记\笔记图片\20\1\10.png)

      父方法由于事务的原因，他的数据并没有成功的插入数据库中；而子方法由于不支持事务，所以插入了一条数据。

3. 总结：

   1. 如果当前有事务，则把事务挂起，自己不用事务去运行数据库操作；
   2. 不支持事务，按非事务方式运行；

#### 2.25 详解事务的传播-Propagation.NEVER

1. **测试案例1**-给子方法添加Propagation.NEVER声明

   1. （mall-service）此时的cn.bravedawn.service.Impl.TestTransServiceImpl文件：

      ```java
      @Transactional(propagation = Propagation.REQUIRED)
      @Override
      // 父方法
      public void testPropagationTrans() {
          stuService.saveParent();
          stuService.saveChildren();
      }
      ```

   2. （mall-service）此时的cn.bravedawn.service.Impl.StuServiceImpl文件:

      ```java
   @Transactional(propagation = Propagation.NEVER)
      // 子方法
      public void saveChildren() {
      	saveChild1();
      	int a = 1 / 0;  //------------------------------报除0异常
      	saveChild2();
      }
      ```
   
      给子方法添加事务声明。

   3. 数据库的保存效果，select * from stu：此时数据库中没有插入任何数据。执行该方法会报**org.springframework.transaction.IllegalTransactionStateException: Existing transaction found for transaction marked with propagation 'never'**异常

2. 总结：**如果当前有事务存在，则抛出异常**。

#### 2.26 详解事务的传播-Propagation.NESTED

1. **测试案例1**-给子方法添加Propagation.NESTED声明，父方法抛出异常

   1. （mall-service）此时的cn.bravedawn.service.Impl.TestTransServiceImpl文件：

      ```java
      @Transactional(propagation = Propagation.REQUIRED)
      @Override
      // 父方法
      public void testPropagationTrans() {
          stuService.saveParent();
          stuService.saveChildren();
          int a = 1 / 0;  //------------------------------报除0异常
      }
      ```

   2. （mall-service）此时的cn.bravedawn.service.Impl.StuServiceImpl文件:

      ```java
   @Transactional(propagation = Propagation.NESTED)
      // 子方法
      public void saveChildren() {
      	saveChild1();
      	saveChild2();
      }
      ```
   
      给子方法添加事务声明。

   3. 数据库的保存效果，select * from stu：此时数据库中没有插入任何数据。子方法使用Propagation.NESTED，他会嵌套在父事务中执行。他与父事务共用一个事务。这个要与Propagation.REQUIRES_NEW的测试案例3相区别。

2. **测试案例2**-给子方法添加Propagation.NESTED声明，子方法抛出异常

   1. （mall-service）此时的cn.bravedawn.service.Impl.TestTransServiceImpl文件：

      ```java
      @Transactional(propagation = Propagation.REQUIRED)
      @Override
      // 父方法
      public void testPropagationTrans() {
          stuService.saveParent();
          stuService.saveChildren();
      }
      ```

   2. （mall-service）此时的cn.bravedawn.service.Impl.StuServiceImpl文件:

      ```java
   @Transactional(propagation = Propagation.NESTED)
      // 子方法
      public void saveChildren() {
      	saveChild1();
          int a = 1 / 0;  //------------------------------报除0异常
      	saveChild2();
      }
      ```
   
      给子方法添加事务声明。

   3. 数据库的保存效果，select * from stu：数据库中没有插入任何数据。子事务异常，父事务也要进行回滚。

3. **测试案例3**-给子方法添加Propagation.NESTED声明，

   1. （mall-service）此时的cn.bravedawn.service.Impl.TestTransServiceImpl文件：

      ```java
      @Transactional(propagation = Propagation.REQUIRED)
      @Override
      // 父方法
      public void testPropagationTrans() {
          stuService.saveParent();
          try {
              // save point
              stuService.saveChildren();
          } catch (Exception e) {
              e.printStackTrace();
          }
      }
      ```

      父方法捕获异常。

   2. （mall-service）此时的cn.bravedawn.service.Impl.StuServiceImpl文件:

      ```java
      @Transactional(propagation = Propagation.NESTED)
      // 子方法
      public void saveChildren() {
      	saveChild1();
          int a = 1 / 0;  //------------------------------报除0异常
      	saveChild2();
      }
      ```

      给子方法添加事务声明。

   3. 数据库的保存效果，select * from stu：

      ![](E:\markdown笔记\笔记图片\20\1\11.png)

      此时数据库中插入一条数据，子方法的异常被父方法捕获。父事务可以不随子事务进行回滚。

4. 总结：

   1. 如果当前有事务，则开启子事务（**嵌套事务**），嵌套事务是独立提交或者回滚；
   2. 如果当前没有事务，则同 REQUIRED。
   3. 如果主事务提交，则会携带子事务一起提交。如果主事务回滚，则子事务会一起回滚。
   4. **相反，子事务异常，则父事务可以回滚或不回滚。**

#### 2.27 为何不使用@EnableTransactionManagement？

在SpringBoot中使用**@SpringBootApplication**注解，会对开启自动装配的设置。其中在META-I**NF/spring.factories**文件中已经对org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration进行了开启，所以不用再使用@EnableTransactionManagement开启Spring的事务管理。

 #### 2.28 复习总结

![](E:\markdown笔记\笔记图片\20\1\12.jpg)

### 第3章 用户登录注册模块开发

#### 3.1 验证用户是否存在接口和注册接口开发

1. 编写验证用户是否存在接口代码

   * 代码：cn.bravedawn.controller.PassportController#usernameIsExist

2. 创建响应信息结构：cn.bravedawn.utils.JsonResult

3. 引入apache工具依赖：

   ```xml
   <!-- apache 工具类 -->
   <dependency>
       <groupId>commons-codec</groupId>
       <artifactId>commons-codec</artifactId>
       <version>1.11</version>
   </dependency>
   <dependency>
       <groupId>org.apache.commons</groupId>
       <artifactId>commons-lang3</artifactId>
       <version>3.4</version>
   </dependency>
   <dependency>
       <groupId>org.apache.commons</groupId>
       <artifactId>commons-io</artifactId>
       <version>1.3.2</version>
   </dependency>
   ```

4. 编写用户注册接口

   * 代码：cn.bravedawn.controller.PassportController#regist

   * 引入ID生成方法：

     * org.n3r.idworker.Sid#nextShort

     * 修改mall-api中的application.java

       ```java
       // 扫描所有包以及相关组件包
       @ComponentScan(basePackages = {"cn.bravedawn", "org.n3r.idworker"})
       ```

   * 引入MD5加密工具：cn.bravedawn.utils.MD5Utils

   * 引入日期处理工具：cn.bravedawn.utils.DateUtil

   * 引入性别处理枚举：cn.bravedawn.enums.Sex

#### 3.2 整合swagger2文档API

1. 在mall项目的Pom文件中添加swagger的相关依赖：

   ```xml
    <!-- swagger2 配置 -->
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger2</artifactId>
        <version>2.4.0</version>
    </dependency>
    <dependency>
        <groupId>io.springfox</groupId>
        <artifactId>springfox-swagger-ui</artifactId>
        <version>2.4.0</version>
    </dependency>
    <dependency>
        <groupId>com.github.xiaoymin</groupId>
        <artifactId>swagger-bootstrap-ui</artifactId>
        <version>1.6</version>
    </dependency>
   ```

2. 对Swagger进行配置：cn.bravedawn.config.Swagger2

#### 3.3 优化Swagger2显示

1. 忽略Controller显示，@ApiIgnore：cn.bravedawn.controller.HelloController
2. 添加接口信息：cn.bravedawn.controller.PassportController
   1. @Api
   2. @ApiOperation
3. 添加请求参数对象注解，@ApiModel：cn.bravedawn.bo.UserBO

#### 3.4 使用Tomcat运行前端源码

将前端源码foodie-shop放置到tomcat的webapp目录下，然后启动Tomcat，在浏览器中输入http://localhost:8080/foodie-shop进行请求。

#### 3.5 设置跨域配置，实现前后端联调

1. 后端对跨域请求进行配置，参见cn.bravedawn.config.CorsConfig

2. 前端的跨域配置，register.html：

   ```js
   // 前端请求，可以携带cookie信息
   axios.defaults.withCredentials = true;
   ```

3. 前端的js\app.js文件中设置后端请求连接：

   ```js
   serverUrl: "http://localhost:8088",
   ```

#### 3.6 回顾cookie与session

1. Cookie
   * 以键值对的形式存储信息在浏览器中，一个cookie保存的大小是4kb
   * cookie不能跨域，当前及其父级域名可以取值
   * cookie可以设置有效期
   * cookie可以设置path
2. session
   * 基于服务器内存的缓存（非持久化），可保存请求会话
   * 每个session通过sessionId来区分不同请求
   * session可设置过期时间
   * session也是以键值对形式存在的
3. 在mall-common中添加JsonUtils和CookieUtils代码。
4. 在mall-api中给注册和登录接口添加Cookie相关的逻辑代码。

#### 3.7 整合log4j打印日志

1. 移除Spring Boot默认日志，修改mall的pom文件：

   ```xml
   <!--排除spring日志模块-->
   <exclusions>
       <exclusion>
       	<groupId>org.springframework.boot</groupId>
       	<artifactId>spring-boot-starter-logging</artifactId>
       </exclusion>
   </exclusions>
   ```

2. 添加log4j日志依赖在mall的pom文件中

   ```xml
   <!--引入日志依赖 抽象层 与 实现层-->
   <dependency>
       <groupId>org.slf4j</groupId>
       <artifactId>slf4j-api</artifactId>
       <version>1.7.21</version>
   </dependency>
   <dependency>
       <groupId>org.slf4j</groupId>
       <artifactId>slf4j-log4j12</artifactId>
       <version>1.7.21</version>
   </dependency>
   ```

3. 创建log4j.properties，放到mall-api的src/main/resources目录下

   ```properties
   log4j.rootLogger=DEBUG,stdout,file
   log4j.additivity.org.apache=true
   
   log4j.appender.stdout=org.apache.log4j.ConsoleAppender
   log4j.appender.stdout.threshold=debug
   log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
   log4j.appender.stdout.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n
   
   log4j.appender.file=org.apache.log4j.DailyRollingFileAppender
   log4j.appender.file.layout=org.apache.log4j.PatternLayout
   log4j.appender.file.DatePattern='.'yyyy-MM-dd-HH-mm
   log4j.appender.file.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n
   log4j.appender.file.Threshold=INFO
   log4j.appender.file.append=true
   log4j.appender.file.File=/workspaces/logs/mall-api/mall.log
   ```

#### 3.8 通过日志监控service执行时间

1. 在mall的pom文件中引入aop依赖：

   ```xml
   <dependency>
   	<groupId>org.springframework.boot</groupId>
   	<artifactId>spring-boot-starter-aop</artifactId>
   </dependency>
   ```

2. 编辑service监控的日志切面，参见cn.bravedawn.aspect.ServiceLogAspect

#### 3.9 开启Mybatis SQL日志打印设置

编辑mall-api中的application.yml文件：

```yaml
mybatis:
    configuration:
        log-impl: org.apache.ibatis.logging.stdout.StdOutImpl  # 开启Mybatis SQL日志打印设置

```

## 2.分类，推荐，搜索，评价，购物车开发

### 第1章 首页轮播图+分类功能实现

#### 1.1获取商品子分类-自定义mapper

1. 编写获取商品子分类SQL

   ```sql
   SELECT
       f.id as id,
       f.`name` as `name`,
       f.type as type,
       f.father_id as fatherId,
       c.id as subId,
       c.`name` as subName,
       c.type as subType,
       c.father_id as subFatherId
   FROM
       category f
   LEFT JOIN
   	category c
   on
   	f.id = c.father_id
   WHERE
   	f.father_id = #{rootCatId}
   ```

2. 编写MyBatis相关文件

   * 在mall-pojo项目中，编写cn.bravedawn.vo.CategoryVO和cn.bravedawn.vo.SubCategoryVO。

   * 在mall-mapper项目中，编写cn.bravedawn.mapper.CategoryMapperCustom，为什么不在原来的CategoryMapper文件中写呢，我觉得作者主要是为了将自定义的mapper内容与框架所包含的内容区别开来，别名重命名的问题。

   * **在mall-mapper项目中，编写mapper/CategoryMapperCustom.xml**

     编写resultMap、collection，该部分主要涉及了自定义对象与Mybatis xml文件如何协作工作。

### 第2章 商品推荐+搜索功能实现

#### 2.1 商品推荐（查询每个一级分类下的最新6条商品数据）-自定义mapper

1. 编写查询每个一级分类下的最新6条商品数据SQL，这里需要查询3张表做联合查询：

   ```sql
   SELECT
   	f.id as rootCatId,
   	f.`name` as rootCatName,
   	f.slogan as slogan,
   	f.cat_image as catImage,
   	f.bg_color as bgColor,
   	i.id as itemId,
   	i.item_name as itemName,
   	ii.url as itemUrl,
   	i.created_time as createdTime
   FROM
   	category f
   LEFT JOIN items i ON f.id = i.root_cat_id
   LEFT JOIN items_img ii ON i.id = ii.item_id
   WHERE
   	f.type = 1
   AND
   	i.root_cat_id = 1
   AND
   	ii.is_main = 1
   ORDER BY
   	i.created_time DESC
   LIMIT 0,6
   ```

2. 在mall-mapper项目中，编写Mapper文件，这里涉及到了Mybatis xml文件中**parameterType="Map"**属性的相关写法。编辑cn.bravedawn.mapper.CategoryMapperCustom文件：

   ```java
   List<NewItemsVO> getSixNewItemsLazy(@Param("paramsMap") Map<String, Object> map);
   ```

3. 在mall-mapper项目中，编辑mapper/CategoryMapperCustom.xml文件：

   ```xml
   <select id="getSixNewItemsLazy" resultMap="myNewItemsVO" parameterType="Map">
   SELECT
       f.id as rootCatId,
       f.`name` as rootCatName,
       f.slogan as slogan,
       f.cat_image as catImage,
       f.bg_color as bgColor,
       i.id as itemId,
       i.item_name as itemName,
       ii.url as itemUrl,
       i.created_time as createdTime
   FROM
       category f
   LEFT JOIN items i ON f.id = i.root_cat_id
   LEFT JOIN items_img ii ON i.id = ii.item_id
   WHERE
       f.type = 1
   AND
       i.root_cat_id = #{paramsMap.rootCatId}
   AND
       ii.is_main = 1
   ORDER BY
       i.created_time
   DESC
   LIMIT 0,6
   </select>
   ```

4. 然后再在mall-service，mall-api中编写相关接口。

### 第3章 商品评价功能开发

#### 3.1 商品评价-编写自定义mapper查询

1. 在mall-mapper项目中src/main/resources/mapper/ItemsMapperCustom.xml。直接看mapper文件的编写

   ```xml
   <select id="queryItemComments" parameterType="Map" resultType="cn.bravedawn.pojo.vo.ItemCommentVO">
           SELECT
           ic.comment_level as commentLevel,
           ic.content as content,
           ic.sepc_name as specName,
           ic.created_time as createdTime,
           u.face as userFace,
           u.nickname as nickname
           FROM
           items_comments ic
           LEFT JOIN
           users u
           ON
           ic.user_id = u.id
           WHERE
           ic.item_id = #{paramsMap.itemId}
           <if test=" paramsMap.level != null and paramsMap.level != '' ">
               AND ic.comment_level = #{paramsMap.level}
           </if>
   </select>
   ```

   在上面的编写方法中并没有编写特定的resultMap，而是在resultType中直接写了参数路径。在后面的开发工作中，值得借鉴。

#### 3.2 商品评价-Spring Boot整合Mybatis-pagehelper

1. 在mall项目的pom文件中引入MyBatis-pageHelper的依赖

   ```xml
   <!--pagehelper -->
   <dependency>
       <groupId>com.github.pagehelper</groupId>
       <artifactId>pagehelper-spring-boot-starter</artifactId>
       <version>1.2.12</version>
   </dependency>
   ```

2. 在mall-api的application.yml文件中配置

   ```yaml
   # 分页插件配置
   pagehelper:
       helperDialect: mysql                   # 配置方言
       supportMethodsArguments: true          # 分页是否支持传递方法参数
   ```

3. 使用分页插件，在查询前使用分页插件。原理：统一拦截SQL，为其通过分页功能

   ```java
   /**
   * page：第几页
   * pageSize：每页显示的条数
   */
   PageHelper.startPage(page, pageSize);
   ```

4. 分装数据到PagedGridResult传给前端

   ```java
   PageInfo<?> pageList = new PageInfo<>(list);
   PagedGridResult grid = new PagedGridResult();
   grid.setPage(page);
   grid.setRows(list);
   grid.setTotal(pageList.getPages());
   grid.setRecords(pageList.getTotal());
   ```

#### 3.3 用户信息脱敏处理

代码详情参见mall-common的cn.bravedawn.utils.DesensitizationUtil。下面我详细的分析一下这个脱敏算法的内部逻辑：

这个代码我debug了一下，感觉不是很难。分别对脱敏字符串长度为1、2、3、7、其他这些进行了处理。

### 第4章 商品搜索功能开发

#### 4.1 商品搜索-自定义mapper

1. 在mall-mapper项目中，mapper/ItemsMapperCustom.xml:29中的sql中演示了Mybatis中的动态SQL功能：

   * if

   * choose

   * when

   * 在Mybatis中如果遇到单引号，该怎么写

     ```xml
     <!-- paramsMap.sort == 'c' -->
     <when test=" paramsMap.sort == &quot;c&quot; ">
     ```

2. 在mall-pojo项目中，在cn.bravedawn.vo.SearchItemsVO中定义的金额的计算方法：

   ```java
   // 这里需要说明的就是，在数据库的价格设计为int，业务端的类型也为int，除100的操作在前端进行
   private int price;
   ```

### 第5章 购物车功能开发

#### 5.1购物车的存储形式

1. Cookie
   * 无需登录、无需查库、保存在浏览器段
   * 优点：性能好、访问快，没有和数据库交互
   * 缺点1：换电脑购物车数据会丢失
   * 缺点2：电脑被其他人登录，隐私安全
2. Session
   * 用户登陆后，购物车数据放入用户会话
   * 优点：初期性能较好，访问快
   * 缺点1：session基于内存，用户量庞大影响服务器性能
   * 缺点2：只能存在于当前会话，不适用集群与分布式系统
3. 数据库
   * 用户登录后，购物车数据存入数据库
   * 优点：数据持久化，可以在任何地点任何时间访问
   * 缺点：频繁读写数据库，造成数据库压力
4. Redis
   * 用户登录后，购物车数据存入redis缓存
   * 优点1：数据持久化，可在任何地点任何时间访问
   * 优点2：频繁的读写只基于缓存，不会造成数据库压力
   * 优点3：适用于集群和分布式系统，可扩展性强

## 3.地址，订单，支付，定时任务开发

### 第2章 确认订单功能开发

#### 1.订单流程梳理与订单状态

* 订单状态流转

  ![](E:\markdown笔记\笔记图片\20\1\13.png)

* 复杂订单状态设计

  ![](E:\markdown笔记\笔记图片\20\1\12.png)

* 聚合支付中心

  ![](E:\markdown笔记\笔记图片\20\1\14.png)

