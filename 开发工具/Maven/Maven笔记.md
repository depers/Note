# LOG

* 2022/08/30 陕西西安 更新 *5. Install local jar with Maven*
* 2022/09/06 陕西西安 更新 *6，7 两篇文章*

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

## 3. Maven Local Repository

> 2022年9月5日 陕西西安

### 概述

本文主要介绍在Maven本地存储库的目录配置。

简单地说，当我们运行 Maven 构建时，我们项目的依赖项（jar、插件 jar、其他工件）都存储在本地以供以后使用。

Maven支持三种类型的存储库：

1. *Local* - 本地开发机器上的文件夹位置
2. *Central* – Maven 社区提供的存储库
3. *Remote* - 组织拥有的自定义存储库

### 本地存储库

Maven 的本地存储库是本地机器上存储所有项目工件的目录。当我们执行 Maven 构建时，Maven 会自动将所有依赖项 jar 下载到本地存储库中。 通常，此目录名为 `.m2`。

以下是基于操作系统的默认本地存储库所在的位置：

```bash
Windows: C:\Users\<User_Name>\.m2
```

```
Linux: /home/<User_Name>/.m2
```

```
Mac: /Users/<user_name>/.m2
```

对于 Linux 和 Mac，我们可以用简短的形式来写：

```
~/.m2
```

### 自定义本地存储库的位置

如果在默认目录下不存在依赖项文件，则可能是因为某些预先存在的配置导致的。该配置文件位于 Maven 安装目录中名为 conf 的文件夹中，名称为 settings.xml。

配置如下：

```
<settings>
    <localRepository>C:/maven_repository</localRepository>
    ...
```

如果修改了这个配置，之前存储在较早位置的依赖项文件不会自动移动。

### 通过命令行传递本地存储库位置

除了在 Maven 的 settings.xml 中设置自定义本地存储库外，mvn 命令还支持 `maven.repo.local` 属性，它允许我们将本地存储库位置作为命令行参数传递：

```
mvn -Dmaven.repo.local=/my/local/repository/path clean install
```

这样，我们就不用修改Maven的settings.xml了。



## 4. Maven Goals and Phases

> 2022年8月26日 陕西西安

### 概述

这篇文章中，主要探讨Maven构建的生命周期及其阶段，还有就是目标（Goals）和阶段（Phases）的核心关系。

### Maven构建的生命周期

Maven的构建遵从一个特定的生命周期去部署和分发目标项目，Maven内置的生命周期有如下三个：

* default：主生命周期，负责项目部署
* clean：清理项目并删除之前构建所生成的所有文件
* site：创建项目的站点文档

每一个生命周期都是由一系列阶段（Phase）组成。default生命周期由23个阶段组成，因为他是构建的主要生命周期。除此之外，clean的生命周期由3个阶段组成，而site生命周期由4和阶段组成。

### Maven Phase

Maven Phase是代表Maven构建生命周期中的一个阶段，下面是default生命周期的最重要的一些阶段：

- *validate* – 检查项目的正确性
- *process-resources* - 将资源复制并处理到目标目录中，准备打包。
- *compile* – 将源代码编译成二进制构件
- *test-compile* - 编译测试源代码
- *test* – 执行单元测试
- *package* – 将编译后的代码打包到归档文件中
- *integration-test* – 执行额外的测试
- *install* – 将打包好的文件安装（保存）到本地仓库中
- *deploy* – 将打包好的文件部署到远程服务器或仓库中

对于每个生命周期阶段的完整列表，参考：[Lifecycle Reference](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Lifecycle_Reference)

**Phase是以特定顺序执行的，这意味着如果我们执行一个特定的Phase，使用的命令是：`mvn <Phase>`，它不仅要执行指定的阶段，还要执行前面的所有阶段。**

```
mvn deploy
```

例如，如果我们运行deploy阶段，这是默认构建生命周期的最后一个Phase，它会在deploy阶段之前执行所有的阶段，其实他执行了default的整个生命周期。

### Maven Goal

每个阶段包含一系列的目标，每个目标负责一个特定的任务。当我们运行一个阶段时，所有绑定到该阶段的目标都会按顺序执行。以下是一些与之相关的阶段和其默认目标：

- *compiler:compile* - 编译器插件的 *compile* 目标绑定到 *compile* 阶段
- *compiler:testCompile* - 绑定到 *test-compile* 阶段
- *surefire:test* - 绑定到 *test* 阶段
- *install:install* is bound to the *install* phase
- *jar:jar* and *war:war* - 绑定到 *package* 阶段

我们可以使用下面的命令列出与特定阶段相关的所有目标及其插件：

```
mvn help:describe -Dcmd=<PHASENAME>
```

例如，要列出与编译阶段有关的所有目标，我们可以运行：

```
mvn help:describe -Dcmd=compile
```

我们得到的输出如下：

```
compile' is a phase corresponding to this plugin: // 编译时与这个插件相对应的一个阶段
org.apache.maven.plugins:maven-compiler-plugin:3.1:compile
```

如上所述，这意味着 *maven-compiler-plugin* 插件的 *compile* 目标绑定到 *compile* 阶段。

### Maven Plugin

**插件是为 Maven 提供目标的工件。**此外，一个插件可能有一个或多个目标，其中每个目标代表该插件的功能。例如，编译器插件有两个目标：`compile` and `testCompile` 。前者编译主代码的源代码，而后者编译测试代码的源代码。Maven 插件是一组目标。 然而，这些目标不一定都绑定到同一个阶段。

例如，这里有一个简单的 Maven 故障安全插件配置，它负责运行集成测试：

```xml
<build>
    <plugins>
        <plugin>
            <artifactId>maven-failsafe-plugin</artifactId>
            <version>${maven.failsafe.version}</version>
            <executions>
                <execution>
                    <goals>
                        <goal>integration-test</goal>
                        <goal>verify</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

正如我们所看到的，故障安全插件在这里有两个主要的配置目标：

- *integration-test*：运行集成测试
- *verify*：验证所有通过的集成测试

我们可以使用下面的命令在一个特定的插件中列出其所有的目标：

```
mvn <PLUGIN>:help
```

例如，要列出*failsafe plugin*的所有目标，我们可以运行:

```
mvn failsafe:help
```

输出结果将是：

```
This plugin has 3 goals:

failsafe:help
  Display help information on maven-failsafe-plugin.
  Call mvn failsafe:help -Ddetail=true -Dgoal=<goal-name> to display parameter
  details.

failsafe:integration-test
  Run integration tests using Surefire.

failsafe:verify
  Verify integration tests ran using Surefire.
```

**要运行一个特定的目标而不执行它的整个阶段(和前面的阶段) ，我们可以使用以下命令：**

```
mvn <PLUGIN>:<GOAL>
```

例如，要运行*failsafe plugin的*  *integration-test* 目标，我们需要运行:

```
mvn failsafe:integration-test
```

### Building a Maven Project

要构建一个 Maven 项目，我们需要通过运行其中一个阶段来执行一个生命周期，这将执行整个*default* 生命周期：

```
mvn deploy
```

或者，我们可以停止在 *install* 阶段:

```
mvn install
```

但通常情况下，我们首先会在新构建之前运行*clean* 生命周期去清理项目：

```
mvn clean install
```

我们也可以只运行插件的一个特定目标:

```
mvn compiler:compile
```

**注意：**如果我们尝试构建 Maven 项目而没有指定阶段或目标，我们将得到一个错误:

```
[ERROR] No goals have been specified for this build. You must specify a valid lifecycle phase or a goal
```

### 总结

* Maven有三个内置的生命周期
* 一个声明周期里面包含若干个阶段；一个阶段里面包含了若干个目标，目标需要绑定到阶段去执行；插件是一组目标。
* 阶段和插件是什么关系呢？我的理解是阶段的执行要借助插件来实现。



## 5. Install local jar with Maven

> 2022年8月24日 陕西西安

### 问题和选择

Maven 是一个非常通用的工具，其可用的公共存储库是首屈一指的。 但是，总会有一个工件（依赖）没有托管在任何地方，或者托管它的存储库存在依赖风险，因为它可能在您需要时你的项目无法正常启动。

当这种情况发生时，有几种解决办法：

1. 安装 Nexus 等成熟的**存储库管理**解决方案
2. 试着把工件（依赖）上传到一个更有声望的公共仓库
3. **使用 maven 插件在本地安装工件（依赖）**

Nexus 当然是更成熟的解决方案，但也更复杂。 配置一个实例来运行 Nexus，设置 Nexus 本身，配置和维护它对于像使用单个 jar 这样简单的问题可能是矫枉过正的。 但是，如果这种场景（托管自定义工件）很常见，那么存储库管理器就很有意义。

将工件直接上传到公共存储库或 Maven 中心也是一个很好的解决方案，但通常会耗时很长。 此外，该库可能根本没有启用 Maven，这使得该过程变得更加困难，因此现在能够使用该工件并不是一个现实的解决方案。

这留下了第三个选项——在源代码控制中添加工件并使用 maven 插件——在这种情况下，`maven-install-plugin`在构建过程需要它之前先在本地安装它。 这是迄今为止最简单、最可靠的选择。

### 通过*maven-intsall-plugin*安装本地jar

让我们从将工件（依赖）安装到本地存储库所需的完整配置开始：

```xml
<plugin>
   <groupId>org.apache.maven.plugins</groupId>
   <artifactId>maven-install-plugin</artifactId>
   <version>2.5.1</version>
   <configuration>
      <groupId>org.somegroup</groupId>
      <artifactId>someartifact</artifactId>
      <version>1.0</version>
      <packaging>jar</packaging>
      <file>${basedir}/dependencies/someartifact-1.0.jar</file>
      <generatePom>true</generatePom>
   </configuration>
   <executions>
      <execution>
         <id>install-jar-lib</id>
         <goals>
            <goal>install-file</goal>
         </goals>
         <phase>validate</phase>
      </execution>
   </executions>
</plugin>
```

让我们分解并分析这个配置的细节。

#### 2.1 工件信息

构件信息被定义为 `< configuration >` 元素的一部分。实际的语法非常类似于声明依赖项-*groupId*、 *artifactId* 和 *version* 元素。

```xml
<groupId>org.somegroup</groupId>
<artifactId>someartifact</artifactId>
<version>1.0</version>
```

配置的下一部分需要定义工件的 *packaging* ——这被指定为 `jar`。

```
<packaging>jar</packaging>
```

接下来，我们需要提供要安装实际 jar 文件的位置——这可以是绝对文件路径，也可以是相对路径，这里可以使用使用 Maven 中可用的属性。 在这种情况下，`${basedir}` 属性代表项目的根，即 `pom.xml` 文件所在的位置。 这意味着 `someartifact-1.0.jar` 文件需要放在根目录下的 `/dependencies/` 目录中。

```xml
<file>${basedir}/dependencies/someartifact-1.0.jar</file>
```

最后，还可以配置其他一些可选细节，具体可以参考：https://maven.apache.org/plugins/maven-install-plugin/install-file-mojo.html

#### 2.2 执行

***install-file goal*** 的执行绑定到标准 Maven 构建生命周期的 ***validate phase***。 因此，在尝试编译之前——您需要显式运行验证阶段：

```
mvn validate
```

在这一步之后，标准编译将起作用：

```
mvn clean install
```

一旦编译阶段执行完毕，我们的 `someartifact-1.0.jar` 就会正确安装在我们的本地存储库中，就像可能从 Maven 中心本身检索到的任何其他工件一样。

#### 2.3 生成pom与提供pom

我们是否需要为工件提供 `pom.xml` 文件的问题主要取决于**工件本身的运行时依赖关系**。

如果运行时**没有依赖关系**，它只是一个简单的工件，此时我们只需要生成`pom`。`install-file` 目标中的 `generatePom` 选项应该足以处理这些类型的工件：

```xml
<generatePom>true</generatePom>
```

如果工件在运行时**依赖于其他 jar**，则这些 jar 也需要在运行时出现在类路径中，此时我们就需要提供`pom`去添加它们，这里我们可以提供自定义的`pom`文件和已安装的工件：

```
<generatePom>false</generatePom>
<pomFile>${basedir}/dependencies/someartifact-1.0.pom</pomFile>
```

这将允许 Maven 解析此自定义 `pom.xml` 中定义的工件的所有依赖项，而无需在项目的主 pom 文件中手动定义它们。

### 总结

本文介绍了如何通过使用 `maven-install-plugin` 将其安装在本地来使用未托管在 Maven 项目中的任何地方的 jar。

参考原文：https://www.baeldung.com/install-local-jar-with-maven/

## 6. Maven Deploy to Nexus

> 2022年9月5日
>
> 没有进行实操验证

### 概述

上一节中我们讨论了如何将以一个第三方jar包安装到本地的存储库，这种方式非常适合小项目。但是随着项目的发展，Nexus 很快成为托管第三方工件以及跨开发流重用内部工件的唯一真正成熟的选择。这篇文章我们主要讨论如何将一个第三方工件通过maven部署到Nexus上。

### 定义存储库信息

为了让 Maven 能够部署在构建的打包阶段创建的工件，它需要通过 `distributionManagement` 元素定义将在其中部署打包工件的存储库信息：

```xml
<distributionManagement>
   <snapshotRepository>
      <id>nexus-snapshots</id>
      <name>snapshot Respository</name>
      <url>http://localhost:8081/nexus/content/repositories/snapshots</url>
   </snapshotRepository>
</distributionManagement>
```

### Plugins

默认情况下，Maven 通过 `maven-deploy-plugin` 处理部署机制——这映射到默认 Maven 生命周期的 *deploy* 阶段：

```xml
<plugin>
   <groupId>org.apache.maven.plugins</groupId>
   <artifactId>maven-deploy-plugin</artifactId>
   <version>2.8.1</version>
   <executions>
      <execution>
         <id>default-deploy</id>
         <phase>deploy</phase>
         <goals>
            <goal>deploy</goal>
         </goals>
      </execution>
   </executions>
</plugin>
```

除了用 *maven-deploy-plugin* 外，我们还可以使用 *nexus-staging-maven-plugin* 这个插件，这个插件为了充分利用 Nexus 提供的功能而构建的。在使用这个插件之前，我们首先要禁用 *maven-deploy-plugin* 的配置，使用 `<skip>true</skip>`（或者直接删除这个配置）

```xml
<plugin>
   <groupId>org.apache.maven.plugins</groupId>
   <artifactId>maven-deploy-plugin</artifactId>
   <version>${maven-deploy-plugin.version}</version>
   <configuration>
      <skip>true</skip>
   </configuration>
</plugin>
```

接着我们定义如下配置：

```xml
<plugin>
   <groupId>org.sonatype.plugins</groupId>
   <artifactId>nexus-staging-maven-plugin</artifactId>
   <version>1.5.1</version>
   <executions>
      <execution>
         <id>default-deploy</id>
         <phase>deploy</phase>
         <goals>
            <goal>deploy</goal>
         </goals>
      </execution>
   </executions>
   <configuration>
      <serverId>nexus</serverId>
      <nexusUrl>http://localhost:8081/nexus/</nexusUrl>
      <skipStaging>true</skipStaging>
   </configuration>
</plugin>
```

这个插件的部署 *goal* 映射到 Maven 构建的 *deploy* 阶段。在将 -SNAPSHOT 工件简单部署到 Nexus 时，我们不需要暂存功能，因此可以通过 `<skipStaging>` 元素完全禁用它。

### 全局配置setting.xml

部署自己的jar包到Nexus是一项安全操作，需要在Maven全局setting.xml文件中配置服务器的凭证，下面是基于原始和明文凭据的定义如下：

* `servers.server.id` 对应`distributionManagement.snapshotRepository.id`的配置

```
<servers>
   <server>
      <id>nexus-snapshots</id>
      <username>deployment</username>
      <password>the_pass_for_the_deployment_user</password>
   </server>
</servers>
```

服务器还可以配置为使用基于密钥的安全性，而不是原始和明文凭据。

### 部署过程

执行部署过程，输入如下指令：

```
mvn clean deploy -Dmaven.test.skip=true
```

其中`-Dmaven.test.skip=true`是在部署作业的上下文中跳过测试，因为该作业应该是项目部署管道中的最后一个作业。这种部署管道的一个常见示例是一系列 Jenkins 作业，每个作业只有在成功完成时才会触发下一个作业。因此，管道中先前作业的责任是运行项目中的所有测试套件——当部署作业运行时，所有测试都应该已经通过。

若我们执行这句命令，之前的测试会在部署之前重新执行一遍：

```
mvn clean deploy
```

### 总结

本文介绍了将 Maven 工件部署到 Nexus 的简单但高效的解决方案：

1. 通过 *maven-deploy-plugin* 进行实现（默认）
2. 通过 *nexus-staging-maven-plugin* 进行实现，暂存功能被禁用（staging functionality）

### 参考文章

* https://zhuanlan.zhihu.com/p/141676033

## 7. Maven Release to Nexus

> 2022年9月6日 陕西西安
>
> 没有进行实操验证

### 概要

在的上一篇文章中，我们设置了一个使用 Maven 到 Nexus 的部署过程。 在本文中，我们将在项目的 pom 以及 Jenkins 作业中使用 Maven 配置发布流程。

### 在pom中配置存储库

为了让 Maven 能够发布到 Nexus Repository Server，我们需要通过 `distributionManagement` 元素定义存储库信息：

```
<distributionManagement>
   <repository>
      <id>nexus-releases</id>
      <url>http://localhost:8081/nexus/content/repositories/releases</url>
   </repository>
</distributionManagement>
```

### 在pom中定义SCM

scm（Source Code/Control Management）源代码版本控制。

发布过程将与项目的源代码控制交互——这意味着我们首先需要在 pom.xml 中定义 `<scm>` 元素：

```xml
<scm>
   <connection>scm:git:https://github.com/user/project.git</connection>
   <url>http://github.com/user/project</url>
   <developerConnection>scm:git:https://github.com/user/project.git</developerConnection>
</scm>
```

`connection`和`developerConnection`元素是一样的，是标记版本控制源代码的地址。

`url`可以是你项目的主页或是github的项目页面。

### 发布的插件

发布进程使用的标准 Maven 插件是 `maven-release-plugin`——这个插件的配置是最少的：

```xml
<plugin>
   <groupId>org.apache.maven.plugins</groupId>
   <artifactId>maven-release-plugin</artifactId>
   <version>2.4.2</version>
   <configuration>
      <tagNameFormat>v@{project.version}</tagNameFormat>
      <autoVersionSubmodules>true</autoVersionSubmodules>
      <releaseProfiles>releases</releaseProfiles>
   </configuration>
</plugin>
```

上面这段配置中，最重要的是`releaseProfiles`元素。这条配置会强制要求一个 Maven 配置文件——发布配置文件。

在这个过程中，`nexus-staging-maven-plugin` 用于执行对 nexus-releases Nexus 存储库的部署：

```xml
<profiles>
   <profile>
      <id>releases</id> # 这里
      <build>
         <plugins>
            <plugin>
               <groupId>org.sonatype.plugins</groupId>
               <artifactId>nexus-staging-maven-plugin</artifactId>
               <version>1.5.1</version>
               <executions>
                  <execution>
                     <id>default-deploy</id>
                     <phase>deploy</phase>
                     <goals>
                        <goal>deploy</goal>
                     </goals>
                  </execution>
               </executions>
               <configuration>
                  <serverId>nexus-releases</serverId>
                  <nexusUrl>http://localhost:8081/nexus/</nexusUrl>
                  <skipStaging>true</skipStaging>
               </configuration>
            </plugin>
         </plugins>
      </build>
   </profile>
</profiles>
```

该插件被配置为在没有暂存机制的情况下执行发布过程，与之前的部署过程相同（`skipStaging=true`）。

除此之外，还需要在全局 settings.xml (`%USER_HOME%/.m2/settings.xml`) 中配置 nexus-releases 服务器的凭据：

```xml
<servers>
   <server>
      <id>nexus-releases</id>
      <username>deployment</username>
      <password>the_pass_for_the_deployment_user</password>
   </server>
</servers>
```

### 发布过程

让我们将发布过程分解为一些小的、重点突出的步骤。

#### 1. Release:Clean

清理版本：

* 删除发布描述符（release.properties）
* 删除任何备份的 POM 文件

#### 2. Release:prepare

发布过程的下一部分是准备发布， 这将：

* 执行一些检查。应该没有未提交的更改，并且项目应该不依赖于 SNAPSHOT 依赖项
* 将 pom 文件中的项目版本更改为完整的版本号（删除 `SNAPSHOT` 后缀）。在我们的示例中为 0.1
* 运行项目测试套件
* 提交并推送更改
* 使用此非 `SNAPSHOT` 版本代码创建标签
* 在 pom 中增加项目的版本。在我们的例子中是`0.2-SNAPSHOT`
* 提交并推送更改

#### 3.  Release:perform

Release过程的后半部分是执行发布，这将：

* 从版本控制中检出release标签
* 构建和部署发布的代码

该过程的第二步依赖于 Prepare 步骤的输出。

### 在Jenkins上

这里对这块先不做了解，后续涉及应用的时候再看。

### 总结

本文展示了如何在有或没有 Jenkins 的情况下实现发布 Maven 项目的过程。 与上一篇部署maven项目类似，此过程使用 `nexus-staging-maven-plugin` 与 Nexus 交互，并专注于一个 git 项目。

### 参考文章

* https://www.baeldung.com/maven-release-nexus
* https://github.com/eugenp/tutorials/tree/master/maven-modules
