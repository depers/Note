# Java web环境配置

## 1. Java配置

1. 清除系统默认自带jdk

   `rpm -qa|grep jdk`搜索已经安装的jdk；

   卸载命令：`yum remove xxx`或`rpm -e --nodeps xxx`

2. 安装OracleJDK

   解压JDK压缩包：`tar -zxvf jdk...`

3. 配置JDK环境变量

   * `vim /etc/profile`

   * 添加Java环境变量：

     ```
     export JAVA_HOME=/usr/local/webapp/software/jdk1.8.0_151
     export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
     export PATH=$PATH:$JAVA_HOME/bin
     ```
     
   * `source /etc/profile`

4. 验证安装成功：`java -version`

## 2. Tomcat配置(以两个tomcat的配置为例)

1. 解压：`tar -zxvf apache-tomcat-8.5.24.tar.gz -C ./../javaweb/`

2. 修改第二个tomcat的server.xml文件

   ```xml
   <Server port="9005" shutdown="SHUTDOWN">
   
   <Connector port="9080" protocol="HTTP/1.1"
                  connectionTimeout="20000"
                  redirectPort="8443" URIEncoding="UTF-8"/>
   
   <Connector port="9009" protocol="AJP/1.3" redirectPort="8443" />
   ```

3. 打开第二个tomcat目录bin下catalina.sh，即${tomcat}/bin/catalina.sh。找到`# OS specific support. $var _must_ be set to either true or false.`，在这行下面编辑，新增配置，保存退出：

   ```co
   export CATALINA_BASE=$CATALINA_2_BASE
   export CATALINA_HOME=$CATALINA_2_HOME
   ```

4. 分别进入两个tomcat的bin目录，启动tomcat，即进入${tomcat}/bin/，执行start.sh。访问[http://localhost:8080]()，[http://localhost:9080]()可以打开tomcat部署的webapps的ROOT项目首页

## 3. MySQL安装（Centos6）

1. 检查是否安装了MySQL：`rpm -qa|grep mysql-server`

2. 若本机没有安装，需到官网下载rpm文件进行安装，安装命令`rpm -Uvh *.rpm --nodeps --force`

3. 安装之后启动mysql，命令`service mysqld start`，接着使用`sudo grep 'temporary password' /var/log/mysqld.log`查看临时密码。然后使用`mysql -uroot -p[temporpary password]`登录，直接进行第7步的配置工作

4. 若在/var/log/mysqld.log没有临时密码，则在/etc/my.conf添加skip-grant-tables，可以不用密码直接登录数据库。如下是my.conf文件：

   ```
   [mysqld]
   datadir=/var/lib/mysql
   socket=/var/lib/mysql/mysql.sock
   user=mysql
   # Disabling symbolic-links is recommended to prevent assorted security risks
   symbolic-links=0
   # fengxiao add
   default-character-set=utf8
   #############
   [mysqld_safe]
   log-error=/var/log/mysqld.log
   pid-file=/var/run/mysqld/mysqld.pid
   skip-grant-tables
   ```

5. 启动MySQL：`service mysqld start`。但是这时报错了：Mysql报错Fatal error: Can't open and lock privilege tables: Table 'mysql.host' doesn't exist。

   解决办法：命令`mysql_install_db`，再启动MySQL就正常了。

7. 输入`sudo mysql -uroot -p`直接进入开始操作数据库，设置管理员和权限。下面是sql：

   ```
   select user,host,password from mysql.user;
   delete from mysql.user where user='';
   
   // 创建用户方法一：该方法创建的用户只有连接数据库权限
   create user '[username]'@'%' identified by '[password]'; // %指任意ip
   
   // 创建用户方法二：insert user
   insert into user (host,user,password) values (’%’,‘john’,password(‘123’));
   
   // 创建用户方法三：grant，若grant命令中的用户若不存在会新创建，若存在则直接授权
   GRANT [权限] ON [数据库名.表名] to [username@hostname] [IDENTIFIED BY ‘PASSWORD’] [WITH GRANT OPTION];
   * 权限：select,insert,update,delete,create,drop,index,alter,grant,references,reload,shutdown,process,file等权限，或者all privileges 为全部权限
   
   // 设置密码
   set password for root@localhost=password('fx1212')
   set password for root@127.0.0.1=password('fx1212')
   
   // 赋权
   GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'fx1212' WITH GRANT OPTION;
   flush privileges;
   ```

8. 重启服务器：`service mysqld restart`

9. 配置防火墙：`vim /etc/sysconfig/iptables`

   ```
   -A INPUT -m state --state NEW -m tcp -p tcp --dport 22 -j ACCEPT
   -A INPUT -m state --state NEW -m tcp -p tcp --dport 21 -j ACCEPT
   -A INPUT -m state --state NEW -m tcp -p tcp --dport 81 -j ACCEPT
   -A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
   -A INPUT -m state --state NEW -m tcp -p tcp --dport 3306 -j ACCEPT
   -A INPUT -m state --state NEW -m tcp -p tcp --dport 8080 -j ACCEPT
   -A INPUT -m state --state NEW -m tcp -p tcp --dport 8081 -j ACCEPT
   ```

10. 重启防火墙：`service iptables restart`

11. mysql自启动设置：1）执行`chkconfig mysqld on`，2）执行`chlconfig --list mysqld`查看，如果2-5位启用on状态即成功。

### 1. MySQL安装过程中出现的问题

1. 在/var/log/mysqld.log文件中出现错误日志：/usr/sbin/mysqld: Can't find file: './mysql/plugin.frm' (errno: 13)

   输入以下命令解决：

   ```shell
   # cd /var/lib/mysql/mysql
   
   // 利用 chown 将指定文件的拥有者改为指定的用户或组
   // 指定该目录下所有文件的拥有者为mysql
   # chown mysql *
   
   // Linux chgrp命令用于变更文件或目录的所属群组。
   // 指定该目录下所有文件的所属群(用户组)为mysql
   # chgrp mysql *
   
   // Linux/Unix 的文件调用权限分为三级 : 文件拥有者、群组、其他。利用 chmod 可以藉以控制文件如何被他人所调用。
   // 设置该目录下所有文件的拥有者(u)和群组(g)，加权限(+)，r 表示可读取，w 表示可写入，x 表示可执行
   # chmod ug+rwx *
   
   # service mysqld start
   ```

## 4. Nginx配置

1. 安装Nginx依赖：`yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel`

2. 解压Nginx：`tar -zxvf nginx`

3. 创建nginx临时目录：`mkdir -p /var/temp/nginx `

4. 创建Makefile文件：`./configure --prefix=/usr/local/myapp/program/nginx`

   下面是更加具体的配置：

   ```sh
   ./configure \
   --prefix=/usr/local/myapp/program/nginx \
   --pid-path=/var/run/nginx/nginx.pid \
   --lock-path=/var/lock/nginx.lock \
   --error-log-path=/var/log/nginx/error.log \
   --http-log-path=/var/log/nginx/access.log \
   --with-http_gzip_static_module \
   --http-client-body-temp-path=/var/temp/nginx/client \
   --http-proxy-temp-path=/var/temp/nginx/proxy \
   --http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
   --http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
   --http-scgi-temp-path=/var/temp/nginx/scgi \
   --add-module=/usr/local/myapp/software/fastdfs/fastdfs-nginx-module-1.22/src
   ```

   配置命令：

   | 命令                           | 说明                                 |
   | ------------------------------ | ------------------------------------ |
   | --prefix                       | 指定nginx安装目录                    |
   | --pid-path                     | 指向nginx的pid                       |
   | --lock-path                    | 锁定安装文件，防止被恶意篡改或误操作 |
   | --error-log-path               | 配置错误日志路径                     |
   | --http-log-path                | 配置http日志路径                     |
   | --with-http_gzip_static_module | 启动gzip模块，在线实时压缩输出数据流 |
   | --http-client-body-temp-path   | 设定客户端请求的临时目录             |
   | --http-proxy-temp-path         | 设定http代理临时目录                 |
   | --http-fastcgi-temp-path       | 设定fastcgi临时目录                  |
   | --http-uwsgi-temp-path         | 设定uwsgi临时目录                    |
   | --http-scgi-temp-path          | 设定scgi临时目录                     |
   | --add-module                   | 添加拓展模块                         |

5. 编译：`make`

6. 安装：`make install`

7. Nginx常用命令：
   * 测试配置文件：`./nginx -t`
   * 启动：`./nginx`
   * 停止：`./nginx -s stop`或`./nginx -s quit`
   * 重启：`./nginx -s reload`
   * 查看进程：`ps -ef|grep nginx`
   * 平滑重启：`kill -HUP [Nginx主进程号（即查看到进程命令查到的PID）]`

## 5. Redis配置

修改第二个redis的redis.conf文件的端口为6380

## 6. MariaDB安装

* 官网安装文档：https://mariadb.com/kb/en/mariadb-installation-version-10121-via-rpms-on-centos-7/

* 具体如下图：

  ![](E:\markdown笔记\笔记图片\20\1\17.png)

* 其中赋予root用户远程连接权限时，输入小写的grant命令时不起作用不知道为什么。若出现这种情况请输入：`GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'IDENTIFIED BY 'fx1212' WITH GRANT OPTION;`

## 7. RabbitMQ安装 

1. 安装基础软件包

   ```
   yum -y install build-essential openssl openssl-devel unixODBC unixODBC-devel make gcc gcc-c++ kernel-devel
   ```

2. 安装Erlang，这里我们使用Zero-dependency Erlang RPM for RabbitMQ

    ```
   rpm -ivh erlang-23.2.3-1.el7.x86_64.rpm
   ```

3. 安装socat和logrotate

   ```
   rpm -ivh socat-1.7.3.2-2.el7.x86_64.rpm
   rpm -ivh socat-1.7.3.2-2.el7.x86_64.rpm
   ```

4. 安装rabbitmq

   ```
   rpm -ivh rabbitmq-server-3.8.11-1.el7.noarch.rpm
   ```

5. 创建/etc/rabbitmq/rabbitmq.conf文件，修改用户登录与连接心跳检测，注意修改

   * 修改点1：设置loopback_users.guest为none，允许guest账户非本地登录
   * 修改点2：heartbeat 为10（用于心跳连接）

   ```
   # DANGER ZONE!
   #
   # allowing remote connections for default user is highly discouraged
   # as it dramatically decreases the security of the system. Delete the user
   # instead and create a new one with generated secure credentials.
   loopback_users.guest = none
   
   ## Set the default AMQP 0-9-1 heartbeat interval (in seconds).
   ## Related doc guides:
   ##
   ## * https://rabbitmq.com/heartbeats.html
   ## * https://rabbitmq.com/networking.html
   ##
   heartbeat = 10
   ```

6. 首先启动服务(后面 | 包含了停止、查看状态以及重启的命令)

   ```
   /etc/init.d/rabbitmq-server start | stop | status | restart
   
   service rabbitmq-server start
   ```

7. 开启web界面管理插件

   ```
   rabbitmq-plugins enable rabbitmq_management
   ```

   这里会报一个错：`{:query, :"rabbit@app-fastdfs_tracker-rabbitmq", {:badrpc, :timeout}}`，这里是因为我的hostname没有在/etc/hosts中进行配置，这里添加：

   ```
   127.0.0.1   app-fastdfs_tracker-rabbitmq
   ```

8. 查看服务有没有启动：`lsof -i:5672` （5672是Rabbit的默认端口）

9. 可查看管理端口有没有启动： 

   ```
   lsof -i:15672 或者 netstat -tnlp | grep 15672
   ```

10. 在浏览器登录web管理端，`http://192.168.156.135:15672/`

## 8. Kafka的安装

1. 下载Kafka：http://kafka.apache.org/downloads

2. 解压命令：`tar -zxvf kafka_2.12-2.1.0.tgz -C /usr/local/`

3. 改名命令：`mv kafka_2.12-2.1.0/ kafka_2.12`

4. 进入解压后的目录，修改server.properties文件：`vim /usr/local/kafka_2.12/config/server.properties`

5. 修改server.properties文件：

   ```
   broker.id=0
   listeners=PLAINTEXT://9092
   host.name=192.168.156.135
   advertised.host.name=192.168.156.135
   log.dirs=/usr/local/kafka_2.12/kafka-logs
   num.partitions=2
   zookeeper.connect=192.168.11.111:2181,192.168.11.112:2181,192.168.11.113:2181
   ```

6. 建立日志文件夹：`mkdir /usr/local/kafka_2.12/kafka-logs`

7. 启动Kafka之前需要启动Zookeeper

8. 启动Kafka：`/usr/local/kafka_2.12/bin/kafka-server-start.sh /usr/local/kafka_2.12/config/server.properties &`
## 9. FileBeat的安装

1. 解压：`tar -zxvf filebeat-7.8.1-linux-x86_64.tar.gz -C ../program/`
2. 重命名：`mv filebeat-7.8.1-linux-x86_64/ filebeat-7.8.1`
3. 配置filebeat，可以参考filebeat.full.yml中的配置
4. 测试配置文件是否正常：`./filebeat test config -c filebeat.yml`
5. 启动filebeat：`/usr/local/filebeat-6.6.0/filebeat &`
6. 查看线程：`ps -ef | grep filebeat`

## 10. Centos自启动配置

修改/etc/rc.local文件

```
#!/bin/sh
#
# This script will be executed *after* all the other init scripts.
# You can put your own initialization stuff in here if you don't
# want to do the full Sys V style init stuff.

touch /var/lock/subsys/local

# start app
export JAVA_HOME=/usr/local/webapp/software/jdk1.8.0_211

# 1.redis start
/usr/local/webapp/javaweb/redis_1/src/redis-server &
/usr/local/webapp/javaweb/redis_2/src/redis-server /usr/local/webapp/javaweb/redis_2/redis.conf &

# 2.tomcat start
/usr/local/webapp/javaweb/mall.bravedawn.cn/bin/startup.sh start
/usr/local/webapp/javaweb/mall2.bravedawn.cn/bin/startup.sh start

# 3.nginx start
/usr/local/webapp/nginx/sbin/nginx
```

## 11. Maven配置

1. 下载maven的二进制压缩包，并解压

2. 修改软件源，在conf/setting.xml文件的mirrors节点添加以下内容：

    ```xml
    <mirror>
        <id>alimaven</id>
        <mirrorOf>central</mirrorOf>
        <name>aliyun maven</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>
    ```

3. 修改本地仓库目录

    ```xml
    <localRepository>/Users/depers/Desktop/package/apache-maven-3.9.1/repository</localRepository>
    ```

## 12. Mysql8在centos7上的安装

1. 检查是否安装了MySQL：`rpm -qa|grep mysql-server`
2. 若本机没有安装，需到官网下载rpm文件进行安装，安装命令`rpm -Uvh *.rpm --nodeps --force`

3. 安装之后启动mysql，命令`service mysqld start`，接着使用`sudo grep 'temporary password' /var/log/mysqld.log`查看临时密码。然后使用`mysql -uroot -p[temporpary password]`登录

4. 使用临时密码登录MySQL，重置root密码

    ```sql
    alter user 'root'@'localhost' identified by 'Fengxiao17393164120-+';
    ```

5. 修改MySQL密码策略

    ```sql
    -- 查看密码配置
    SHOW VARIABLES LIKE 'validate_password%';
    
    -- 修改密码策略为较低
    set global validate_password_policy=0;
    
    -- 重新设置一个简单的密码
    alter user 'root'@'localhost' identified by 'fx12345678';
    ```

6. 为用户授权

    ```sql
    -- 查看用户是否有grant授权的权限
    select user ,grant_priv, host from user ;
    
    -- 更新用户主机名字段
    update user set host='%' where user='root' ;
    
    -- [数据库.数据表] TO [用户名@hostname]
    GRANT ALL PRIVILEGES ON *.* TO 'root'@'%';
    
    flush privileges;
    
    -- 创建其他的用户(MySQl8将用户的创建和授权分开了)
    #创建账户
    create user 'root'@'%' identified by '123456';
    
    -- 修改密码
    ALTER user 'depers'@'%' IDENTIFIED BY 'fengxiao1313';
    
    #赋予权限
    grant all privileges on *.* to 'root'@'%' with grant option;
    
    #刷新
    flush privileges;
    ```

7. 重启mysql

