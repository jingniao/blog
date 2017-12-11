---
title: 学习笔记（二）
mathjax: false
date: 2017-12-11 20:56:21
tags: [linux,mysql,sqlalchemy]
categories: 
---

## 正则

* 用户名 `"/^[a-zA-Z][a-zA-Z0-9_]{5,15}$/"` 字母开头，包含数字字母下划线 总长度为6-16位
* 用户姓名 `"/^[\x{4e00}-\x{9fa5}A-Za-z0-9_@、.\[\]\":\-]{1,100}$/u"` 中文，数字字母符号，1到100位
* 手机号 `"/^[1]{1}[0-9]{10}$/"` 以1开头，11位数字

<!-- more -->

## mysql编码

* 查看字符集 `show variables like '%char%';`   
!['mysql编码'](/image/mysql_bianma.png)    
`character_set_results`、`character_set_client` 、`character_set_connection` 这三者会被mysql配置文件里`[client]`里的`default-character-set`影响，也会被`set names utf8` 影响


* mysql 5.5.3 版本之后支持`utf8mb4`，也就是支持4字节的utf8编码，目前常用的emoji字符有不少都是4字节的，目前支持utf8mb4编码的mysql驱动只有；['pymysql', 'mysqldb']
* 如果要设置需要存储utf8mb4字符，可以进行如下设置：
```ini
[mysqld]

character-set-server=utf8mb4 
# 服务器默认编码，会影响新创建的数据库的编码，数据库的编码会影响新创
# 建的表的编码，表的编码一般是存储数据的事迹编码，当然还可以设置某列
# 的编码

collation-server = utf8mb4_unicode_ci #排序规则

# 根据语义，初始化链接的时候，执行某些语句
init_connect='SET NAMES utf8mb4' 
init_connect='SET collation_connection = utf8mb4_unicode_ci'
character-set-client-handshake=true # 忽略客户端的编码设置

[client]
default-character-set=utf8mb4 # 客户端默认编码设置
# 与 set names utf8mb4一样的想过
```
* 如果有条件的话，还是将数据库设置为同一编码，数据库连接也设置位同一编码，否则有时候是自找麻烦

## flask
* `flask`默认自带的`session`，是将数据加密存放在`client`端的`cookie`里的
* `python2` 中`flask`，`jinja2` 内容默认使用的都是`unicode`，在使用`jinja2`模板的时候，会将传入的`str`类型进行`decode(sys.getdefaultencoding)`操作，所以如果传入 `'中文'` 而不是 `u'中文'`，则j`inja2`会抛出异常
* `flask-sqlalchemy` 在使用`mysql`作为后端的时候，会自动添加`charset=utf8`参数，在某些编码情况下，会出现比较恼人的编码问题
*  `sqlite3` 在存取数据只接受`unicode`，不接受`str`，mysql大部分驱动会接受`str`跟`unicode`