# Maven Basics

## 1. Apache Maven Tutorial

> 2022年8月23日 陕西西安

### 介绍

Maven是Java项目的软件构建工具，负责工程的依赖管理、编译、运行测试和打包部署。

### Maven的优势

* 遵循最佳实践的简单项目设置，使用比较简单
* 依赖管理方便
* 可以隔离项目依赖项和插件
* 拥有本地仓库和远程仓库加载依赖项

### 项目对象模型

Maven 项目的配置是通过项目对象模型(Project Object Model，POM)完成的，由 POM.xml 文件表示。

pom文件的基本结构如下：

```xml
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.baeldung</groupId>
    <artifactId>baeldung</artifactId>
    <packaging>jar</packaging>
    <version>1.0-SNAPSHOT</version>
    <name>com.baeldung</name>
    <url>http://maven.apache.org</url>
    <dependencies>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>5.8.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
            //...
            </plugin>
        </plugins>
    </build>
</project>
```

根据上面的文件定义，主要拆分成下面的模块：

1. 项目定义

   Maven 使用一组标识符(也称为坐标)来唯一标识项目，并指定项目工件应该如何打包：

   * *groupId*
   * *artifactId*
   * *version* 
   * *packaging*

   一般而言，前三个(groupId: artifactId: version)组合起来形成唯一标识符，并且是您指定项目将使用哪个外部库版本(例如 JAR)的机制。

2. 依赖

   项目使用的这些外部库称为依赖项。Mavne会下载Spring Core的包到本地仓库中。

   ```xml
   <dependency>
       <groupId>org.springframework</groupId>
       <artifactId>spring-core</artifactId>
       <version>5.3.16</version>
   </dependency>
   ```

3. 仓库

   Maven 中的存储库用于保存不同类型的构建构件和依赖项。默认的本地存储库位于。用户主目录下的 `.m2/`仓库文件夹下。

   如果一个构件或是插件在本地仓库中是可用的，Maven会直接使用它。否则，Maven会去远程仓库下载它到本地。默认的远程仓库是 [Maven Central](https://search.maven.org/search?q=centra)。

   有些库(比如 JBoss 服务器)在中央存储库中不可用，但在备用存储库中可用。对于这些库，您需要在 pom.xml 文件中提供指向备用存储库的 URL:

   ```xml
   <repositories>
       <repository>
           <id>JBoss repository</id>
           <url>http://repository.jboss.org/nexus/content/groups/public/</url>
       </repository>
   </repositories>
   ```

   **请注意，您可以在项目中使用多个存储库。**

4. 自定义属性

   自定义属性有助于使 pom.xml 文件更容易读取和维护。

   Maven 属性是值占位符，可以通过使用符号`${name}`在 pom.xml 中的任何地方访问，其中name是属性名。

   一般我们会在依赖的版本号和文件路径中使用自定义属性。像这样：

   ```xml
   <properties>
       <spring.version>5.3.16</spring.version>
   </properties>
   
   <dependencies>
       <dependency>
           <groupId>org.springframework</groupId>
           <artifactId>spring-core</artifactId>
           <version>${spring.version}</version>
       </dependency>
       <dependency>
           <groupId>org.springframework</groupId>
           <artifactId>spring-context</artifactId>
           <version>${spring.version}</version>
       </dependency>
   </dependencies>
   ```

   ```xml
   <properties>
       <project.build.folder>${project.build.directory}/tmp/</project.build.folder>
   </properties>
   
   <plugin>
       //...
       <outputDirectory>${project.resources.build.folder}</outputDirectory>
       //...
   </plugin>
   ```

5. 构建

   构建部分提供了默认的Maven *goal*，编译输出的文件目录，以及应用程序的最终名称等相关信息的描述。默认的构建如下：

   ```xml
   <build>
       <defaultGoal>install</defaultGoal>
       <directory>${basedir}/target</directory>
       <finalName>${artifactId}-${version}</finalName>
       <filters>
         <filter>filters/filter1.properties</filter>
       </filters>
       //...
   </build>
   ```

   默认的编译构件目录是target，应用程序的打包名称是artifactId和version的组合。

6. 通过profile配置环境隔离

   配置文件基本上是一组配置值。通过使用配置文件，您可以为不同的环境(比如生产/测试/开发)定制构建。

   ```xml
   <profiles>
       <profile>
           <id>production</id>
           <build>
               <plugins>
                   <plugin>
                   //...
                   </plugin>
               </plugins>
           </build>
       </profile>
       <profile>
           <id>development</id>
           <activation>
               <activeByDefault>true</activeByDefault>
           </activation>
           <build>
               <plugins>
                   <plugin>
                   //...
                   </plugin>
               </plugins>
           </build>
        </profile>
    </profiles>
   ```

   使用下面的命令切换到 production 配置

   ```
   mvn clean install -Pproduction
   ```

 ### Maven构建的生命周期

每个 Maven 构建都遵循特定的生命周期。

1. Maven生命周期的阶段

   下面列出了 Maven 生命周期中最重要的阶段:

   * *validate* – 检查项目的正确性
   * *compile* – 将源代码编译成二进制构件
   * *test* – 执行单元测试
   * *package* – 将编译后的代码打包到归档文件中
   * *integration-test* – 执行额外的测试
   * *verify* – 检查包是否可用
   * *install* – 将打包好的文件安装（保存）到本地仓库中
   * *deploy* – 将打包好的文件部署到远程服务器或仓库中

2. 插件(*plugins*)和目标(*goal*)

   Maven *plugins* 是一个或多个*goal*的集合。*goal*是分阶段执行的，这有助于确定*goal*的执行顺序。

   官方支持的丰富的插件列表：https://maven.apache.org/plugins/

   为了更好的理解默认情况下哪些目标在哪些阶段运行，查看Maven默认的生命周期绑定：[default Maven lifecycle bindings.](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Built-in_Lifecycle_Bindings)

   插件的使用文章：[How to Create an Executable JAR with Maven](https://www.baeldung.com/executable-jar-with-maven)

   ```
   mvn <phase>
   ```

   例如，`mvn clean install`将删除先前创建的 jar/war/zip 文件和编译类(clean) ，并执行安装新归档文件(install)所需的所有阶段。

   **请注意，插件提供的*goal*可以与生命周期的不同阶段相关联。**

### 编写自己的第一个Maven项目

1. 执行下面的命令，建议使用git bash来执行，powershell会报错

   ```
   mvn archetype:generate \
     -DgroupId=com.baeldung \
     -DartifactId=baeldung \
     -DarchetypeArtifactId=maven-archetype-quickstart \
     -DarchetypeVersion=1.4 \
     -DinteractiveMode=false
   ```

   version默认是*1.0-SNAPSHOT*

   如果不知道要提供哪些参数，可以始终指定 interactiveMode = true，这样 Maven 就会在命令行上面让你提供所有必需的参数。如果我们直接执行`mvn archetype:generate`命令，maven就会让我们提供相应的参数。

   上面的命令执行之后，我们就可以看到pom.xml的样子如下：

   ```xml
   <project>
       <modelVersion>4.0.0</modelVersion>
       <groupId>com.baeldung</groupId>
       <artifactId>baeldung</artifactId>
       <version>1.0-SNAPSHOT</version>
       <name>baeldung</name>
       <url>http://www.example.com</url>
       <dependencies>
           <dependency>
               <groupId>junit</groupId>
               <artifactId>junit</artifactId>
               <version>4.11</version>
               <scope>test</scope>
           </dependency>
       </dependencies>
   </project>
   ```

2. 编译和打包一个项目

   * 编译

     Maven 将贯穿编译阶段所需的所有生命周期阶段，以构建项目的源代码。

     ```
     mvn compile
     ```

   * 测试

     如果只想运行测试阶段，可以使用:

     ```
     mvn test
     ```

   * 打包

     调用打包阶段，它将生成已编译的归档 jar 文件：

     ```
     mvn package
     ```

3. 执行程序

   我们通过exec-maven-plugin来执行我们的Java项目，配置文件如下：

   ```xml
   <build>
       <sourceDirectory>src</sourceDirectory>
       <plugins>
           <plugin>
               <artifactId>maven-compiler-plugin</artifactId>
               <version>3.6.1</version>
               <configuration>
                   <source>1.8</source>
                   <target>1.8</target>
               </configuration>
           </plugin>
           <plugin>
               <groupId>org.codehaus.mojo</groupId>
               <artifactId>exec-maven-plugin</artifactId>
               <version>3.0.0</version>
               <configuration>
                   <mainClass>com.baeldung.java.App</mainClass>
               </configuration>
           </plugin>
       </plugins>
   </build>
   ```

   其中：maven-compiler-plugin负责使用 Java 版本1.8编译源代码。exec-maven-plugin 在我们的项目中搜索主函数。

   运行下面的命令执行程序：`mvn exec:java`


## 2. Apache Maven Standard Directory Layout

> 2022年8月24日 陕西西安

促进项目之间的统一目录结构也是Maven的重要功能之一。

### 目录布局

Maven项目的标准目录布局：

```
└───maven-project
    ├───pom.xml
    ├───README.txt
    ├───NOTICE.txt
    ├───LICENSE.txt
    └───src
        ├───main
        │   ├───java
        │   ├───resources
        │   ├───filters
        │   └───webapp
        ├───test
        │   ├───java
        │   ├───resources
        │   └───filters
        ├───it
        ├───site
        └───assembly
```

### 根目录

根目录每一个目录的功能和含义如下：

* *maven-project/pom.xml* - 定义 Maven 项目构建生命周期中所需的依赖项和模块
* *maven-project/LICENSE.txt* - 项目的许可证资料
* *maven-project/NOTICE.txt* - 项目中使用的第三方库的信息
* *maven-project/src/main* - 作为打包构建主要部分，包含源代码和资源文件
* *maven-project/src/test* - 存放所有的测试代码和资源
* *maven-project/src/it* - 通常保留用于 Maven 故障安全插件所使用的集成测试
* *maven-project/src/site* - 使用 Maven 站点插件创建的站点文档
* *maven-project/src/assembly* - 封装二进制程序的组装配置

### src/main 目录

src/main 是 Maven 项目中最重要的目录。

* *src/main/java* - 构件的Java源代码
* *src/main/resources* - 配置文件和其他文件，例如 i18n 文件、每个环境配置文件和 XML 配置
* *src/main/webapp* - 对于 Web 应用程序，包含 JavaScript、 CSS、 HTML 文件、视图模板和图像等资源
* *src/main/filters* - 包含在构建阶段将值注入到资源文件夹中的配置属性的文件

### src/test目录

src/test 是Maven存放测试代码的地方。

值得注意是，这些目录或文件都不会成为构件的一部分。

* *src/test/java* - 测试的Java源代码
* *src/test/resources* - 用于测试的配置文件和其他文件
* *src/test/filters* - 包含在测试阶段将值注入到资源文件夹中的配置属性的文件

