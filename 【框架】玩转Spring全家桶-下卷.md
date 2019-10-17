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

  * 在浏览器中输入http://localhost:8080/login，输入在sba-server-demo的appliation.properties中配置的用户名和密码。就可以登录到Spring admin的主页，如下图：

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

通过java提供的keytool生成证书。

![](E:\markdown笔记\笔记图片\13-2\36.png)

![37](E:\markdown笔记\笔记图片\13-2\37.png)

#### 1.项目代码，配置容器支持https

本节有ssl-waiter-service和ssl-customer-service两个程序，这个采用new module的方式将这两个项目都引入IDEA中。

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

  