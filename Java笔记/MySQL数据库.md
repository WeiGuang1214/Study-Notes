### 关系型数据库MySQL

1、SQL通用语法

单行、多行、以分号结尾

有空格和缩进

建议关键字用大写，虽然SQL不区分大小写

单行注释：-- 或者 # 

多行注释：/* **xxxxx***/

DDL数据定义、定义

```sql
# 创建
CREATE DATABASE [IF NOT EXISTS] NAME [DEFAULT CHARSET 字符集] [COLLATE 排序规则];
# 删除
DROP DATABASE [IF EXISTS] name;
# 查询
SHOW DATABASES;
# 查询当前数据库
SELECT DATABASE();
# 使用
USE 数据库名;


# 查询库里面的所有表
SHOW TABLES;

# 创建表
CREATE TABLE name(
    字段1 字段1类型 COMMENT 字段1注释,
	字段1 字段1类型 COMMENT 字段1注释,
    字段1 字段1类型 COMMENT 字段1注释
)COMMENT 表注释

CREATE TABLE table_user(
	id BIGINT COMMENT 'id',
    name VARCHAR(128) COMMENT '名字',
    user_id BIGINT NOT NULL COMMENT 'userID'
) COMMENT '用户表';

# 查询表结构,查看字段、字段类型、null、default、extra
DESC name;

# 展示完整的创建表的语句
SHOW CREATE TABLE name;

```

DML数据操作、增删改

```sql

```

DQL数据查询

DCL数据控制，创建数据库用户、控制数据库访问权限

