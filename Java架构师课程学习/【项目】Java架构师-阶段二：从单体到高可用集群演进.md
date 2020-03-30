# 阶段二：从单体到高可用集群演进

## 01.LVS+Nginx实现高可用集群

### 第1章 Nginx快速认知

#### 1-1.集群阶段开篇概述

* 阶段规划
  * 从单体到集群过度
  * Nginx入门
  * Nginx实现集群和负载均衡
  * 高可用Nginx方案与实现
  * 生产环境Nginx替换tomcat
* 项目规模的演进
  1. 项目初期
     * 2-3人团队
     * 用户量：100-500
  2. 小型项目
     * 5-10人团队
     * 用户量：1000-10000
  3. 中型项目
     * 20-30人团队
     * 用户量：50000-500000
  4. 大型项目
     * 50-100人团队
     * 用户量：百万级
  5. 巨型项目
     * 100+团队
     * 用户量：千万级
  6. 大厂级项目
     * 1000+团队
     * 用户量：亿级

* 单体架构的优点
  * 小团队成型即可完成开发-测试-上线
  * 迭代周期短，速度快
  * 打包方便，运维省事
* 单体架构面临的挑战
  * 单节点宕机造成所有服务不可用
  * 随着项目的迭代，代码的耦合度太高（迭代，测试，部署）
  * 单节点并发能力有限
* 集群概念
  * 计算机“群体”构成整个系统
  * 这个“群体”构成一个整体，不能独立存在
  * “人多力量大”，集群可以提升并发与可用性
  * 多台计算机构成的系统中，每台计算机做相同的事情称为集群；每台计算机做不同的事情称之为分布式
* 使用集群的优势
  * 提高系统性能
  * 提高系统可用性
  * 可扩展性高
* 使用集群的注意点
  * 用户会话- 分布式会话
  * 定时任务-单独用一台服务器来做定时任务
  * 内网互通

#### 1-2.什么是Nginx？常用的Web服务器有哪些？

1. 什么是Nginx
   * Nginx（engine x）是一个高性能的HTTP和反向代理web服务器，同时也提供IMAP/POP3/SMTP服务
   * 主要功能是反向代理
   * 通过配置文件可以实现集群和负载均衡
   * 静态资源虚拟化
2. 常用的web服务器
   * MS IIS——asp .net
   * Weblogic、Jboss——传统行业 ERP/物流/电信/金融
   * Tomcat、Jetty——J2EE
   * Apache、Nginx——静态服务，反向代理
   * Netty——高性能服务器编程

#### 1-3.什么是反向代理？

1. 什么是正向代理
   * 客户端请求目标服务器之间的一个代理服务器
   * 请求会先经过代理服务器，然后在转发请求到目标服务器，获得内容后最后响应给客户端
2. 什么是反向代理
   * 用户请求目标服务器，由代理服务器决定访问那个ip
3. 反向代理之路由
   * 通过Nginx可以规避端口访问的问题

####1-7 Nginx进程模型解析

* Nginx的两个进程

  * master进程：主进程
  * worker进程：工作进程

  master进程是一个领导者，worker进程是为master进程服务的。

  当程序员通过nginx命令进行操作后时，主进程会将命令**分发给worker进程**进行执行，worker进程之间是相互独立的。

* Nginx中worker进程数量配置

  1. 编辑/usr/local/myapp/program/nginx/conf/nginx.conf，编辑worker_processes参数进行配置，默认只有一个工作进程。

     ```
     worker_processes  1;
     ```

  2. 保存后，`ngxin -s reload`

#### 1-8 Nginx处理Web请求机制解析

1. worker抢占机制

   当我们提供80端口的http服务时，一个连接请求过来，每个进程都有可能处理这个连接。

   每个worker进程都是从master进程fork过来，在master进程里面，先建立好需要listen的socket（listenfd）之后，这句话的意思就是说master进程会监听来自客户端的请求，然后再fork出多个worker进程。且所有worker进程的listenfd会在新连接到来时变得可读，为了保证只有一个进程处理该连接，所有worker进程在注册listenfd读事件前抢**accept_mutex的锁**，worker进程处理请求的过程大概如下图所示：

   ![](E:\markdown笔记\笔记图片\20\2\1.jpg)

2. 传统服务器事件处理

   在传统服务器时间处理模型是同步阻塞，如下图中三个客户端进行请求，client1被worker1阻塞，client2被worker2阻塞。以此类推，每当请求过来之后，master进程不得不重新fork一个进程去做处理。

   ![](E:\markdown笔记\笔记图片\20\2\2.png)

3. Nginx事件处理

   nginx的高并发就是通过worker的抢占机制同时搭配了异步非阻塞方式来实现的。在 Linux 下，Nginx 使用 epoll 的 I/O 多路复用模型进行实现。

   编辑/usr/local/myapp/program/nginx/conf/nginx.conf对其进行配置：

   ```nginx
   events {
      # 默认使用epoll
      use epoll;
      # 每个worker进程允许连接的客户单最大连接数
      worker_connections  1024;
   }
   ```

#### 1-9 nginx.conf 配置结构与指令语法

1. nginx.conf配置结构
   1. main 全局配置
   2. event 配置工作模式以及连接数
   3. http http模块相关配置
      1. server 虚拟主机配置，可以有多个
         1. localtion 路由规则，表达式
         2. upstream 集群，内网服务器

#### 1-10 同步与异步，阻塞与非阻塞

下图中：

* BIO：（Blocking IO）同步并阻塞
* NIO：（Non-Blocking）同步非阻塞
* AIO：（Asynchronous IO）异步非堵塞

![](E:\markdown笔记\笔记图片\20\2\3.png)

#### 1-11 nginx.conf 核心配置文件

![](E:\markdown笔记\笔记图片\20\2\4.png)

#### 1-12 nginx.conf 核心配置文件详解

1. 修改worker进行执行用户，将默认的nobady修改为root

   ```nginx
   user root;
   ```

2. 修改worker工作进程数，设置为等于 CPU 的核数，若该服务器上还有别的应用在部署，可以设置为N-1个

   ```nginx
   worker_processes  1;
   ```

3. 修改错误日志配置，notice是nginx的日志级别分别是debug、info、notice、warn、error和crit

   ```nginx
   error_log  logs/error.log  notice;
   ```

4. 修改nginx的进程号，可以直接注释

   ```nginx
   pid logs/nginx.pid;
   ```

5. 修改nginx事件模式和连接数

   ```nginx
   events {
      # 默认使用epoll
      use epoll;
      # 每个worker进程允许连接的客户单最大连接数
      worker_connections  1024;
   }
   ```

6. http是指令块，针对http网络传输的一些配置指令

   ```nginx
   http {
   }
   ```

7. include引入外部配置，增加可读性。避免单个文件过大

   ```nginx
   include mime.types;
   ```

#### 1-15 nginx.pid打开失败以及失效的解决方案

nginx默认会将pid放在logs文件下的，启动之后才有这个文件。

1. 错误一：nginx:[error] open() "/var/run/nginx/nginx.pid" failed（2: No such file or directory）

   1. 方案一：可能是/var/run/nginx这个目录不存在

      使用mkdir命令新建这个文件，然后`nginx -s reload`

   2. 方案二：可能是nginx.pid文件不存在

      这个问题请参考错误的解决方法

2. 错误二：nginx:[error] invalid PID number "" in "/var/run/nginx/nginx.pid"

   通过下面命令重新制定nginx的核心配置文件然后`nginx -s reload`：

   ```sh
   ./nginx -c ../conf/nginx.conf
   ```

#### 1-16 Nginx常用命令解析

* `nginx -s stop`：暴力关闭nginx（饭店吃饭，关门赶顾客走）
* `nginx -s quit`：优雅关闭nginx（饭店吃饭，等顾客吃完再关门）
* `nginx -t`：测试配置文件语法
* `nginx -v`：nginx版本
* `nginx -V`：nginx版本并展示详细的配置信息
* `nginx -h`/`nginx -?`：nginx命令帮助
* `nginx -c`：设置nginx核心配置文件

#### 1-17 Nginx日志切割 - 手动

其中`kill -USR1`命令中USR1的解释：

USR1亦通常被用来告知应用程序重载配置文件；例如，向Apache HTTP服务器发送一个USR1信号将导致以下步骤的发生：停止接受新的连接，等待当前连接停止，重新载入配置文件，重新打开日志文件，重启服务器，从而实现相对平滑的不关机的更改。

这里实现了nginx不关机，重新打开日志文件的功能。

![](E:\markdown笔记\笔记图片\20\2\5.png)

