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

    

  

