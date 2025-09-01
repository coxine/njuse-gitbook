# 04-数据库安全性

## 存取控制

* 定义用户权限，将用户权限登记至数据字典中
* 用户发出存取请求时检查合法权限
* DBMS 的存取控制子系统：用户权限定义 + 合法权检查机制

### Discretionary Access Control (DAC)

* 用户对不同对象拥有不同的存取权限
* 不同用户对同对象的存取权限不同
* 用户可以将权限授予其他用户或收回权限：SQL 中的 `GRANT` `REVOKE`
* 缺点：数据本身无安全性标记，可能存在数据的泄露

### Mandatory Access Control (MAC)

* 数据对象被标以密级
* 用户被授予许可证
* 只有拥有合法许可证的用户才能访问数据
* 用户无法直接感知/进行控制
* 实体分类
  * 主体：系统中的活动实体，实际用户、进程
  * 客体：系统中的被动实体，文件、基本表、索引、视图
* 敏感度标记：Top Secret>=Secret>=Confidential>=Public
  * 主体的敏感度标记：许可证级别
  * 客体的敏感度标记：密级
* 控制规则
  * 读：主体的敏感度标记 >= 客体的敏感度标记，低级别的用户不能读高级别的对象，以防高级别数据泄漏
  * 写：主体的敏感度标记 <= 客体的敏感度标记，高级别的用户不能写低级别的对象，以防高级别数据泄漏
* 实现 MAC 前需要先实现 DAC

## SQL 中的授权机制

* 授权：定义用户的存取权限 `GRANT` `REVOKE`
* 用户权限：数据库对象 + 操作类型

### 用户权限

* 数据库管理员
  * 拥有所有对象的所有权限
  * 根据实际情况不同的权限授予不同的用户
* 用户
  * 拥有自己建立的对象的全部操作权限
  * 可以使用 `GRANT` 将权限授予其他用户
* 被授权的用户
  * 若拥有继续授权的许可，则可以将权限授予其他用户

### 授权语句

#### GRANT

```sql
GRANT <privilege> [, <privilege>...]
ON <object type> <object name> [, <object type> <object name>...]
TO <user> [, <user>...]
[WITH GRANT OPTION]
```

* `privilege`：权限
  * 数据库模式：`CREATE SCHEMA` `CREATE TABLE` `CREATE VIEW` `CREATE INDEX` `ALTER TABLE`
  * 基本表：`SELECT` `INSERT` `UPDATE` `DELETE` `REFERENCES` `ALL PRIVILEGES`
  * 属性列：`SELECT` `INSERT` `UPDATE` `REFERENCES` `ALL PRIVILEGES`
* 可发出 `GRANT` 的用户
  * 数据库管理员
  * 数据库对象创建者
  * 拥有权限的用户
* 接受权限的用户
  * 一个/多个具体用户
  * `PUBLIC`：所有用户
* `WITH GRANT OPTION`：被授权的用户可以将权限授予其他用户
* 不允许循环授权
* 对属性列授权时必须指出列名如`UPDATE(col1, col2)`

#### REVOKE

```sql
REVOKE <privilege> [, <privilege>...]
ON <object type> <object name> [, <object type> <object name>...]
FROM <user> [, <user>...]
[CASCADE|RESTRICT]
```

* `CASCADE`：删除所有引用该对象的权限
  * 若该对象从其他对象获得该权限，则不删除

#### 创建数据库模式的权限

* 在创建用户时实现

```sql
CREATE USER <user> [IDENTIFIED BY <password>]
[WITH] [DBA|RESOURCE|CONNECT]
```

* `DBA`：拥有所有权限
* `RESOURCE`：可创建基本表、视图，不可创建模式、新用户
* `CONNECT`：只可连接数据库

#### 角色

* 角色权限的集合

```sql
CREATE ROLE <role name>
-- 为角色授权
GRANT <privilege> [, <privilege>...]
ON <object type> <object name> [, <object type> <object name>...]
TO <role name>
-- 授权给其他角色/用户
GRANT <role name> [, <role name>...]
TO <user> [, <user>...]
[WITH ADMIN OPTION]
-- `REVOKE` 语法相同，省略
```

* `WITH ADMIN OPTION`：被授权的用户可以将权限授予其他用户
* `REVOKE` 的执行者
  * 角色创建者
  * 拥有在角色上 `ADMIN OPTION`的用户

## 视图

* 另一种管理权限的实现
* 语法和基本表相同，相见第三章笔记

## 审计

* 将用户对 DB 的操作记录在日志上以供审计
* 审计事件
  * 服务器事件
  * 系统权限
  * 语句事件
  * 模式对象事件
* 审计功能：规则、报表、管理、查询视图
* 审计级别
  * 用户级审计：任何用户均可设置，用户自己创建的对象
  * 系统级审计：只能由管理员设置，全局性

```sql
AUDIT <privilege> [, <privilege>...] ON <object type> <object name> 
NOAUDIT <privilege> [, <privilege>...] ON <object type> <object name>
```

## 加密

* 储存加密
  * 透明存储加密：写入时加密，读出时解密
  * 非透明存储加密
* 传输加密
  * 链路加密
  * 端到端加密

## 其他安全性保护

* 推理控制：避免利用能够访问的数据推知更高级的数据
* 隐蔽信道：间接数据传递
* 数据隐私保护
