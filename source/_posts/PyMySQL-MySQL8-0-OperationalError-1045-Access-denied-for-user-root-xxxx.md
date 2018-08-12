---
title: '[PyMySQL&MySQL8.0]OperationalError: (1045, ''Access denied for user ''root''xxxx'
date: 2018-06-29 09:09:49
categories:
- 技术笔记
tags:
---
* 具体报错：`OperationalError: (1045, "Access denied for user 'root'@'localhost' (using password: NO)")`
* 连接数据库的代码如下：   
```python
import pymysql
config = {
    'host': 'localhost',
    'user': 'root',
    'password': 'passwd',
    'db': 'metagene',
}
connection = pymysql.connect(**config)
```
* 明明填写了密码，却提示**using password: NO**，到 google 各处转了一大圈，终于锁定了问题所在：使用了最新的 MySQL8.0，而 
  > MySQL8.0 用到了新的密码插件验证方式，5.7叫做mysql_native_password，8.0叫做caching_sha2_password，这种加密方式让很多和MySQL连接的界面工具(如Navicat)或编程语言(如PHP)mysqli接口失效。
  引用自[掘金](https://juejin.im/entry/5adb5deff265da0b9d77cb3b)。
* 两种解决办法，一种是使用 mysql 官方实现的 python 接口 [mysql-connector-python](https://github.com/mysql/mysql-connector-python)，一种是将 MySQL 降级为 5.7 版本。
