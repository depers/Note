# Java web环境配置

## Java配置

1. 清除系统默认自带jdk

   `rpm -qa|grep jdk`搜索已经安装的jdk；

   卸载命令：`yum remove xxx`

2. 安装OracleJDK

   解压JDK压缩包：`tar -zxvf jdk...`

3. 配置JDK环境变量

   * `vim /etc/profile`

   * 添加Java环境变量：

     ```
     export JAVA_HOME=/usr/local/webapp/software/jdk1.8.0_151
     export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
     export PATH=$JAVA_HOME/bin
     ```
     
   * `source /etc/profile`

4. 验证安装成功：`java -version`

## Tomcat配置(以两个tomcat的配置为例)

1. 解压：`tar -zxvf apache-tomcat-8.5.24.tar.gz -C ./../javaweb/`

2. 修改第二个tomcat的server.xml文件

   ```
   <Server port="9005" shutdown="SHUTDOWN">
   
   <Connector port="9080" protocol="HTTP/1.1"
                  connectionTimeout="20000"
                  redirectPort="8443" URIEncoding="UTF-8"/>
   
   <Connector port="9009" protocol="AJP/1.3" redirectPort="8443" />
   ```

3. 打开第二个tomcat目录bin下catalina.sh，即${tomcat}/bin/catalina.sh。找到`# OS specific support. $var _must_ be set to either true or false.`，在这行下面编辑，新增配置，保存退出：

   ```
   export CATALINA_BASE=$CATALINA_2_BASE
   export CATALINA_HOME=$CATALINA_2_HOME
   ```

4. 分别进入两个tomcat的bin目录，启动tomcat，即进入${tomcat}/bin/，执行start.sh。访问[http://localhost:8080]()，[http://localhost:9080]()可以打开tomcat部署的webapps的ROOT项目首页

## MySQL安装（Centos6）

1. 检查是否安装了MySQL：`rpm -qa|grep mysql-server`

2. 若本机没有安装，需到官网下载rpm文件进行安装，安装命令`rpm -Uvh *.rpm --nodeps --force`

3. 安装之后启动mysql，命令`service mysqld start`，接着使用`sudo grep 'temporpary password /var/log/mysqld.log`查看临时密码。然后使用`mysql -uroot -p[temporpary password]`登录，直接进行第7步的配置工作

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

7. 输入`sudo mysql uroot -p`直接进入开始操作数据库，设置管理员和权限。下面是sql：

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
   grant all privileges on *.* to 'root'@'%' identified by 'fx1212' with grant option;
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

## Nginx配置

1. 安装Nginx依赖：`yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel`
2. 解压Nginx：`tar -zxvf nginx`
3. 依次输入：`./configure --prefix=/usr/local/webapp/`，`make`，`make install`
4. Nginx常用命令：
   * 测试配置文件：`./nginx -t`
   * 启动：`./nginx`
   * 停止：`./nginx -s stop`或`./nginx -s quit`
   * 重启：`./nginx -s reload`
   * 查看进程：`ps -ef|grep nginx`
   * 平滑重启：`kill -HUP [Nginx主进程号（即查看到进程命令查到的PID）]`

## Redis配置

修改第二个redis的redis.conf文件的端口为6380

## Centos自启动配置

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





