# Spring Security开发安全的REST服务

项目github地址：

## 第1章 课程导学

### 1.导学

* 企业级安全认证和授权模块考虑的问题和实现的特性

  ![](E:\markdown笔记\笔记图片\19\1.png)

* 企业级认证和授权的组成

  ![](E:\markdown笔记\笔记图片\19\2.png)

* 课程目标

  ![](E:\markdown笔记\笔记图片\19\3.png)

## 第2章 开始开发

### 1.环境配置

* JDK1.8
* IDE
* MySQL

###  2.代码结构介绍

*  代码结构

  ![](E:\markdown笔记\笔记图片\19\4.png)

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

### 1 Restful简介

![](E:\markdown笔记\笔记图片\19\5.png)

![6](E:\markdown笔记\笔记图片\19\6.png)

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

      