---
title: MySQL使用备忘录
date: 2018-06-09 13:05:35
categories:
- 技术笔记
tags:
---
#### 创建数据库，并指定该数据库的字符集编码为 utf8mb4  
```mysql
CREATE DATABASE `irenshi` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci
```

#### 用 root 权限创建名为 Vent 的数据库    
```mysql
mysqladmin -u root -p create Vent
```


#### 把数据库'test'中的所有表的所有权限给用户'Frank'  
```mysql
mysql> grant all privileges on test.* to 'Frank'@'localhost'
    -> ;
Query OK, 0 rows affected (0.01 sec)
```

#### 登录'root'用户  
```mysql
mysql -u root -p
```

#### 收回权限  
"grant" -> "revoke"  
"to" -> "from"

#### 使用数据库'test'  
`use  test;`

#### 创建用户  
```mysql
CREATE USER 'username'@'host' IDENTIFIED BY 'password';
```

#### 删除用户  
```mysql
DROP USER 'username'@'host';
```

#### 通过 brew 安装，启动`mysql`服务  
```
brew install mysql
```
```bash
xiefengdeAir:~ Frank$ mysql.server stop
Shutting down MySQL
.. SUCCESS! 
xiefengdeAir:~ Frank$ mysql.server start
Starting MySQL
. SUCCESS! 
xiefengdeAir:~ Frank$ mysql.server restart
Shutting down MySQL
.... SUCCESS! 
Starting MySQL
. SUCCESS! 
```

#### 修改默认值
```mysql
alter table table_name alter column_name set default default_value;
```


#### 修改指定表的字符集编码为 utf8mb4  
```mysql
ALTER TABLE [table_name] CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```
