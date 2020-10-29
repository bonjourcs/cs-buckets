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


