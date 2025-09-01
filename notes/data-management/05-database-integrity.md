# 05-数据库完整性

## 基本概念

* 数据库完整性
  * 数据的正确性：数据符合现实语义
  * 数据的相容性：同一对象在不同表中表示符合逻辑
* 完整性机制
  * 定义完整性约束条件的机制
  * 完整性检查的方法
  * 违约处理

{% hint style="info" %}
- 完整性：防范不合语义、不正确的数据
- 安全性：防范非法用户和非法操作
{% endhint %}

## 实体完整性（定义主键）

* 单个属性为主键：可在列/表级定义

```sql
-- 在列级定义
CREATE TABLE student (
    sno INT PRIMARY KEY,
    sname VARCHAR(20),
    sage INT
);
-- 在表级定义
CREATE TABLE student (
    sno INT,
    sname VARCHAR(20),
    sage INT,
    PRIMARY KEY (sno)
);
```

* 多个属性为主键：只能在表级定义

```sql
CREATE TABLE student (
    sno INT,
    sname VARCHAR(20),
    sage INT,
    PRIMARY KEY (sno, sname)
);
```

### 检查实体完整性

* 检查
  * 主键是否唯一
  * 主键的各个属性是否非空
* 检查手段
  * 全表遍历：$$O(n)$$
  * B+ 树索引：$$O(\log n)$$

## 参照完整性（定义外键）

* `FOREIGN KEY`：定义外键的列
* `REFERENCES`：定义外键引用的表和列（必须是唯一的）

```sql
CREATE TABLE student (
    sno INT PRIMARY KEY,
    sname VARCHAR(20),
    sage INT
);
CREATE TABLE course (
    cno INT PRIMARY KEY,
    cname VARCHAR(20),
    sno INT,
    FOREIGN KEY (sno) REFERENCES student(sno)
);
```

### 检查参照完整性

| 操作         | 触发条件                    | 触发操作         |
| ---------- | ----------------------- | ------------ |
| 插入违反外键约束   | 插入新记录时，外键值在被引用的表中不存在    | 拒绝插入         |
| 更新违反外键约束   | 更新外键值时，外键值在被引用的表中不存在    | 拒绝更新         |
| 删除被引用的主键记录 | 删除主键记录时，引用表中仍有使用该外键值的记录 | 拒绝删除/级联删除/置空 |
| 更新被引用的主键记录 | 更新主键记录时，引用表中仍有使用该外键值的记录 | 拒绝更新/级联更新/置空 |

#### 违约处理语法

* `NO ACTION`：拒绝操作，默认
* `CASCADE`：级联操作，删除/更新主键时，删除/更新引用表中所有外键值相同的记录
* `SET NULL`：设置为空

```sql
...
    FOREIGN KEY (sno) REFERENCES student(sno) 
      ON UPDATE CASCADE
      ON DELETE SET NULL
...
```

## 用户定义的完整性（语义要求）

* 单个属性的取值条件
  * `UNIQUE`：唯一性约束
  * `NOT NULL`：非空约束
  * `CHECK`：取值范围约束
* 元组的取值条件：属性之间存在依赖，需要约束

```sql
CREATE TABLE student(
    sno UNIQUE,
    sname NOT NULL,
    ssex CHAR(1) CHECK (ssex IN ('M', 'F')),
    CHECK(ssex='F' OR sname NOT LIKE 'Ms.%'),
)
```

## 完整性约束命名子句

* `CONSTRAINT`：为约束规则显式命名，便于后续更改/删除
* 不用也能正常定义规则，但是为修改和维护带来不便

```sql
CREATE TABLE student(
    sno INT,
    sname VARCHAR(20) CONSTRAINT name_check NOT NULL,
    ssex CHAR(1) CONSTRAINT sex_check CHECK (ssex IN ('M', 'F')),
    CONSTRAINT name_sex_check CHECK(ssex='F' OR sname NOT LIKE 'Ms.%'),
);
-- 修改约束
ALTER TABLE student
    DROP CONSTRAINT name_check;
ALTER TABLE student
    ADD CONSTRAINT name_check UNIQUE (sname);
```

## 断言

* 若断言为假，拒绝执行
* 性能开销较大

```sql
CREATE ASSERTION max_enrollment_check
    CHECK (60 >= (SELECT COUNT(*) 
      FROM course,sc 
      WHERE sc.cno = course.cno AND course.name='DB'));
```

## 触发器

* Event-Condition-Action：当特定事件发生，且满足触发条件时，激活触发器，执行定义的操作
* 保存在数据库服务器中
* 表的拥有者才可以在表上创建触发器
* 触发器名
  * 同一模式下必须唯一
  * 触发器名称和表名称必须在同一模式下
* 表名：触发器只能定义在基本表上
* 触发事件
  * `INSERT` `UPDATE` `DELETE`
  * `UPDATE OF <column_name>`：只在指定列上触发
* 激活顺序：`BEFORE`触发器、SQL 语句、`AFTER`触发器
* 执行频率
  * `FOR EACH ROW`：每行触发一次
  * `FOR EACH STATEMENT`：每个语句触发一次
* 动作：可以是
  * SQL 语句
  * PL/SQL 语句块
* 触发器的删除者必须有对应的权限

```sql
CREATE TRIGGER <trigger_name>
  {BEFORE | AFTER} <event> ON <table_name>
  REFERENCING {OLD | NEW} ROW AS <alias>
  FOR EACH {ROW|CONDITION}
  [WHEN <condition>] <action>
DROP TRIGGER <trigger_name> ON <table_name>;
```

```sql
CREATE TRIGGER Insert_Or_Update_Sal ​
BEFORE INSERT OR UPDATE ON Teacher  ​
REFERENCING NEW row AS newTuple​
FOR EACH ROW
  BEGIN​
    IF (newTuple.Job='教授') AND (newTuple.Sal < 4000) ​
      THEN newTuple.Sal :=4000;                ​
    END IF;​
  END;
-- 删除触发器
DROP TRIGGER Insert_Or_Update_Sal ON Teacher;
```
