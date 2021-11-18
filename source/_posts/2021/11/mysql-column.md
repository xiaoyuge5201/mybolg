---
title: Mysql添加、修改、删除字段
date: 2021-11-15 21:13:42
tags: mysql
comments: false
categories:
- 数据库
- mysql
translate_title: mysql_add_modify_drop_column
---
### 1. 添加字段
#### 1.1 在末尾添加字段
1）语法：
```sql
ALTER TABLE <表名> ADD <字段名> <数据类型> [约束条件];
```
语法格式的说明：
- <表名> 为数据表的名字；
- <字段名> 为所要添加的字段的名字；
- <数据类型> 为所要添加的字段能存储数据的数据类型；
- [约束条件] 是可选的，用来对添加的字段进行约束。
- 这种语法格式默认在表的最后位置（最后一列的后面）添加新字段

2）示例：在user表末尾添加字段phone
```sql
ALTER TABLE user ADD phone VARCHAR(11) DEFAULT NULL COMMENT '电话号码';
```
#### 1.2 在开头添加字段
1）语法：
```sql
ALTER TABLE <表名> ADD <字段名> <数据类型> [约束条件] FIRST;
```
FIRST 关键字一般放在语句的末尾
2）示例：在user表开头添加字段user_id
```sql
ALTER TABLE user ADD user_id VARCHAR(32) NOT NULL COMMENT '用户主键' FIRST;
```
#### 1.3 在中间添加字段
1）语法：
```sql
ALTER TABLE <表名> ADD <字段名> <数据类型> [约束条件] AFTER <已经存在的字段名>;
```
AFTER 的作用是将新字段添加到某个已有字段后面。
注意：只能在某个已有字段的后面添加新字段，不能在它的前面添加新字段

2）示例：在user表的user_id字段后添加username字段
```sql
ALTER TABLE user ADD username VARCHAR(30) DEFAULT NULL COMMENT '用户名' AFTER `user_id`;
```
### 2. 修改字段
#### 2.1 修改字段属性
1）语法：
```sql
ALTER TABLE <表名> MODIFY <字段名> <数据类型> [约束条件];
```
2）示例1：修改字段属性
```sql
-- 将email字段VARCHAR(50)修改成VARCHAR(200)
ALTER TABLE user MODIFY email VARCHAR(200) NOT NULL DEFAULT 'email@163.com';
```
注意：修改时如果不带完整性约束条件，原有的约束条件将丢失，如果想保留修改时就得带上完整性约束条件

3）示例2： 将email移到phone后面
```sql
ALTER TABLE user MODIFY email VARCHAR(50) AFTER `phone`;
```
4）示例3：放置第一个，保留原完成性约束条件
```sql
ALTER TABLE user`MODIFY email VARCHAR(50) NOT NULL DEFAULT 'test@163.com' FIRST;
```
5）示例4：修改成大小写敏感，即查询区分大小写
```sql
ALTER TABLE user MODIFY username VARCHAR(30) BINARY CHARACTER SET utf8 COLLATE utf8_bin DEFAULT NULL COMMENT '用户名';
```
#### 2.2 修改字段名称和属性
1）语法：
```sql
ALTER TABLE <表名> CHANGE <原字段名> <新字段名> <数据类型> [约束条件];
```
2）示例：将username字段修改成user_name
```sql
ALTER TABLE user CHANGE username user_name VARCHAR(30) DEFAULT NULL COMMENT '用户名';
```

#### 2.3 添加、删除默认值
1）语法：
```sql
-- 添加默认值
ALTER TABLE <表名> ALTER <字段名> SET DEFAULT <默认值>;

-- 删除默认值
ALTER TABLE <表名> ALTER <字段名> DROP DEFAULT;
```
2）示例：给sex添加默认值
```sql
ALTER TABLE USER ALTER sex SET DEFAULT  '难';
```
3）示例：删除sex默认值
```sql
ALTER TABLE user ALTER sex DROP DEFAULT;
```

#### 2.4 添加、删除主键
1) 语法：
```sql
-- 添加主键
ALTER TABLE <表名> ADD [CONSTRAINT <约束名>] PRIMARY KEY (<字段名称,...>);

-- 删除主键
ALTER TABLE <表名> DROP PRIMARY KEY;
```
2）示例：添加主键
```sql
ALTER TABLE user ADD PRIMARY  KEY (user_id)
```

3）示例：添加复合主键
```sql
ALTER TABLE  user_role ADD PRIMARY KEY (user_id, role_id);
```
4）示例：删除主键
```sql
ALTER TABLE user DROP PRIMARY KEY;
```
5）示例：删除带自增长属性的主键
```sql
-- 先用MODIFY删除自增长属性，注意MODIFY不能去掉主键属性
ALTER TABLE test MODIFY id INT UNSIGNED;
-- 再来删除主键
ALTER TABLE test DROP PRIMARY KEY;
```
#### 2.5 添加、删除唯一索引
1）语法：
```sql
-- 添加唯一性约束
ALTER TABLE <表名> ADD [CONSTANT <约束名>] UNIQUE [INDEX | KEY] [索引名称](<字段名称,...>)

-- 删除唯一性约束
ALTER TABLE <表名> DROP [INDEX | KEY] [索引名称];
```
2）示例：为username添加唯一性约束，如果没有指定索引名称，系统会以字段名建立索引
```sql
ALTER TABLE user ADD UNIQUE(username);
```
3）示例：为username添加唯一性约束，并指定索引名称
```sql
ALTER TABLE user ADD UNION KEY uni_username(username);
```
4）示例：查看索引
```sql
SHOW CREATE TABLE user;
```
5）示例：添加联合UNIQUE
```sql
ALTER TABLE user ADD UNIQUE INDEX uni_nickname_username(nickname, username);
```
6）示例：删除索引
```sql
ALTER TABLE user DROP INDEX username;
ALTER TABLE user DROP KEY uni_username;
ALTER TABLE user DROP INDEX uni_nickname_username;
```
#### 2.6 修改表的存储引擎
1）语法：
```sql
ALTER TABLE <表名> ENGINE=<存储引擎名称>
```
2）示例：
```sql
ALTER TABLE user ENGINE=MyISAM;
ALTER TABLE user ENGINE=INNODB;
```
#### 2.7 修改自增长值
1）语法：
```sql
ALTER TABLE <表名> AUTO_INCREMENT=[值];
```
2）示例：
```sql
ALTER TABLE user AUTO_INCREMENT= 100;
```


博客原文链接：https://www.cnblogs.com/Jimc/p/12979319.html
如有侵权，请联系删除！
