# web 应用（2018/12/26）

### 传统servlet应用

#### 依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

#### servlet组件

* Servelet

* Filter

* Listener
#### Servlet注册

* 实现
  * @WebServlet
  * HttpServlet
  * 注册 
* URL映射
  * `@WebServlet(urlPatterns = "/my/servlet")` 
* 注册
  * `@ServletComponentScan(basePackages = "cn.fx.driveinspringboot.web.servlet")` 

### 异步非阻塞Servlet代码示例

```java
@WebServlet(urlPatterns = "/my/servlet", asyncSupported = true)
public class MyServlet extends HttpServlet {

    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {

        AsyncContext asyncContext = req.startAsync();

        asyncContext.start(()->{
            try {
                resp.getWriter().println("hello world");
                
                // 触发完成
                asyncContext.complete();
            } catch (IOException e) {
                e.printStackTrace();
            }
        });
    }
}
```







