# 简介

## 1. 基本概念

### 1.1 数据库

- **持久化(Persistence)**: 把数据保存到可掉电式存储设备中，将内存中数据保存到硬盘上固化
- **持久化介质**：关系型数据库，非关系性数据库，磁盘文件，XML文件

### 1.2 术语

- **DB**:      Database, 数据结构化存储仓库，保存数据
- **DBMS**:    Database Management System，数据库管理系统。  类似 txt文件 和notepad软件
- **RDBMS**：  Relational Database Management System，关系型数据管理系统
- **SQL**：    Structured Query Language, 结构化查询语言, 用来和数据库通讯

### 1.3 RDBMS/非RDBMS

####  RDBMS

- 数据结构归结为二维表格形式
- 包含Database, Table, Row, Column
- 不同Table对应实体，并且不同Table可以进行联查(**关系模型** )
- 支持**复杂查询，事务**

#### 非RDBMS

- 可以看成RDBMS的精简版，基于k-v存储数据，不需要经过SQL层的解析，性能高

```bash
# 不能通过类似mysql的where来进行过滤
- k-v类型数据库(Redis),
- 文档类型数据库(mongodb, aws dynomoDB), 存储xml或json
- 图形类型数据库
```

## 2. SQL

- Structured Query Language: 使用滚戏模型的数据库应用语言，与数据库直接交互
- 美国国家标准局(ANSI)制定，其中SQL92和SQL99影响深远
- SQL类似普通话，不同的数据库厂商类似各地方言

### 2.1 分类

```bash
DDL
- Data Definition Languages, 数据定义语言
- CREATE   ALTER   DROP   RENAME  TRUNCATE
- 定义不同的库，表，视图，索引等数据库对象

DML
- Data Manipulation Language，数据操作语言
- INSERT   DELETE   UPDATE  SELECT
- 添加，删除，更新，查询数据库记录
- SELECT 是SQL的基础，最为重要

DCL
- Data Control Language, 数据控制语言
- GRANT,  REVOKE, COMMIT, ROLLBACK,  SAVEPOINT
- 定义库，表，字段，用户的访问权限和安全级别
```

### 2.2 规则

- SQL结束用 ； 同一个SQL必要时候可以换行
- 同一行输入多条指令，用  ； 隔开即可，多条sql可以一起执行
- \G 作为sql语句结束符，也可以使用 \G来结束，查询结果垂直显示
- \c : 输入前面的较长的sql，如果不想执行了，则 \c 来取消当前sql的执行
- 不区分大小写，为方便调试，关键字使用大写，库名，表名，字段名使用小写

```sql
--  如果输入的语句比较短，则 \G没有什么意义，如果较长，则方便查看；
--   只在mysql自带的cli中生效
SELECT user(),now(),version()\G

SELECT now(),version(),user()\c

SELECT now();SELECT version();SELECT user();
```

## 3. 安装-8.0.26

- 安装mysql，其实就是安装DB和对应的DBMS
- 开源RDBMS,  使用标准的SQL规范
- 图形化界面工具： JetBrains的 DataGrip

```bash
docker pull mysql:8.0.26
#  1.1 -e: 配置数据库的密码，数据库用户名默认为root；
#  1.2 修改时区
docker run --name erick_mysql -e TZ=Asia/Shanghai -e MYSQL_ROOT_PASSWORD=123456 -d -p 3307:3306 mysql:8.0.26 

# 2. 进入容器，通过容器名称或者容器id
docker exec -it erick_mysql bash
# 3. 默认用户名root打开mysql的命令行,输入密码
mysql -u root -p             mysql -u root -p123456
select version();
# 4. 退出命令行
exit
quit
ctrl + d

# Docker中存放对应的数据库的目录
/var/lib/mysql
```

```bash
SELECT now();   # 当前mysql的时间
SELECT user();  # 当前账户的用户名，返回当前用户@服务端的内网ip   root@117.35.135.138  
SELECT version(); # 
```

# 数据类型

```sql
整数类型：        TINYINT    SMALLINT  MEDIUMINT   INT    INTEGER   BIGINT
浮点类型：        FLOAT      DOUBLE
定点数类型：      DECIMAL
位类型：          BIT
日期类型：        YEAR  TIME    DATE   DATETIME    TIMESTAMP
文本字符串类型：   CHAR  VARCHAR TINYTEXT  TEXT  MEDIUMTEXT  LONGTEXT
枚举类型：        ENUM
集合类型          SET
二进制字符串：     BINARY   VARBINARY  TINYBLOB   BLOB   MEDIUMBLOB     LONGBLOB
JSON类型：       JSON对象    JSON数组
```

## 1. 整数类型

- 无符号类型：加上UNSIGNED
- 一个字节8位
- 建议使用大点的数据，空间换系统稳定

![img](https://cdn.nlark.com/yuque/0/2022/png/26882770/1653059227660-165272c2-5e12-493c-af5c-5d19d1e58e3e.png)

```sql
-- 应用场景
TINYINT:  一般用于枚举类型
SMALLINT: 较小范围的统计数据
MEDIUMINT: 较大整数的计算
INT INTEGER：  取值范围足够大，一般就用这个
BIGINT: 只有处理特别巨大的整数才会用到
```

## 2. 浮点类型

- FLOAT: 单精度浮点数，4字节
- DOUBLE: 双精度浮点数，8字节
- DECIMAL(m,n): 更加精确

```sql
-- UNSIGNED: 并不会改变正数数据的范围，因为符号位也占用空间，因此没必要显示声明UNSIGNED 

-- 1. 四舍五入： 如果存入的数据的精度超过了指定的精度，小数位会四舍五入

-- 2.DECIMAL(5,2): -999.99~999.99
--                 底层是用字符串存储
-- 不指定的默认的是（10，0

-- 金融类的推荐用Decimal,一分钱都不能差
```

## 3. 日期时间

- 添加格式为字符串或者数字，日期函数，会自动转换

![img](https://cdn.nlark.com/yuque/0/2022/png/26882770/1653063414547-dd2c129f-65a0-4621-9963-788b5f6e4974.png) 

```sql
-- DATETIME: 插入的数据和读取的数据一样， 建议使用
-- TIMESTAMP: 会根据时区变化， 底层存储的是毫秒数，查询日期时间差比较快
```

## 4. 文本类型

![img](https://cdn.nlark.com/yuque/0/2022/png/26882770/1653065021718-15f76e1c-919e-4069-96e0-708a1c239bf9.png)

### 4.1char/varchar

![img](https://cdn.nlark.com/yuque/0/2022/png/26882770/1653097133973-0391ba8d-529e-4020-9127-49884a0ccb46.png)

**char**

- 需要预定义长度，默认为1个字符，显示定义就是占用字节数
- 保存的数据比声明的长度小，则在右侧填充空格达到指定长度，检索时，char类型会自动去除尾部空格

**varchar**

- 必须指定长度
- varchar(n), 保存20个字符
- 检索时，保留数据尾部的空格，存储空间为字符串实际长度加一个字节
- 因为影响性能的主要因素是存储总量，推荐varchar

![img](https://cdn.nlark.com/yuque/0/2022/png/26882770/1653097976712-523adba1-80eb-49ce-b2f8-9577c458b530.png)

### 4.2 text

- 主要用来存储大文本，比如评论，博客记录等
- 频繁使用的表，不建议使用text，可以把该字段单独分表出去

### 4.3 enum/set

```sql
ADD season ENUM ('春', '夏', '秋', '冬') null;
ADD season SET ('春', '夏', '秋', '冬') null;
```

## 5. 二进制字符串

- 存储图片，音频等数据
- 一般都是用S3/OSS等存储资源，mysql中保留对应的url

![img](https://cdn.nlark.com/yuque/0/2022/png/26882770/1653098685735-d7788dcd-3b3b-4483-9021-d2e94ac47e25.png)

![img](https://cdn.nlark.com/yuque/0/2022/png/26882770/1653098878471-44b03851-62b2-47a0-a1c7-f4c6def68273.png)

# 基本指令

## 1. 库表操作

### 库

```sql
-- 1. 查看当前用户账户下所有库
SHOW DATABASES;

-- 2. 查看当前使用的库，   一般当前为null
SELECT DATABASE();

-- 3. 切换库, 每次从mysql断开重新打开，当前使用的库就是null
USE lucycat; 


-- A. 创建库
-- 默认为utf8mb4
CREATE DATABASE shuzhan_db;
CREATE DATABASE my_db CHARACTER SET 'utf8mb4';
CREATE DATABASE IF NOT EXISTS shuzhan_db; 


-- B. 修改库的字符集
ALTER DATABASE shuzhan_db CHARACTER SET 'utf8mb4';
-- 数据库名不能改


-- C. 删除库
DROP DATABASE my_db;
DROP DATABASE IF EXISTS my_db;
```

### 表

- 表操作，要么库名.表名，要么在sql脚本前面指定用那个库

#### 创建

```sql
-- 1. 列出当前数据库所有表
SHOW TABLES;
SHOW TABLES  FROM shuzhan_db;

-- 2. 创建普通表
CREATE TABLE people (name char(20),weight INT,sex ENUM('F','M'));
CREATE TABLE IF NOT EXISTS  people (name char(20),weight INT,sex ENUM('F','M'));

-- 存储引擎名字不区分大小写，如果该存储引擎在mysql中未启用，则会使用默认存储引擎；
CREATE TABLE `people` (
  `name` char(20) COLLATE utf8mb4_unicode_ci DEFAULT NULL,
  `weight` int(11) DEFAULT NULL,
  `sex` enum('F','M') COLLATE utf8mb4_unicode_ci DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

#### 描述

```sql
-- 1. 描述一个表
DESC/DESCRIBE/EXPLAIN people;
SHOW COLUMNS/FIELDS FROM people;

DESC/DESCRIBE/EXPLAIN president '%name';       -- 只描述带有name的字段
SHOW COLUMNS/FIELDS FROM president LIKE '%name';  

-- 2.  查看行数据的详细信息
SHOW FULL FIELDS/COLUMNS FROM people;

-- 3. 查看表
SHOW CREATE TABLE people;
```

#### 修改表

```sql
-- 1. 添加field
ALTER TABLE employee ADD status boolean;
ALTER TABLE employee ADD home varchar(50) FIRST ;
ALTER TABLE employee ADD hobby varchar(50) AFTER home;

-- 2. 修改field
ALTER TABLE employee MODIFY home VARCHAR(100);
ALTER TABLE employee MODIFY status BOOLEAN DEFAULT 1;

 -- 3. 重命名field
 ALTER TABLE employee CHANGE status is_enabled BOOLEAN;
 
 -- 4. 删除field
 ALTER TABLE employee DROP is_enabled;
 
 
 -- 5.重命名表
 RENAME TABLE employee TO employees;
 ALTER TABLE  employees RENAME TO employee;
 
 
-- 6. 修改存储引擎
ALTER TABLE people ENGINE = MyISAM;
```

#### 删除

```sql
-- 1. 删除一个表： 删除表结构和表数据，释放表空间
--    删除后不能回滚
DROP TABLE cnip;
DROP TABLE cnip,phone;
DROP TABLE IF EXISTS cnip;


-- 2. 清空表: 表数据清除，表结构保留
TRUNCATE TABLE people;
DELETE FROM people; 


-- TRUNCAT: 不可回滚                           
-- 其他DDL 也适用,   DDL执行完成后会自动COMMIT
-- DDL在8.0中支持事务， 比如删除多张表，如果失败则回滚

-- DELETE: 可以部分删除， 可以回滚(), 利用事务              其他DML 也适用

● TRUNCATE比DELETE速度快，且使用的系统和事务日志资源少
● 但TRUNCATE无事务且不触发TRIGGER，有可能造成事故，因此不建议使用
```

#### 复制

```sql
-- 1. 仅仅复制表的结构
CREATE TABLE cn_phone LIKE phone;

-- 2. 复制表结构和数据
CREATE TABLE us_phone AS SELECT * FROM phone;

-- 3. 原表的部分复制
CREATE TABLE us_apple_phone AS SELECT * FROM phone WHERE brand = 'Apple';  

-- 4. 原表的结构复制
CREATE TABLE us_apple_phone AS SELECT * FROM phone WHERE 1=2; 
```

#### 分区表

- 大数据量表，读写操作效率低，将一个大表的行数据，分配到不同的表

```sql
-- 1.按照时间分区
-- 在var/lib/mysql/库名 目录下，会创建 phone#P#p1.ibd类似的数据保存数据文件
CREATE TABLE phone (id INT,name CHAR(20),created_time DATETIME NOT NULL)

PARTITION BY RANGE(YEAR(created_time))

(PARTITION p0 VALUES LESS THAN (2010),
PARTITION p1 VALUES LESS THAN (2011),
PARTITION p2 VALUES LESS THAN (2012),
PARTITION p3 VALUES LESS THAN (2013),
PARTITION pmax VALUES LESS THAN MAXVALUE);

--2. 如果后续数据都超过了2013，则可以对pmax分区进行再分区
ALTER TABLE phone REORGANIZE PARTITION pmax
INTO(
PARTITION p4 VALUES LESS THAN(2014),
PARTITION p5 VALUES LESS THAN(2015),
PARTITION pmax VALUES LESS THAN MAXVALUE
)
```

### 行

#### 插入

```sql
--  全列插入：必须包含所有的列数据。不推荐这种方式
-- 批量插入执行效率更高
INSERT INTO president VALUES (1,"shu","zhan");
INSERT INTO president VALUES (2,"wu","wang"),(3,"si","li");

-- 指定列名插入， 对于自增的键，如果赋值就取赋值的，不赋值则取默认的
INSERT INTO president (last_name,first_name,id) VALUES ("八","王",10);
INSERT INTO president (last_name,first_name,id) VALUES ("六","赵",5),("七","田",6);

-- set赋值方法：主键就没有给值，采取默认的即可
INSERT INTO president SET first_name = "朱", last_name = "院长";

-- 查询结果插入数据
INSERT INTO phone(name, created_time)
SELECT name, created_time
FROM phone;
```

#### 删除/更新

```sql
-- 1. 删除: 删除所有数据
DELETE FROM cnip;
-- 指定删除： 一般删除时候，必须加where子句，避免误删
DELETE FROM cnip WHERE name = 'zhangsan';


-- 2. 更新操作: 更新所有数据的name
UPDATE cnip SET name = 'sz';
-- 指定更新：更新时候，一般必须加where子句，避免误更新
UPDATE cnip SET name = 'sz' WHERE id = 4;
-- 指定更新多列数据
UPDATE cnip SET name = 'sz', color = 'red' WHERE id = 4;
-- 更新某列的值为NULL
UPDATE cnip SET name = NULL where id = 4;
```



# 查询

## 单表查询

### 1.基本查询

#### 1.1 基本/别名/去重/拼接/着重号

```sql
-- 1. 基本使用
SELECT * FROM cnip; 
SELECT name,age FROM cnip;
SELECT 100;  SELECT 'mick';   -- 常量,可以给每行添加一个对应的字段    
SELECT (2+2); -- 表达式
SELECT 1+1 FROM dual; -- 伪表
SELECT VERSION(); -- 函数
SELECT * FROM cnip_db.cnip WHERE cnip.name = '李明'; -- 全限定表名，列名


-- 2.别名：改变结果集中字段显示名称；    多表联查，重复字段进行区分
SELECT VERSION() AS version;    -- 建议加AS
SELECT VERSION() version;       -- 可以不写
SELECT VERSION() 'my version';  -- 别名中存在空格，用‘’或“”


-- 3. 去重：select后单独使用
SELECT DISTINCT name FROM phone;   -- 获取不重复的某个字段的value
SELECT DISTINCT name,age FROM cnip; -- 用于后面多列，即行中name和age都重复了，才去重


-- 4. 拼接： 不能用 + ， sql中 + 只能当作运算符
SELECT CONCAT('姓名:',first_name,'---',last_name) AS name FROM president;

-- 5. 着重号： 如果表，字段名和mysql关键字重复了，可以使用着重号
SELECT `order` FROM people;
```

#### 1.2. 计算运算符

```sql
-- 1. 算数运算符：
-- +  -  *  /(DIV)  %(MOD)
SELECT 100 + 99;
SELECT '100' + 99;  -- 尝试将字符转换为数字并做运算，如果转换不成功则将字符视为0
SELECT 'dfa' + 99;
SELECT 100 DIV 20;   -- 5

-- 2. null参与运算：  null不等同于0，‘’，‘null’
SELECT null + 10;   --只要其中一个为null，则结果为null

-- 可以通过流程函数来控制: 如果为null，则转换为0
SELECT IFNULL(salary,0) * 10
```

### 2. 比较运算符

#### 2.1 基本比较

```sql
-- 1. = 返回的是0或者1     {！=,   < ,   >,    <=,   >= } 
-- 字符串参与比较运算，隐式转换，转换不成功则看作0
SELECT 1 = 1, 1 = 2, 1 = '1', 1 = 'a', 0 = 'a';

-- 两边都是字符串，则按照ANSI的比较规则
SELECT 'A' = 'A', 'Ab' = 'AB', 'a' = 'b'; -- 1 1 0

-- 只要有NULL参与运算，则结果都是null, 而不是0或1
SELECT 1 = NULL, NULL = NULL;
SELECT * FROM cnip WHERE name=null; -- 不会返回一条记录



-- 2. <=>:  安全等于，null不参与时和=没区别， 为null而生
-- 两边都是null时为1，只有一个null时为0
SELECT NULL <=> NULL, 1 <=> NULL;
SELECT * FROM cnip WHERE name<=>NULL; -- 会返回对应name为null的记录
```

#### 2.2 高级比较

```sql
-- 1. 非空判断
-- IS NULL         IS NOT NULL      和<=>相同
SELECT * FROM phone WHERE name IS NULL;
SELECT * FROM phone WHERE name IS NOT NULL;


-- 2. 按照字典来比较
-- LEAST()     GREATEST()   
SELECT LEAST('A', 'B', 'F', 'Z', 'G');
SELECT GREATEST('A', 'B', 'F', 'Z', 'G');
SELECT LEAST('erick', 'edjk', 'adf'); -- 首先按照第一个字母比较，以此类推
SELECT LEAST(LENGTH('erick'), LENGTH('dfg'), LENGTH('dgadfaf')); -- 比较长度


-- 3. 范围查找
-- BETWEEN(下限) AND(上限)      NOT BETWEEN AND
SELECT * FROM phone WHERE age BETWEEN 20 and 100; --  包含临界值
SELECT * FROM phone WHERE age NOT BETWEEN 32 AND 100;


-- 4. 离散查找
-- IN     NOT IN
SELECT * FROM phone WHERE name in ('tony','jack','lucy');


-- 5. 模糊查询
-- % 代表0-n个字符，  - 只匹配一个字符；
-- 匹配时： 字符的一定要用‘’,     数值类型，可用可不用''
-- 不能匹配null值
-- 英文匹配时候不区分大小写；
SELECT * FROM cnip WHERE content LIKE '%zte%';
SELECT * FROM cnip WHERE content LIKE 'z%e';
SELECT * FROM cnip WHERE content LIKE '_te';
SELECT * FROM cnip WHERE content LIKE '__te';
SELECT * FROM phone WHERE  name like '%\_%';   -- 转义符号   \
SELECT * FROM phone WHERE name like '%^_%' ESCAPE '^';   -- 自定义转义符号为 ^
```

### 3. 逻辑运算符

```sql
-- 逻辑运算符
-- OR ｜｜     AND &&    NOT!
SELECT * FROM people WHERE name = 'erick' || name = 'eri';
SELECT * FROM people WHERE name = 'erick' AND name = 'eri';
```

### 4. 分页/排序

```sql
-- 1. 分页： 语法最后，执行顺序最后
-- 查前五条数据
SELECT * FROM cnip LIMIT 5;
-- 跳过前面5个，检索2个
SELECT * FROM cnip LIMIT 5,2;
-- 跳过前面5个，检索2个
SELECT * FROM cnip LIMIT 2 OFFSET 5;



-- 2. 排序
-- 默认升序/ASC   DESC/降序
SELECT * FROM people ORDER BY name DESC;

-- order by 后 <支持别名>
SELECT *, name AS 'MYNAME' FROM people ORDER BY MYNAME;

-- 多字段(二级排序)： 先按第一个字段排序，第一个字段相同再按第二个排序
SELECT * FROM president ORDER BY first_name DESC, last_name DESC;

-- 区分null值的排序： 先按照null和非null排序，再按照字段排序
-- IF(ccondtion,0,1): true则为0，false 则为1, 三元表达式
SELECT * FROM president ORDER BY IF(address IS NULL,0,1) DESC, address DESC;
```

## 多表联查

### 1 笛卡尔积

- 假如存在两个集合X, Y,  笛卡尔积就是X和Y的全排列组合
- CROSS JOIN,  INNER JOIN, JOIN
- A表包含n条数据，B表包含m条数据，则笛卡尔积为n*m条数据；

```sql
-- 表别名区分不同表的相同字段，起别名后字段必须用别名的字段
-- 出现笛卡尔积错误的几种方式
-- 如果要避免笛卡尔积错误，那么需要加上链接条件
SELECT * FROM people,phone;
SELECT * FROM people CROSS JOIN phone;
SELECT * FROM people INNER JOIN phone;
SELECT * FROM people JOIN phone;

-- 加上别名：避免不同表字段重复，同时sql优化提升效率
SELECT peo.name,peo.`order`, pho.name FROM people AS peo,phone AS pho;
```

### 2. 连接条件

- 两个表在笛卡尔积的条件上，增加一个条件

#### 2.1 等值链接/非等值链接

```sql
-- 1 等值链接，  笛卡尔积的各种join都适用
SELECT * FROM student stu,president pre WHERE stu.id = pre.id;

-- 2. 非等值链接
SELECT emp.first_name, sal.grade
FROM employee AS emp,
     salary_level AS sal
WHERE emp.salary > sal.low_level
  AND emp.salary < sal.high_level;
```

#### 2.2 自链接/非自链接

```sql
-- 1. 自链接
SELECT emp.employ_id, emp.first_name, manage.employ_id, manage.first_name
FROM employee AS emp,
     employee AS manage
WHERE emp.manager_id = manage.employ_id;
```

### 3. 内连接/外连接

- SQL99 标准
- 采用(A表)   JOIN (B表)  ON 

#### 3.1 内连接

- 合并具有同一列的两个以上的表的行，结果集中不包含一个表与另一个表不匹配的行
- 笛卡尔积加上对应的条件过滤得到的数据

```sql
-- (A表) JOIN (B表)  ON 
-- JOIN,  INNER JOIN, 其中Inner可以省略 
SELECT * FROM employee JOIN salary_level ON employ_id=salary_level.high_level;
SELECT * FROM employee INNER JOIN salary_level ON employ_id=salary_level.high_level;
```

#### 3.2 外连接

- 除了应该有的结果集，还包含不匹配的比如null的结果集

#### 3.3 LEFT JOIN

- 在笛卡尔积的基础上，进行on条件的筛选
- 再加上左表中剩下的数据，右边数据用null表示



```sql
-- (A表) LEFT OUTTER JOIN (B表)  ON 
-- OUTTER可以省略
SELECT * FROM student stu LEFT JOIN president pre on stu.id = pre.id;
SELECT * FROM student stu LEFT OUTTER JOIN president pre on stu.id = pre.id;
```



![img](https://img-blog.csdnimg.cn/20200816173507406.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzM3NDU3OA==,size_16,color_FFFFFF,t_70#pic_center)

#### 3.4 RIGHT JOIN



```sql
-- OUTTER可以省略
SELECT * FROM student stu RIGHT JOIN president pre on stu.id = pre.id;
SELECT * FROM student stu RIGHT OUTTER JOIN president pre on stu.id = pre.id
```



![img](https://img-blog.csdnimg.cn/20200816173812906.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzM3NDU3OA==,size_16,color_FFFFFF,t_70#pic_center)

#### 3.5 OUTER JOIN

- 满外链接
- 再加上左表中剩下的数据，右边数据用null表示
- 再加上右表中剩下的数据，左边数据用null表示 

```sql
SELECT * FROM student stu LEFT JOIN president pre on stu.id = pre.id

UNION

SELECT * FROM student stu RIGHT JOIN president pre on stu.id = pre.id;


-- UNION   UNION ALL
-- UNION 会进行去重， 但是去重操作会耗费时间
-- UNION ALL: 存在重复的数据， 一般尽可能的去用，效率高
```

![img](https://img-blog.csdnimg.cn/20200816174323369.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzM3NDU3OA==,size_16,color_FFFFFF,t_70#pic_center)

## 4. SQL99新特性

### 4.1 USING子句

```sql
-- 如果两个表中的on条件的列名相同，则可以用using来简化sql
SELECT * FROM student stu RIGHT JOIN president pre USING(id);
```

### 4.2  NATURAL

- 自动查询两个表中**所有相同**的字端，然后进行等值连接
- 两个表中只有一列的表字段相同，则用NATURAL  JOIN简化sql
- 相同的列只会展示一个

```sql
-- 左连接，右连接，内连接
-- NATURAL LEFT JOIN, NATURAL RIGHT JOIN, NATURAL  JOIN

SELECT * FROM student stu NATURAL LEFT JOIN president pre;
```



![img](https://img-blog.csdnimg.cn/20200816180642864.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzM3NDU3OA==,size_16,color_FFFFFF,t_70#pic_center)

## 5. 子查询

```sql
-- 子查询先执行，外查询后执行
-- 单行操作符对应单行子查询， 多行操作符对应多行子查询
-- 子查询可以作用于where，having，case中

-- 1. 子查询： 单行子查询
-- 操作符号： =      >      >=       <       >=     !=
-- 建议先写子查询，再去写外查询
-- 子查询如果没有数据，那么整体查出来也没数据
SELECT *
FROM employee
WHERE salary > (SELECT salary FROM employee WHERE first_name = '王'); 
          


-- 2. 多行子查询： 内查询返回多条数据
-- IN: 列表中任意一个
-- ANY: 需要和单行操作符一起使用，和字查询返回的某一个值比较
-- ALL/SOME: 需要和单行操作符一起使用，和字查询返回的所有值比较

SELECT *
FROM employee
WHERE salary in (SELECT salary FROM employee);

-- 比任一的都小就可以查
SELECT *
FROM employee
WHERE salary < ANY (SELECT salary FROM employee WHERE address = '上海');

-- 比所有的小才行
SELECT *
FROM employee
WHERE salary < ALL (SELECT salary FROM employee WHERE address = '上海');
```

# 函数

## 1. 简单函数

### 1.1 数值函数

```sql
SELECT ABS(1)               -- 绝对值
CEIL(1.5)  CEILING(1.5)     -- 向上取整  >= 该参数的最小整数
FLOOR(1.5)                  -- 向下取整
PI()                        -- 圆周率的值
RAND()                      -- 0-1 的随机数,可以乘法的到对应的值
RAND(X)                     -- 0-1 的随机数，如果X因子一样，产生的随机数相同
ROUND(1.5)                  -- 四舍五入
ROUND(5.32, 1)              -- 四舍五入，保留1位小数 5.3
TRUNCATE(1.345,2);          -- 取小数点后几位  1.34
SQRT(2)                     -- 返回平方根，x值为负，返回NULL
LEAST(e1,e2,e3)             -- 返回最小的
GREATEST(e1,e2,e3)          -- 返回最大的
```

### 1.2 字符串函数

```sql
SELECT LENGTH(name) from phone;                 -- 字符长度
SELECT CONCAT(name,'====',brand) from phone;    -- 字符拼接
SELECT UPPER(name) from phone;                  -- 大写转换
SELECT LOWER(name) from phone;                  -- 小写转换
SELECT CONCAT( LOWER(name), '===',UPPER(brand)) from phone; -- 函数嵌套

SELECT TRIM('   LUCY     ') as result;   -- TRIM()   LTRIM().  RTRIM()
```

## 2. 日期函数

### 2.1 基本类型

```sql
-- 1. 获取日期，时间
SELECT CURDATE();        -- 2022-05-18
SELECT CURRENT_DATE();

CURTIME();               -- 10:21:58
CURRENT_TIME;

SELECT NOW();            -- 2022-05-18 10:22:45
SYSDATE()
CURRENT_TIMESTAMP()
LOCALTIME()
LOCALTIMESTAMP()

UTC_DATE                -- 世界标准时间 2022-05-18
UTC_TIME                -- 02:27:40
UTC_TIMESTAMP           -- 2022-05-18 02:27:41


-- 2. 获取年月日周天
YEAR(NOW()), MONTH(NOW()),  DAY(NOW());  -- 年月日,参数date
MONTHNAME(NOW()),WEEKDAY(NOW()),QUARTER(NOW()); 
WEEK(NOW()),WEEKDAY(NOW()),WEEKOFYEAR(NOW());
DAYOFMONTH(NOW()),DAYOFYEAR(NOW()),DAYOFWEEK(NOW());

HOUR(CURRENT_TIME), MINUTE(CURRENT_TIME),SECOND(CURRENT_TIME); -- 时分秒，参数time


-- 3. 日期操作函数 EXTRACT(type from date)
SELECT EXTRACT(SECOND FROM NOW());
SELECT EXTRACT(MINUTE_MICROSECOND FROM NOW());

-- 4. 日期与时间戳转换
SELECT UNIX_TIMESTAMP();        -- 以UNIX时间戳的形式返回当前时间 1652841099
SELECT UNIX_TIMESTAMP(NOW());   -- 将Data转换为UNIX时间戳
SELECT UNIX_TIMESTAMP('2021-09-01: 10:24');

SELECT FROM_UNIXTIME(1652841243); -- 将UNIX时间戳转换普通格式时间 2022-05-18 10:34:03

-- 5. 时间和秒转换
SELECT TIME_TO_SEC(CURRENT_TIME); -- time参数， 小时*3600+分钟*60
SELECT SEC_TO_TIME(39421);        -- seconds参数


-- 6. 修改时间
SELECT DATE_ADD(NOW(), INTERVAL 1 YEAR);   -- 加的具体的值可为负数
SELECT ADDDATE(NOW(), INTERVAL 1 YEAR);
SELECT DATE_SUB(NOW(), INTERVAL 1 MONTH);
SELECT SUBDATE(NOW(), INTERVAL 1 MONTH);
SELECT DATE_ADD(NOW(), INTERVAL '1_1' YEAR_MONTH ); -- 同时加年和月
SELECT DATEDIFF(NOW(),SUBDATE(NOW(),INTERVAL 1 YEAR)); -- 天数
```

### 2.2 字符串和日期转换

```sql
-- 1.日期转换为字符串，按照指定格式
DATE_FORMAT(date, fmt);
SELECT DATE_FORMAT(NOW(),'%Y-%m-%d');    -- 2022-05-18
SELECT DATE_FORMAT(NOW(),'%Y-%m-%d %H:%i:%s'); -- 2022-05-18 11:25:11



-- 2.字符串转为日期
-- 字符串和对应的格式需要匹配，不然会报错
STR_TO_DATE(str, fmt)
SELECT STR_TO_DATE('2021-09-05 12:38:16', '%Y-%m-%d %H:%i:%s');
%Y -- 4位数年份
%y -- 2位数年份

%M -- 月名表示月份 January
%m -- 两位数表示月份 01，02
%b -- 缩写的月名  Jan Feb
%c -- 数字表示月份  1，2，11

%D -- 英文后缀月中的天数  1st
%d -- 2位数表示月中的天数 01，22
%e -- 数字表示月中天数。 1，2，3

%H -- 2位数字表示小时   24小时制
%h -- 2位数表示小时,12小时制
%k -- 数字形式的小时， 1，12

%i -- 2位数字表示分钟 00，01
%S %s -- 2位数字表示秒
```

## 3. 流程控制函数

### 3.1 三元运算符

```sql
-- 1. IF(condition,result1, result2)
-- 条件中可以用 = <= >=  is null等判断
SELECT IF( 1=1,'positive','negative' ) as result;  

-- 2. IFNULL(value,0);  
SELECT IFNULL(name,0) FROM phone;
```

### 3.2 SWITCH 函数

```sql
-- 1. case 函数 ： 类似switch，是用在from之前的，对于某些字段进行计算
SELECT *,
       CASE brand
           WHEN 'Apple' THEN price * 0.9
           WHEN 'Huawei' THEN price * 0.8
           WHEN 'XiaoMi' THEN price * 0.7
           WHEN 'Oppo' THEN price * 0.6
           ELSE price
           END AS discount_price
FROM phone;

-- 2. case 函数 ： 类似多重if，  适用于区间比较。  case 后没有字段 ， 
SELECT *,
       CASE
           WHEN price >= 5000 THEN 'LEVEL-A'
           WHEN price >= 4000 THEN 'LEVEL-B'
           WHEN price >= 3000 THEN 'LEVEL-C'
           WHEN price >= 2000 THEN 'LEVEL-D'
           WHEN price >= 1000 THEN 'LEVEL-E'
           ELSE 'LEVEL-LOW'
           END AS product_level
FROM phone;
```

## 4. 聚合函数

```sql
-- 统计函数都会忽略null值：      字段为NULL时候，统计时候忽略该行（分子和分母都忽略该行）
-- SUM()  AVG():              参数必须是数值类型                
-- MAX()  MIN()，COUNT():     参数为任何类型


SELECT MAX(age) from phone;
SELECT SUM(DISTINCT(brand)) FROM phone;  -- 和distinct搭配


-- COUNT(*)    COUNT(field)    COUNT(1) 

-- COUNT(*)，COUNT(1)：没区别，统计不为null的行数目， 一行中一个字段不为null，总数就会计算该行

-- COUNT(field):  如果某个字段不为null，就统计

-- 性能： 
INNODB:  COUNT(*) 和 COUNT(1)的效率差不多，  比COUNT(filed)效率高
MYISAM:  三个都一样
```

## 5.  GROUP BY

```sql
-- 单个列分组
SELECT AVG(salary), SUM(salary), nation
FROM employee
GROUP BY nation;

-- 多个列分组: 先按照一个字段分为一个小组，再根据另外一个字段再分组，统计的是最小组的数据
-- 字段前后无区别
SELECT nation, address, SUM(salary)
FROM employee
GROUP BY nation, address;

-- 结合聚合函数使用: from前面的字段只能是对应的聚合函数，group by的字段
-- from前面的字段，可以不用写，一般建议写
```

## 6. HAVING

```sql
-- 1. 过滤条件中使用了聚合函数，则必须使用HAVING来替换WHERE
--    HAVING 必须在 GROUP BY 后面
--    使用HAVING的前提： 使用GROUP BY, 不然就没有太多意义
SELECT nation, MAX(salary)
FROM employee
GROUP BY nation
HAVING MAX(salary) > 20000;

-- 2. 加where和having的区别: 
-- 推荐，where, 实现效率高
SELECT address, MAX(salary)
FROM employee
WHERE address = '上海'
GROUP BY address
HAVING MAX(salary) > 10;

SELECT address, MAX(salary)
FROM employee
GROUP BY address
HAVING MAX(salary) > 10
   AND address = '上海';
   
```

## 7. 执行过程

```sql
-- 书写顺序： 
SELECT>>>FROM>>>WHERE>>>GROUP BY>>>HAVING>>>ORDER BY>>>LIMIT

-- 执行顺序：
FROM-> WHERE->GROUP BY->HAVING->SELECT字段->DINSTINCT->ORDER BY ->LIMIT

-- 尽可能每次执行，都最多的去过滤掉数据
```

# 约束

```sql
-- 查看
SELECT * FROM information_schema.TABLE_CONSTRAINTS WHERE TABLE_NAME='employee';
```

## NOT NULL

- 空字符串‘’不等于null

```sql
CREATE TABLE erick (name VARCHAR(20) NOT NULL)

ALTER TABLE erick
    ADD COLUMN address VARCHAR(20) NOT NULL;
 
ALTER TABLE erick
    MODIFY address VARCHAR(40) NOT NULL;
    
 -- 2. 删除约束
 ALTER TABLE erick
    MODIFY address VARCHAR(40) NULL;
```

## UNIQUE

- 一个表中可以存在多个
- 可以是某个列的值唯一，也可以是多个列组合的值唯一
- 允许列值为多个null值
- 唯一约束命名默认名：和列名相同
- 唯一约束默认会创建一个唯一索引

```sql
-- 建表后添加
ALTER TABLE erick
    ADD UNIQUE name_unique_index (name);
    
ALTER TABLE erick
    MODIFY address varchar(20) UNIQUE;
    
-- 多行数据的约束    
CREATE TABLE user
(
    id       INT,
    name     VARCHAR(20),
    password VARCHAR(20),
    CONSTRAINT uk_user_name UNIQUE (name, password)
);

-- 删除：只能通过删除对应的索引
ALTER TABLE user
    DROP INDEX uk_user_name;
```

## PRIMARY KEY

- 主键约束：相当于非空+唯一约束
- 一个表最多只能包含一个主键约束，并且一般都会包含一个
- 主键约束对应表中的一列或多列， 多列的时候每个列都不允许为null
- 主键名永远是 PRIMARY，就算自定义名字都没用
- 主键约束对应主键索引

```sql
-- 创建表
CREATE TABLE my_erick
(
    id   INT PRIMARY KEY,
    name varchar(20)
)

ALTER TABLE people
    MODIFY id int PRIMARY KEY;

-- 复合主键
CREATE TABLE my_erick_tb
(
    id   INT,
    name varchar(20),
    PRIMARY KEY (id, name)
)

-- 删除主键约束
ALTER TABLE my_erick_tb
    DROP PRIMARY KEY;
```

## AUTO_INCREMENT

- 一个表最多只能有一个自增长列
- 必须作用在唯一或主键上
- 约束的必须是整数类型
- 主键声明有自增，添加数据时，不要给主键赋值

```sql
-- 必须作用在唯一或主键上
CREATE TABLE erick
(
    id   INTEGER PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(20)
);

-- 修改
ALTER TABLE erick
    MODIFY id int AUTO_INCREMENT;
    
-- 删除
ALTER TABLE erick
    MODIFY id int;
    
-- 自增变量持久化
删除一条数据后(如id=10)，新增的数据，会沿用之前的值，下一个值就是11，即使重启服务器也是
```

## CHECK

- 5.8的新特性

```sql
-- check约束
CREATE table erick_shu
(
    id     int PRIMARY KEY,
    name   varchar(10),
    salary int CHECK ( salary > 200 )
);
```

## DEFAULT

- 建表时，加not null defalut‘’或default 0 ，不想让表中出现null值
- null不好比较，同时效率不高，影响索引效果

```sql
CREATE table erick_shu
(
    id     int PRIMARY KEY,
    name   varchar(10) DEFAULT 'ERICK',
    salary int CHECK ( salary > 200 )
);

-- 修改时添加
ALTER TABLE erick_shu
    MODIFY name VARCHAR(20) DEFAULT 'LUCY';

-- 删除
ALTER TABLE erick_shu
    MODIFY name VARCHAR(20);
```

# 二、视图

## 1. 基本介绍

- 只把数据的一部分展示给用户，可以使用视图，类似一种权限管理
- 视图是一种虚拟表，本身不具有数据，占用很少的内存空间
- 视图建立在已有表的基础上，原表称为基表

## 2. 特性

- 视图的创建和删除只会影响视图本身，不影响对应的基表。但是对视图中的数据进行增删改操作时，数据表中的数据会响应发生变化，反之亦然
- 小型项目的数据库可以不使用视图。但是在大型项目中，以及数据表比较复杂的情况下，可以把经常查询的结果放到虚拟表中，提升使用效率
  ![img](https://img-blog.csdnimg.cn/e3814fc5e2734c43ae5dfe51616451fd.png)

## 3. 使用

```sql
-- 1. 创建
CREATE VIEW employee_view AS
SELECT *
FROM employee;

-- select中的别名会当作view中的列名
CREATE VIEW employee_view_second AS
SELECT first_name AS f_name, last_name AS l_name
FROM employee;

CREATE VIEW employee_view_third(f_name, l_name) AS
SELECT first_name, last_name
FROM employee;

-- 基于视图创建视图
CREATE VIEW employ_last_view AS
SELECT *
FROM employee_view_second;

-- 任何select中的数据结果集，都可以作为view
-- 2. 查询
SELECT * FROM employee_view;

-- 3. 查询视图
SHOW TABLES;  --查看到所有的表和视图 
DESC employee_view_second; -- 查看视图结构
SHOW CREATE VIEW employee_view_second;  -- 查看视图
```



```sql
-- 修改了视图中的数据，对应的表中数据也会变化  UPDATE/DELETE
-- 只有视图和对应的表中的数据对应起来的才可以
UPDATE employee_view_second SET f_name='SHU' WHERE l_name='展';

-- 删除视图, 基于A view的B view，A没了B也查不到
DROP VIEW employ_last_view; 
DROP VIEW IF EXISTS employ_last_view; 

-- 修改视图
CREATE OR REPLACE VIEW employee_view_second AS
SELECT *
FROM employee;   --如果有就替换
```

## 4. 优缺优点

```bash
## 优点
- 经常使用的查询定义为视图，不用关心对应的数据表结构，表关系，简化开发
- 通过视图来控制权限，相当于对视图具有隔离性
##  缺点
- 如果表结构变了，那么视图就要修改，维护成本高
- 特别是嵌套的视图
- 小项目建议不要用视图，大项目考虑情况去使用
```