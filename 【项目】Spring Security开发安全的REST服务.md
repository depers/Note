# Spring Security开发安全的REST服务

项目github地址：

## 第1章 课程导学

### 1.导学

* 企业级安全认证和授权模块考虑的问题和实现的特性

  ![](.\笔记图片\19\1.png)

* 企业级认证和授权的组成

  ![](.\笔记图片\19\2.png)

* 课程目标

  ![](.\笔记图片\19\3.png)

## 第2章 开始开发

### 1.环境配置

* JDK1.8
* IDE
* MySQL

###  2.代码结构介绍

*  代码结构

  ![](.\笔记图片\19\4.png)

* 分别在IDEA中新建spring-security、security-core、security-broewer、security-app和security-demo项目，按照项目示例代码的pom文件对上述的几个项目进行配置。

* spring-security module项目的pom文件解析

  ```xml
  <dependencyManagement>
      <dependencies>
          <dependency>
              <groupId>io.spring.platform</groupId>
              <artifactId>platform-bom</artifactId>
              <version>Brussels-SR4</version>
              <type>pom</type>
              <scope>import</scope>
          </dependency>
          <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-dependencies</artifactId>
              <version>Dalston.SR2</version>
              <type>pom</type>
              <scope>import</scope>
          </dependency>
      </dependencies>
  </dependencyManagement>  
  ```

  这里引入了spring.io和spring-cloud的作用：管理项目各依赖的版本，在开发过程中无需开发人员自己去选择合适的依赖版本。

### 3. Hello Spring Security项目实战

* 如何让maven项目打出spring boot 的可执行jar

  ```xml
  <build>
      <plugins>
          <plugin>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-maven-plugin</artifactId>
              <version>1.3.3.RELEASE</version>
              <executions>
                  <execution>
                      <goals>
                          <goal>repackage</goal>
                      </goals>
                  </execution>
              </executions>
          </plugin>
      </plugins>
      <finalName>demo</finalName>
  </build>
  ```

## 第三章 使用Spring MVC开发RESTful API

* Restful请求编写
* bernate.validator校验框架
* Controller测试案例的编写

### 1 Restful简介

![](.\笔记图片\19\5.png)

![6](.\笔记图片\19\6.png)

### 2.查询请求

* 编写controller的测试案例

  ```java
  // 使用springRunner来运行这个类
  @RunWith(SpringRunner.class)
  @SpringBootTest
  public class UserControllerTest {
  
      @Autowired
      private WebApplicationContext wac;
  
      private MockMvc mockMvc;
  
      @Before
      public void setup(){
          // 伪造mvc环境
          mockMvc = MockMvcBuilders.webAppContextSetup(wac).build();
      }
  
      @Test
      public void whenQuerySuccess() throws Exception {
          mockMvc.perform(MockMvcRequestBuilders.get("/user")
          // 期望接收的参数是json格式     
          .contentType(MediaType.APPLICATION_JSON_UTF8))
          // 期望得到的响应状态码为200
          .andExpect(MockMvcResultMatchers.status().isOk())
          // 期望响应数据集合的长度是3
          .andExpect(MockMvcResultMatchers.jsonPath("$.length()").value(3));
      }
  }
  ```

* 使用注解声明RestFul API

  * 常用注解

    * @RestController标明此Controller提供RestAPI
    * @RequestMapping及变体。映射http请求url到java方法
    * @RequestParam映射请求参数到java 方法的参数
    * @PageableDefault指定分页参数默认值

  * 编写测试代码whenQuerySuccess方法的Controller代码

    ```java
    @GetMapping("/user")
    public List<User> getUserList(User user,
        @PageableDefault(size = 15, page = 1, sort = "age,asc") Page page){
        // 利用反射打印Object信息
        System.out.println(ReflectionToStringBuilder.toString(user));
    
        List<User> userList = Lists.newArrayList();
        userList.add(new User());
        userList.add(new User());
        userList.add(new User());
        return userList;
    }
    ```

* MockMvcResultMatchers.jsonPath()

  * JsonPath的github网站：https://github.com/json-path/JsonPath 

* @GetMapping注解中params参数详解

  * ```java
    @RequestMapping(value="/admin", method = RequestMethod.GET, params = "add")
    public String createCourse(){
        return "course_admin/edit";
    }
    ```

    * 映射地址/admin?add
    * 映射地址/admin?add=0  
    * 映射地址/admin?add=33
    * 映射地址/admin?add=*

### 3 用户详情请求

* @PathVariable：映射url片段到Java方法的参数

  ```java
  // 在url声明中使用正则表达式
  @GetMapping("/{id:\\d+}")
  @JsonView(User.UserInfoView.class)
  public User getUserInfo(@PathVariable String id){
      User user = new User();
      user.setUsername("tom");
      return user;
  }
  ```

* @JsonView控制json输出内容

  * @JsonView的使用步骤

    1. 使用接口来声明多个视图
    2. 在值对象的get方法上指定视图（如果项目中使用了lombok，则在对象的属性上指定视图）
    3. 在Controller方法上指定视图

  * 实战

    * cn.bravedawn.dto.User文件中

      ```java
      @Data
      public class User {
      
          // 使用接口来声明多个视图
          public interface UserSimpleView {};
          public interface UserInfoView extends UserSimpleView {};
      
          // 在对象的属性上指定视图
          @JsonView(UserSimpleView.class)
          private String username;
      
          @JsonView(UserInfoView.class)
          private String password;
      
          @JsonView(UserSimpleView.class)
          private int age;
      
      }
      ```

    * 在cn.bravedawn.web.controller.UserController

      ```java
      @GetMapping
      @JsonView(User.UserSimpleView.class)
      public List<User> getUserList(User user,
          @PageableDefault(size = 15, page = 1, sort = "age,asc") Pageable page){
          // 利用反射打印Object信息
          System.out.println(ReflectionToStringBuilder.toString(user));
      
          List<User> userList = Lists.newArrayList();
          userList.add(new User());
          userList.add(new User());
          userList.add(new User());
          return userList;
      }
      ```

### 4.用户创建请求

* @RequestBody映射请求题到Java方法的参数

  * 用法
    * 该注解适用于POST请求，因为他会接收content方法体中的参数，而GET方式无请求体 
    
    * 在post请求中，前端请求的参数会绑定到@RequestBody修饰的对象属性中。若在post请求中没有使用@RequestBody修饰的对象，则相关参数是不会绑定到对象的属性上
    
    * @RequestBody接收http请求中包裹在content中的参数
    
    * @RequestParam接收http请求中包裹在param中的参数
    
    * 若一个Controller方法中，即用了@RequestParam又用了@RequestBody。此时在该方法的参数中应该先写@RequestParam参数后写@RequestBody参数。例如：
    
      ```java
      public User create(@RequestParam String username, @Valid @RequestBody User user, BindingResult result){
      ```

* 日期类型参数的处理

  * 使用时间戳进行输出，在后端并不进行日期格式的装换

  * 通过日期获取时间戳：

    ```java
    Date date = new Date();
    System.out.println(date.getTime());
    ```

* @Valid注解和BindingResult验证请求参数的合法性并处理校验结果

  * controller方法：

    ```java
    public User create(@RequestParam String username, @Valid @RequestBody User user, BindingResult result){
    
    if (result.hasErrors()){
    	result.getAllErrors().stream().forEach(error -> System.out.println(error.getDefaultMessage()));
    }
    ```

  * User对象定义：

    ```java
    @Data
    public class User {
    
        // 使用接口来声明多个视图
        public interface UserSimpleView {};
        public interface UserInfoView extends UserSimpleView {};
    
        // 在对象的属性上指定视图
        @JsonView(UserSimpleView.class)
        @NotBlank
        private String username;
    
        @JsonView(UserInfoView.class)
        @NotBlank
        private String password;
    
        @JsonView(UserSimpleView.class)
        private int age;
    
        @JsonView(UserSimpleView.class)
        @NotBlank
        private String id;
    
        @JsonView(UserSimpleView.class)
        private Date birthday;
    }
    ```

### 5.修改和删除请求

hibernate.validator校验框架的学习：

1. 常用的验证注解

   ![](E:\markdown笔记\笔记图片\19\7.png)

   ![8](E:\markdown笔记\笔记图片\19\8.png)

2. 自定义消息

   ```java
   @Past(message = "生日必须是过去的时间")
   private Date birthday;
   ```

3. 自定义校验注解

   1. 编辑校验注解cn.bravedawn.validator.MyConstraint

      ```java
      @Target({ElementType.METHOD, ElementType.FIELD})
      @Retention(RetentionPolicy.RUNTIME)
      @Constraint(validatedBy = MyConstraintValidator.class)
      public @interface MyConstraint {
      
          String message();
      
          Class<?>[] groups() default {};
      
          Class<? extends Payload>[] payload() default {};
      }
      ```

   2. 编辑校验规则实现类cn.bravedawn.validator.MyConstraintValidator。

      * 该校验规则类不用使用@Bean注解，该类会自动被spring注入；
      * 该类中也可以注入其他的类并调用他的方法；

      ```java
      public class MyConstraintValidator implements ConstraintValidator<MyConstraint, Object> {
      
          @Autowired
          private HelloService helloService;
      
          @Override
          public void initialize(MyConstraint myConstraint) {
              System.out.println("my validator init");
          }
      
          // 验证成功返回true，错误返回false
          @Override
          public boolean isValid(Object o, ConstraintValidatorContext constraintValidatorContext) {
              System.out.println("my validator value is : " + o);
              helloService.greeting("tom");
              return false;
          }
      }
      ```

   3. 在User对象中使用该注解

      ```java
      @MyConstraint(message = "测试message")
      private String username;
      ```
### 6.服务异常处理

1. Spring Boot中默认的错误处理机制

   1. 在Spring Boot中是如何处理error请求的？
      * 若用户是通过浏览器请求到了错误的页面，则会返回html页面。Spring Boot中代码实现org.springframework.boot.autoconfigure.web.BasicErrorController#errorHtml
      * 若用户是通过客户端发送的请求遇到错误，则会返回Json信息。Spring Boot中代码的实现org.springframework.boot.autoconfigure.web.BasicErrorController#error

2. 自定义异常处理

   自定义异常处理类cn.bravedawn.web.controller.ControllerExceptionHandler：

   ```java
   @ControllerAdvice
   public class ControllerExceptionHandler {
   
       @ExceptionHandler(UserNotExistException.class)
       @ResponseBody
       @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
       public Map<String, Object> handleUserNotExistException(UserNotExistException ex) {
           Map<String, Object> result = new HashMap<>();
           result.put("id", ex.getId());
           result.put("message", ex.getMessage());
           return result;
       }
   
   }
   ```

### 7.使用切片拦截REST服务

1. 过滤器（Filter）

   1. 自定义请求计时的Filter，参见cn.bravedawn.filter.TimeFilter，所有的请求都会被这个Filter拦截。

   2. 如果项目中引用第三方的Filter，但是第三方的Filter是不能添加@Component注解的，如何在Spring Boot项目中引入第三方的Filter？

      编写配置类cn.bravedawn.config.WebConfig：

      ```java
      @Configuration
      public class WebConfig {
      
          @Bean
          public FilterRegistrationBean timeFilter() {
      
              FilterRegistrationBean registrationBean = new FilterRegistrationBean();
      
              TimeFilter timeFilter = new TimeFilter();
              registrationBean.setFilter(timeFilter);
      
              // 设置请求非拦截的路径，该处设置为所有请求路径
              List<String> urls = new ArrayList<>();
              urls.add("/*");
              registrationBean.setUrlPatterns(urls);
              return registrationBean;
          }
      }
      ```

   3. 采用Filter的不足：

      使用Filter是不能获取到具体是**那个Controller的那个方法**处理某一个请求。

2. 拦截器（Interceptor）

   1. 编写自定义拦截器cn.bravedawn.interceptor.TimeInterceptor，记得将该类声明为spring组件：

      ```java
      @Component
      public class TimeInterceptor implements HandlerInterceptor {
      
          /**
           * 该方法在进入控制器之前调用，该方法返回true之后，才会进入控制器
           * @param request
           * @param response
           * @param handler
           * @return
           * @throws Exception
           */
          @Override
          public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
              System.out.println("preHandle");
      
              System.out.println(((HandlerMethod)handler).getBean().getClass().getName());
              System.out.println(((HandlerMethod)handler).getMethod().getName());
      
              request.setAttribute("startTime", new Date().getTime());
              return true;
          }
      
          /**
           * 该方法在请求正常结束后调用，若报异常则不会调用该方法
           * @param request
           * @param response
           * @param o
           * @param modelAndView
           * @throws Exception
           */
          @Override
          public void postHandle(HttpServletRequest request, HttpServletResponse response, Object o, ModelAndView modelAndView) throws Exception {
              System.out.println("postHandle");
              Long start = (Long) request.getAttribute("startTime");
              System.out.println("time interceptor 耗时:"+ (new Date().getTime() - start));
          }
      
          /**
           * 在请求结束之后调用，若之前定义的ControllerExceptionHandler自定义异常处理类将异常处理之后，则该处的Exception e参数则为null
           * @param request
           * @param response
           * @param o
           * @param e
           * @throws Exception
           */
          @Override
          public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object o, Exception e) throws Exception {
              System.out.println("afterCompletion");
              Long start = (Long) request.getAttribute("startTime");
              System.out.println("time interceptor 耗时:"+ (new Date().getTime() - start));
              System.out.println("ex is "+ e);
          }
      }
      ```

   2. 对拦截器进行注册：

      ```java
      @Configuration
      public class WebConfig extends WebMvcConfigurerAdapter {
      
          @Autowired
          private TimeInterceptor timeInterceptor;
          
          @Override
          public void addInterceptors(InterceptorRegistry registry) {
              registry.addInterceptor(timeInterceptor);
          }
      }
      ```
      
   3. 采用拦截器的不足

      从上面的实例你可以看到，通过preHandle方法的handle方法，我们可以获取请求调用的类和方法名。但是并**不能获取请求的调用方法的具体参数**。

      在这里我们可以看下源码，在org.springframework.web.servlet.DispatcherServlet#doDispatch方法中：

      ```
      // 如果preHandle方法返回false，则会直接return
      if (!mappedHandler.applyPreHandle(processedRequest, response)) {
      	return;
      }
      
      // 如果preHandle方法返回true，才会执行真正的
      // Actually invoke the handler.
      mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
      ```

      

3. 切片（Aspect）

  
  
   

