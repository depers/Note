# 玩转Spring全家桶-下卷——极客时间

## 第十章：运行中的Spring Boot

### 75.认识 Spring Boot 的各类 Actuator Endpoint

![](E:\markdown笔记\笔记图片\13-2\1.png)

![2](E:\markdown笔记\笔记图片\13-2\2.png)

![3](E:\markdown笔记\笔记图片\13-2\3.png)

![4](E:\markdown笔记\笔记图片\13-2\4.png)

![5](E:\markdown笔记\笔记图片\13-2\5.png)

访问查看方式：

* 浏览器
* JConsole

### 76.动手定制自己的 Health Indicator

![](E:\markdown笔记\笔记图片\13-2\6.png)

上图是应用程序的健康状态与其对应的http状态码。如果返回503说明系统的健康状态就是不好的。

![](E:\markdown笔记\笔记图片\13-2\7.png)

![8](E:\markdown笔记\笔记图片\13-2\8.png)

#### 1.Spring Boot自带的Health Indicator的源码分析

* org.springframework.boot.actuate.jdbc.DataSourceHealthIndicator
* org.springframework.boot.actuate.system.DiskSpaceHealthIndicator

#### 2.自定义Health Indicator

![](E:\markdown笔记\笔记图片\13-2\9.png)

项目代码：

* 地址： [https://github.com/depers/geektime-spring-code/tree/master/Chapter%2010/indicator-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 10/indicator-demo) 

* 项目代码

  * 修改了pom文件

    ```xml
    <dependency>
    	<groupId>org.springframework.boot</groupId>
    	<artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    
    <dependency>
    	<groupId>io.micrometer</groupId>
    	<artifactId>micrometer-registry-prometheus</artifactId>
    </dependency>
    ```

    其中 [Prometheus](https://prometheus.io/docs/introduction/overview/)是最近几年开始流行的一个新兴监控告警工具 。

  * 修改application.properties

    ```properties
    management.endpoints.web.exposure.include=*
    management.endpoint.health.show-details=always
    
    info.app.author=DigitalSonic
    info.app.encoding=@project.build.sourceEncoding@
    ```

  * 编写了geektime.spring.springbucks.waiter.support.CoffeeIndicator

  * 启动项目，访问  http://localhost:8080/actuator/health 

    ![](E:\markdown笔记\笔记图片\13-2\10.png)

    上图所示，coffeeIndicator的status是UP。我们在data.sql中修改sql为：

    ```
    select 1
    ```

    此时，再访问  http://localhost:8080/actuator/health 

    ![](E:\markdown笔记\笔记图片\13-2\11.png)

    此时，coffeeIndicator的status是DOWN，整个health的status也是DOWN，说明系统出现了问题。

### 77.通过 Micrometer 获取运行数据

![](E:\markdown笔记\笔记图片\13-2\12.png)

Micrometer为最流行的监视系统提供了一个基于仪表客户端，使您无需供应商锁定即可仪表化基于JVM的应用程序代码。 考虑使用SLF4J，但是是度量界。（slf4j是日志界的门面，Micrometer是度量界的门面）

官网： http://micrometer.io/ 

![](E:\markdown笔记\笔记图片\13-2\13.png)

![](E:\markdown笔记\笔记图片\13-2\14.png)

![15](E:\markdown笔记\笔记图片\13-2\15.png)

![](E:\markdown笔记\笔记图片\13-2\16.png)



其中具体的配置信息可以参考： https://docs.spring.io/spring-boot/docs/2.2.0.RELEASE/reference/html/production-ready-features.html#production-ready-metrics 

![](E:\markdown笔记\笔记图片\13-2\17.png)

![18](E:\markdown笔记\笔记图片\13-2\18.png)

#### 1.自定义度量指标

* 项目代码地址： [https://github.com/depers/geektime-spring-code/tree/master/Chapter%2010/metrics-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 10/metrics-demo) 

* 编写了geektime.spring.springbucks.waiter.service.CoffeeOrderService#bindTo

* 然后启动项目，访问metrics： http://localhost:8080/actuator/metrics/order.count 

  ![](E:\markdown笔记\笔记图片\13-2\20.png)

* 访问prometheus： http://localhost:8080/actuator/prometheus 

  ![](E:\markdown笔记\笔记图片\13-2\19.png)

### 78.通过 Spring Boot Admin 了解程序的运行状态

![](E:\markdown笔记\笔记图片\13-2\21.png)

![22](E:\markdown笔记\笔记图片\13-2\22.png)

![23](E:\markdown笔记\笔记图片\13-2\23.png)

![24](E:\markdown笔记\笔记图片\13-2\24.png)

#### 1.配置spring-boot-admin-server和spring-boot-admin-client

这里需要将sba-server-demo和sba-client-demo以module的形式放在idea的统一目录下面。

* sba-server-demo项目代码

  1. 编辑pom文件

     ```xml
     <dependency>
         <groupId>org.springframework.boot</groupId>
     	<artifactId>spring-boot-starter-security</artifactId>
     </dependency>
     <dependency>
         <groupId>org.springframework.boot</groupId>
     	<artifactId>spring-boot-starter-web</artifactId>
     </dependency>
     <dependency>
         <groupId>de.codecentric</groupId>
     	<artifactId>spring-boot-admin-starter-server</artifactId>
     </dependency>
     ```

  2. 编辑application.properties

     ```properties
     spring.application.name=sba-server
     server.port=8080
     
     spring.security.user.name=geektime
     spring.security.user.password=sba-server-password
     ```

  3. 在geektime.spring.boot.sba.SbaServerApplication进行安全配置

  4. 代码地址： [https://github.com/depers/geektime-spring-code/tree/master/Chapter%2010/sba-server-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 10/sba-server-demo) 

* sba-client-demo项目

  1. 编辑pom文件

     ```xml
     <dependency>
     	<groupId>org.springframework.boot</groupId>
     	<artifactId>spring-boot-starter-actuator</artifactId>
     </dependency>
     <dependency>
     	<groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-security</artifactId>
     </dependency>
     <dependency>
     	<groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter-web</artifactId>
     </dependency>
     <dependency>
     	<groupId>de.codecentric</groupId>
     	<artifactId>spring-boot-admin-starter-client</artifactId>
     </dependency>
     ```

  2. 配置application.properties

  3. 代码地址： [https://github.com/depers/geektime-spring-code/tree/master/Chapter%2010/sba-client-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 10/sba-client-demo) 

* 分别启动这两个项目

  * 在浏览器中输入http://localhost:8080/login，输入在sba-server-demo的appliation.properties中配置的用户名和密码。老师也说了，并不支持在产线上面使用spring boot admin，应该把监控的这部分工作和自己的运维监控系统结合起来。就可以登录到Spring admin的主页，如下图：

    ![](E:\markdown笔记\笔记图片\13-2\25.png)
### 79.如何定制 Web 容器的运行参数

![](E:\markdown笔记\笔记图片\13-2\26.png)

#### 1.通过配置的方式修改容器的配置

  ![27](E:\markdown笔记\笔记图片\13-2\27.png)

  ![28](E:\markdown笔记\笔记图片\13-2\28.png)

  ![29](E:\markdown笔记\笔记图片\13-2\29.png)

#### 2.通过编程的方式修改容器的配置

![](E:\markdown笔记\笔记图片\13-2\30.png)

#### 3.项目代码

* 地址： [https://github.com/depers/geektime-spring-code/tree/master/Chapter%2010/tomcat-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 10/tomcat-demo) 

* 配置方式：

  1. application.properties中的配置

     ```
     # 开启压缩
     server.compression.enabled=true
     # 压缩的最小大小为512b
     server.compression.min-response-size=512B
     
     # 显示exception的信息
     ## 是否包含异常的堆栈信息，默认是NEVER，其他ALWAYS，ON_TRACE_PARAM
     server.error.include-stacktrace=always
     ## 当服务器错误是，是否显示错误信息，默认为true
     server.error.include-exception=true
     ```

  2. 注释geektime.spring.springbucks.waiter.WaiterServiceApplication#customize方法

  3. 然后启动项目

  4. 下图是使用crul命令测试配置效果的截图，该效果演示了application.properties配置的前两行配置

     * 第一个红圈：正常的请求
     * 第二个红圈：使用gzip压缩的方式返回数据
     * 第三个红圈：Accept-Encoding 和Content-Encoding是HTTP中用来对采用哪种编码格式传输正文进行协定的一对头部字段
     * 第四个红圈：是压缩的内容
       ![](E:\markdown笔记\笔记图片\13-2\32.png)
     
  5. 下图是演示开启打印异常的配置的效果，该效果演示了application.properties配置的后两行配置
     ![](E:\markdown笔记\笔记图片\13-2\33.png)
* 编码配置方式：

  1. 在geektime.spring.springbucks.waiter.WaiterServiceApplication里面编写
  
     ```java
     @Override
     public void customize(TomcatServletWebServerFactory factory) {
     	Compression compression = new Compression();
     	compression.setEnabled(true);
     	compression.setMinResponseSize(DataSize.ofBytes(512));
     	factory.setCompression(compression);
     }
     ```
  
  2. 然后对application.properties中的配置进行注释
  
     ```properties
     # 开启压缩
     #server.compression.enabled=true
     ## 压缩的最小大小为512b
     #server.compression.min-response-size=512B
     ```
  
  3. 通过代码和上面的properties的配置效果是一样的

### 80.如何配置容器支持 HTTP 2（上）

![](E:\markdown笔记\笔记图片\13-2\34.png)

![35](E:\markdown笔记\笔记图片\13-2\35.png)

通过java提供的keytool生成证书。命令：`keytool -genkey -alias springbucks -storetype PKCS12 -keyalg RSA -keysize 2048 -keystore springbucks.p12 -validity 365`![](E:\markdown笔记\笔记图片\13-2\36.png)

![37](E:\markdown笔记\笔记图片\13-2\37.png)

#### 1.项目代码，配置容器支持https

本节有ssl-waiter-service和ssl-customer-service两个程序，这个采用new module的方式将这两个项目都引入IDEA中。

* 项目地址：
  *  [https://github.com/depers/geektime-spring-code/tree/master/Chapter%2010/ssl-waiter-service](https://github.com/depers/geektime-spring-code/tree/master/Chapter 10/ssl-waiter-service) 
  *  [https://github.com/depers/geektime-spring-code/tree/master/Chapter%2010/ssl-customer-service](https://github.com/depers/geektime-spring-code/tree/master/Chapter 10/ssl-customer-service) 

* ssl-waiter-service项目

  1. 编辑application.properties

     ```
     server.port=8443
     server.ssl.key-store=classpath:springbucks.p12
     server.ssl.key-store-type=PKCS12
     server.ssl.key-store-password=spring
     ```

  2. 启动项目，在terminal中输入命令`curl -k -v https://localhost:8443/coffee/1`，就可以顺利的访问这个链接。

     * curl命令简单介绍
       * -k：发送https请求的时候，不做认证（当用https请求出错的时候，可以试下加-k）
       * -v：看到详细的请求头中的信息

     ![](E:\markdown笔记\笔记图片\13-2\38.png)

* ssl-customer-service项目

  1. 编辑application.properties

     ```properties
     waiter.service.url=https://localhost:8443
     
     security.key-store=classpath:springbucks.p12
     security.key-pass=spring
     ```

  2. 编辑org.springframework.http.client.HttpComponentsClientHttpRequestFactory

     * SSLContextBuilder构造sslContext
     * NoopHostnameVerifier: 这个主机名验证器基本上就是把主机名验证关闭了,它接受任何有效的SSL会话来匹配目标主机。
     * 配置HttpComponentsClientHttpRequestFactory

  3. 编辑geektime.spring.springbucks.customer.CustomerRunner#url

  4. 然后启动项目就可以看到效果了

### 81.如何配置容器支持 HTTP 2（下）

![](E:\markdown笔记\笔记图片\13-2\39.png)

Spring Boot不支持明文的http/2，只支持https的http/2。

![40](E:\markdown笔记\笔记图片\13-2\40.png)

RestTemplate支持的http库中，目前只有OkHttp是支持Http/2的。所以在本节的演示代码中，使用了okhttp来代替之前的Apache HttpComponent。

#### 1.代码演示，配置容器支持http/2

本节有http2-waiter-service和http2-customer-service两个程序，这个采用new module的方式将这两个项目都引入IDEA中。

* 代码地址：
  *  [https://github.com/depers/geektime-spring-code/tree/master/Chapter%2010/http2-waiter-service](https://github.com/depers/geektime-spring-code/tree/master/Chapter 10/http2-waiter-service) 
  *  [https://github.com/depers/geektime-spring-code/tree/master/Chapter%2010/http2-customer-service](https://github.com/depers/geektime-spring-code/tree/master/Chapter 10/http2-customer-service) 

* http2-waiter-service项目

  1. 配置application.properties

     ```properties
     // 开启spring对http2的支持
     server.http2.enabled=true
     ```

  2. 启动项目，在terminal中输入`curl -k -I https://localhost:8443/coffee/1`，`-I`参数的意思是只显示响应报文首部信息。

     ![](E:\markdown笔记\笔记图片\13-2\44.png)
  
  3. 也可以输入`curl -k -I -v https://localhost:8443/coffee/1`查看更为详细的信息。
  
* http2-customer-service项目
  
  1. 编辑org.springframework.http.client.ClientHttpRequestFactory，将httpComponent换成OKhttp。
  2. 启动项目，效果与上一节相同。
  
### 82.如何编写命令行运行的程序

  ![](E:\markdown笔记\笔记图片\13-2\41.png)

![](E:\markdown笔记\笔记图片\13-2\42.png)

![43](E:\markdown笔记\笔记图片\13-2\43.png)

其中ApplicationRunner与CommandLineRunner的作用是一样的，都是在SpringApplication的run方法之前执行一段代码，不同的是他们的参数不同。

#### 1.源码分析

* org.springframework.boot.SpringApplication#exit
* org.springframework.boot.ExitCodeGenerators#getExitCode

#### 2.项目代码

1. 项目地址： [https://github.com/depers/geektime-spring-code/tree/master/Chapter%2010/command-line-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 10/command-line-demo) 

2. 项目注意点：

   * 故意在pom文件中引入web依赖

     ```xml
     <dependency>
     	<groupId>org.springframework.boot</groupId>
     	<artifactId>spring-boot-starter-web</artifactId>
     </dependency>
     ```

   * 编辑application.properties

     ```properties
     spring.main.web-application-type=none
     ```

   * 编辑main方法

     ```java
     public static void main(String[] args) {
     	// 法一：如果 application.properties中没有spring.main.web-application-type=none，则需这样配置
     	// new SpringApplicationBuilder(CommandLineApplication.class)
     	//				.web(WebApplicationType.NONE)
         //				.run(args);
     	// 法二：根据 application.properties 里的配置来决定 WebApplicationType
     	SpringApplication.run(CommandLineApplication.class, args);
     }
     ```

   * 编辑返回码：geektime.spring.hello.MyExitCodeGenerator

   * 编辑geektime.spring.hello.FooCommandLineRunner

   * 编辑ApplicationRunner

     * geektime.spring.hello.BarApplicationRunner
     * geektime.spring.hello.ExitApplicationRunner

3. 启动项目

   ![](E:\markdown笔记\笔记图片\13-2\45.png)

### 83.了解可执行 Jar 背后的秘密

![](E:\markdown笔记\笔记图片\13-2\46.png)

![](E:\markdown笔记\笔记图片\13-2\47.png)

上图是可执行jar的结构图。

![48](E:\markdown笔记\笔记图片\13-2\48.png)

![](E:\markdown笔记\笔记图片\13-2\49.png)

其中`<executable>true</executable>`参数是直接可执行jar的关键配置，通过同名.conf文件可以配置一些参数，在启动项目时spring会将.conf文件的配置读取，添加到启动命令后面。

![50](E:\markdown笔记\笔记图片\13-2\50.png)

#### 1.项目代码

* 项目地址： [https://github.com/depers/geektime-spring-code/tree/master/Chapter%2010/jar-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 10/jar-demo) 

![51](E:\markdown笔记\笔记图片\13-2\51.png)

在上图中我们使用`unzip -l waiter-service.jar`可以看到jar文件有一个头文件。其中`unzip -l`意思是显示压缩文件内所包含的文件；

然后，使用`less`命令查看这个头文件，可以看到在头文件中读取了同名.conf文件，然后在java 命令后面这些参数加了进去。

![](E:\markdown笔记\笔记图片\13-2\52.png)

![53](E:\markdown笔记\笔记图片\13-2\53.png)

然后我们查看.conf文件：

![](E:\markdown笔记\笔记图片\13-2\54.png)

然后运行jar，通过`ps ux|grep java`查看刚才java项目的启动信息，发现.conf的文件信息已经添加到了启动命令里面：

![](E:\markdown笔记\笔记图片\13-2\55.png)

#### 疑问：

视频中，说这个黑魔法的实现是因为利用了**zip文件是从后往前读，而shell文件是从前往后读**。这句话我还是不太理解。

### 84.如何将 Spring Boot 应用打包成 Docker 镜像文件

![](E:\markdown笔记\笔记图片\13-2\56.png)

![57](E:\markdown笔记\笔记图片\13-2\57.png)

![58](E:\markdown笔记\笔记图片\13-2\58.png)

![59](E:\markdown笔记\笔记图片\13-2\59.png)

#### 1.项目代码

* 项目地址：
* 注意点：
  1. 编辑pom文件，添加dockerfile-maven-plugin
  2. 编辑Dockerfile文件
  3. 编译项目，命令：`mvn clean package -Dmaven.test.skip=true`，执行构建
  4. `docker images`，就可以看到springbucks的镜像了
  5. `docker run --name waiter-service -d -p 8080:8080 springbucks/waiter-service:0.0.1-SNAPSHOT`，启动镜像
  6. `docker log waiter-service`，查看日志
  7. 启动后，就可以使用命令：`curl http://localhost:8080/coffee/1`测试了

### 85.SpringBucks 实战项目进度小结

![](E:\markdown笔记\笔记图片\13-2\61.png)

![](E:\markdown笔记\笔记图片\13-2\61.png)

## 第十一章：Spring Cloud及Cloud Native概述

### 86.简单理解微服务

![](E:\markdown笔记\笔记图片\13-2\62.png)

![63](E:\markdown笔记\笔记图片\13-2\63.png)

![64](E:\markdown笔记\笔记图片\13-2\64.png)

![65](E:\markdown笔记\笔记图片\13-2\65.png)

### 87.如何理解云原生(Cloud Native)

![](E:\markdown笔记\笔记图片\13-2\66.png)

pivotal认为云原生应具备下面四个条件：

![67](E:\markdown笔记\笔记图片\13-2\67.png)

![](E:\markdown笔记\笔记图片\13-2\68.png)

![69](E:\markdown笔记\笔记图片\13-2\69.png)

CNCF官网： https://www.cncf.io/ 

### 88.12-Factor App（上）

![](E:\markdown笔记\笔记图片\13-2\70.png)

官网： https://12factor.net/zh_cn/ 

![71](E:\markdown笔记\笔记图片\13-2\71.png)

![72](E:\markdown笔记\笔记图片\13-2\72.png)

![72](E:\markdown笔记\笔记图片\13-2\73.png)

### 89.12-Factor App（下）

本节就是针对上一节中的十二条中的几条最佳实践来做展开说明：

![](E:\markdown笔记\笔记图片\13-2\74.png)

![75](E:\markdown笔记\笔记图片\13-2\75.png)

![76](E:\markdown笔记\笔记图片\13-2\76.png)

![77](E:\markdown笔记\笔记图片\13-2\77.png)

![78](E:\markdown笔记\笔记图片\13-2\78.png)

![79](E:\markdown笔记\笔记图片\13-2\79.png)

### 90.认识Spring Cloud的组成部分

![](E:\markdown笔记\笔记图片\13-2\80.png)

![81](E:\markdown笔记\笔记图片\13-2\81.png)

![82](E:\markdown笔记\笔记图片\13-2\82.png)

![83](E:\markdown笔记\笔记图片\13-2\83.png)

## 第十二章：服务注册与发现

### 91.使用Eureka作为服务注册中心

![](E:\markdown笔记\笔记图片\13-2\84.png)

其中，老师不建议在非AWS服务器上使用Eureka，Eureka在阿里云上的实现的版本也已经不再开源了。

![85](E:\markdown笔记\笔记图片\13-2\85.png)

![86](E:\markdown笔记\笔记图片\13-2\86.png)

其中@EnableDiscoveryClient与@EnableEurekaClient的作用是一样，demo项目中我们使用了@EnableDiscoveryClient。

![87](E:\markdown笔记\笔记图片\13-2\87.png)

#### 1.项目代码

本节有eureka-server和eureka-waiter-service两个程序，这个采用new module的方式将这两个项目都引入IDEA中。

* 项目地址：

  *  [https://github.com/depers/geektime-spring-code/tree/master/Chapter%2012/eureka-server](https://github.com/depers/geektime-spring-code/tree/master/Chapter 12/eureka-server) 
  *  [https://github.com/depers/geektime-spring-code/tree/master/Chapter%2012/eureka-waiter-service](https://github.com/depers/geektime-spring-code/tree/master/Chapter 12/eureka-waiter-service) 

* eureka-server项目

  1. 编辑pom文件，引入相关依赖

  2. 编辑application.properties

     ```properties
     # 配置注册中心端口
     server.port=8761
     
     # Eureka是为注册中心,是否需要将自己注册到注册中心上
     eureka.client.register-with-eureka=false
     # Erueka是为注册中心，不需要检索服务信息
     eureka.client.fetch-registry=false
     ```

  3. 在geektime.spring.cloud.eureka.EurekaServerApplication中添加`@EnableEurekaServer`

  4. 启动项目，访问http://localhost:8761

* eureka-waiter-service项目

  1. 编辑pom文件，引入相关依赖

  2. 配置bootstrap.properties

     ```properties
     # 声明服务名
     spring.application.name=waiter-service
     ```

  3. 配置application.properties

     ```properties
     # 配置服务端口为0，表示随机就好
     server.port=0
     ```

  4. 在geektime.spring.springbucks.waiter.WaiterServiceApplication中添加`@EnableDiscoveryClient`

  5. 启动项目，效果如下：

     ![](E:\markdown笔记\笔记图片\13-2\88.png)

### 92.使用Spring Cloud Loadbalancer访问服务

![](E:\markdown笔记\笔记图片\13-2\89.png)

![90](E:\markdown笔记\笔记图片\13-2\90.png)

上图中我们可以通过添加一个@LoadBalaced的注解来为RestTemplate与WebClient提供负载均衡的支持。

#### 1.源码分析

* org.springframework.cloud.client.loadbalancer.LoadBalancerInterceptor#intercept
* org.springframework.cloud.client.loadbalancer.LoadBalancerAutoConfiguration
* org.springframework.cloud.netflix.ribbon.RibbonLoadBalancerClient#execute
* org.springframework.cloud.netflix.ribbon.RibbonAutoConfiguration

#### 2.项目代码

本节有ribbon-customer-service、eureka-server和eureka-waiter-service三个程序，这个采用new module的方式将这两个项目都引入IDEA中。其中eureka-server和eureka-waiter-service是ribbon-customer-service依赖的项目，在启动ribbon-customer-service应该先启动这两个项目。

* 项目地址： [https://github.com/depers/geektime-spring-code/tree/master/Chapter%2012/ribbon-customer-service](https://github.com/depers/geektime-spring-code/tree/master/Chapter 12/ribbon-customer-service) 

* ribbon-customer-service项目

  1. 编辑pom文，引入依赖

     ```xml
     <dependency>
     	<groupId>org.springframework.cloud</groupId>
     	<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
     </dependency>
     ```

  2. 配置bootstrap.properties

     ```properties
     spring.application.name=customer-service
     ```

  3. 编辑application.properties

     ```properties
     # 配置服务端口为0，表示随机就好
     server.port=0
     ```

  4. 在geektime.spring.springbucks.customer.CustomerServiceApplication中添加`@EnableDiscoveryClient`

  5. 在geektime.spring.springbucks.customer.CustomerServiceApplication#restTemplate添加`@LoadBalanced`，说明我要使用loadBalanced的相关功能的。

  6. 添加了geektime.spring.springbucks.customer.CustomerRunner#showServiceInstances，读取服务实例

  7. 修改readMenu()、 orderCoffee()、queryOrder()方法，url变为http://waiter-service/开头

  8. 修改geektime.spring.springbucks.customer.CustomerServiceApplication#main方法，将该项目变为web项目，注册到服务

     ```java
     public static void main(String[] args) {
     	SpringApplication.run(CustomerServiceApplication.class, args);
     }
     ```

  9. 启动项目，可以在控制台看到服务实例信息的打印

     ![](E:\markdown笔记\笔记图片\13-2\92.png)
     
  9. 还有新的服务被发现
  
  ![](E:\markdown笔记\笔记图片\13-2\91.png)
  
### 93.使用Feign访问服务

![](E:\markdown笔记\笔记图片\13-2\93.png)

![94](E:\markdown笔记\笔记图片\13-2\94.png)

![95](E:\markdown笔记\笔记图片\13-2\95.png)

![96](E:\markdown笔记\笔记图片\13-2\96.png)

#### 1.项目代码

* 项目地址： [https://github.com/depers/geektime-spring-code/tree/master/Chapter%2012/feign-customer-service](https://github.com/depers/geektime-spring-code/tree/master/Chapter 12/feign-customer-service) 

* feign-customer-service项目

  1. 编辑pom文件，引入相关依赖

     ```xml
     <dependency>
     	<groupId>org.springframework.cloud</groupId>
     	<artifactId>spring-cloud-starter-openfeign</artifactId>
     </dependency>
     <dependency>
     	<groupId>io.github.openfeign</groupId>
     	<artifactId>feign-httpclient</artifactId>
     </dependency>
     ```

  2. 编辑bootstrap.properties

     ```properties
     spring.application.name=customer-service
     ```

  3. 编辑application.properties

     ```properties
     # 端口号随机
     server.port=0
     
     # 连接超时
     feign.client.config.default.connect-timeout=500
     # 读超时
     feign.client.config.default.read-timeout=500
     ```

  4. 编写geektime.spring.springbucks.customer.integration

     * CoffeeOrderService

     * CoffeeService

     * @FeignClient(name = "waiter-service", contextId = "coffee", path = "/coffee")

       * name：服务名，用于请求url
       * contextId：标识FeignClient并作以区分
       * path：为请求的路径，注意不要在接口上加@RequestMapping
     
4. 编辑geektime.spring.springbucks.customer.CustomerRunner，通过调用FeignClient实现
  
  6. 在geektime.spring.springbucks.customer.CustomerServiceApplication中添加@EnableDiscoveryClient和@EnableFeignClients
  
  7. 编辑org.apache.http.impl.client.CloseableHttpClient
  
  8. 启动项目，还有新的服务注册
  
### 94.深入理解服务发现背后的DiscoveryClient

![](E:\markdown笔记\笔记图片\13-2\97.png)

![98](E:\markdown笔记\笔记图片\13-2\98.png)

#### 1.源码分析

* ServiceRegistry
  * org.springframework.cloud.netflix.eureka.serviceregistry.EurekaServiceRegistry#register
  * org.springframework.cloud.netflix.eureka.serviceregistry.EurekaServiceRegistry#deregister
* EurekaAutoServiceRegistration
  * org.springframework.cloud.netflix.eureka.serviceregistry.EurekaAutoServiceRegistration#start
  * org.springframework.cloud.netflix.eureka.serviceregistry.EurekaAutoServiceRegistration#stop()
* DiscoveryClient
  * org.springframework.cloud.client.discovery.DiscoveryClient
  * org.springframework.cloud.netflix.eureka.EurekaDiscoveryClient

如果后期自己想阅读源码的话，可以先从spring-cloud-commons入手。

### 95.使用Zookeeper作为服务注册中心

![](E:\markdown笔记\笔记图片\13-2\99.png)

![100](E:\markdown笔记\笔记图片\13-2\100.png)

![](E:\markdown笔记\笔记图片\13-2\101.png)

如果要在多个机房里面使用zookeeper做服务的发现和注册的话，zookeeper带来的脑裂问题还是比较严重的。老师建议应该在同一个机房或是可溶区里面使用一组zookeeper集群。

![102](E:\markdown笔记\笔记图片\13-2\102.png)

运行zookeeper的命令：

```
# 启动zookeeper
docker start zookeeper

# 查看zookeeper日志
docker logs zookeeper

# 运行zookeeper bash
docker exec -it zookeeper bash

# 运行zookeeper cli
./bin/zkCli.sh

# 查看节点信息
ls /services/waiter-service/a5db66f9-fa16-41ee-b435-3838190e36ff
get /services/waiter-service/a5db66f9-fa16-41ee-b435-3838190e36ff
```

#### 1.项目代码

本节zk-waiter-service和zk-customer-service两个程序，这个采用new module的方式将这两个项目都引入IDEA中。

* 项目地址：

  *  [https://github.com/depers/geektime-spring-code/tree/master/Chapter%2012/zk-waiter-service](https://github.com/depers/geektime-spring-code/tree/master/Chapter 12/zk-waiter-service) 
  *  [https://github.com/depers/geektime-spring-code/tree/master/Chapter%2012/zk-customer-service](https://github.com/depers/geektime-spring-code/tree/master/Chapter 12/zk-customer-service) 

* zk-waiter-service项目

  1. 编辑pom文件

     ```xml
     <dependency>
     	<groupId>org.springframework.cloud</groupId>
     	<artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
     </dependency>
     ```

  2. 修改application.properties

     ```properties
     spring.cloud.zookeeper.connect-string=192.168.156.128:2181
     ```

* zk-customer-service项目

  1. 编辑pom文件

     ```xml
     <dependency>
     	<groupId>org.springframework.cloud</groupId>
     	<artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
     </dependency>
     ```

  2. 修改application.properties

     ```properties
     spring.cloud.zookeeper.connect-string=192.168.156.128:2181
     ```

  3. 分别启动zk-waiter-service和zk-customer-service两个程序，使用zookeeper的命令查看服务信息。

### 96.使用Consul作为服务注册中心

![](E:\markdown笔记\笔记图片\13-2\103.png)

![](E:\markdown笔记\笔记图片\13-2\104.png)

![105](E:\markdown笔记\笔记图片\13-2\105.png)

![106](E:\markdown笔记\笔记图片\13-2\106.png)

![107](E:\markdown笔记\笔记图片\13-2\107.png)

consul的常用命令，这里我们使用8600端口绑定udp：

```
docker run --name consul -d -p 8500:8500 -p 8600:8600/udp consul
```

#### 1.项目代码

本节consul-waiter-service和consul-customer-service两个程序，这个采用new module的方式将这两个项目都引入IDEA中。

* 项目地址：

  *  [https://github.com/depers/geektime-spring-code/tree/master/Chapter%2012/consul-waiter-service](https://github.com/depers/geektime-spring-code/tree/master/Chapter 12/consul-waiter-service) 
  *  [https://github.com/depers/geektime-spring-code/tree/master/Chapter%2012/consul-customer-service](https://github.com/depers/geektime-spring-code/tree/master/Chapter 12/consul-customer-service) 

* consul-waiter-service项目

  1. 编辑pom文件

     ```xml
     <dependency>
     	<groupId>org.springframework.cloud</groupId>
     	<artifactId>spring-cloud-starter-consul-discovery</artifactId>
     </dependency>
     ```

  2. 编辑application.properties

     ```properties
     spring.cloud.consul.host=localhost
     spring.cloud.consul.port=8500
     # 表示注册时使用IP而不是hostname
     spring.cloud.consul.discovery.prefer-ip-address=true
     ```

  3. 启动项目，访问 http://192.168.156.128:8500/ui/dc1/nodes/df21f9834706 就可以看到该节点的相关信息。

  4. dig 命令主要用来从 DNS 域名服务器查询主机地址信息。通过DNS的方式获取waiter-service这个命令要在linux中输，通过8600端口来解析waiter-service.service.consul这个域名：

     ![](E:\markdown笔记\笔记图片\13-2\108.png)

* consul-customer-service项目

  1. 编辑pom文件

     ```xml
     <dependency>
     	<groupId>org.springframework.cloud</groupId>
     	<artifactId>spring-cloud-starter-consul-discovery</artifactId>
     </dependency>
     ```

  2. 修改application.properties

     ```properties
     spring.cloud.consul.host=192.168.156.128
     spring.cloud.consul.port=8500
     spring.cloud.consul.discovery.prefer-ip-address=true
     ```

  3. 启动项目，访问 http://192.168.156.128:8500/ui/dc1/services ，就可以看到效果了

     ![](E:\markdown笔记\笔记图片\13-2\109.png)

### 97.使用Nacos作为服务注册中心

![](E:\markdown笔记\笔记图片\13-2\110.png)

![111](E:\markdown笔记\笔记图片\13-2\111.png)

![112](E:\markdown笔记\笔记图片\13-2\112.png)

![113](E:\markdown笔记\笔记图片\13-2\113.png)

nacos命令：

```
docker run --name nacos -d -p 8848:8848 -e MODE=standalone nacos/nacos-server
```

然后在浏览器里面访问： http://192.168.156.128:8848/nacos/#/ 

#### 1.项目代码

本节nacos-consul-waiter-service和nacos-customer-service两个程序，这个采用new module的方式将这两个项目都引入IDEA中。

* 项目地址：
  *  [https://github.com/depers/geektime-spring-code/tree/master/Chapter%2012/nacos-waiter-service](https://github.com/depers/geektime-spring-code/tree/master/Chapter 12/nacos-waiter-service) 
  *  [https://github.com/depers/geektime-spring-code/tree/master/Chapter%2012/nacos-customer-service](https://github.com/depers/geektime-spring-code/tree/master/Chapter 12/nacos-customer-service) 

* nacos-consul-waiter-service项目

  1. 编辑pom文件

    ```xml
    <dependency>
    	<groupId>org.springframework.cloud</groupId>
    	<artifactId>spring-cloud-alibaba-dependencies</artifactId>
    	<version>${spring-cloud-alibaba.version}</version>
    	<type>pom</type>
    	<scope>import</scope>
    </dependency>
    ```

    ```xml
    <dependency>
    	<groupId>org.springframework.cloud</groupId>
    	<artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    ```
    
  2. 编辑application.properties

     ```properties
     spring.cloud.nacos.discovery.server-addr=127.0.0.1:8848
     ```

  3. 启动项目，效果如下图：

     ![](E:\markdown笔记\笔记图片\13-2\114.png)

* nacos-customer-service项目

  1. 该项目的pom配置和application.properties是相同的。
  
  2. 启动项目，效果如下图：
  
     ![](E:\markdown笔记\笔记图片\13-2\115.png)

#### 2.源码解析——服务的注册与发现

* spring-cloud-alibaba-nacos-discovery-0.2.1.RELEASE.jar!/META-INF/spring.factories
* org.springframework.cloud.alibaba.nacos.NacosDiscoveryAutoConfiguration
* org.springframework.cloud.alibaba.nacos.registry.NacosServiceRegistry
* org.springframework.cloud.alibaba.nacos.NacosDiscoveryClient

### 98.如何定制自己的DiscoveryClient

![](E:\markdown笔记\笔记图片\13-2\116.png)

![117](E:\markdown笔记\笔记图片\13-2\117.png)

![118](E:\markdown笔记\笔记图片\13-2\118.png)

#### 1.具体实现

* 项目地址： [https://github.com/depers/geektime-spring-code/tree/master/Chapter%2012/fixed-discovery-client-demo](https://github.com/depers/geektime-spring-code/tree/master/Chapter 12/fixed-discovery-client-demo) 

1. 编辑pom文件

   ```xml
   <dependency>
   	<groupId>org.springframework.cloud</groupId>
   	<artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
   </dependency>
   ```

2. 实现自己的DiscoveryClient：geektime.spring.springbucks.customer.support.FixedDiscoveryClient

3. 实现ServerList接口：geektime.spring.springbucks.customer.support.FixedServerList

4. 编辑geektime.spring.springbucks.customer.CustomerServiceApplication

   ```java
   @Bean
   public DiscoveryClient fixedDiscoveryClient() {
   	return new FixedDiscoveryClient();
   }
   
   @Bean
   public FixedServerList fixedServerList() {
   	return new FixedServerList();
   }
   ```

5. 通过new moudle方式，导入chapter 6的springbucks项目，然后启动它。

6. 接着再启动fixed-discovery-client-demo

   ![](E:\markdown笔记\笔记图片\13-2\119.png)

### 99.SpringBucks实战项目进度小结

![](E:\markdown笔记\笔记图片\13-2\120.png)

![121](E:\markdown笔记\笔记图片\13-2\121.png)

