# 基本概念

- 数据库：以某种有组织的方式存储数据的集合，可以看成是存放文件的书柜。
- 数据表：存储数据的文件，可以看成是包含资料的文件。
- 模式（Schema）：描述数据库和数据表的布局和特性的信息。
- 列：可以类比一张表格中的列。
- 行：包含一列或多列的一条记录。
- 数据类型：限定存储在列中的数据种类。
- SQL: 与数据库沟通的语言。
    * DDL(Data Definition Language): 数据定义语言，包括：CREATE、ALTER、DROP、RENAME、TRUNCATE、COMMENT等
    * DML(Data Manipulation Language): 数据操作语言，包括：INSERT、UPDATE、DELETE、MERGE、CALL、EXPLAIN PLAN、LOCK TABLE等
    * DCL(Data Control Language): 数据控制语言，包括：GRANT、EVOKE等
    * DQL(Data Query Language): 数据查询语言：SELECT

# 基础操作

假设表 `tb_student` 的结构如下：
id|name|gender|birth
--|--|--|--
1|Zhang|M|1990-10-01
2|Li|F|1990-01-01
3|Liu|M|1998-12-11
4|Li|M|1996-07-21
4|Song|M|1998-07-21

完成下列需求对应的 SQL 如下：

- 选择姓名列
```sql
SELECT name FROM tb_student;
```

- 选择姓名和性别列
```sql
SELECT name, gender FROM tb_student;
```

- 选择所有列
```sql
SELECT * FROM tb_stduent;
```

- 选择姓名列并去重
```sql
SELECT DISTINCT name from tb_student;
```

- 选择前 3 条记录
```sql
SELECT * FROM tb_student limit 0, 3;
```

- 选择所有列，并按姓名升序、出生日期降序排列
```sql
SELECT * FROM tb_student ORDER BY name, birth DESC;
```

- 查询 id < 3 的数据
```sql
SELECT * FROM tb_student WHERE id < 3;
```

- 选择姓名是 LI 或者 LIU 的数据
```sql
SELECT * FROM tb_student WHERE name IN('LI', 'LIU');
```

- 选择姓名以 L 开头的数据
```sql
SELECT * FROM tb_student WHERE name LIKE 'L%';
```

# 使用简单的函数

函数能帮我们简化操作，MySQL 内置很多函数，下面举例说明：

- TRIM(): 去字符开始和结束的空格
```sql
SELECT TRIM('   hello world   '); -- 输出'hello world'
```

- CONCAT(): 拼接字符串
```sql
SELECT CONCAT('hello', ' ', 'world'); -- 输出'hello world'
```

- LENGTH(): 获取字符串长度
```sql
SELECT LENGTH('hello world'); -- 输出11
```

# 使用聚合函数

聚合函数是指对某行进行聚合计算，并返回一个值。

- AVG(): 计算平均值
```sql
SELECT AVG(id) FROM tb_student;
```
**注意** `AVG()` 会忽略包含 NULL 的行。

- MIN(): 获取最小值
```sql
SELECT MIN(LENGTH(name)) FROM tb_student;
```

- MAX(): 获取最大值
- SUM(): 求和
```sql
SELECT SUM(id) FROM tb_student;
```

## 聚合和分组

通过 GROUP BY 和 HAVING 可以对数据进行分组

```sql
SELECT name, COUNT(*) AS times 
FROM tb_student 
WHERE id < 10
GROUP BY name
HAVING times > 1;
```

**注意** WHERE 和 HAVING 有些区别，有一种理解方式是：WHERE 用于分组前的过滤，HAVING 用于分组后的结果过滤。即，WHERE 用来过滤行，HAVING 用来过滤分组。

## 子查询

如果需要将一条 SQL 的结果作为另外一条 SQL 的查询条件，可以使用子查询（嵌套查询），下面是一个简单的例子：
```sql
SELECT * FROM
tb_student
WHERE LENGTH(name) = 
(
    SELECT MAX(LENGTH(name)) FROM tb_student
);
```

# 联结查询

联结（JOIN）是关系型数据库中重要的一种查询手段，其作用是使用一条复杂的 SQL 将多张表的数据检索出来。联结和好的关系型数据库设计息息相关。联结有以下几种：

- 交叉联结（cross join）: 返回笛卡尔积的联结
```sql
SELECT t1.col1, t1.col2, t2.col1, t2.col2
FROM t1, t2;
```
- 内联结（inner join）: 基于表值相等的联结
```sql
SELECT t1.col1, t1.col2, t2.col1, t2.col2
FROM t1
INNER JOIN t2
ON t1.col1 = t2.col1;
```

- 自联结（self join）: 表将自己本身看作其他表的联结操作
```sql
SELECT t1.col1, t1.col2
FROM t1, t1 as t2
WHERE t1.col2 = t2.col1
AND t1.col1 = 'some_value';
```
**注意** 自联结可能比子查询要高效

- 外联结（outer join）: 当需要获取未关联的行时，可以使用该联结方式
```sql
SELECT t1.col1, t1.col2, t2.col2
FROM t1 LEFT JOIN t2
ON t1.col1 = t2.col1
```
上面的查询如果 t1 中存在 t2 表里没有的 col1 列，将会展示如下结果

t1.col1|t1.col2|t2.col2
--|--|--
xx|xx|xx
xx|xx|null

上面的示例 SQL 使用的是左外联结，如果用右外联结（RIGHT JOIN），下面的 SQL 和示例 SQL 等效
```sql
SELECT t1.col1, t1.col2, t2.col2
FROM t2 RIGHT JOIN t1
ON t1.col1 = t2.col1
```

- UNION: 组合查询
```sql
SELECT t1.col1, t1.col2
FROM t1
WHERE t1.col1 = 'some_value'
UNION
SELECT t1.col1, t1.col2
WHERE t1.col2 = 'some_value';
```
上面的 SQL 会将两条查询的结果合并。
**注意** 使用 UNION 有一些限制条件

- 必须是两条以上的 SQL 才可以
- 每条 SQL 拥有相同的列、表达式或者聚合表达式
- 列的类型必须兼容

默认情况下，UNION 是会取消重复行，如果不需要自动取消，使用 UNION ALL 提到 UNION

# 基础数据操作

## 数据插入

- 插入指定行
```sql
INSERT INTO t1(col1, col2, col3)
VALUE
('value1', 'value2', 'value3'),
('value1', 'value2', 'value3');
```

- 插入选出的行
```sql
INSERT INTO t1(col1, col2, col3)
SELECT col1, col2, col3
FROM t2;
```

- 复制表
```sql
CREATE TABLE t1_copy AS 
SELECT * FROM t1;
```

## 数据更新
```sql
UPDATE TABLE t1
SET col1='value1', col2='value2'
WHERE col1='some-value';
```

## 数据删除
```sql
DELETE FORM t1
WHERE col1='some-value';
```

## 数据表创建
```sql
CREATE TABLE t1(
    id INT PRIMARY KEY NOT NULL AUTO_INCREMENT,
    col1 VARCHAR(100) NOT NULL DEFAULT '-',
    col2 DATETIME NOT NULL DEFUALT CURRENT_TIMESTAMP 
)
```

## 数据表更改
```sql
ALTER TABLE t1 DROP COLUMN  col1;
```

## 试图的创建
```sql
CREATE VIEW AS 
SELECT t1.col1, t1.col2, t2.col1
FROM t1 INNER JOIN
t2 ON t1.col1 = t2.col2;
```

# 存储过程

## 创建存储过程
```sql
CREATE PROCEDURE count_student(IN id INTEGER)
BEGIN
    SELECT COUNT(*) FROM tb_student
    WHERE id < id;
END$;
```

## 调用存储过程
```sql
CALL count_student(1);
```

## 使用游标
使用游标可以处理存储过程的结果集，下面是在 MySQL 中使用游标的示例：
```sql
CREATE PROCEDURE build_email_list (INOUT email_list varchar(1000))
BEGIN
    DECLARE v_finished INTEGER DEFAULT 0;
    DECLARE v_email VARCHAR(100) DEFAULT "";
    DECLARE email_cursor CURSOR FOR 
        SELECT email FROM tb_employee;
    DECLARE CONTINUE HANDLER FOR 
        NOT FOUND SET v_finished = 1;
    OPEN email_cursor;
    get_mail: LOOP
    FETCH email_cursor INTO v_email;
    IF v_finished = 1 THEN
        LEAVE get_email;
    END IF; 
    SET email_list = CONCAT(v_email, ";", email_list);
    END LOOP get_email;
    CLOSE email_cursor;
END;
```
可以通过下面的 SQL 语句进行调用：
```sql
SET @email_list="";
CALL build_email_list(@email_list);
SELECT @email_list;
```
# 事务

事务需要满足四个条件：

- 原子性(Atomicity)
- 一致性(Consistency)
- 隔离性(Isolation)
- 持久性(Durability)

MySQL 中事务相关的关键字

- BEGIN/START TRANSACTION: 开启事务
- COMMIT: 提交事务
- ROLLBACK: 事务回滚
- SAVEPOINT: 事务保留点，用于回滚到此处

下面是 MySQL 在命令提示符下的使用事务的示例:

```shell script
mysql> set autocommit=0;
mysql> begin;
mysql> delete from tb_student where id = 1;
mysql> rollback;
mysql> select * from tb_student where id = 1;
```

# 高级 SQL 特性

## 约束

约束是用来保障数据表中数据的有效性。

```sql
CREATE TABLE tb_demo(
    id INT PRIMARY KEY NOT NULL,
    email VARCHAR(100) UNIQUE
)
```
也可以为表增加约束
```sql
ALETR TABLE tb_demo ADD CONSTARINT unq_demo_email UNIQUE(email);
```
使用下面语句删除约束
```sql
ALTER TABLE tb_demo DROP INDEX unq_demo_email;
```

## 索引

使用下面语句为 tb_demo 表的 email 字段创建索引
```sql
ALTER TABLE tb_demo ADD INDEX idx_email(email);
```

下面的语句删除该索引
```sql
ALTER TABLE tb_demo DROP INDEX idx_email;
```

## 触发器

触发器可以在执行 SQL 语句时进行额外的操作。下面是创建触发器的示例：
```sql
CREATE TRIGGER t_upper AFTER INSESRT
ON tb_demo FOR EACH ROW
BEGIN
    -- 这里是业务逻辑
END;
```
删除触发器使用下面的语句：
```sql
DROP TRIGGER t_upper;
```
