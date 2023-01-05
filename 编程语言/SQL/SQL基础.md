#### 创建一个数据库

```sql
CREATE DATABASE IF NOT EXISTS db_test_1 DEFAULT CHARSET utf8 COLLATE utf8_general_ci;
```
```sql
SHOW CREATE DATABASE db_test_1;
```

# 创建一个数据表
## 创建一个新表
```sql
CREATE TABLE `mytable1` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `col1` int(11) NOT NULL DEFAULT '1',
  `col2` varchar(45) DEFAULT NULL,
  `col3` date DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```
查看建表语句：

```sql
SHOW CREATE TABLE mytable;
```
## 依据现有表结构创建新表
### 只复制表结构
```
create table `mytable2` like `mytable`;
```
### 复制数据和表结构
```
create table `mytable2` select * from `mytable`;
```

# 插入

## 插入一条数据
```sql
INSERT INTO mytable(col1, col2) VALUES(1811942438, 'FENGXIAO'); --其中col1的长度定为11，其中一位为符号位，所以只能插入10位数字。
```

## 插入多条数据

```sql
INSERT INTO mytbale (col1, col2)
VALUES
(val1, val2),
(val1, val2);
```

## 插入检索出的数据

```sql
INSERT INTO mytable1(col1, col2)
SELECT col1, col2
FROM mytable2;
```

## 将一个表的内容复制到一个新表
```sql
CREATE TABLE newtable AS
SELECT * FROM mytable;
```

# 更新
```sql
UPDATE mytable SET col2 = 'xiaoming' WHERE id = 1;
```

# 删除
```sql
DELETE FROM mytable1 WHERE id = 1;
```

# 清空表
```sql
TRUNCATE TABLE newtable;
```

# 修改表

#### 添加列
```sql
ALTER TABLE mytable ADD col4 CHAR(20);
```

#### 修改列
```sql
ALTER TABLE test MODIFY col1 col_1 CHAR(20);
ALTER TABLE test CHANGE col1 col_1 CHAR(20);
```

#### 删除列
```sql
ALTER TABLE mytable DROP COLUMN col3;
```

# 删除表

```sql
DROP TABLE newtable;
```

# 查询

## DISTINCT 

去重查询关键字

```sql
SELECT DISTINCT col1 FROM mytable;
```

## LIMIT 

限制返回的行数。两个参数，第一个起始行，从0开始；第二个参数为返回的总函数；

* 返回前五行：

```sql
SELECT * FROM mytable LIMIT 5;

SELECT * FROM mytable LIMIT 0, 5;
```
* 返回3~5行：

```sql
SELECT * FROM mytable LIMIT 2, 5;
```

## 排序

* `ASC`：升序（默认）

* `DESC`：降序

可以按多个列进行排序，并且为每个列指定不同的排序方式：

```
SELECT * FROM mytable
ORDER BY col DESC, col2 ASC;
```

## 过滤

### 比较运算符

过滤条件就是在WHERE 后面要写的查询条件，主要有以下几类：

1. `=`、 `<`、 `>`、 等于 小于 大于

2. `<>`、`!= `不等于

3. `<=`、`!> `小于等于

4. `>=`、`!<` 大于等于

5. `BETWEEN...AND` 在两个值之间

6. `IS NULL `为NULL值

注意：`NULL`与 `0 `、空字符串的含义并不相同；

### 逻辑运算符

AND OR 用于连接多个过滤条件。优先处理 AND，因此当一个过滤表达式涉及到多个 AND 和 OR 时，应当使用 () 来决定优先级。
​    IN 操作符用于匹配一组值，其后也可以接一个 SELECT 子句，从而匹配子查询得到的一组值。
​    NOT 操作符用于否定一个条件。

--------------------通配符-------------------------
1）通配符也是用在过滤语句中，但它只能用于文本字段。

2）% 匹配 >=0 个任意字符，类似于 *；

3）_ 匹配 ==1 个任意字符，类似于 .；

4）[] 可以匹配集合内的字符，例如 [ab] 将匹配字符 a 或者 b。用脱字符 ^ 可以对其进行否定，也就是不匹配集合内的字符。

5）使用 Like 来进行通配符匹配。
```
SELECT *
FROM mytable
WHERE col LIKE '[^AB]%' -- 不以 A 和 B开头的任意文本
```
注意：不要滥用通配符，通配符位于开头处匹配会非常慢。


--------------------计算字段--------------------------
#### 在数据库服务器上完成数据的转换和格式化的工作往往比客户端上快得多，并且转换和格式化后的数据量更少的话可以减少网络通信量

#### 计算字段通常需要使用 AS 来取别名，否则输出的时候字段名为计算表达式。
```
SELECT col * col2 AS alias
FROM mytable;
```

#### Concat() 用于连接多个字段。许多数据库会使用空格把一个值填充为列宽，因此连接的结果会出现一些不必要的空格，使用 TRIM() 可以去除首尾空格。
```
SELECT Concat(TRIM(col1), '(', TRIM(col2), ')')
FROM mytable;
```

--------------------函数---------------------------

#### 各个 DBMS 的函数都是不相同的，因此不可移植。

```
/*------------1、文本处理函数-----------*/
LEFT() RIGHT()	左边或者右边的字符
LOWER() UPPER()	转换为小写或者大写
LTRIM() RTIM()	去除左边或者右边的空格
LENGTH()	    长度
SUNDEX()	    转换为语音值
```

#### LEFT()函数有两个参数，第一个是数据库字段，第二个是要截取的字段长度：
```
SELECT LEFT(col2, 5) FROM mytable;
```

#### 其中， SOUNDEX() 是将一个字符串转换为描述其语音表示的字母数字模式的算法，它是根据发音而不是字母比较。
```
SELECT * 
FROM mytable
WHERE SOUNDEX(col1) = SOUNDEX("apple");
```

/*------------2、日期和时间处理-----------*/
#### 日期格式：YYYY-MM-DD
#### 时间格式：HH:MM:SS

函 数|说 明
--|--
AddDate()|[增加一个日期（天、周等）] (http://www.w3school.com.cn/sql/func_date_add.asp)
AddTime()|增加一个时间（时、分等）
CurDate()|返回当前日期
CurTime()| 返回当前时间
Date()|返回日期时间的日期部分
DateDiff()|计算两个日期之差
Date_Add()|高度灵活的日期运算函数
Date_Format()|返回一个格式化的日期或时间串
Day()|返回一个日期的天数部分
DayOfWeek()|对于一个日期，返回对应的星期几
Hour()| 返回一个时间的小时部分
Minute()|返回一个时间的分钟部分
Month()| 返回一个日期的月份部分
Now()|返回当前日期和时间
Second()|返回一个时间的秒部分
Time()| 返回一个日期时间的时间部分
Year()| 返回一个日期的年份部分 

/*------------3、数值处理-----------*/

函数| 说明
--|--
SIN()|正弦
COS()|余弦
TAN()|正切
ABS()|绝对值
SQRT()|平方根
MOD()|余数
EXP()|指数
PI()|圆周率
RAND()|随机数

/*------------4、汇总-----------*/

函 数|  说 明
--|--
AVG()|返回某列的平均值
COUNT()|返回某列的行数
MAX()|返回某列的最大值
MIN()|返回某列的最小值
SUM()|返回某列值之和
AVG() |会忽略 NULL 行。

#### 使用 DISTINCT 可以汇总函数值汇总不同的值。
```
SELECT AVG(DISTINCT col1) AS avg_col
FROM mytable
```

#### 在 SQL 中增加 HAVING 子句原因是，WHERE 关键字无法与合计函数一起使用。

/*-------------------------------分组-------------------------*/
#### 当前的数据表数据
SELECT * FROM mytable；;

| id | col1       | col2       | col4 | col5 |
|----|------------|------------|------|------|
|  1 | 1811942438 | kanglirong | NULL |   99 |
|  2 | 1811942438 | fengxiao   | aaa  |   66 |

#### （如上表信息）根据col1进行分组查询，切记GROUP BY后面的字段名必须和SELECT 查询语句后面的字段值相同。
```
SELECT col1, COUNT(*) AS num
FROM mytable
GROUP BY col1;
```

#### WHERE 过滤行，HAVING 过滤分组。行过滤应当先与分组过滤；
```
SELECT col, COUNT(*) AS num
FROM mytable
WHERE col > 2
GROUP BY col
HAVING COUNT(*) >= 2;
```

#### GROUP BY 的排序结果为分组字段，而 ORDER BY 也可以以聚集字段来进行排序。
```
SELECT col, COUNT(*) AS num
FROM mytable
GROUP BY col
ORDER BY num;
```

#### （如上表信息） 可以通过GROUP BY分组使用函数计算平均值。
```
SELECT col1, AVG(col5) AS num FROM mytable GROUP BY col1;
```

#### 分组规定

1. GROUP BY 子句出现在 WHERE 子句之后，ORDER BY 子句之前；

2. 除了汇总计算语句的字段外，SELECT 语句中的每一字段都必须在 GROUP BY 子句中给出；

3. NULL 的行会单独分为一组；

4. 大多数 SQL 实现不支持 GROUP BY 列具有可变长度的数据类型。


/*--------------------子查询----------------------*/
#### 子查询中只能返回一个字段的数据。
#### 可以将子查询的结果作为 WHRER 语句的过滤条件：
```
SELECT *
FROM mytable1
WHERE col1 IN (SELECT col2
​                 FROM mytable2);
```

#### 下面的语句可以检索出客户的订单数量，子查询语句会对第一个查询检索出的每个客户执行一次：
```
SELECT cust_name, (SELECT COUNT(*)
​                   FROM Orders
​                   WHERE Orders.cust_id = Customers.cust_id)
​                   AS orders_num
FROM Customers
ORDER BY cust_name;
```

# 连接

连接用于连接多个表，使用 JOIN 关键字，并且条件语句使用 ON 而不是 Where。

连接可以替换子查询，并且比子查询的效率一般会更快。

可以用 AS 给列名、计算字段和表名取别名，给表名取别名是为了简化 SQL 语句以及连接相同表。

## 内连接

内连接又称等值连接(比如有两张表，一张表是学生学号表，一张是选课表，通过内连接查询可以将两张表中学号相同的信息拼在一起)，也可以不使用 INNER JOIN 关键字。**若学生学号表一条记录为null或是选课表一条记录为null，这种情况下这条记录就会被丢弃。**

```sql
SELECT a.col1, a.col2, a.col5, b.col3 FROM mytable AS a, mytable1 AS b WHERE a.id = b.id;
```

## 使用 INNER JOIN 关键字

```sql
SELECT a.col1, a.col2, a.col5, b.col3 FROM mytable AS a inner join mytable1 AS b ON a.id = b.id;
```

注意：inner join后面是可以追加and条件的：

```sql
SELECT a.col1, a.col2, a.col5, b.col3 FROM mytable AS a inner join mytable1 AS b ON a.id = b.id and a.col1 = 1;
```

上面这句sql等价于：

```sql
SELECT a.col1, a.col2, a.col5, b.col3 FROM mytable AS a inner join mytable1 AS b ON a.id = b.id where a.col1 = 1;
```

## 自连接

自连接可以看成内连接的一种，只是连接的表是自身而已。

```
SELECT * FROM mytable WHERE col1 = col1 AND col2 = 'fengxiao';
```

## 自然连接

自然连接是把两个表中同名列通过等值测试连接起来的，同名列可以有多个。

内连接和自然连接的区别：内连接提供连接的列，而自然连接自动连接所有同名列，他比内连接更智能。

```
SELECT * FROM mytable natural join mytable1;
```

## 外连接

外连接保留了没有关联的那些行。分为左外连接，右外连接以及全外连接.

### 左外连接

左外连接，左外连接就是保留左表没有关联的行。

mysql> select * from mytable;

| id | col1       | col2       | col4 | col5 |
|----|------------|------------|------|------|
|  1 | 1811942438 | kanglirong | NULL |   99 |
|  2 | 1811942438 | fengxiao   | aaa  |   66 |

2 rows in set (0.00 sec)

mysql> select * from mytable1;

| id | col1       | col2     | col3       |
|-|-|-|-|
|  2 | 1811942438 | fengxiao | 2018-03-23 |
|  3 | 1811942431 | xiaoming | 2018-03-23 |

2 rows in set (0.00 sec)

SELECT a.col2, a.col4, a.col5, b.col1, b.col2, b.col3 mytable AS a LEFT OUTER JOIN mytable1 AS b ON a.col2 = b.col2;

| col2       | col4 | col5 | col1       | col2     | col3       |
|------------|------|------|------------|----------|------------|
| fengxiao   | aaa  |   66 | 1811942438 | fengxiao | 2018-03-23 |
| kanglirong | NULL |   99 |       NULL | NULL     | NULL       |

### 右外连接

mysql> SELECT a.col2, a.col4, a.col5, b.col1, b.col2, b.col3 as a RIGHT OUTER JOIN mytable1 AS b ON a.col2 = b.col2;

| col2     | col4 | col5 | col1       | col2     | col3       |
|----------|------|------|------------|----------|------------|
| fengxiao | aaa  |   66 | 1811942438 | fengxiao | 2018-03-23 |
| NULL     | NULL | NULL | 1811942431 | xiaoming | 2018-03-23 |

2 rows in set (0.00 sec)

## 组合查询

使用 UNION 来组合两个查询，如果第一个查询返回 M 行，第二个查询返回 N 行，那么组合查询的结果为 M+N 行。

每个查询必须包含相同的列、表达式或者聚集函数。

默认会去除相同行，如果需要保留相同行，使用 UNION ALL。

只能包含一个 ORDER BY 子句，并且必须位于语句的最后。

```
SELECT col
FROM mytable
WHERE col = 1
UNION
SELECT col
FROM mytable
WHERE col =2;
```

# 函数

## 日期计算函数

1. `date_add('某个日期时间', interval 1 时间种类名称)`，日期时间加计算

   时间种类名称：`quarter`-季度，`week`-周，`day`-天，`hour`-小时，`minute`-分钟，`second`-秒，`microsecond`-毫秒

   示例：

   * `select date_add('2022-07-20', interval 1 day)`
   * `select date_add('2022-07-20 14:20:21', interval 1 day)`

2. `date_sub()`，时期时间减计算

   示例：

   * `select date_sub('2022-07-20', interval 1 day)`
   * `select date_sub('2022-07-20 14:20:21', interval 1 day)`

3. `datediff()`，日期相减计算差值

4. `timediff()`，时间相减计算差值

5. `now()`，当前具体的日期和时间

6. `curdate()`，当前日期

7. `curtime()`，当前时间