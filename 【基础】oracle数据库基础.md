# Oracle数据库学习

## 一、Oracle数据库开发必备利器之SQL基础

### 1.Oracle的安装与卸载

* 安装

  在Oracle官网下载安装oracle11g2版本，安装在D:/oracle目录下。用户名有sys，system；密码为fx1212。

* 卸载

  执行D:\program_file\oracle\product\11.2.0\dbhome_1\deinstall下的deinstall.bat文件。

### 2.用户和表空间

* 用户

  * 登录SQL Plus

    * 系统用户

      * sys：权限大于system，必须使用系统管理员登录

      * system：可以直接登录

      * sysman：操作企业管理器的用户

      * scott

        前三个用户的密码在安装是统一设置，scott用户密码为tiger。在sys，system，sysman，scott四个用户权限中，scott用户最低。

    * 使用系统用户登录

      * 使用system登录

        \[username/password]\[@server]\[as sysdba|sysoper]

        例如：system/root @orcl as sysdba，其中orcl就是自己设置的服务名

      * 命令：打开sqlPlus输入`system/root `，因为数据库和服务器在同一机器，所以不需要写服务名

      * 命令：打开sqlPlus输入`sys/root as sysdba`，在sql环境中使用`connect sys/root as sysdba`

  * 查看登录用户

    * `show user`命令，不需要分号结尾
    * 通过数据字典查看其它的用户信息
      * dba_users数据字典，数据字典是数据库提供的表，用户查看数据库的信息
      * `desc dba_users`查看用户表
      * `select username from dba_users`

  * 启用scott用户

    * 启用用户语句

      `alter user [name] account unlock`

      例如：`alter user scott accout unlock`

    * 使用scott登录sql Plus：`scott/tiger`
  
* 表空间
  
  * 表空间概述
  
    * 表空间与数据库之间的关系：表空间是数据库的逻辑存储空间，我们可以把表空间理解为在数据库的开辟的空间，用户存放数据库的对象。一个数据库可以由多个表空间构成。
    * 表空间与数据文件的关系：表空间是由一个或多个数据文件构成的。数据文件的位置和大小可以由用户自己来定义。我们存放的数据、表和其他的数据对象都是存放在表空间的数据文件中的。
  
  * 表空间的分类
  
    * 永久表空间
  
      存放的是数据库当中要永久化存储的对象，例如表、视图、存储过程
  
    * 临时表空间。
  
      存放数据库操作当中中间执行的过程，当操作结束后存放的内容就会被自动的释放掉，不进行永久化的存储。
  
    * UNDO表空间
  
      保存数据所修改数据的旧值，也就是被修改之前的数据。方便我们对数据进行回滚。
  
  * 查看用户的表空间
  
    不同用户登录oracle所拥有的表空间也不同。
  
    * dba_tablespaces数据字典
  
      针对系统用户来查看的数据字典
  
      查看数据字典字段：`desc dba_tablespaces`
  
      ​								`select tablespace_name from dba_tablespaces;`
  
    * user_tablespaces数据字典
  
      针对普通用户登录之后查看的数据字典
  
    * dba_users数据字典
  
      针对系统用户来查看的数据字典
  
    * user_users数据字典
  
      针对普通用户登录之后查看的数据字典
  
  * 设置用户的默认或临时表空间（普通用户没有修改表空间的权限）
  
    > 在Oracle数据库安装完成后，system用户的默认表空间和临时表空间分别是system,temp
  
    **ALTER USER** username 
  
    **DEFAULT|TEMPORARY**
  
    **TABLESPACE** tablespace_ame
  
    例如修改system表空间为system：`ALTER USER system DEFAULT TABLESPACE system`
  
  * 创建、修改、删除表空间
  
    * 创建表空间：
  
      **CREATE [TEMPORARY] TABLESPACE**
  
      tablespace_name
  
      **TEMPFILE|DATAFILE** 'xx.dbf' **SIZE** xx
  
      例子：
  
      * 创建永久表空间：`create tablespace test1_tablespace datafile 'testfile.dbf' size 10m;`
      * 创建临时表空间：`create temporary tablespace temtest1_tablespace tempfile 'tempfile1.dbf' size 10m;`
      * 查看永久表空间信息：
        * 使用命令`desc dba_data_files`查看存储表空间的表的字段
        * 使用命令`select file_name from dba_data_files where tablespace_name='TESTFILE1_TABLESPACE';`查看我们刚才常见的表空间所存储的位置，注意表空间名称为大写
      * 查看临时表空间信息：
        * 使用命令`desc dba_temp_files`查看存储表空间的表的字段
          * 使用命令`select file_name from dba_temp_files where tablespace_name='TEMPFILE1_TABLESPACE';`查看我们刚才常见的表空间所存储的位置，注意表空间名称为大写
  
    * 修改表空间的状态
  
      * 设置联机或是脱机状态：**ALTER TABLESPACE** tablespace_name **ONLINE|OFFLINE**;
  
        创建一个表空间之后他的默认状态是联机状态。
  
        如果一个表空间设置为脱机状态的话，我们就不能使用它了。
  
        例如：`alter tablespace test1_tablespace offline`；
  
        通过`select status from dba_tablespaces where tablespace_name='TEST1_TABLESPACE'; `查看表空间的状态是联机还是脱机
  
      * 设置只读或可读写状态：**ALTER TABLESPACE** tablespace_name **READ ONLY|READ WRITE**（设置该属性的前提是表空间是联机状态的）
  
        默认联机状态就是读写状态。
  
        例如：`alter tablespace test1_tablespace read only`
  
        通过`select status from dba_tablespaces where tablespace_name='TEST1_TABLESPACE'; `查看表空间的状态是否是只读状态
  
    * 修改数据文件
  
      * 增加数据文件
  
        **ALTER TABLESPACE** tablespace_name
  
        **ADD DATAFILE** 'xxx.dbf' **SIZE** xx;
  
        例如：`alter tablespace test1_tablespace add datafile 'test2_file.dbf' size 10m;`
  
        通过命令`select file_name data_data_files where tablespace_name='TEST1_TABLESPACE'`查看我们刚才常见的表空间所存储的位置有哪些
  
      * 删除数据文件
  
        **ALTER TABLESPACE** tablespace_name **DROP DATAFILE **'filename.dbf'
  
        注意：不能删除我们创建表空间时的第一个数据文件。如果要删除的话，我们需要把整个的表空间删除。
  
        例如：`alter tablespace test1_tablespace drop datafile 'test2_file.dbf';`
  
    * 删除表空间
  
      **DROP TABLESPACE**
  
      tablespace_name **[INCLUDEING CONTENTS]**
  
      当你删除表空间时要把相应的数据文件也要删除的话应该加上INCLUDEING CONTENTS。
  
### 3.管理表

#### 3.1认识表

* 表是基本存储单位
* 是二维结构
* 包括行和列
* 表的约定
  * 每一列数据必须具有相同数据类型
  * 列名唯一
  * 每一行数据的唯一性

#### 3.2数据类型

* 字符型

  * 固定长度类型：CHAR(n)，NCHAR(n)
  * 可变长度：VARCHAR2(n)其中n最大为4000，NVARCHAR(n)其中n最大为2000

* 数值型

  * NUMBER(p,s)其中p为有效数字，s为小数点后的位数。s若正数他表示小数点到最低有效数字的位数，若为负数他表示最高有效数字到小数点的位数

    例如：NUMBER(5,2)表示有效数字的为5位，保留两位小数，如123.45

  * FLOAT(n)

* 日期型

  * DATE：DATE表示的范围公元前4712年到公元9999年12月31日，他可以精确到秒
  * TIMESTAMP：时间戳，他可以精确到小数秒

* 其他类型

  * BLOB：可以保存4G二进制形式存储的数据
  * CLOB：可以保存4G字符创形式存储的数据

#### 3.3管理表

* 创建表的基本语法：

  CREATE TABLE table_name

  (

  ​	column_name datatype,....

  )

  注意：同一登录数据库用户的表名是唯一的

* 实例

  ```
  create table userinfo
  (
  	id number(6,0),
  	username varchar2(20),
  	userpwd varchar2(20),
  	email varchar2(30),
  	regdate date
  )
  ```

  查看表描述：`desc userinfo`

#### 3.4管理表之修改表

#### 3.5管理表之删除表

## 二、Oracle数据库开发必备利器之PL/SQL基础

### 1.PL/SQL的安装和介绍



​    

​    