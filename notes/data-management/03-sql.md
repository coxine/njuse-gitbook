# 03-SQL

* SQL 结构化查询语言：关系数据库的标准语言
  * 综合统一
  * 命令式语言，非过程化
  * 面向集合的操作方式

## SQL 与 关系数据库三级模式

* 存储文件：对应内模式
* 基本表：对应模式
  * 一个关系对应一个基本表
  * 一个表可带若干索引
* 视图：对应外模式
  * 从基本表导出的表
  * 数据库中只存放视图的定义而不存放视图对应的数据
  * 视图是一个虚表
  * 用户可以在视图上再定义视图

## SQL 数据定义

### 层次化机制

* 一个数据库实例可以建立多个数据库
* 一个数据库可以建立多个模式
* 一个模式包含多个表、视图、索引等对象

### 数据定义语法

| 对象 | 创建              | 删除            | 修改            |
| -- | --------------- | ------------- | ------------- |
| 模式 | `CREATE SCHEMA` | `DROP SCHEMA` |               |
| 表  | `CREATE TABLE`  | `DROP TABLE`  | `ALTER TABLE` |
| 视图 | `CREATE VIEW`   | `DROP VIEW`   |               |
| 索引 | `CREATE INDEX`  | `DROP INDEX`  | `ALTER INDEX` |

* 没有修改模式：避免增加复杂性
* 注：MySQL 创建数据库时，模式和数据库是同义词，会创建同名的模式

#### 模式

*   定义模式

    ```sql
      CREATE SCHEMA <schema_name> AUTHORIZATION <user_name> 
        [ CREATE TABLE <table_name> ] 
        [ CREATE VIEW <view_name> ]
        [ GRANT <privilege> ON <object_name> TO <user_name> ]
    ```
*   删除模式

    * `CASCADE`：删除模式下的所有对象
    * `RESTRICT`：不删除模式下的对象，如果有对象存在则拒绝执行，仅当没有对象存在时才执行

    ```sql
      DROP SCHEMA <schema_name> <CASCADE | RESTRICT>
    ```

#### 表

*   定义基本表

    * 约束条件
      * 列约束：
        * `NOT NULL`：非空约束
        * `UNIQUE`：唯一约束
        * `CHECK (<condition>)`：检查约束
        * `DEFAULT <default_value>`：默认值约束
        * `PRIMARY KEY`：主键约束
      * 表约束：
        * `PRIMARY KEY (<column_name>, [<column_name>])`：主键约束，可以包含多个列
        * `FOREIGN KEY (<column_name>) REFERENCES <table_name> (<column_name>)`：外键约束

    ```sql
      CREATE TABLE <table_name> (
        <column_name> <data_type> [<column_constraint>],
        ...
        [<table_constraint>]
      )
    ```
*   修改基本表：

    * `ADD`：添加列，新增列一律为空值
    * `DROP`：删除列
    * `ALTER`：修改列
    * `RENAME`：重命名表

    ```sql
      ALTER TABLE <table_name> 
        [ ADD[COLUMN]  <column_name> <data_type> [<column_constraint>]] 
        [ ADD <table_constraint> ]
        [ DROP [COLUMN] <column_name> [CASCADE | RESTRICT] ]
        [ DROP CONSTRAINT <constraint_name> [CASCADE | RESTRICT] ]
        [ ALTER COLUMN <column_name> <data_type> ]
    ```
*   删除基本表

    * `CASCADE`：删除表和相关的所有依赖对象
    * `RESTRICT`：若该表存在依赖对象/引用，则拒绝删除

    ```sql
      DROP TABLE <table_name> [CASCADE | RESTRICT]
    ```

#### 索引

* 索引：加快查询速度
* 由数据库系统自动维护
* 常见索引
  * 顺序文件索引
  * B+树索引
  * 哈希索引
  * 位图索引：bitmap
*   建立索引

    * 索引可以建立在一列/多列上
    * 次序：`ASC`：升序（默认），`DESC`：降序
    * `UNIQUE`：唯一索引，索引对应唯一数据记录
    * `CLUSTER`：聚簇索引，根据索引重新排列物理储存聚簇，提高查询效率

    ```sql
      CREATE [UNIQUE] [CLUSTER] INDEX <index_name> ON <table_name> (<column_name> [ASC | DESC], ...)
    ```
*   修改索引

    ```sql
      ALTER INDEX <index_name> RENAME TO <new_index_name>
    ```
*   删除索引

    ```sql
      DROP INDEX <index_name>
    ```

## SQL 数据类型

* 字符串
  * `CHAR(n)`, `CHARACTER(n)`：定长字符串
  * `VARCHAR(n)`, `CHARACTER VARYING(n)`：变长字符串
  * `NCHAR(n)`, `NVARCHAR(n)`: 定长/变长字符串，Unicode 字符集
* `CLOB(n)`, `TEXT`: 大文本
* `BLOB(n)`: 二进制大对象
* 数值
  * `SMALLINT`: 2字节整数
  * `INT`, `INTEGER`: 4字节整数
  * `BIGINT`: 8字节整数
  * `REAL`: 单精度浮点数（取决于机器精度）
  * `DOUBLE PRECISION`: 双精度浮点数（取决于机器精度）
  * `FLOAT(n)`: n位精度浮点数
  * `DECIMAL(p, d)`, `NUMERIC(p,d)`, `DEC(p, d)`: 定点数，p: 位数，d: 小·数位数
* 时间
  * `DATE`: 日期
  * `TIME`: 时间
  * `TIMESTAMP`: 时间戳
  * `INTERVAL`: 时间间隔
* `BOOLEAN`: 布尔值
* 选取属性的原则
  * 数据的取值范围
  * 数据需要做的运算

## 数据字典

* 数据库管理系统的内部表
* 记录关系模式、视图、索引、完整性约束、操作权限、统计信息等定义和信息
* SQL 语句执行时更新数据字典的内容

## SQL 数据查询

* `SELECT` 要显示的列
  * 目标列表达式可以是表达式/函数，如`<column_name> + 1`，`LOWER(<column_name>)`
  * 可以使用`AS`给列起别名，也可以不用`AS`，如`SELECT <column_name> AS <alias_name>`，`SELECT <column_name> <alias_name>`
  * `DISTINCT`：去重
  * `ALL`：保留重复值（默认）
  * `*`：所有列
* `FROM` 要查询的表/视图
* `WHERE` 查询条件
  * 比较运算符：`=`, `<>`, `!=`, `<`, `<=`, `>`, `>=`，`!<`, `!>`, `NOT + 运算符`
  * 范围：`BETWEEN <low> AND <high>`，`NOT BETWEEN <low> AND <high>`
  * 包含：`IN (<value>, ...)`，`NOT IN (<value>, ...)`
  * 字符匹配
    * `LIKE`：模糊匹配，`%`表示0个或多个任意字符，`_`表示单个字符
    * `NOT LIKE`
    * 指定转义符：`ESCAPE <escape_char>`，如`LIKE 'A\%' ESCAPE '\'`查询`A%`
  * 空值：`IS NULL`，`IS NOT NULL`
  * 逻辑运算符：`AND`, `OR`, `NOT`
    * 优先级：`NOT` > `AND` > `OR`
    * 可使用括号更改优先级
* `GROUP BY` 按照指定列的值分组
* `HAVING` 分组后的条件
* `ORDER BY` 排序
  * 根据多个列排序：前面的列优先级高

```sql
SELECT [ALL | DISTINCT] <column_name_expr> [ [AS] <column_alias_name>], ...
FROM <table_name|view_name>, ...| (SELECT ...) [AS <alias_name>]
WHERE <condition>
GROUP BY <column_name>, ...
HAVING <condition>
ORDER BY <column_name> [ASC | DESC]
```

{% hint style="warning" %}
关系运算中的投影需要去重，对应于 SQL 中的 `DISTINCT`
{% endhint %}

### 聚集函数

* `COUNT(*)`：统计行数
* `COUNT([DISTINCT|ALL] <column_name>)`：统计列数，默认为`ALL`
* `SUM([DISTINCT|ALL] <column_name>)`：求和，默认为`ALL`
* `AVG([DISTINCT|ALL] <column_name>)`：平均值，默认为`ALL`
* `MIN([DISTINCT|ALL] <column_name>)`：最小值，默认为`ALL`
* `MAX([DISTINCT|ALL] <column_name>)`：最大值，默认为`ALL`
* 除了`COUNT(*)`，其他聚集函数跳过空值

### 对查询结果分组

* `GROUP BY`：分组
* 对查询结果分组后，聚集函数将分别作用于每个组
*   `HAVING`作用于分组后的结果，`WHERE`作用于分组前的所有结果

    * 例：查询平均成绩大于等于90分的学生学号和平均成绩

    ```sql
     SELECT SNO, AVG(SCORE) AS AVG_SCORE
     FROM SC
     GROUP BY SNO
     HAVING AVG(SCORE) >= 90
    ```

### 连接查询

* 连接查询：同时涉及两个以上的表的查询

#### 连接查询求解方法

* 嵌套循环法：在表中找到第一个元组后，在第二个表中找到符合条件的元组，直到找到所有符合条件的元组，时间复杂度为$$O(n^2)$$
* 排序合并法：（用于等值连接）
  * 将两个表按照连接条件的属性进行排序
  * 将两个表进行合并，时间复杂度为$$O(nlogn)$$
* 索引链接法：
  * 对表 2 按连接字段建立索引
  * 对表 1 中的每个元组，利用索引在表 2 中查找符合条件的元组进行拼接
  * 时间复杂度为$$O(nlogm)$$，$$m$$为表 2 的记录数
*   等值连接：连接条件是等值关系

    * 例：查询每个学生及其选修课程的情况

    ```sql
    SELECT Student.*, SC.*
    FROM Student, SC
    WHERE Student.SNO = SC.SNO
    ```
*   自身连接：表格与自身连接，由于属性名相同，必须使用别名

    * 例：查询每一门课的间接先修课

    ```sql
    SELECT FIRST.Cno, SECOND.Cpno
    FROM C FIRST, C SECOND
    WHERE FIRST.Cpno = SECOND.Cno
    ```
* 外连接：保留悬浮元组，空值填充 NULL
  * 左外连接：`FROM <table_name> LEFT OUT JOIN <table_name> ON <condition>`
  * 右外连接：`FROM <table_name> RIGHT OUT JOIN <table_name> ON <condition>`
  * 这里写了`<condition>`，就不用`WHERE`了
* 多表连接：两个以上的表进行连接

### 嵌套查询

* 嵌套查询：在一个查询中嵌套另一个查询

```sql
SELECT <column_name>, ...
FROM <table_name>
WHERE <column_name> IN | EXISTS | ANY | ALL (
  SELECT <column_name> 
  FROM <table_name> 
  WHERE <condition>
  )
```

* 上层：父查询，下层：子查询
* 子查询不能使用`ORDER BY`
* 嵌套查询可以通过连接替代：谨慎使用嵌套查询
* 若确定内存查询返回单个值，可以使用比较运算符代替`IN`
* `ANY`：子查询返回的值中有一个符合条件即可
* `ALL`：子查询返回的值中所有符合条件
* 可尝试将`ANY`和`ALL`替换为聚集函数，提升性能
* `EXISTS`：子查询返回的结果集非空
  * `EXISTS`和`NOT EXISTS`只能用于子查询
  * `EXISTS`：子查询返回的结果集非空
  * `NOT EXISTS`：子查询返回的结果集为空
*   实现 $$\forall$$：使用两次 `NOT EXISTS`，$$(\forall x) P \equiv \neg((\exists x) \neg P)$$，都符合条件=不存在不符合条件的元组

    * 例：查询选修了所有课程的学生

    ```sql
    SELECT SNO
    FROM STUDENT S
    WHERE NOT EXISTS (
      SELECT Cno
      FROM C
      WHERE NOT EXISTS (
        SELECT Cno
        FROM SC
        WHERE SC.SNO = S.SNO AND SC.Cno = C.Cno
      )
    )
    ```
*   实现蕴含：$$P \Rightarrow Q \equiv \neg P \lor Q$$

    * 例：查询了至少选修了学生 201215122 选修的全部课程的学生号码
    * $$p$$：201215122 选修了课程 y
    * $$q$$：学生 x 选修了课程 y
    * $$\forall y (p \Rightarrow q)$$
    * $$\neg \exists y (p \land \neg q)$$

    ```sql
    SELECT DISTINCT SNO
    FROM SC S1
    WHERE NOT EXISTS ( -- 不存在有课程，在 201215122 选的同时，不存在其他人选了这门课 -> 对于每一门 201215122 选的课，筛选出来的人都选了 (描述的主语是 `SELECT` 后的结果)
      SELECT CNO
      FROM SC S2
      WHERE S2.SNO = '201215122'
      AND NOT EXISTS ( -- 不存在有 S3(S1) 选了 201215122 选的课
        SELECT CNO
        FROM SC S3
        WHERE S3.SNO = S1.SNO AND S3.CNO = S2.CNO
      )
    )
    ```

#### 嵌套查询求解方法

* 不相关子查询
  * 子查询的结果不依赖于父查询
  * 从内到外逐层处理
* 相关子查询
  * 子查询的结果依赖于父查询
  * 逐个选取外层表的元组，处理内层查询
  * 返回真保留，否则舍弃

### 集合查询

* `SELECT xxx op SELECT yyy`
* `UNION`：并集，默认去重
  * `UNION ALL`：并集，保留重复值
* `INTERSECT`：交集
* `EXCEPT`：差集
* 参与集合操作的查询结果必须列数相同，数据类型相同

### 基于派生表的查询

* `FROM (SELECT ...) AS <alias_name>`：在`FROM`中创建子查询

## SQL 数据插入

### 插入元组

```sql
INSERT INTO <table_name> [<column_name>, ...]
VALUES (<value>, ...)
```

* 属性列的顺序可以和表中定义的顺序不一致
* 未指定属性列时默认插入完整的元组，顺序按照表中顺序
* 未指定属性列时，未指定的列取空值
* `VALUES`中值的个数和类型必须和表中一致

#### 插入查询结果

```sql
INSERT INTO <table_name> [<column_name>, ...]
SELECT <column_name>, ...
FROM <table_name> [WHERE <condition>]
```

* `SELECT`中列数和类型必须和表中一致

## SQL 数据更新

```sql
UPDATE <table_name>
SET <column_name> = <value>, ...
WHERE <condition>
```

* `SET`中列数和类型必须和表中一致
* 若省略`WHERE`，则更新所有元组
* 关系数据库管理系统执行修改语句时，会检查修改操作是否破坏表上已定义的完整性规则
  * 实体完整性
  * 参照完整性
  * 用户定义的完整性：`NOT NULL`，`UNIQUE`，值域

## SQL 数据删除

```sql
DELETE FROM <table_name>
WHERE <condition>
```

* 若未指定`WHERE`，则删除所有元组，保留表的定义

## 空值

* 判断：`IS NULL`，`IS NOT NULL`

### 产生条件

* 应该有值，但是不知道具体值
* 不应该有值
* 因为某种原因不便于填写：~~性别类型是`BOOLEAN`，但是用户是小男娘~~

### 空值的约束条件

* `NOT NULL`：不允许空值
* `UNIQUE`：不允许重复值
* 码属性不能为空

### 空值的计算

* `NULL`参与算术计算时，结果为`NULL`
* `NULL`参与比较时，结果为`UNKNOWN`
* 含有`UNKNOWN`的逻辑运算:`TRUE` > `UNKNOWN` > `FALSE`
  * `NOT UNKNOWN`：`UNKNOWN`
  * `AND`：取“小”的
  * `OR`：取“大”的

## 视图

* 视图：从一个或多个基本表（或视图）导出的表
* 只存放定义，不重复存储数据
* 查询的数据随基本表的变化而变化
* 作用
  * 封装不同表的复杂性，简化用户操作
  * 为不同用户提供看待数据的不同角度
  * 便于重构底层数据库，无需更改用户的外模式
  * 安全防护：权限设置
  * 适当的利用视图可以更清晰的表达查询

### 建立视图

```sql
CREATE VIEW <view_name> [<column_name>, ...]
AS <query>
[WITH CHECK OPTION]
```

* `<query>`：任意查询语句 `SELECT ... FROM ... WHERE ...`
* `WITH CHECK OPTION`：检查视图的完整性约束，视图的插入、删除、更新操作必须满足视图定义的谓词条件（`WHERE`条件）
* 列名
  * 全部省略：视图的列为查询所得的全部字段
  * 明确指定：
    * 某个目标列是聚集函数或列表达式​
    * 多表连接时选出了几个同名列作为视图的字段​
    * 需要在视图中为某个列启用新的更合适的名字
* 行列子集视图：从单个基本表导出，只去掉某些行列，保留主码

### 删除视图

```sql
DROP VIEW <view_name> [CASCADE]
```

* `CASCADE`：删除视图的同时删除该视图导出的视图

### 查询视图

* 用户命令：和查询表相同
* 系统实现
  1. 进行有效性检查
  2. 转化为等价的对基本表的查询
  3. 执行修正后的查询
* 部分情况下无法正常转换：`GROUP BY`等

### 更新视图

```sql
UPDATE <view_name>
SET <column_name> = <value>, ...
WHERE <condition>
```

* 不可更新视图：对视图的更新无法有意义地转换为对基本表的更新
* 不可更新视图上定义的视图也不可更新
