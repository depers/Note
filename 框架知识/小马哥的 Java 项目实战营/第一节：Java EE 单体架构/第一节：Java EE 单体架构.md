# 第一节：Java EE 单体架构

学习《[小马哥的 Java 项目实战营](https://u.geekbang.org/subject/java2nd)》第一课的笔记记录：

* ~~JAX-RS注解~~

* ~~进程和线程的区别~~

* Servlet是必须要学的

* Servlet规范的重点章节

  * CHAPTER 2 The Servlet Interface
  * CHAPTER 3 The Request
    * getParameter(String name)
    * no blocking io
  * CHAPTER 4 Servlet Context
    * Reloading Considerations
  * CHAPTER 5 The Response
    * buffering
  *  CHAPTER 9 Dispatching Requests
  * CHAPTER 11Application Lifecycle Events
  * CHAPTER 12 Mapping Requests to Servlets

* tomcat编译之后的项目中，在conf/web.xml.0文件中有两个servlet.

  * `development`，Is **Jasper** used in development mode? If true, the frequency at which JSPs are checked for modification may be specified via the modificationTestInterval parameter. [true] 

  * JSP Servlet

     做jsp渲染的时候回用到这个Servlet。如果你的项目中用不到servlet的话，可以把下面这段代码注释掉，避免性能浪费。

      ```xml
      <servlet>
          <servlet-name>jsp</servlet-name>
          <servlet-class>org.apache.jasper.servlet.JspServlet</servlet-class>
          <init-param>
              <param-name>fork</param-name>
              <param-value>false</param-value>
          </init-param>
          <init-param>
              <param-name>xpoweredBy</param-name>
              <param-value>false</param-value>
          </init-param>
          <load-on-startup>3</load-on-startup>
      </servlet>
     
      <!-- The mappings for the JSP servlet -->
      <servlet-mapping>
          <servlet-name>jsp</servlet-name>
          <url-pattern>*.jsp</url-pattern>
          <url-pattern>*.jspx</url-pattern>
      </servlet-mapping>
      ```
     
  * default Servlet

      这个也是tomcat自己设置的Servlet。
      
      ````xml
      <servlet>
          <servlet-name>default</servlet-name>
          <servlet-class>org.apache.catalina.servlets.DefaultServlet</servlet-class>
          <init-param>
              <param-name>debug</param-name>
              <param-value>0</param-value>
          </init-param>
          <init-param>
              <param-name>listings</param-name>
              <param-value>false</param-value>
          </init-param>
          <load-on-startup>1</load-on-startup>
      </servlet>
      
      <servlet-mapping>
          <servlet-name>default</servlet-name>
          <url-pattern>/</url-pattern>
      </servlet-mapping>
      ```
      ````
      
      **Servlet 处理静态内容：**我们编码过程中，会有一些静态的资源文件，我们该怎么处理呢？
      
      如果把应用部署到tomcat里面，我们需要在应用的web.xml文件中配置`org.apache.catalina.servlets.DefaultServlet`的Servlet。
      
      ```xml
      <!-- The mapping for the default servlet -->
      <servlet-mapping>
          <servlet-name>default</servlet-name>
          <url-pattern>*.css</url-pattern>
      </servlet-mapping>
      
      <servlet-mapping>
          <servlet-name>default</servlet-name>
          <url-pattern>*.js</url-pattern>
      </servlet-mapping>
      ```

* 接着上面的讨论：为什么Spring Boot 将 CSS 和 JS 文件放在 static 目录下，可以读取被读取？

  上面对于处理Servlet静态资源的问题，针对于Tomcat容器，我们可以使用org.apache.catalina.servlets.DefaultServlet，但是针对于Tomcat（JBoss）、Jetty、Resin、WebLogic、WebSphere这些容器，Spring是怎么做的？

  答案就在Spring的org.springframework.web.servlet.resource.DefaultServletHttpRequestHandler中。Spring通过判断不同的容器，然后借助不同容器自己的Servlet去处理静态资源，Spring自己没有处理。

* Servlet Forward技术

  1. 案例一：Spring Framework Web DefaultServletHttpRequestHandler forword 到容器文件Servlet 实例
  2. 案例二：自主研发 Web MVC
  3. 案例三：Spring Web MVC模板渲染

* HttpServletRequest的RequestDispatcher和ServletContext的RequestDispatcher是有区别的

  若是`HttpServletRequest`的`RequestDispatcher`会在当前请求地址的基础上，转发到新地址。比如`/hello`跳转到`/world`，最后的结果是转发到`/hello/world`。

  若是`ServletContext`的`RequestDispatcher`会从根目录上直接跳转的，转发到新地址，最后的结果是转发到`/world`。

* 我们可以使用`ServletContext.getRequestDisPatcher(String Path)`方法在Servlet中获得`RequestDisPatcher`。该路径必须以一个`/`开头，并将其解释为相对于当前上下文的根。

