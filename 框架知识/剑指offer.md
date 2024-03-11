# 剑指offer

## 第一章 网络

### 1.OSI开放式互联参考模型

主要讲了OIS七层模型的作用

![7-1](.\..\笔记图片\7\7-1.png)

### 2.TCP/IP四层模型

![7-2](..\笔记图片\7\7-2.png)

![7-3](..\笔记图片\7\7-3.png)

### 3.TCP的三次握手

![7-4](..\笔记图片\7\7-4.png)

TCP报文头字段的含义

![7-5](..\笔记图片\7\7-5.png)

![7-6](..\笔记图片\7\7-6.png)

![7-7](..\笔记图片\7\7-7.png)

![7-8](..\笔记图片\7\7-8.png)

* 为什么需要三次握手才能建立起连接呢？

  为了初始化Sequence Number的初始值

* 首次握手的隐患——SYN超时

  * Server收到Client的SYN，回复SYN-ACK的时候，Clinet掉线导致Server未收到Client发送的ACK确认
  * Server如果收不到ACK确认的话，他就会不断重试直至超时，Linux下会重发五次，每次重试的间隔时间翻倍，从1s，2s，4s，8s，16s，32s总共等待63秒后才会断开连接

* SYN超时会带来[SYN Flood](https://baike.baidu.com/item/syn%20flood/5342784?fr=aladdin)攻击，防护措施:

  SYN Flood攻击：问题就出在TCP连接的三次握手中，假设一个用户向服务器发送了SYN[报文](https://baike.baidu.com/item/%E6%8A%A5%E6%96%87)后突然死机或掉线，那么服务器在发出SYN+ACK应答报文后是无法收到客户端的ACK报文的（第三次握手无法完成），这种情况下服务器端一般会重试（再次发送SYN+ACK给客户端）并等待一段时间后丢弃这个未完成的连接，这段时间的长度我们称为SYN Timeout，一般来说这个时间是分钟的数量级（大约为30秒-2分钟）；一个用户出现异常导致服务器的一个线程等待1分钟并不是什么很大的问题，但如果有一个恶意的攻击者大量模拟这种情况，服务器端将为了维护一个非常大的半连接列表而消耗非常多的资源----数以万计的半连接，即使是简单的保存并遍历也会消耗非常多的CPU时间和内存，何况还要不断对这个列表中的IP进行SYN+ACK的重试。实际上如果服务器的TCP/IP栈不够强大，最后的结果往往是堆栈溢出崩溃---即使服务器端的系统足够强大，服务器端也将忙于处理攻击者伪造的TCP连接请求而无暇理睬客户的正常请求（毕竟客户端的正常请求比率非常之小），此时从正常客户的角度看来，服务器失去响应，这种情况我们称作：服务器端受到了SYN Flood攻击（SYN[洪水攻击](https://baike.baidu.com/item/%E6%B4%AA%E6%B0%B4%E6%94%BB%E5%87%BB)）。

  * Server的SYN队列满后，通过tcp_syncookies参数回发SYN Cookie
  * 若为正常连接则Client会回发SYN Cookie，直接建立连接

* 建立连接后，Client出现故障怎么办？——TCP保活机制

  * 连接处于非活动状态，开启保活功能的一端将向对方发送保活探测报文，如果未收到响应则继续发送
  * 尝试发送次数达到保活探测数仍未收到响应则中断连接

### 4.TCP的四次挥手

![7-9](..\笔记图片\7\7-9.png)

![7-10](..\笔记图片\7\7-10.png)

* 上图中为什么Client会有TIME_WAIT状态？

  * 1.可靠的终止TCP连接，若处于time_wait的client发送给server确认报文段丢失的话，server将在此又一次发送FIN报文段，那么client必须处于一个可接收的状态就是time_wait而不是close状态。 
  * 2.保证迟来的TCP报文段有足够的时间被识别并丢弃，linux 中一个TCPport不能打开两次或两次以上。当client处于time_wait状态时我们将无法使用此port建立新连接，假设不存在time_wait状态，新连接可能会收到旧连接的数据。time_wait持续的时间是2MSL，保证旧的数据能够丢弃。

* 为什么四次握手才能断开连接

  * 为保证单向通信的可行性，所以多一次握手。
    * 主动断开方发送FIN时，被动断开方要回复ACK，意思是“我收到你的FIN了”；
    * 主动断开方发送FIN并不意味着立即关闭TCP连接，而是告诉对方自己没有更多的数据要发送了，只有当对方发完自己的数据再发送FIN后，才意味着关闭TCP连接；
    * 被动断开方收到FIN并回复ACK后，此时TCP处于“半关闭”状态，为保证被动断开方可以继续发送数据，所以第二个FIN并不会伴随ACK发送，所以比连接时多一个报文段。

* 服务器出现大量CLOSE_WAIT状态的原因

  在主动断开方关闭socket连接，被动断开方方忙于读或写，没有及时关闭连接

  措施：

  * 检查代码，特别是释放资源的代码
  * 检查配置，特别是处理请求的线程配置

* netstat指令

  * `netstat -n|awk '/^tcp/{++S[$NF]}END{for(a in S) print a, S[a]}'`

### 5.TCP滑动窗口

* RTT和RTO
  * RTT：发送一个数据包到收到对应的ACK所花费的时间
  * TRO：重传时间间隔
* TCP使用滑动窗口做流量控制与乱序重排，TCP的滑动窗口主要有两个作用：
  * 保证TCP的可靠性
  * 保证TCP的流控特性

* 滑动窗口的计算

  ![7-11](..\笔记图片\7\7-11.png)

  * LastByteAcked：已发送并收到确认的最后一个字节位置
  * LastByteSent：已发送未收到确认的最后一个字节位置
  * LastByteWritten：上层应用已写完的，准备发送的最后一个字节的位置
  * LastByteRead：上层应用已读到的，已向发送方发送回执信息的最后一个字节位置
  * NextByteExpected：指向连续最大Sequence的位置，已经收到但未收到回执信息位置
  * LastByteRcvd：已经收到最后一个字节
  * AdvertisedWindow：接收方可以处理数据的量
  * MaxRecvBuffer：接收方最大可以处理数据的量，也可以理解为接收方缓存池的大小
  * EffectiveWindow：发送方剩余可发送数据量的大小

* 滑动窗口原理

  * TCP会话发送方

    ![7-12](..\笔记图片\7\7-12.png)

    数据分类：

    * Sent and Acknowledged：已经发送成功并已经被确认的数据

    * Send But Not Yet Acknowledged：发送但没有被确认

    * Not Sent，Recipient Ready to Receive：将要发送的数据，这部分数据已经被加载到缓存中，也就是窗口中了，等待发送

    * Not Sent，Recipient Not Ready to Receive： 这些数据属于未发送，同时接收端也不允许发送的，因为这些数据已经超出了发送端所接收的范围

  * 滑动窗口

    ![7-13](..\笔记图片\7\7-13.png)

    原滑动窗口为32-51，现在接收到确认的是32-35，所以窗口滑动，使得52-55进入滑动窗口。

  * TCP会话接收方

    ![7-14](..\笔记图片\7\7-14.png)

    在TCP的接收缓存内有三种状态：

    * Received and Acknowledge：已接收已发送回执状态
    * Not Yet Received，Transmiter Permitted To Send：未接收，准备接收的状态
    * Not Yet Received Transmitter May Not Send：未接收，但不能接收的状态

    **TCP的数据传输可靠性来源于确认重传机制，TCP滑动窗口的可靠性也来源于确认重传机制**
### 6.UDP与TCP的区别

* UDP报文结构

  ![7-15](..\笔记图片\7\7-15.png)

* UDP特点

  ![7-16](..\笔记图片\7\7-16.png)

* TCP与UDP的区别

  * TCP面向连接，UDP面向无连接
  * 可靠性：TCP利用握手，确认重传机制保障数据传输的可靠性，UDP则可能会丢失数据
  * 有序性：TCP利用序列号保障消息报的有序交付，UDP不具备有序性
  * 速度：TCP慢，创建连接，保证数据的可靠性和有序性需要做额外的事情，UDP快
  * 量级：TCP是重量级的，TCP首部20个字节，UDP首部则8个字节

### 7.HTTP

* HTTP超文本传输协议的主要特点：

  ![7-17](..\笔记图片\7\7-17.png)

  ![7-18](..\笔记图片\7\7-18.png)

* HTTP请求结构：请求行，请求头部，请求正文

  ![7-19](..\笔记图片\7\7-19.png)

* HTTP响应结构：状态行，响应头部，响应正文

  ![7-20](..\笔记图片\7\7-20.png)

* HTTP请求/响应的步骤

  ![7-21](..\笔记图片\7\7-21.png)

* 在浏览器地址栏键入URL，按下回车之后经历的流程

  * DNS解析：浏览区会根据URL，逐层查询DNS缓存，解析URL中的域名，查找IP地址。DNS缓存由近到远依次是浏览器缓存，系统缓存，路由器缓存，IPS服务器缓存，域名服务器缓存，顶级域名服务器缓存。若在某层找到对应的IP地址，则停止查找。
  * TCP链接：通过查询到的IP地址和对应端口号（默认是80端口），与服务器建立TCP连接（这里可以结合前面的三次握手来讲解）
  * 发送HTTP请求：浏览器会发出读取文件的HTTP请求，该请求会发送给服务器
  * 服务器处理请求并返回HTTP报文：服务器对浏览器的请求作出响应，将带有html文本的HTTP响应报文发送给浏览器
  * 浏览器解析渲染页面：浏览器接收html文本
  * 连接结束：浏览器释放TCP连接（这里可以结合前面的四次握手来讲解）

* HTTP状态码

  ![7-22](..\笔记图片\7\7-22.png)

* 常见状态码

  ![7-23](..\笔记图片\7\7-23.png)

* GET请求和POST请求的区别

  * HTTP报文层面：GET将请求信息放在URL中，POST放在报文体中
  * 数据库层面：GET符合幂等性（对数据库的一次或多次操作，获得的结果是一致的）和安全性（对数据库的操作没有改变数据库的数据），POST不符合
  * 其他层面：GET可以被缓存、被存储，而POST不行

* Cookie和Session的区别

  * Cookie

    ![7-24](..\笔记图片\7\7-24.png)

  * Cookie的设置以及发送过程

    ![7-25](..\笔记图片\7\7-25.png)

  * Session

    ![7-26](..\笔记图片\7\7-26.png)

  * Session的实现方式

    ![7-27](..\笔记图片\7\7-27.png)

  * 区别

    ![7-28](..\笔记图片\7\7-28.png)

### 8.HTTP和HTTPS的区别

* HTTPS在HTTP下面加了一层SSL或TLS

  ![7-29](..\笔记图片\7\7-29.png)

* SSL

  ![7-30](..\笔记图片\7\7-30.png)

* 加密方式

  ![7-31](..\笔记图片\7\7-31.png)

* HTTPS数据传输流程

  ![7-32](..\笔记图片\7\7-32.png)

* 区别

  ![7-33](..\笔记图片\7\7-33.png)

* HTTPS真的很安全吗

  ![7-34](..\笔记图片\7\7-34.png)
### 9.Socket相关

* Socket简介

  ![7-35](..\笔记图片\7\7-35.png)

* Socket通信流程

  ![7-36](..\笔记图片\7\7-36.png)

* Socket编程

  详情请参见E:\Dome\Java EE\interview\src\socket

## 第二章 数据库

![7-38](..\笔记图片\7\7-38.png)

* 一道面试题：如何设计一个关系型数据库？

  ![7-39](..\笔记图片\7\7-39.png)

  要设计一个关系型数据库，要将其划分为两个部分。一个是程序实例部分，一个是存储部分，该部分类似于一个文件系统，将数据持久化到存储设备当中。除了存储部分，我们还需要通过程序实例部分对存储部分进行逻辑上的管理。程序实例部分包括将数据的逻辑关系转换为物理存储关系的存储模块；优化执行效率的缓存模块；将SQL语句进行解析的SQL解析模块；记录操作的日志管理模块；进行多于户管理的权限划分模块；灾难恢复的容灾机制模块；优化数据查询效率的索引管理模块；使得数据库支持并发操作的锁模块。

### 1.索引模块

在开始本节之前请阅读本篇文章：[再有人问你为什么MySQL用B+树做索引，就把这篇文章发给她]( https://mp.weixin.qq.com/s?__biz=MzI3NzE0NjcwMg==&mid=2650125030&idx=1&sn=1c2a09a80547159b336e7ea25d7f4955&chksm=f36bafc7c41c26d14cf685d42369e82b3ead547a919b6e86b8068ef88464cb8ac13f72271a74&scene=27#wechat_redirect )![7-40](..\笔记图片\7\7-40.png)

* **为什么要是用索引？**

  索引（Index）是帮助MySQL高效获取数据的数据结构。一个没有索引的数据库表就相当于一本没有索引的新华字典，索引的一个主要目的就是加快检索表中数据，亦即能协助信息搜索者尽快的找到符合限制条件的记录ID的辅助数据结构。

* **什么样的信息能称为索引？**

  能将该记录限定在一定查找范围内的字段。包括主键、唯一键以及普通键等

* **索引的数据结构**

  * 生成索引，建立二叉查找树进行二分查找
  * 生成索引，建立B-Tree结构进行查找
  * 生成索引，建立B+Tree结构进行查找
  * 生成索引，建立Hash结构进行查找

* 二叉查找树，也称二叉排序树

  ![7-41](..\笔记图片\7\7-41.png)

  二叉查找树的特点：（1）若左子树不空，则左子树上所有结点的值均小于它的根结点的值；（2）若右子树不空，则右子树上所有结点的值均大于它的根结点的值；（3）左、右子树也分别为二叉排序树；（4）没有键值相等的节点。

  但是如果数据增多，二叉查找树的二分查找的时间复杂度就会增大，每查找到一个节点的子树就要进行一次IO操作，继而又增加了IO的次数。

* B-Tree，又称多路搜索树

  ![7-42](..\笔记图片\7\7-42.png)

  * 定义（制定这样的约束为的是让每个节点存储更多的数据，尽可能较少IO的次数）

    * 根结点至少包括两个孩子

    * 树中每个结点最多含有m个孩子（m>2）

    * 除根节点和叶节点外，其他每个节点只收有ceil(m/2)个孩子

      > ceil()失去一个数的上限的函数，例如ceil(1.3)=2,ceil(1.8)=2

    * 所有叶子结点都位于同一层

    * 假设每个非终端节点中包含有n个关键字信息，其中

      * a) Ki(i=1...n)为关键字，且关键字按照顺序升序排序K(i-1)<Ki
      * b)关键字的个数n必须满足：[ceil(m/2)-1]<=n<=m-1
      * c)非叶子节点的指针：Pi(1...n)为指向子树根结点的指针；其中P(i)指向关键字小于K[i]，P[i]指向关键字大于K[i-1]，其他P[i]指向关键字大小范围为(K[i-1],K[i])
      
    * 上面的内容太过冗长，简而言之： **B树相对于平衡二叉树，每个节点存储了更多的键值(key)和数据(data)，并且每个节点拥有更多的子节点，子节点的个数一般称为阶，上述图中的B树为3阶B树，高度也会很低。**  

* B+Tree

  B+树是B树的变体，其定义基本与B树相同，除了：

  ![7-43](..\笔记图片\7\7-43.png)

  * 非叶子节点的子树指针与关键字的个数相同(可以存储更多的关键字)
  * 非叶子节点的子树指针P[i]，指向关键字值大小范围为[K[i], K[i+1])
  * 非叶子结点仅用来索引，不存储数据；数据都保存在叶子节点中
  * 所有叶子结点均有一个链指针指向下一个叶子结点

* B+Tree更适合用来做存储索引

  * B+树的磁盘读写代价更低：单节点可以存储更多的元素，使得查询磁盘IO次数更少。
  * B+树的查询效率更稳定：所有查询都要查找到叶子节点，查询性能稳定。
  * B+树更有利于对数据库的扫描：所有叶子节点形成有序链表，便于范围查询。

* Hash索引

  * 优点：

    * 哈希索引就是采用一定的哈希算法，把键值换算成新的哈希值，检索时不需要类似B+树那样从根节点到叶子节点逐级查找，只需一次哈希算法即可立刻定位到相应的位置，速度非常快。

  * 缺点：

    * Hash索引仅仅能满足”=”,”IN”和”<=>”查询，不能使用范围查询。

      由于 MySQL Hash索引比较的是进行 Hash 运算之后的 Hash 值，所以它只能用于等值的过滤，不能用于基于范围的过滤，因为经过相应的 Hash 算法处理之后的 Hash 值的大小关系，并不能保证和Hash运算前完全一样。

    * Hash索引无法被用来避免数据的排序操作。

      由于Hash索引中存放的是经过 Hash 计算之后的 Hash 值，而且Hash值的大小关系并不一定和 Hash 运算前的键值完全一样，所以数据库无法利用索引的数据来避免任何排序运算；

    * Hash索引不能利用部分索引键查询。

      对于组合索引，Hash 索引在计算 Hash 值的时候是组合索引键合并后再一起计算 Hash 值，而不是单独计算 Hash 值，所以通过组合索引的前面一个或几个索引键进行查询的时候，Hash 索引也无法被利用。

    * Hash索引遇到大量Hash值相等的情况后性能并不一定就会比B-Tree索引高。

      对于选择性比较低的索引键，如果创建 Hash 索引，那么将会存在大量记录指针信息存于同一个 Hash 值相关联。这样要定位某一条记录时就会非常麻烦，会浪费多次表数据的访问，而造成整体性能低下。

  * BitMap索引（MySQL不支持）

    ![7-44](..\笔记图片\7\7-44.png)

    所谓的BitMap就是用一个bit位来标记某个元素所对应的value，而key即是该元素，由于BitMap使用了bit位来存储数据，因此可以大大节省存储空间。

* 密集索引和稀疏索引的区别

  ![7-45](..\笔记图片\7\7-45.png)

  * 区别
    * 密集索引文件中的每个搜索码值都对应一个索引值
    * 稀疏索引文件只为索引码的某些值建立索引项

  * 定义
    * 密集索引的定义：叶子节点保存的不只是键值，还保存了位于同一行记录里的其他列的信息，由于密集索引决定了表的物理排列顺序，一个表只有一个物理排列顺序，所以一个表只能创建一个密集索引
    * 稀疏索引：叶子节点仅保存了键位信息以及该行数据的地址，有的稀疏索引只保存了键位信息机器主键

  * MySQL

    在MySQL中表结构信息存储在.frm文件中，若使用InnoDB引擎，他的数据和索引都是存储在.idb文件下的；若使用的是MyISAM引擎，他的数据存储在.MYD，索引存储在.MYI中。

    ![7-46](..\笔记图片\7\7-46.png)

    * mysam存储引擎，不管是主键索引，唯一键索引还是普通索引都是稀疏索引

    * innodb存储引擎：有且只有一个密集索引。密集索引的选取规则如下：
      * 若主键被定义，则主键作为密集索引
      * 如果没有主键被定义，该表的第一个唯一非空索引则作为密集索引
      * 若不满足以上条件，innodb内部会生成一个隐藏主键（密集索引）
      * 非主键索引存储相关键位和其对应的主键值，包含两次查找

### 2.索引额外的问题之如何调优Sql

以MySQL为例

* 根据慢日志定位慢查询sql

  * 在MySQL数据库中查询，输入`show variables like '%query%'`，然后我们需要关注这三个点：

    * slow_query_log：是否开启慢日志。默认为关闭OFF
    * slow_query_log_file：慢日志的存储位置
    * long_query_time：设置SQL查询时间，默认为10s，意思是一条SQL语句执行超过10s，就会被记录到慢日志当中

  * 查看慢查询的数量Slow_queries`show status like '%slow_queries%'`，该数值为本次客户端会话时慢SQL的条数，一旦关闭会话，下次打开时该值就会被清零

  * 打开慢日志记录：`set GLOBAL slow_query_log=on;`

  * 设置慢日志记录时间为1s，`set GLOBAL long_query_time=1;`,然后重新连接数据库，该项设置才能生效

  * 输入SQL：`select name from person_info_large order by name desc`就会产生一条慢日志

    ![7-47](..\笔记图片\7\7-47.png)

* 使用explain等工具分析SQL

  输入下列指令`explain select product_name from mmall_order_item order by product_name desc;`:

  下面出现的两个属性，我们需要注意：

  * type

    在下图中如果type结果为index和all，说明sql有待优化

    ![7-48](..\笔记图片\7\7-48.png)

  * Extra

    ![7-49](..\笔记图片\7\7-49.png)

* 修改SQL或者尽量让SQL走索引

  我们先来看下表结构：

  ![7-50](..\笔记图片\7\7-50.png)

  在表结构中account是唯一索引，所以我们使用account唯一索引进行检索：

  ![7-51](..\笔记图片\7\7-51.png)

  在上图中我们可以看到type，key，Extra都发生了变化，是因为我们使用了索引。

  接下来我们给name字段添加索引`alter table person_info_large add index idx_name(name)`。然后我们再输入sql`select name from person_info_large order by name desc`，就可以看到下图：

  ![7-52](..\笔记图片\7\7-52.png)

  添加索引后，sql执行的速度有了明显的提升。

### 3.索引额外问题之最左匹配原则的成因

![7-53](..\笔记图片\7\7-53.png)

在上面的建表语句中我们可以看到index_area_title的组合索引。

![7-54](..\笔记图片\7\7-54.png)

通过explain可以看到key为index_area_title，sql执行使用了组合索引。

![7-55](..\笔记图片\7\7-55.png)

此时，我们单独使用area进行检索，可以看到key依然为index_area_title，sql执行依然使用了组合索引。

![7-56](..\笔记图片\7\7-56.png)

但是，如果我们单独使用title进行检索却没有使用组合索引。这就与mysql最左匹配原则有关系了。

![7-57](..\笔记图片\7\7-57.png)

 其中第二条等于可以乱序的意思就是说，使用`select * from person_info_large where area='' and title=''`与`select * from person_info_large where title='' and area='' `效果是一样的，都会使用联合索引进行查询。

### 4.索引是建立的越多越好吗

* 数据量小的表是不需要建立索引的，建立索引会增加额外的索引开销。
* 数据变更需要维护索引，因此更多的索引意味着更多的维护成本。
* 更多的索引意味着也需要更多的空间。

### 5.锁模块之MyISAM与InooDB关于锁方面的区别（1）

![7-58](..\笔记图片\7\7-58.png)

* MyISAM与InnoDB关于锁方面的区别是什么

  * MyISAM默认用的是表级锁，不支持行级锁（MyISAM不支持事务）
  * InnoDB默认用的是行级锁，也支持表级锁

* MyISAM引擎锁的实验（下面的两个操作都是使用两个session进行的）
  * 先执行读锁，后执行写锁

    MyISAM引擎下，当我们使用select语句对表进行查询时，他会为查询添加一个读锁（表锁），当我们进行增删改时，他会为增删改添加一个写锁（表锁）。如果我们运行读锁，此时读锁还没运行结束然后在运行写锁，当读锁还没有释放时，他会阻塞写锁，直到读锁结束。

    显式的设置锁和释放锁：

    ```sql
    lock tables table_name read/write;
    unlock tables;
    ```

  * 先执行读锁，后也执行读锁

    MyISAM引擎下，我先执行select语句，然后我再执行一条select语句。可以发现在第一条读锁还没有结束的时候，第二条读锁就可以快速的查询并结束。其中读锁也称**共享锁**。

  * 先执行写锁，后执行读锁

    MyISAM引擎下，我们先执行更新操作，接着我们在另一个session中执行读操作。此时之前的更新操作还没有执行完，MyISAM会为这个表加一个写锁。写锁会阻塞读锁，直到写锁释放，读操作才能进行。
    
  * 先执行写锁，后执行写锁
  
    MyISAM引擎下，我们先执行更新操作接着我们继续执行更新操作。此时我们就会发现我们第一次的更新操作还未完成，他会阻塞第二次的更新操作。所以读锁又称为**排他锁**。
  
  * select语句也可以设置**排他锁**
  
    通过为select语句添加**for update**语句，来为他添加排他锁。先执行下列语句：
  
    ```sql
    select * from person_info_myisam where id between 1 and 2000000 for update;
    ```
  
    接着执行:
  
    ```sql
    select * from person_info_myisam where id in (20000001);
    ```
  
    此时第一条语句还未执行完，执行第二条语句。此时你就会发现第二条查询被阻塞了。只有第一条语句执行完后，第二条数据才执行。
  
* InnoDB引擎锁的实验

  1. 关闭数据库的自动提交。（预先准备）

    ```sql
    show variables like 'autocommit';
    
    set autocommit = 0; -- 关闭自动提交，该操作只针对当前的查询session
    ```

    如果你不想显示的关闭自动提交，可以使用以下语句：

    ```sql
    begin transaction
    -- update .....sql
    ```

  2. 实验一：InnoDB并未为select语句添加共享锁（先读后写）
  
     首先关闭两个session的自动提交，然后在session1中执行：
  
     ```sql
     set autocommit = 0;
     select * from person_info_large where id = 3;
     ```
  ```
     
  接着再session2中执行更新操作：
     
     ```sql
     set autocommit = 0;
  update person_info_large set title='test3' where id = 3;
  ```
  
     按照常理来说在select操作中InnoDB会给改行数据添加共享锁，共享锁未释放之前session中的写锁是会被堵塞的，只有commit之后才会被释放。但是session2中的更新操作却并未被阻塞。**原因是InnoDB会并没有为select语句添加锁，所以更新操作可以顺畅执行**。
  
  3. 实验二：InnoDB为select显式添加**行级共享锁**（先读后写）
  
     在session1中显式添加共享锁：
  
     ```sql
     set autocommit = 0;
     select * from person_info_large where id = 3 lock in share mode;
     ```
  ```
     
  在session2中更新：
     
     ```sql
     set autocommit = 0;
  update person_info_large set title='test3' where id = 3;
  ```
  
     此时我们就会发现更新操作被阻塞了。只有在session1中commit之后，共享锁才会被释放。
  
  4. 实验三：InnoDB是否真的支持行级锁（先读后写）
  
     在session1中显式添加共享锁，为id=3添加行锁：
  
     ```sql
     set autocommit = 0;
     select * from person_info_large where id = 3 lock in share mode;
     ```
  ```
     
  在session2中更新操作：
     
     ```sql
     set autocommit = 0;
  update person_info_large set title='test4' where id = 4;
  ```
  
     此时我们顺利执行了id=4的数据，该行数据并没有被阻塞。
  
  5. 实验四：先写后读
  
     在session2中执行更新操作：
  
     ```sql
     set autocommit = 0;
     update art_article set title = title where id = 4181;
     ```
  
     在session1中执行查询操作：
  
     ```sql
     set autocommit = 0;
     select * from art_article where id = 4181;
     ```
  
     此时我们可以从数据库中查询到数据。
  
     如果我们在session1中显式的添加共享锁：
  
     ```sql
     set autocommit = 0;
     select * from art_article where id = 4181 lock in share mode;
     ```
  
     此时只有session2中的数据commit之后，session1中的查询才能执行。
  
  6. 实现五：先读后读
  
     在session1中执行：
  
     ```sql
     set autocommit = 0;
     select * from art_article where id = 4181;
     ```
  
     在session2中执行：
  
     ```sql
     set autocommit = 0;
     select * from art_article where id = 4181;
     ```
  
     他们都不会阻塞，能够顺利的进行。
  
## 第三章 Redis

### 1.Redis的五种数据结构

* String：最基础的数据类型，二进制安全
  * `set`
  * `get`
  * `incr`：将值为数值类型的键，每次加1，`incr key`
* Hash：String元素组成的字典，使用用于存储对象
  * `hmset`：`hmset map filed1 value1 field2 value2`设置哈希表map的多个域与值
  * `hset`
  * `hget`
* List：列表，按照String元素插入顺序排序（类似栈，后进先出）
  * `lpush`
  * `lrange`：`LRANGE key start stop` 返回存储在 key 的列表里指定范围内的元素 
* Set：String元素组成的无需集合，通过哈希表实现，不允许重复
  * `sadd`
  * `smembers`： 返回key集合所有的元素 
* Sorted Set：通过分数来为集合中的成员进行从小到大的排序
  * `zadd`
  * `zrangebyscore`：返回有序集合中指定分数区间内的成员，分数由低到高排序
* 其他：用于计数的HyperLogLog，用于支持存储地理位置信息的Geo

#### Redis底层数据类型

1. 简单动态字符串
2. 链表
3. 字典
4. 跳跃表
5. 整数集合
6. 压缩列表
7. 对象

### 2.从海量数据里查询某一固定前缀的key

注意：此时应先搞清楚有多少数据

#### 使用keys对线上的业务的影响

KEYS pattern：查找所有符合给定模式pattern的key

* KEYS指令一次性返回所有匹配的key
* 健的数量过大会使服务卡顿

#### SCAN cursor [MATCH pattern] [Count count]

```
scan 0 match k1* count 10
```

* 基于游标的迭代器，需要基于上一次的游标延续之前的迭代过程
* 以0作为游标开始一次新的迭代，知道命令返回游标0完成一次遍历
* 不保证每次执行都返回某个给定数量的元素，有可能返回0个，支持模糊查询
* 一次返回的数量不可控，只能是大概率符合count参数

### 3.如何通过Redis实现分布式锁

#### 分布式锁需要解决的问题

* 互斥性：同一时刻，只能有一个客户端能够获取锁
* 安全性：锁只有拥有该锁的客户端才能删除
* 死锁：避免死锁
* 容错：当redis节点宕机的情况下，客户端仍然能够获取锁

#### SETNX key value：如果key不存在，则创建并赋值

* 时间复杂度：O(1)
* 返回值：设置成功，返回1；设置失败，返回0。

#### 如何解决SETNX长期有效的问题

* EXPIRE key seconds

  * 设置key的生存时间，当key过期时（生存时间为0），会自动删除
  * 缺点：原子性得不到满足

* 分布式锁实现的伪代码

  ```java
  RedisService redisService = SpringUtils.getBean(RedisService.class);
  long status = redisService.setnx(key, "1");
  
  if(status == 1){
      redisService.expire(key, expire);  // 在执行这句的时候该服务就宕机了，那redis中的key就不会自动删除，原子性得不到满足
      // 执行独占资源逻辑
      doOcuppiedWork();
  }
  ```

#### SET key value [EX seconds] [PX milliseconds] [NX|XX]

* EX second：设置键的过期时间为second秒

* PX millisecond：设置键的过期时间为millisecond毫秒

* NX：只在键不存在时，才对键进行设置操作

* XX：只在键已经存在时，才对键进行设置操作

* SET操作成功完成时，返回OK，否则返回nil

* 伪代码：

  ```java
  RedisService redisService = SpringUtils.getBean(RedisService.class);
  long status = redisService.set(key, requestId, SET_IF_NOT_EXIST, SET_WITH_EXPIRE_TIME, expireTime);
  
  if("OK".equals(status)){
      redisService.expire(key, expire);
      // 执行独占资源逻辑
      doOcuppiedWork();
  }
  ```

#### 大量的key同时过期的注意事项

集中过期，由于清除大量的key很耗时，会出现短暂的卡顿现象

* 解决方案：在设置key的过期时间的时候，给每个key的时间加上随机值

### 4.如何使用Redis做异步队列

#### 使用List作为队列，RPUSH生产消息，LPOP消费消息

* 缺点：没有等待队列里有值就直接消费
* 弥补：可以通过在应用层引入Sleep机制去调用LPOP重试

#### Blpop key [key ...] timeout：阻塞直到队列有消息或者超时

```
# 从testlist中取值，等待20秒，如果没有则返回超时
blpop testlist 20
```

* 缺点：只能提供一个消费者消费，不能一次发布多个消费。

#### pub/sub：主题订阅者模式

* 发送者(pub)发送消息，订阅者(sub)接收消息

* 订阅者可以订阅任意数量的频道

  ```
  session1订阅mytopic:
  subscribe mytopic
  
  session2订阅mytopic：
  subscirbe mytopic 
  
  session3订阅mytopic:
  subscirbe othertopic
  
  session4发布消息到mytopic，此时只要session1和session2能够接收到这条消息：
  publish mytopic "hello"
  ```

* 缺点：消息的发布是五状态的，无法保证可达。

### 5.持久化方式之RDB

#### RDB（快照）持久化：保存某个时间点的全量数据快照

配置在redis.conf中：

```
# 在900秒之内有一次写入就触发一次快照
save 900 1
save 300 10
save 60 10000
# 禁用rdb配置
save ""

# 当备份进程出错的时候，主进程停止新的内容的写入操作。
stop-writes-on-bgsave-error yes

# 将rdb文件压缩之后采去做保存，建议设置为no，因为这样会加大对cpu的消耗
rdbcompression yes
```

快照文件：dump.rdb

RBD文件的创建方式：

* SAVE：阻塞Redis的服务器进程，直到RDB文件被创建完成。
  * `lastsave`：上次执行save操作的时间
* BGSAVE：Fork出一个子进程来创建RDB文件，不阻塞服务器进程。（异步备份）

#### 自动化触发RDB持久化的方式

* 根据redis.conf配置里的SAVE m n定时触发（用的是BGSAVE）
* 主从复制时，主节点自动触发
* 执行Debug Reload
* 执行Shutdown且没有开启AOF持久化

#### BGSAVE原理

* 系统调用fork()：创建进程，实现Copy-on-Write写时复制优化策略

* Copy-on-Write

  如果有多个同时要求相同资源（如内存或磁盘上的数据存储），他们会共同获取相同的指针指向相同的资源，直到某个调用者试图修改资源的内容时，系统才会真正复制一份专用副本给该调用者，而其他调用者所见到的最初的资源仍然保持不变。

#### RDB持久化的缺点

* 内存数据的全量同步，数据量大会使得I/O操作严重影响性能
* 可能会因为Redis挂掉而丢失从当前至最后一次快照期间的数据

### 6.持久化方式之AOF

#### AOF(Append-Only-File)持久化：保存写状态

* 记录下除了查询以外的所有变更数据库状态的指令
* 以append的形式追加保存到AOF文件中（增量）

#### AOF的相关配置

在nginx.conf中进行相关的配置：

```
# 开启AOF，设置为yes即可。也可以使用命令开启：config set appendonly yes
appendonly no

# 配置AOF文件的位置：
appendfilename "appendonly.aof"

# 配置AOF文件的保存方式，主要有三种
# 1.如果缓存区的内容发生变化，就及时将缓存区的内容写入到AOF中
# appendfsync always
# 2.将缓存区的内容每隔一秒就写入到AOF文件里面，建议使用这种方式
appendfsync everysec
# 3.让操作系统去确定写入的时机，一般操作系统会等待缓存区填满之后才往AOF写入数据
# appendfsync no
```

#### Redis日志重写

Redis可以不中断主进程的情况下，进行日志重写。日志重写解决了AOF文件大小不断增大的问题，原理如下：

* 调用fork()，创建一个子进程
* 子进程把新的AOF写到一个临时文件里，不依赖原来的AOF文件
* 主进程持续将新的变动同时写到内存和原来的AOF里
* 主进程获取子进程重写AOF的完成信号，往新AOF同步增量变动
* 使用新的AOF文件替换掉旧的AOF文件

#### Redis数据的恢复

RDB和AOF文件共存情况下的恢复流程

#### RDB和AOF的优缺点

* RDB优点：全量数据快照，文件小，恢复快
* RDB缺点：无法保存最近一次快照之后的数据
* AOF优点：可读性高，适合保存增量数据，数据不易丢失
* AOF缺点：文件体积大，恢复时间长

#### RDB-AOF混合持久化方式

* BGSAVE做镜像全量持久化，AOF做增量持久化

### 7.Pipeline及主从同步

#### 使用Pipeline和Linux的管道类似

* Pipeline和Linux的管道类似
* Redis基于请求/响应模型，单个请求处理需要一一应答
* Pipeline批量执行指令，节省多次IO往返的时间
* 有顺序依赖的指令建议分配发送

#### Redis的同步机制

* 主从同步原理

  master与多个slave之间的数据要达到最终一致性。

* 全同步过程：

  * Slave发送sync命令到Master
  * Master启动一个后台进程，将Redis中的数据快照保存到文件中
  * Master将保存数据快照期间接收到的写命令缓存起来
  * Master完成写文件操作后，将该文件发送给Slave
  * 使用新的AOF文件替换旧的AOF文件
  * Master将这期间收集的增量写命令发送给Slave端

* 增量同步过程

  * Master接收到用户的操作指令，判断是否需要传播到Salve
  * 将操作记录追加到AOF文件
  * 将操作传播到其他Slave：
    * 1.对齐主从库
    * 2.往响应缓存写入指令
  * 将缓存中的数据发送给Slave

#### Redis Sentinel哨兵

* 监控：检查主从服务器是否运行正常
* 提醒：通过API向管理员或者其他应用程序发送故障通知
* 自动故障迁移：主从切换

#### 流言协议Gossip

在杂乱无章中寻求一致

* 每个节点都随机地与对方通信，最终所有节点的状态达成一致
* 种子节点定期所及向其他节点发送节点列表以及需要传播的消息
* 不保证信息一定会传递给所有节点，但是最终会趋于一致

### 8.Redis集群

#### 如何从海量数据里快速找到所需？

* 分片：按照某种规则去划分数据，分散存储在多个节点上
* 常规的按照哈希划分无法实现节点的动态增减

#### 一致性哈希算法

* 对2的32次取模，将哈希值空间组织成虚拟的圆环
* 将数据key使用相同的函数Hash计算出哈希值
* 当其中的一台节点宕机后，受影响的数据只有该节点逆时针到上一个节点的数据，其他数据不受影响。
* 当新增一台节点后，受影响的数据只有新节点逆时针到上一个节点之间的数据，其他数据不受影响。

#### Hash环的数据倾斜问题

引入虚拟节点解决数据倾斜问题。

### 9.Redis客户端Jedis与Lettuce

#### Jedis客户端

1. Core: 核心模块，实现RESP协议
   - Jedis/BinaryJedis: 入口类，封装Redis的各种命令。
   - Client/BinaryClient/Connection: 与Redis进行具体的交互工作。
   - Protocol, RedisInputStream, RedisOutputStream: 实现RESP协议。
2. Sharding: 提供Partitioning支持
   *  ShardedJedis/BinaryShardedJedis: 首先对传入的Key进行Hash计算（默认使用高性能、低碰撞率的MurmurHash算法），然后根据计算结果找到相应的Jedis实例，最后执行命令。 
3. Pool: 提供连接池和Sentinel支持
   * JedisPool: 基于Apache Commons Pool实现的连接池，通过JedisFactory获取Jedis实例。
   * JedisSentinelPool: 通过侦听”switch-master”事件，每当master切换时，调用JedisFactory重新初始化master连接信息。
   * ShardedJedisPool: 与JedisPool类似，通过ShardedJedisFactory获取ShardedJedis实例。
4. Pipeline: 提供Pipelining和事务支持
   * Pipeline: 通过Jedis#pipelined()获取实例。以类型安全的方式获取执行结果，通过BuilderFactory将Object类型的Response转化为期望的结果类型。
     - 非事务模式：构建Response Queue，然后通过Client#getMany()批量获取结果。
     - 事务模式：通过MultiResponseBuilder缓存Response，然后批量获取结果。
   * Transaction: 通过Jedis#multi()获取实例。天然的事务属性，通过Client#getMany()批量获取结果，但无法获取单条命令的结果，且类型非安全。
   * ShardedJedisPipeline: ShardedJedis#pipelined()获取实例。不同于Pipeline和Transaction，由于请求可能落到多个Client上，只能通过Client#getOne()挨个获取结果，类型非安全。
5. Cluster: 提供Cluster支持
   * JedisCluster/BinaryJedisCluster: 通过JedisClusterConnectionHandler获取Jedis实例，然后执行命令。
   * JedisClusterConnectionHandler & JedisClusterInfoCache: 通过Collections#shuffle()随机返回一个Jedis实例。使用ReentrantReadWriteLock保证更新Cluster的Jedis实例列表时的线程安全性。
   * JedisClusterCommand: 通过retry机制获取有效的Jedis实例，然后再执行命令。

#### Lettuce客户端

 Lettuce 是 一种可伸缩，线程安全，完全非阻塞的Redis客户端，多个线程可以共享一个RedisConnection,它利用Netty NIO 框架来高效地管理多个连接，从而提供了异步和同步数据访问方式，用于构建非阻塞的反应性应用程序。 

目前Lettuce已经是Spring-Data-redis中默认的底层实现。

* 支持`Redis`的新增命令`ZPOPMIN, ZPOPMAX, BZPOPMIN, BZPOPMAX`。

* 支持通过`Brave`模块跟踪`Redis`命令执行。

* 支持`Redis Streams`。

* 支持异步的主从连接。

* 支持异步连接池。

* 新增命令最多执行一次模式（禁止自动重连）。

* 全局命令超时设置（对异步和反应式命令也有效）。

