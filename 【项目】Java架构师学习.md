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

<img src="E:\markdown笔记\笔记图片\20\1\1\4.png" style="zoom:70%;" />

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

![](E:\markdown笔记\笔记图片\20\1\1\5.jpg)

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

   







