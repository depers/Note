# Maven的使用知识点

## \<scope>

* **Compile:**

  默认就是compile. Compile表示当前依赖在编译/测试/运行这三种classpaths下都有效，是一个比较强的依赖。

* **Provided:**

  provided在编译/测试的classpath下有效，在运行的classpath下无效。

* **runtime:**

  在运行/测试对应的classpaths下有效，在编译对应的classpaths下无效。

* **Test:**

  Test在测试的classpath下有效。

* **System:**

  System元素与provided元素使用范围相同，但它的依赖不会从maven仓库中下载，而是从本地路径获取，通过systempath指定本地文件jar包路径。

* **Import:**

  这个scope只支持`<dependencyManagement>`中dependency的类型为pom的节点。

> 参考博客：<https://blog.csdn.net/qq_31071543/article/details/91350906>