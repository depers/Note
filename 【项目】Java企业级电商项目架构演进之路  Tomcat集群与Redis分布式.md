typora-root-url: 笔记图片

# Java企业级电商项目架构演进之路

## Lombok

### 1.Lombok介绍

* [Lombok官网](https://projectlombok.org/)
* 通过简单的注解来精简代码达到消除冗长代码的目的。

### 2.Lombok的优点

* 精简代码，消除冗长代码
* 避免修改字段名字时完成修改方法名

**注意**：IDE必须要安装Lombox插件

### 3.Lombok原理

![4-1](E:/markdown笔记/笔记图片/4-1.png)

![4-2](E:/markdown笔记/笔记图片/4-2.png)

Java的编译器对源代码进行分析，生成抽象语法树AST。然后Lombox Annotation Processor对AST进行处理。例如：我们在一个类中声明了Data的注解，Lombox Annotation Handler就会找到Data注解所在的类的AST，然后修改语法树，添加了get和set方法的相应的树节点。之后再对语法树进行分析，生成字节码文件。

### 4.Lombok引入项目

* maven引入：

  ```xml
    <dependency>
              <groupId>org.projectlombok</groupId>
              <artifactId>lombok</artifactId>
              <version>1.16.18</version>
          </dependency>
  ```

### 5.安装Lombox插件

* IDEA
   ![4-4](E:/markdown笔记/笔记图片/4-4.png)

   ![4-3](E:/markdown笔记/笔记图片/4-3.png)

* Eclipse

   * 下载[lombok.jar](https://projectlombok.org/download)
   * 双击运行lombok.jar

### 6.Lombok实战

* @Data——包含了@Getter，@Setter，@ToString，@EqualsAndHashCode
* @Getter
* @Getter(AccessLevel.PROTECTED)
* @Setter
* @Setter(AccessLevel.PROTECTED)
* @NoArgsConstructor——无参构造器
* @AllArgsConstructor——全参构造器
* @ToString
  * @ToString(exclude="column")——exclude是排除column
  * @ToString(ecxlude = {"column", "column"})
  * @ToString(of = "column")——of是指定column
  * @ToString(of = {"column", "column"})
* @EqualsAndHashCode——重写equals和hashCode方法
  * @EqualsAndHashCode(exclude="column")
  * 其他方法同@ToString
* @Slf4j(当项目使用logback日志框架时使用)
* @Log4j(当项目使用log4j日志框架的时使用)

### 7.反编译大法

> 通过Java Decompiler验证Class文件。其中编译的命令是`mvn clean package -Dmaven.test.skip=true`,找到target/classes下的文件进行查看。

* [Java Decompiler](http://jd.benow.ca/)

* JD-GUI

  JD-GUI本机安装地址：D:\Program Files\java_test\jd-gui-windows-1.4.0

* JD-Eclipse

* JD-IntelliJ

### 8.Lombok实际使用时需注意的点

* 在类需要序列化、反序列化时详细的控制字段时，要注意是否使用Lombok。例如：Jackson json序列化
* Lombok降低了源码的可阅读性
* 使用@Slf4j还是@Log4j看项目使用的日志框架。
* 选择合适的地方使用Lombok，例如POJO是个好地方，POJO很单纯。

## Maven环境隔离

### 1.实际的项目环境

* 本地开发环境（Local)
* 开发环境（Dev)
* 测试环境（Beta）
* 线上环境（Prod)

### 2.隔离环境之间各种配置存在的差异

* FTP服务器相关的配置不一样
* 数据库配置不一样
* ...

### 3.Maven环境隔离解决的问题

* 避免人工修改的弊端，即容易犯错
* 轻松分环境编译、打包、部署

### 4.Maven环境隔离配置及原理

* pom.xml中build节点添加

  ```xml
          <resources>
              <resource>
                  <directory>src/main/resources.${deploy.type}</directory>
                  <excludes>
                      <exclude>*.jsp</exclude>  <!--排除*.jsp文件-->
                  </excludes>
              </resource>
              <resource>
                  <directory>src/main/resources</directory> <!--公用的文件-->
              </resource>
          </resources>
  ```

* pom.xml中添加profiles节点

  ```xml
      <profiles>
          <profile>
              <id>dev</id>
              <activation>
                  <activeByDefault>true</activeByDefault> <!--当编译时没有指定环境时，他作为默认环境-->
              </activation>
              <properties>
                  <deploy.type>dev</deploy.type>  <!--对应于上面定义的${deploy.type}-->
              </properties>
          </profile>
  
          <profile>
              <id>beta</id>
              <properties>
                  <deploy.type>beta</deploy.type>
              </properties>
          </profile>
  
          <profile>
              <id>prod</id>
              <properties>
                  <deploy.type>prod</deploy.type>
              </properties>
          </profile>
      </profiles>
  ```

### 5.Maven环境隔离目录初始化

新建对应的文件夹，并把要隔离的文件分开，公共的留下

![4-6](E:/markdown笔记/笔记图片/4-6.png)

### 6.Maven环境隔离IDEA中设置默认环境

* 在IDEA右侧Maven Projects，刷新，选中（单选）本地开发环境对应的环境，点击import change进行更新。

![4-5](E:/markdown笔记/笔记图片/4-5.png)

* 下图中，dev前的对勾建议只选一个实心的，他表示在IDEA中发布时所用的环境。

  ![4-7](E:\markdown笔记\笔记图片\4-7.png)

### 7.Maven环境隔离编译打包命令

* 命令是：`mvn clean package -Dmaven.test.skip=true -Pdev`

* 参数是：-P${环境标识}，分别有`dev`、`beta`、`prod`

### 8.Maven环境隔离验证

`E:\Dome\Java EE\fmall>mvn clean package -Dmaven.test.skip=true -Pdev`

## Tomcat集群

### 1.一期Tomcat配置简要回顾

* Mac/Linux
* Windows

### 2.一期Nginx配置简要回顾

具体配置tianxianyigou.bravedawn.cn.conf：

```nginx
server {
        listen 80;
        autoindex on;
        server_name 39.106.115.25 tianxianyigou.bravedawn.cn;
        access_log /usr/local/nginx/logs/access.log combined;
        index index.html index.htm index.jsp index.php;
        #root /devsoft/apache-tomcat-7.0.73/webapps/mmall;
        #error_page 404 /404.html;
        if ( $query_string ~* ".*[\;'\<\>].*" ){
                return 404;
        }
        location / {
                proxy_pass http://127.0.0.1:8080/;
                add_header Access-Control-Allow-Origin '*';
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header REMOTE-HOST $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                client_max_body_size 33m;
        }
}
```

### 3.Tomcat集群能带来什么？

* 提高服务的性能，并发能力，以及高可用性

  一台机器配置一个Tomcat，提高服务的性能；

  多台Tomcat机器就可以提高并发能力；

  Nginx下挂了多台Tomcat服务器，一台宕机，nginx就可以摘掉这个服务器，选择新的可用的服务器，从而提高可用性；

* 提供项目架构的横向扩展能力

  通过升级一台机器的内存，CPU和硬盘，是纵向扩展，但成本指数上升；

  通过添加Tomcat服务器节点，从而达到横向扩展的能力；

### 4.Tomcat集群实现原理

通过Nginx负载均衡进行请求转发

### 5.Tomcat集群新旧架构对比解析

* 一期架构

  ![4-8](E:\markdown笔记\笔记图片\4-8.png)

* 二期架构

  * “想当然“二期架构
     ![4-9](E:\markdown笔记\笔记图片\4-9.png)

  * 真二期架构

     ![4-10](E:\markdown笔记\笔记图片\4-10.png)
### 6.Tomcat集群带来了什么问题？

* Session登录信息存储及读取的问题

  解决方法：

  * 采用nginx ip hash policy

    **优点：**可以不改变现有技术架构，直接实现横向扩展（省事）

    **缺点**：1.导致服务器请求（负载）不平均，完全依赖ip hash 的结果。2.在IP变化的环境下无法服务。

  ​	

* 服务器定时任务并发的问题

* ...

### 7.Tomcat单机部署多应用/多机部署多应用

* Tomcat单机部署多应用-Mac/Linux（具体步骤如下）

  * 修改`/etc/profile`增加tomcat环境变量，修改完成后，保存退出。执行：`source /etc/profile`使配置文件生效，可以使用命令`echo $CATALINA_2_BASE`，看是否有环境变量的输出。

    ```
    export CATALINA_BASE = /Users/imooc/tomcat1
    export CATALINA_HOME = /Users/imooc/tomcat1
    export TOMCAT_HOME = /Users/imooc/tomcat1
    export CATALINA_2_BASE = /Users/imooc/tomcat2
    export CATALINA_2_HOME = /Users/imooc/tomcat2
    export TOMCAT_2_HOME = /Users/imooc/tomcat2
    ```

  * 第一个tomcat不变

  * 打开第二个tomcat目录bin下catalina.sh，即${tomcat}/bin/catalina.sh

  * 找到`# OS specific support. $var _must_ be set to either true or false.`，在这行下面编辑，新增配置，保存退出：

    ```
    export CATALINA_BASE = $CATALINA_2_BASE
    export CATALINA_HOME = $CATALINA_2_HOME
    ```

  * 打开第二个tomcat的conf目录下的server.xml进行修改，即：${tomcat}/conf/server.xml。**注意：此处有三个端口需要修改。**

    * Server port节点端口号修改

      ![4-11](E:\markdown笔记\笔记图片\4-11.png)

      **注意：**8005是默认的第一个tomcat的8005，第二个tomcat要修改成9005或者其他的端口号，但是在多个tomcat之间一定不能重复。

    * Tomcat的访问端口Connector port="8080"节点端口号修改

      ![4-12](E:\markdown笔记\笔记图片\4-12.png)

    * Connector port="8009" protocol=...节点端口号修改

      ![4-13](E:\markdown笔记\笔记图片\4-13.png)

   * 分别进入两个tomcat的bin目录，启动tomcat，即进入${tomcat}/bin/，执行start.sh

   * 检查两个tomcat的启动日志

      ![4-14](E:\markdown笔记\笔记图片\4-14.png)

   * 访问[http://localhost:8080]()，[http://localhost:9080]()可以打开tomcat部署的webapps的ROOT项目首页

   * 如果想部署多个Tomcat实例，请依照此法

      **注意：**端口号在系统中必须不能重复，必须是系统没有使用的

* Tomcat单机部署多应用-Windows（具体步骤如下）

  * 配置环境变量

    ![4-15](E:\markdown笔记\笔记图片\4-15.png)

  * 添加新的Tomcat相关的**系统变量**

    ```
    export CATALINA_BASE = C:\tomcat1
    export CATALINA_HOME = C:\tomcat1
    export TOMCAT_HOME = C:\tomcat1
    export CATALINA_2_BASE = C:\tomcat2
    export CATALINA_2_HOME = C:\tomcat2
    export TOMCAT_2_HOME = C:\tomcat2
    ```

  * 第一个tomcat不变

  * 打开第二个tomcat目录bin下catalina.bat，即${tomcat}/bin/catalina.bat

  * 打开第二个tomcat目录bin下startup.bat，即${tomcat}/bin/startup.bat

  * 在第二个tomcat中的catalina.bat，startup.bat进行操作，替换这两个文件中的：

    `CATALINA_BASE`替换为`CATALINA_2_BASE`,`CATALINA_HOME`替换为`CATALINA_2_HOME`

  * 打开第二个tomcat的conf目录下的server.xml进行修改，即：${tomcat}/conf/server.xml。**注意：此处有三个端口需要修改。**

    - Server port节点端口号修改

      ![4-11](E:\markdown笔记\笔记图片\4-11.png)

      **注意：**8005是默认的第一个tomcat的8005，第二个tomcat要修改成9005或者其他的端口号，但是在多个tomcat之间一定不能重复。

    - Tomcat的访问端口Connector port="8080"节点端口号修改

      ![4-12](E:\markdown笔记\笔记图片\4-12.png)

    - Connector port="8009" protocol=...节点端口号修改

      ![4-13](E:\markdown笔记\笔记图片\4-13.png)

  * cmd分别进入两个tomcat的bin目录，启动tomcat，即进入${tomcat}/bin/，执行`startup.bat`

  * 检查tomcat启动的日志

    ![4-16](E:\markdown笔记\笔记图片\4-16.png)

  * 访问[http://localhost:8080]()，[http://localhost:9080]()可以打开tomcat部署的webapps的ROOT项目首页

  * 如果想部署多个Tomcat实例，请依照此法

    **注意：**端口号在系统中必须不能重复，必须是系统没有使用的


​	

* Tomcat多机部署多应用

  * 必须在每台机器上安装一个Tomcat即可
  * 如果一个机器部署一个tomcat实例，不用修改任何配置文件。如果一个机器部署多个tomcat实例，请依照单机部署多实例的方法

  **注意：**多个服务器并且每个服务器只安装一个Tomcat要保证他们之间的网络是互通的，方可集群。Nginx装在任一一台服务器上即可，也可单独把Nginx服务独立出来一台，nginx和tomcat之间也要保证网络互通。

### 8.Nginx负载均衡配置、策略、场景及特点

* 轮循（默认）

  > 就是Nginx将请求轮流分发给各个服务器。

  * 优点：实现简单

  * 缺点：不考虑每台服务器处理能力

  * 配置：

    ```nginx
    upstream www.happymmall.com{
        server www.happymmall.com:8080;  
        server www.happymmall.com:9080;
    }
    ```

* 权重

  > 在多服务器集群中，将更多的请求分发给配置和性能较高的服务器。

  * 优点：考虑了每台服务器处理能力的不同

  * 配置：

    ```nginx
    upstream www.happymmall.com{
        server www.happymmall.com:8080 weight=15;
        server www.happymmall.com:9080 weight=10;
    }
    ```

  * 注意：在本课程中采用权重配置负载均衡，weight默认是1。如果多个配置权重的节点，比较相对值，例如上面的配置一个是15，一个是10只代表请求访问8080的概率是访问8090的1.5倍。

* ip hash

  > 将请求的ip做一次散列，然后根据hash值将请求分到相应的服务器上。

  * 优点：能实现同一用户访问同一个服务器

  * 缺点：根据ip hash不一定平均，有可能多个ip只访问同一个服务器节点

  * 配置：

    ```nginx
    upstream www.happymmall.com{
    	ip_hash;
    	
        server www.happymmall.com:8080; 
        server www.happymmall.com:9080;
    }
    ```

* url hash（第三方插件）

  > 根据访问的url进行hash，相同的url只会访问同一个服务器。

  * 优点：能实现同一个服务访问同一个服务器

  * 缺点：根据url hash分配请求会不平均，请求频繁的url会请求道同一个服务器上

  * 配置：

    ```nginx
    upstream www.happymmall.com{
        server www.happymmall.com:8080;   
        server www.happymmall.com:9080;
        
        hash $request_uri;
    }
    ```

* fair（第三方差将）

  > 按后端服务器的响应时间来分配请求，响应时间短的优先分配。

  * 配置:

    ```nginx
    upstream www.happymmall.com{
        server www.happymmall.com:8080;   
        server www.happymmall.com:9080;
        
        fair;
    }
    ```

    ​	

* 负载均衡参数讲解（扩展知识点）

  ![4-17](E:\markdown笔记\笔记图片\4-17.png)

### 9.Nginx+Tomcat搭建集群

* Mac/Linux

  * 启动两个Tomcat

  * 执行`startup.sh`

    ![4-18](E:\markdown笔记\笔记图片\4-18.png)

  * 替换一个Tomcat2的Logo，即tomcat.png(方便验证)

    * 路径是：${tomcat2}/webapps/ROOT/tomcat.png
    * 注：如果该tomcat部署过项目则找不到此文件，需重新安装Tomcat。
    * 这里给一个测试图片：[慕课网logo](http://www.imooc.com/static/img/index/logo_new.png)

  * 如果没有购买配置域名的话，可修改系统host(浏览器所在机器的host)，`sudo vim /etc/hosts`,添加下面一行：

    ```
    127.0.0.1 www.happymmall.com
    ```

  * Ping host配置的域名，验证host生效，打开终端：`ping www.happymmall.com`

    ![4-19](E:\markdown笔记\笔记图片\4-19.png)

  * 访问两个tomcat的首页进行验证(8080端口和8090端口)

    [http://localhoust:8080]()

    [http://www.happymmall.com:8080]()

    [http://localhoust:9080]()

    [http://www.happymmall.com:9080]()

  * 启动Nginx，`/usr/local/nginx/sbin/nginx.sh`

  * **注意：**Nginx服务默认端口是本机80端口（不要修改），如果安装了IIS，请停止服务，保证80端口可用。

  * 访问[http://localhost]()或[http://www.imooc.com]()

  * 编辑在${nginx}/conf/nginx.conf配置文件，在http节点下面增加`include vhost/*.conf;`目的是为了要增加Tomcat集群的负载均衡配置，并且把域名配置文件分开，方便后期管理。

  * 在${nginx}/conf目录下创建文件夹**vhost**，在vhost目录下新增**www.happymall.com.conf**配置文件。内容如下：

    ```nginx
    upstream www.happymmall.com{ 
    	server 127.0.0.1:8080 weight=1; 
    	server 127.0.0.1:8081 weight=1;
    
    	# 配置多个服务器
    	# server 192.168.1.1:8080;
    	# server 192.168.1.2:8080; 
    } 
    server { 
    	listen 80; 
    	autoindex on; 
    	server_name www.happymmall.com happymmall.com; 
    	access_log /usr/local/nginx/logs/access.log combined; 
    	index index.html index.htm index.jsp index.php; 
    	if ( $query_string ~* ".*[\;'\<\>].*" ){ 
    		return 404; 
    	} 
    
    	location / {
    		proxy_pass http://www.happymmall.com/;
    		add_header Access-Control-Allow-Origin *;
    	} 
    }
    ```

    截图：

    ![4-21](E:\markdown笔记\笔记图片\4-21.png)

  * 重新加载nginx的配置，在${nginx}/sbin下，执行`sudo ./nginx -s reload`

  * 打开浏览器输入[http://www.happymmall.com]()，狂刷新，观察页面的区别（替换的图片），可以观察到一会访问的是8080的tomcat，一回事9080的tomcat。

* Windows

  * 启动两个Tomcat

  * 执行`startup.bat`

  * 替换一个Tomcat2的Logo，即tomcat.png(方便验证)

    - 路径是：${tomcat2}/webapps/ROOT/tomcat.png
    - 注：如果该tomcat部署过项目则找不到此文件，需重新安装Tomcat。
    - 这里给一个测试图片：[慕课网logo](http://www.imooc.com/static/img/index/logo_new.png)

  * 如果没有购买配置域名的话，可修改系统host(浏览器所在机器的host)，在C:\Windows\System32\drivers\etc\hosts，增加一行：

    ```host
    127.0.0.1 www.happymmall.com
    ```

  * Ping host配置的域名，验证host生效，打开终端：`ping www.happymmall.com`

    ![4-20](E:\markdown笔记\笔记图片\4-20.png)

  * 访问两个tomcat的首页进行验证(8080端口和8090端口)

    [http://localhoust:8080]()

    [http://www.happymmall.com:8080]()

    [http://localhoust:9080]()

    [http://www.happymmall.com:9080]()

  * 启动Nginx，用cmd`${tomcat}/nginx.exe`

  * **注意：**Nginx服务默认端口是本机80端口（不要修改），如果安装了IIS，请停止服务，保证80端口可用。

  * 访问[http://localhost]()或[http://www.imooc.com]()

  *  编辑在${nginx}/conf/nginx.conf配置文件，在http节点下面增加`include vhost/*.conf;`目的是为了要增加Tomcat集群的负载均衡配置，并且把域名配置文件分开，方便后期管理。在${nginx}/conf目录下创建文件夹**vhost**，在vhost目录下新增**www.happymall.com.conf**配置文件。内容如下： 

    ```nginx
    upstream www.happymmall.com{ 
    	server 127.0.0.1:8080; 
    	server 127.0.0.1:9080;
    
    	# 配置多个服务器
    	# server 192.168.1.1:8080;
    	# server 192.168.1.2:8080; 
    } 
    server { 
    	listen 80; 
    	autoindex on; 
    	server_name www.happymmall.com happymmall.com; 
    	access_log c:/access.log combined; 
    	index index.html index.htm index.jsp index.php; 
    	if ( $query_string ~* ".*[\;'\<\>].*" ){ 
    		return 404; 
    	} 
    
    	location / {
    		proxy_pass http://www.happymmall.com;
    		add_header Access-Control-Allow-Origin *;
    	} 
    }
    ```

    截图：

    ![4-22](E:\markdown笔记\笔记图片\4-22.png)

  * 重新加载nginx配置，之前启动的nginx服务窗口不变，Windows比较特殊，如果关闭之前的窗口，nginx服务就会停止。所以需要重新打开cmd窗口，进入${nginx}，执行`nginx.exe -s reload`
  * 打开浏览器输入[http://www.happymmall.com]()，狂刷新，观察页面的区别（替换的图片），可以观察到一会访问的是8080的tomcat，一回事9080的tomcat。

## Redis基础强化

### 1.Redis简介

* Redis——REmote DIctionary Server
* Redis是一个使用ANSI C语言编写的开源数据库
* 高性能的Key-Value数据库
* **内存数据库，支持数据持久化**
* 官网
  * [Redis官网](https://redis.io/)
  * [Redis中国官网](https://redis.cn/)
* Redis支持多语言的客户端
* 从2010年起，Redis的开发工作由VMware
* 从2013年开始，Redis的开发由Pivotal赞助

### 2.Redis常用数据类型

> Redis Object信息主要有：数据类型(type)，编码方式(encoding)，数据指针(ptr)，虚拟内存(vm)，其他信息

![4-23](E:\markdown笔记\笔记图片\4-23.png)

* String（字符串）
* list（链表）
* set（无序集合）
* sorted set（有序集合）
* hash（Hash集合）

### 3.Redis开发语言客户端介绍

客户端的下载：[http://redis.cn/clients.html](http://redis.cn/clients.html)

### 4.课程Redis版本

[redis-2.8.0](http://download.redis.io/releases/)

### 5.Redis安装（Windows/Linux/Mac）

* Windows版本的Redis
  * Redis-x64-2.8.2402
  * 由Microsoft Open Tech group维护
  * [https://github.com/MicrosoftArchive/redis](https://github.com/MicrosoftArchive/redis)
* Linux安装Redis
  * 相关命令：
    * `tar -zxvf redis-2.8.0.tar.gz`
    * 进入解压后的redis目录输入：`make`
    * 测试redis的安装是否成功：`make test`
    * 进入redis目录的src文件夹，执行`./redis-server`，但这种启动方式会占用命令行。如果想不占用命令行，应执行`./redis-server &`
    * 通过`./redis-cli`查看redis安装是否是成功的。
* Windows安装Redis
  * 注意：在Windows上需要编译redis
  * 在redis目录下直接执行`redis-server.exe`，然后在redis的根目录下执行`redis-cli`

### 6.Redis单实例配置

* redis.conf配置文件
* port端口
* requirepass密码
* masterauth主从同步中在slave配置master密码

### 7.Redis单实例服务端、客户端、启动及关闭

* redis单实例服务端启动
  * `redis-server`
  * `redis-server ${redis.conf}` 指定配置文件（推荐）
  * `redis-server --port ${port}` `redis-server --port 6380` 指定端口
* redis单实例客户端的启动
  * `redis-cli`
  * `redis-cli -p ${port}` 指定端口，默认是3679
  * `redis-cli -h ${ip}` 指定ip
  * `redis-cli -a ${password}` 指定auth认证密码
  * `redis-cli -p ${port} -h ${ip} -a ${password}`组合使用
* redis单实例服务端及客户端关闭，redis在`shutdown`的时候会进行持久化，执行：
  * `redis-cli shutdown`
  * `redis-cli -p ${port} shutdown`
  * `redis-cli -h ${port} shutdown`
  * `redis-cli -p ${port} -h ${ip} shutdown`

### 8.Redis单实例环境验证（在redis-cli中执行以下操作）

* ping，此时会返回`PONG`
* 执行redis set命令相关
* 查看redis get命令获取到的值

### 9.Redis基础命令

* info——查看系统信息

  其中比较重要的信息就是Keyspace:

  ```
  # Keyspace
  db0:keys=1,expires=0,avg_ttl=0
  ```

  db0是默认的第一个database，可以在redis.conf中设置database的数量，通过`select <dbid>`来选择使用的database。如果使用db0的话，输入`select 0`即可。

* ping——测试redis-cli连接是否ok

* quit——退出redis-cli连接

* save——人工对redis进行持久化

* dbsize——当前database的key的数量

* select——选择切换的database

* flushdb——清除当前的database的key

* flushall——清除所有的database的key

* monitor——查看redis的日志

### 10.[Redis键命令](http://redisdoc.com/index.html)

* `set`——设置键值对，`set key value`

* `setnx`——`setnx newkey newvalue`如果返回值为0说明键值对newkey已存在，如果返回1则说明键值对newkey已经设置成功

* `strlen`——字符串的长度，`strlen key`返回key对应值的长度

* `get`——取键对应的值，`get a`，如果键已过其生存时间，则返回`(nil)`

* `del`——删除键值对，`del key`

* `keys`——查看所有的键值对的键

* `exists`——判断是否存在键值对，返回1是存在，返回0为不存在，`exists key`

* `expire`——`expire key 10`设置key的生存时间为10s

* `ttl`——( Time To Live)`ttl key`看一个key的剩余生存时间，当返回-1的时候，key是永久存在的；当返回-2是说明不存在

* `type`——`type key`查看一个key对应value的类型

* `randomkey`——返回随机key

* `hset`——`hset key field value`将哈希表 `key` 中的域 `field` 的值设为 `value` 

* `hexists`——`hexists map name`查看哈希表map的域name是否存在

* `hget`——`hget map name`取到哈希表map的域name的值，若果值不存在，则返回`(nil)`

* `hgetall`——`hgetall map`取出哈希表map的全部域和值

* `hkeys`——`hkeys map`取出哈希表map中的全部域

* `hvals`——`hvals map`取出哈希表map的全部值

* `hlen`——`hlen map`查看哈希表map的域的个数

* `hmget`——`hmget map field1 field2`获取哈希表map多个域的值

* `hmset`——`hmset map filed1 value1 field2 value2`设置哈希表map的多个域与值

* `hdel`——`hdel map field1 field2`删除哈希表map的多个域，返回值为删除域的个数

* `hsetnx`——`hsetnx map field1 value1`设置哈希表map的域与值，如果filed1已经存在，则返回0，表明设置失败；如果filed1不存在，则返回1设置成功

* `rename`——给key重命名，`rename lastkey newkey`

* `renamenx`——`renamenx a b`如果键b存在，则会返回0，修改失败；如果键a存在，则会返回1，键值对的键a被就改为b。

* `setex`——设置键值对的同时设置生存时间`setex a 100 tom`设置键a值tom键值对的生存时间为100s

* `psetex`——设置键值对的同时设置生存时间，只不过单位是ms毫秒，`psetex a 100 tom`设置键a值tom键值对的生存时间为100ms

* `getrange`——取键对应值的一定范围的值：

  ```
  127.0.0.1:6379> set word happy
  OK
  127.0.0.1:6379> keys *
  1) "a"
  2) "word"
  127.0.0.1:6379> get word
  "happy"
  127.0.0.1:6379> getrange word 0 2  取键word的值的0到2的值
  "hap"
  ```

* `getset`——先返回键当前对应的值，然后在设定新的值

  ```
  127.0.0.1:6379> get a
  "b"
  127.0.0.1:6379> set a a
  OK
  127.0.0.1:6379> get a
  "a"
  127.0.0.1:6379> getset a aaa
  "a"
  127.0.0.1:6379> get a
  "aaa"
  ```

* `mset`——设置多个键值对，`mset key1 value1 key2 value2 key3 value3`

* `msetnx`——设置多个键值对，`msetnx key1 value1 key2 value2`,只要有一个键值对已经存在，就不能存储，具有原子性

* `mget`——获取多个键的值，`get key1 key2 key3`

* `incr`——将值为数值类型的键，每次加1，`incr key`

* `incrby`——将值为数值类型的键，每次加一个步长，`incrby key 100`

* `decr`——将值为数值类型的键，每次减1，`decr key`

* `decrby`——将值为数值类型的键，每次减一个步长，`decrby key 100`

* `append`——在键对应值的后面拼接一个字符串，`append key wrod`

### 11.五种数据结构Redis命令快速上手

- String（字符串）
  - `set`
  - `setex`
  - `psetex`
  - `get`
  - `getrange`
  - `getset`
  - `mset`
  - `mget`
  - `setnx`
  - `strlen`
  - `incr`
  - `decr`
  - `incrby`
  - `decrby`
  - `append`
- list（链表）：里面是可以允许重复值的
  - `lpush`
  - `llen`
  - `lrange`
  - `lset`
  - `lindex`
  - `pop`
  - `lpop`
  - `rpop`
- set（无序集合）：里面不允许有重复值
  - `sadd`
  - `scard`
  - `smembers`
  - `sdiff`
  - `sinter`
  - `sunion`
  - `srandmember`
  - `sismenber`
  - `srem`
  - `spop`
- sorted set（有序集合）
  - `zadd`
  - `zcard`
  - `zscore`
  - `zcount`
  - `zrank`
  - `zincrby`
  - `zrange`
  - `withsorces`
- hash（Hash集合）
  - `hset`
  - `hexists`
  - `hget`
  - `hgetall`
  - `hkeys`
  - `hvals`
  - `hlen`
  - `hmget`
  - `hmset`
  - `hdel`
  - `hsetnx`

## Redis+Cookie+Jackson+Filter实现单点登录

### 1.用户模块一期回顾

* 用户登录
  * /user/login.do
* 用户登出
  * /user/logout.do
* 获取用户信息
  * /order/create.do
* 判断用户是否登录
  * /order/create.do

### 2.项目集成Redis客户端Jedis

* 引入jedis pom

### 3.Redis连接池构建及调试

* JedisPoolConfig源码解析

* JedisPool源码解析

* JedisPool回收资源

* 封装RedisPool

  详情请参考common目录下的**RedisPool.java**

### 4.Jedis API封装及调试

详情请参考util下的**RedisPoolUtil.java**

* 封装RedisPoolUtil
* 集成测试验证

### 5.Jackson封装JsonUtil及调试

详情请参考util下的**JsonUtil.java**

* Jackson封装JsonUtil及调试
* 多泛型序列化和反序列化

### 6.Jackson ObjectMapper源码封装

* Inclusion.ALWAYS

  将对象所有的字段都序列化。

* Inclusion.NON_NULL

  只有非空(NULL)的字段才会被序列化。

* Inclusion.NON_DEFAULT

  序列化的字段中不包含有默认值的字段，类似StringUtil.isEmpty()。

* Inclusion.NON_EMPTY

  序列化中不包含空字符串和空集合，类似StringUtil.isBlank()。

* SerializationConfig.Feature.WRITE_DATES_AS_TIMESTEMP

  若值设为true，则时间就会被序列化成时间戳(毫秒数)；若值设为false，则时间就是默认值(2019-01-20T12:07:49.608+0000)。

* SerializationConfig.Feature.FAIL_ON_EMPTY_BEANS

  若值设为false，则忽略空Bean转json的错误；若值为true，就会不会忽略Bean转json的错误。

* DeserializationConfig.Feature.FAIL_ON_UNKNOWN_PROPERTIES

  若值设为false，则忽略某些字段在json字符串中存在，但是在java对象中不存在对应属性的情况，防止错误；若值设为true，则会产生错误。

* ObjectMapper DateFormat

  所有的日期格式都统一为一个格式。

### 7.Cookie封装及使用

详情请参考util下的CookieUtil.java

* 写Cookie（新增、更新）

* 读Cookie

* 删Cookie

* domian

      //X.domain:".happyname.com"--a,b,c,d,e都可以拿到X的cookie
      //a:A.happymall.com                 cookie:domain=A.happymall.com; path="/"
      //b:B.happymall.com                 cookie:domain=B.happymall.com; path="/"
      //c:A.happymall.com/test/cc         cookie:domain=A.happymall.com; path="/test/cc"
      //d:A.happymall.com/test/bb         cookie:domain=A.happymall.com; path="/test/bb"
      //e:A.happymall.com/test            cookie:domain=A.happymall.com; path="/test"
      
      /**
       * a和b是同级域名两个人都拿不到对方的cookie
       * c和d可以共享a的，e的cookie
       * c拿不到d的，d也拿不到c的
       * c和d拿不到b的
       *
       * 总结：若A域名中包含B域名，那A就能拿到B域名的cookie;
       *      若A域名和B域名同级，则互相都拿不到对方的cookie;
       */

* path

  代表设置在根目录下，只有在该目录下的代码才能访问到cookie

* maxAge

  * maxAge单位是秒，如果这个maxage不设置的话，cookie就不会写入硬盘，而是写入内存，这样的话如果页面关了cookie也就消失了，只在当前的页面有效。
            

  * maxage如果设置为-1的话，代表永久

* httponly

  防止脚本攻击造成信息泄露风险

### 8.SessionExpireFilter重置session有效期

原本在redis存储的时间是30分钟，但是如果用户在网站上停留的时间超过了30分钟，这不乱套了吗。所以我们需要在用户使用网站的时候重置redis中session的存储时间。

详情代码的编写请参见：com.fmall.controller.common.SessionExpireFilter

### 9.用户session相关模块重置

就是将以下代码：

```java
User user = (User) session.getAttribute(Const.CURRENT_USER);
```

替换为：

```java
String loginToken = CookieUtil.readLoginToken(request);
if (StringUtils.isEmpty(loginToken)){
    return ServerResponse.createByErrorMessage("用户未登录，无法获取当前用户的信息。");
}

String userJsonStr = RedisPoolUtil.get(loginToken);
User currentUser = JsonUtil.string2Obj(userJsonStr, User.class);
```

* 用户登录重构
* 用户登出重构
* 获取用户信息重构
* 判断用户是否登录重构

### 10.Guava cache迁移Redis

* 目前Guava Cache的用法

  * 原来Guava的配置com.fmall.common.TokenCache

  * ```java
    TokenCache.setKey(TokenCache.TOKEN_PREFIX + username, forgeToken);
    ```

  * ```java
    String token = TokenCache.getKey(TokenCache.TOKEN_PREFIX + username);
    ```

* 集群后Guava Cache的不足

  Guava Cache会存储在Tomcat中，如果再集群环境里，在验证答案时TokenCache如果存储在tomcat1中，但是在修改密码的时候我们访问到tomcat2，就拿不到这个TokenCache。

* Guava Cache迁移Reids缓存

  * ```java
    RedisPoolUtil.setEx(Const.TOKEN_PREFIX + username, forgeToken, 60*60*12);
    ```

  * ```java
    String token = RedisPoolUtil.get(Const.TOKEN_PREFIX + username);
    ```

## Redis分布式

### 1.Redis分布式算法原理

#### (1)传统分布式算法

根据取余来决定将图片放在哪个节点上。

![4-24](E:\markdown笔记\笔记图片\4-24.png)

如果有4个Redis节点，20个数据。

![4-25](E:\markdown笔记\笔记图片\4-25.png)

现将这20个结点利用取余算法分配到指定的redis节点。

![4-26](E:\markdown笔记\笔记图片\4-26.png)

如果增加一个redis节点，到5个。

![4-27](E:\markdown笔记\笔记图片\4-27.png)

可以发现只有20,1,2,3这四个数据的位置没有变化。

![4-28](E:\markdown笔记\笔记图片\4-28.png)

结论：计算命中率。

![4-29](E:\markdown笔记\笔记图片\4-29.png)

![4-30](E:\markdown笔记\笔记图片\4-30.png)

#### (2)Consistent hashing一致性算法原理

![4-31](E:\markdown笔记\笔记图片\4-31.png)

我们将数轴卷起来构成圆环，他的取值范围是从0到2的32次减1。

![4-32](E:\markdown笔记\笔记图片\4-32.png)

![4-33](E:\markdown笔记\笔记图片\4-33.png)

![4-34](E:\markdown笔记\笔记图片\4-34.png)

![4-35](E:\markdown笔记\笔记图片\4-35.png)

在下图中我们将三个Cache也映射到hash空间中，其中数据对象object1沿着顺时针找到CaheA，映射到CacheA上；object2和object3映射到Cache映射到CacheC上；object4映射到CacheB上。这样四个数据对象就找到了所要存储的Cache节点。

![4-36](E:\markdown笔记\笔记图片\4-36.png)

增加和移除一个Cache节点，传统取余的算法直接将数据存储到服务器节点上，对服务器的冲击很大，很多缓存都没有命中。

移除CacheB

![4-37](E:\markdown笔记\笔记图片\4-37.png)

Object4就会存储到CacheC上。

![4-38](E:\markdown笔记\笔记图片\4-38.png)

移除CacheB后并不会影响object2，他影响的范围是沿着逆时针计算到上一个Cache节点的范围的数据，影响范围小，如下图透明红色框所示：

![4-39](E:\markdown笔记\笔记图片\4-39.png)

添加Cache

添加CacheD节点，object2就会存储到cacheD上，而他影响的范围是沿着逆时针计算到上一个Cache节点的范围的数据，影响范围小，如下图透明红色框所示：

![4-40](E:\markdown笔记\笔记图片\4-40.png)

#### (3)hash倾斜性

理想的Cache节点分布，分布是十分均匀的：

![4-41](E:\markdown笔记\笔记图片\4-41.png)

现实中呢，ABC挨得非常近，大量的数据节点会落在CacheA上，B和C上面不会有很多的数据存储在上面的：

![4-42](E:\markdown笔记\笔记图片\4-42.png)

在下图中1,2,3,4节点会落在CacheA上，5会落在CacheB上，CacheC上没有数据节点，从而导致Hash的倾斜性：

![4-43](E:\markdown笔记\笔记图片\4-43.png)

#### (4)虚拟节点

下图中，Object1和Object2经过Hash分别落到了虚拟节点V1和V5上，我们在对虚拟节点进行Hash，将V1和V2存储在实际节点N1...

![4-44](E:\markdown笔记\笔记图片\4-44.png)

在下图中蓝色的A，B，C是实际节点，淡蓝色的A，B，C是虚拟节点。这样的话数据节点的分布就会均匀一些，但是虚拟节点进行rehash时也存在Hash倾斜性的问题，只要我们将实际节点和虚拟节点的数量分配一个良好的比例，分布就会均匀一些，增加和减少一个节点的时候，产生的影响也会越来越多。

![4-45](E:\markdown笔记\笔记图片\4-45.png)

#### (5)Consistent hashing命中率

说明：服务器台数是n，新增服务器台数是m。

![4-46](E:\markdown笔记\笔记图片\4-46.png)

### 2.Redis分布式环境配置

![4-47](E:\markdown笔记\笔记图片\4-47.png)

### 3.Redis分布式服务端及客户端启动

* 修改两个redis的配置文件redis.conf

* 修改端口一个为6379，一个为6380

* 通过配置文件启动redis-server

* ```
  redis-server ${redis.conf}
  ```

### 4.封装分布式Shard Redis API

* ShardedJedis源码解析

  ![4-48](E:\markdown笔记\笔记图片\4-48.png)

* 封装RedisSharedPool

  详情请参见：

  com.fmall.common.RedisShardedPool

  com.fmall.util.RedisShardedPoolUtil

* 编写测试代码

* 集成测试验证

### 5.Redis分布式环境验证

* 查看reids-servre中的值

  ![4-49](E:\markdown笔记\笔记图片\4-49.png)

### 6.集群和分布式概念讲解

* 集群：是物理形态，多个tomcat，多个redis都是集群，但是他们的工作方式不同。一个任务由10个子任务组成，一个子任务单独执行1个小时。十台服务器，每台安装一个tomcat。有10个大任务同时到达，十台服务器同时工作，十个小时后十个大任务完成。整体来看一个小时完成一个任务，每个tomcat完成一个任务。

* 分布式：是工作方式，一个任务由10个子任务组成，一个子任务单独执行1个小时，将这10个组任务分给10台服务器，每台服务器一个子任务执行一小时。一个小时之后10个子任务执行完成，这个大任务也执行完成。整体来看10个小时候后，也完成10个任务。所以我们称多台redis为分布式。
## Spring Session
### 1.Spring Session简介

* Spring Session是Spring的项目之一
* Spring Session提供了一套创建和管理Servlet HttpSession的方案
* Spring Session提供了集群 Session（Clustered Sessions）
* 默认采用外置的Redis来存储Session数据，以此来解决Session共享的问题
* **要实现单点登录且对业务没有侵入的Session框架，但是他的劣势就是他对redis没有分片的操作**

### 2.Spring Session官方地址、文档、源码

* [官网](https://spring.io/projects/spring-session)
* [官方文档](https://docs.spring.io/spring-session/docs/2.1.3.BUILD-SNAPSHOT/reference/html5/)
* [Github地址](https://github.com/spring-projects/spring-session)
* [Spring Session1.2.x官方Demo](https://github.com/spring-projects/spring-session/tree/1.2.x/samples)

### 3.Sprint Session项目集成

* 引入Spring Session POM
* 在applicationContext-spring-session.xml配置：
  * 配置JedisConnectFactory
  * 配置RedisHttpSessionConfiguration
  * 配置DefaultCookieSerializer
  * 配置JdeisPoolConfig
* 在web.xml中配置DelegatingFilterProxy

### 4.Redis Desktop Manager介绍及使用
### 5.Spring Session源码解析

* DelegatingFilterProxy
* DefaultCookieSerializer
* SessionReponsitoryFilter
* RedisOperationsSessionRepository
* AbstractHttpSessionApplicationInitializer
* SessionReponsitoryRequestWrapper
* SessionReponsitoryResponseWrapprt
* CookieHttpSessionStrategy

### 6.Coding

详情请参见com.fmall.controller.portal.UserSpringSessionController

### 7.阅读源码的技巧

* 学习官方的例子
* debug代码
* 看方法的实现，类的继承关系(Diagrams)
### 8.遇到的坑

我在使用Spring session时在配置defaultCookieSerializer时，写了`<property name="domainName" value=".happymall.com"/>`，但是运行起来后tomcat报错**An invalid domain was specified for this cookie**。

最后总结，视频中老师使用的Tomcat7，而我用的是Tomcat8。[An invalid domain was specified for this cookie](https://stackoverflow.com/questions/42524002/an-invalid-domain-was-specified-for-this-cookie)和[How to change Cookie Processor to LegacyCookieProcessor in tomcat 8](https://stackoverflow.com/questions/38696081/how-to-change-cookie-processor-to-legacycookieprocessor-in-tomcat-8)的解决方法是在tomcat的conf文件的context.xml中添加`<CookieProcessor className="org.apache.tomcat.util.http.LegacyCookieProcessor" />`

## Spring MVC全局异常

### 1.Spring MVC全局异常

* 无SpringMVC全局异常时的流程图

  ![4-50](E:\markdown笔记\笔记图片\4-50.png)

* SpringMVC全局异常流程图

  ![4-51](E:\markdown笔记\笔记图片\4-51.png)

### 2.Spring及Spring MVC包扫描隔离

* 在applicationContext.xml中

  ```xml
  <!--spring容器中包扫描，排除controller注解，将controller注解由Springmvc配置-->
      <context:component-scan base-package="com.fmall" annotation-config="true">
          <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
      </context:component-scan>
  ```

* 在applicationContext-datasource.xml中

  ```xml
  <!-- 使用@Transactional进行声明式事务管理需要声明下面这行 -->
      <!--<tx:annotation-driven transaction-manager="transactionManager" proxy-target-class="true" />-->
      <!-- 事务管理 -->
      <!--<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">-->
          <!--<property name="dataSource" ref="dataSource"/>-->
          <!--&lt;!&ndash;当提交失败时是否进行回滚&ndash;&gt;-->
          <!--<property name="rollbackOnCommitFailure" value="true"/>-->
      <!--</bean>-->
  ```

* 在dispatcher-servlet.xml中

  ```xml
  <!--springmvc配置文件中添加注解扫描的包-->
      <context:component-scan base-package="com.fmall.controller" annotation-config="true" use-default-filters="false">
          <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
      </context:component-scan>
  ```

### 3.@Component注解

将有该注解的类声明为Bean，注入到spring容器中。

### 4.ModelAndView和MappingJacksonView

详情请参见com.fmall.common.ExceptionResolver

## SpringMVC 拦截器实现权限统一校验

### 1.一期管理员权限判断演进代码回顾

包com.fmall.controller.backend下的代码都有这段重复代码：

```java
String loginToken = CookieUtil.readLoginToken(request);
if (StringUtils.isEmpty(loginToken)){
return ServerResponse.createByErrorMessage("用户未登录，无法获取当前用户的信息。");
}

String userJsonStr = RedisShardedPoolUtil.get(loginToken);
User user = JsonUtil.string2Obj(userJsonStr, User.class);

if (user == null){
return ServerResponse.createByErrorCodeMessage(ResponseCode.NEED_LOGIN.getCode(), "用户未登录，请登录管理员");
}
```

### 2.Spring MVC拦截器流程图

![4-52](E:\markdown笔记\笔记图片\4-52.png)

### 3.HandlerInterceptor(preHandle(),postHandler(),afterCompletion())

### 4.拦截器配置及类初始化

* 在dispatcher-servlet.xml配置

  ```xml
  <mvc:interceptors>
          <!--定义在这里的，所有的都会拦截-->
          <!--manage/product/save.do  /manage/**-->
          <!--manage/order/detail.do  /manage/**-->
          <!--manage/a.do  /manage/*-->
          <!--manage/b.do  /manage/*-->
  
          <mvc:interceptor>
              <mvc:mapping path="/manage/**"/>
              <bean class="com.fmall.controller.common.interceptor.AuthorityInterceptor"/>
          </mvc:interceptor>
  </mvc:interceptors>
  ```

* com.fmall.controller.common.interceptor.AuthorityInterceptor

### 5.拦截器执行流程

详情请看com.fmall.controller.common.interceptor.AuthorityInterceptor

先执行preHandle()---->Controller---->postHandle()---->afterCompletion()

### 6.关键配置

* <mvc:mapping path="" />
  * /**所有路径及里面的子路径，如遇到这种路径：
    * manage/product/save.do
    * manage/order/detail.do
  * /*当前路径下的所有路径，不含子路径，如遇到这种路径：
    * manage/a.do
    * manage/b.do
  * /web项目的根目录请求

* \<mvc:interceptors>

* \<mvc:interceptor>

* \<mvc:exclude-mapping path=""/>

### 7.重置HttpServletResponse

### 8.解决拦截登录循环问题

在我们编写拦截器的时候需要注意，不要使拦截器陷入一个死循环。例如我们要访问http://www.happymall.com/manage/order/list.do，但是我门必须先要登录。我们登录的时候，发现拦截器将登录http://www.happymall.com/user/springsession/login.do?username=admin&password=admin1212也给拦截了。

解决办法：

* 配置dispatcher-servlet.xml

  ```xml
   <mvc:exclude-mapping path="/manage/user/login.do"/>
  ```

* 在代码配置

  ```java
  if (StringUtils.equals(className, "UserManagerController") && StringUtils.equals(methodName, "login")){
  log.info("权限拦截器拦截到请求，className:{}, methodName:{}", className, methodName);
  // 吐过是拦截到登录请求，不打印参数，因为里面有密码，全部都会打印到日志中，防止日志泄露
  return true;
  }
  ```

### 9.富文本上传拦截器处理及自测验证

在com.fmall.controller.common.interceptor.AuthorityInterceptor:

```java
if (user == null){
                if (StringUtils.equals(className, "ProductManageController") && StringUtils.equals(methodName, "richTextImgUpload")){
                    Map resultMap = Maps.newHashMap();
                    resultMap.put("success", false);
                    resultMap.put("msg", "请登录管理员");
                    out.print(JsonUtil.obj2String(request));
                }else{
                    out.print(JsonUtil.obj2String(ServerResponse.createByErrorMessage("拦截器拦截，用户未登录。")));
                }
            }else{
                if (StringUtils.equals(className, "ProductManageController") && StringUtils.equals(methodName, "richTextImgUpload")){
                    Map resultMap = Maps.newHashMap();
                    resultMap.put("success", false);
                    resultMap.put("msg", "用户无权限操作");
                    out.print(JsonUtil.obj2String(request));
                }else{
                    out.print(JsonUtil.obj2String(ServerResponse.createByErrorMessage("拦截器拦截，用户无权限操作。")));
                }
            }
```

### 10.拦截器相关代码重构

将包com.fmall.controller.backend下的代码重构，删除拦截器中已经配置的代码。
## Spring MVC RESTful原理及改造实战
### 1.SpringMVC RESTful介绍及讲解
![4-53](E:\markdown笔记\笔记图片\4-53.png)

![4-54](E:\markdown笔记\笔记图片\4-54.png)

![4-55](E:\markdown笔记\笔记图片\4-55.png)
* Content-Type，内容类型，一般是指网页中存在的**Content-Type**，用于定义网络文件的类型和网页的编码，决定文件接收方将以什么形式、什么编码读取这个文件，这就是经常看到一些Asp网页点击的结果却是下载到的一个文件或一张图片的原因。
* Accept，请求头用来告知客户端可以处理的内容类型，这种内容类型用[MIME类型](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types)来表示。
![4-56](E:\markdown笔记\笔记图片\4-56.png)

![4-57](E:\markdown笔记\笔记图片\4-57.png)

### 2.一起商品详情及列表代码的回顾
### 3.SpringMVC RESTful配置

```xml
<!--<servlet-mapping>-->
        <!--<servlet-name>dispatcher</servlet-name>-->
        <!--<url-pattern>*.do</url-pattern> &lt;!&ndash;会拦截所有的.do结尾的请求&ndash;&gt;-->
    <!--</servlet-mapping>-->

    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/</url-pattern> <!--配置RESTful-->
    </servlet-mapping>
```

### 4.RequestMapping写法注意事项

* 非RESTful接口

  ```
  @RequestMapping("detail.do")
  ```

* RESTful接口

  ```
  @RequestMapping(value = "/{productId}", method = RequestMethod.GET)
  ```

  ```
  @RequestMapping("/keyword/{keyword}/{pageNum}/{pageSize}/{orderBy}")
  ```

### 5.detail接口改造RESTful

参见com.fmall.controller.portal.ProductController#detailRESTful

### 6.list接口改造为RESTful

* 成功的案例

  com.fmall.controller.portal.ProductController#listRESTful
* 失败的案例
  * ```
    @RequestMapping("/{categoryId}/{pageNum}/{pageSize}/{orderBy}")
    ```

  * ```
     @RequestMapping("/{keyword}/{pageNum}/{pageSize}/{orderBy}")
     ```

上面这两句声明是错的，因为请求的链接是http://www.happymall.com/product/100012/1/10/price_asc，程序不知道`100012`是categoryId还是keyword，会报错**Ambiguous handler methods mapped for HTTP path http://www.happymall.com/product/100012/1/10/price_asc**


## Spring Schedule定时关单
### 1.Spring Schedule介绍

作业调度，如定时任务。

### 2.Spring Schedule Cron表达式快速入门

Cron表达式的格式（七位）:秒 分 时 日 月 周 年(可选)

第一位：Seconds

第二位：Minutes

第三位：Hours

第四位：Day-of-Month

第五位：Month

第六位：Day-of-Week

第七位：Year（可选）

![4-58](E:\markdown笔记\笔记图片\4-58.png)

![4-59](E:\markdown笔记\笔记图片\4-59.png)

![4-60](E:\markdown笔记\笔记图片\4-60.png)

其中需要说明的是：

`0 */1 * * *`其中0/1与*/1是一样的。比如我们在五点三十分23秒启动，他会等待37秒到五点三十一开始执行。

`0 0 */6 * *`:

![4-62](E:\markdown笔记\笔记图片\4-62.png)

![4-61](E:\markdown笔记\笔记图片\4-61.png)

### 3.Spring Schedule Cron生成器

[Cron生成器](http://cron.qqe2.com/)

### 4.Spring Schedule配置

* applicationContext.xml中

  ```xml
  <!--Spring Schedule配置 -->
  <task:annotation-driven/>
  ```

  会出现：

  ```xml
  xmlns:task="http://www.springframework.org/schema/task"
  ```

  ```xml
  http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task.xsd
  ```

* 在applicationContext-datasource.xml中显示参数信息

  ```xml
  <!-- 在applicationContext-datasource.xml中显示参数信息-->
  <context:property-placeholder location="classpath:datasource.properties"/>
  ```

* @Component

  将bean加载到Srping容器中（com.fmall.task.CloseOrderTask）

* @Schedule

  定义一个定时任务，关闭过期订单的定时任务的第一个版本请参照com.fmall.task.CloseOrderTask#closeOrderTaskV1

* 注意

  * 在编写\fmall\src\main\resources\mappers\OrderMapper.xml的时候发现`and create_time <= #{date}`报错

    ```xml
    <select id="selectOrderStatusByCreateTime" resultMap="BaseResultMap" parameterMap="map">
        SELECT
        <include refid="Base_Column_List"/>
        from mmall_order
        where status = #{status}
        and create_time <= #{date}
        order by create_time desc
      </select>
    ```

    解决办法：

    ```xml
    <select id="selectOrderStatusByCreateTime" resultMap="BaseResultMap" parameterMap="map">
        SELECT
        <include refid="Base_Column_List"/>
        from mmall_order
        where status = #{status}
        <![CDATA[
        and create_time <= #{date}
        ]]>
        order by create_time desc
      </select>
    ```

  * 【疑问】悲观锁：一定要使用主键

    ```xml
    <select id="selectStockByProductId" resultType="int" parameterType="java.lang.Integer">
        select
        stock
        from mmall_product
        where id = #{id}
        for update
      </select>
    ```

    


### 5.MySQL 行锁、表锁

* SELECT ... FOR UPDATE(悲观锁)

  在where条件中没有主键是表锁，有是行锁。

* 使用InnoDB

* Row-Level  Lock(明确的主键)

* Table-Level Lock(无明确的主键)


​    ![4-65](E:\markdown笔记\笔记图片\4-65.png)

![4-66](E:\markdown笔记\笔记图片\4-66.png)

![4-67](E:\markdown笔记\笔记图片\4-67.png)

![4-68](E:\markdown笔记\笔记图片\4-68.png)

![4-69](E:\markdown笔记\笔记图片\4-69.png)

## Redis分布式锁原理
### 1.Redis分布式锁命令

* ​	`setnx`——`setnx newkey newvalue`如果返回值为0说明键值对newkey已存在，如果返回1则说明键值对newkey已经设置成功，该命令具有原子性。
* `getset`——先返回键当前对应的值，然后在设定新的值，该命令具有原子性。

  ```
  127.0.0.1:6379> get a
  "b"
  127.0.0.1:6379> set a a
  OK
  127.0.0.1:6379> get a
  "a"
  127.0.0.1:6379> getset a aaa
  "a"
  127.0.0.1:6379> get a
  "aaa"
  ```
* `expire`——`expire key 10`设置key的生存时间为10s
* `del`——删除键值对，`del key`
### 2.Redis分布式锁的流程图

* 通过setnx()传入key为一个锁的名字，value为当前时间戳加被动释放的时间；
* 两边的两个箭头分别指的是两个tomcat；

![4-70](E:\markdown笔记\笔记图片\4-70.png)

### 3.Redis分布式锁优化版流程图

![4-71](E:\markdown笔记\笔记图片\4-71.png)

## Spring Schedule+Redis分布式锁构建分布式任务调度

### Coding

* Redis分布式锁防死锁

  按照Redis分布式锁的流程图，关闭过期订单的定时任务的第二个版本请参照：com.fmall.task.CloseOrderTask#closeOrderTaskV2

* Redis分布式锁双重防死锁演进（优化版）

  按照Redis分布式锁优化版流程图，关闭过期订单的定时任务的第三个版本请参照：

  com.fmall.task.CloseOrderTask#closeOrderTaskV3

## Redisson框架讲解及项目集成

### 1.Redisson介绍

![4-72](E:\markdown笔记\笔记图片\4-72.png)

![4-73](E:\markdown笔记\笔记图片\4-73.png)

### 2.Redisson官方网站

* [Redisson官网](https://redisson.org/)
* [Redisson Gtithub](https://github.com/redisson/redisson)
* [Redisson Wiki](https://github.com/redisson/redisson/wiki)

### 3.Redisson框架集成

修改pom.xml：

```xml
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson</artifactId>
    <version>2.9.0</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-avro</artifactId>
    <version>2.9.0</version>
</dependency>
```

## Spring Schedule+Redisson分布式锁构建分布式任务调度

### 1.Redisson初始化

详情请参照com.fmall.common.RedissonManager

### 2.Redisson分布式锁实战

请参照com.fmall.task.CloseOrderTask#closeOrderTaskV4

### 3.Redisson分布式锁实战解决wait_time之坑

先看下面的这段代码：

```java
getLock = lock.tryLock(1, 50, TimeUnit.SECONDS)
```

在tryLock方法中，我们将waitTime设置为1秒，但是一旦一个tomcat获取锁之后，定时任务执行的非常快，小于1秒的时候，tomcat就会将锁释放，从而出现同一分钟两个tomcat都获得锁。

改进的方法：

```java
getLock = lock.tryLock(0, 50, TimeUnit.SECONDS)
```

将waitTime设置为0.

### 4.Redis主从配置及验证

现在我们有两台redis服务器，端口分别是6379,6380。现在我们将6379作为主，6380作为从。只需修改6380的配置文件，做以下修改：

```redis
slaveof 127.0.0.1 6379
```
## 云服务器线上部署及验证

### 1.代码修改

修改com.fmall.task.CloseOrderTask，选择分布式redis的锁方法closeOrderTaskV3()

### 2.日志配置

修改src/main/resources.prod/logback.xml

### 3.环境变量配置

修改src/main/resources.prod/mmall.properties

### 4.Redis配置

安装两个redis，修改其中一个的redis.conf文件的端口为6380

### 5.Tomcat+Nginx集群环境配置（在一台虚拟机放置两个tomcat）

* 编辑`vim /etc/sudoers`文件对用户进行授权

  ```
  ## Allow root to run any commands anywhere
  root    ALL=(ALL)       ALL
  ```

* 复制两个tomcat，修改其中一个的server.xml文件

  ```
  <Server port="9005" shutdown="SHUTDOWN">
  
  
  <Connector port="9080" protocol="HTTP/1.1"
                 connectionTimeout="20000"
                 redirectPort="8443" URIEncoding="UTF-8"/>
  
  <Connector port="9009" protocol="AJP/1.3" redirectPort="8443" />
  ```

* 配置两个tomcat的环境变量

  ```
  export CATALINA_HOME=/usr/local/software/web/www.happymall.com
  export CATALINA_BASE=/usr/local/software/web/www.happymall.com
  export TOMCAT_HOME=/usr/local/software/web/www.happymall.com
  
  export CATALINA_2_HOME=/usr/local/software/web/www.happymall.com_2
  export CATALINA_2_BASE=/usr/local/software/web/www.happymall.com_2
  export TOMCAT_2_HOME=/usr/local/software/web/www.happymall.com_2
  ```

* 编辑第二个tomcat的catalina.sh文件

* 配置nginx配置文件

* 验证访问

* 关闭8080和9080端口

* 修改第二个tomcat的logback.xml文件，将`${CATALINA_HOME}`改为`${CATALINA_2_HOME}`

### 6.自动化发布shell脚本优化

编写了deploy.sh文件，使用的命令：

```
./deploy.sh v2.0 www.happymall.com
```

### 7.Logback热加载
### 8.tail和ps命令

* [tail命令](http://www.runoob.com/linux/linux-comm-tail.html)

  * 验证tomcat访问

    ```
    tail -f localhost_access_log.2019-02-05.txt
    ```

  * 验证nginx访问

    ```
    tail -f access.log
    ```

* [ps命令](http://www.runoob.com/linux/linux-comm-ps.html)

  * `ps -ef|grep tomcat`

* [less命令](http://www.runoob.com/linux/linux-comm-less.html)


