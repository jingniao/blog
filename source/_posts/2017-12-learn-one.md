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

## pymysql/sqlalchemy
为了保证数据库的中文编码正确，以下是几点建议：
* 数据表字段的编码应该兼容中文的，例如`latin1`就不兼容`中文`，尽量不要使用这种编码，不是说一定不能用，如果禁止使用`mysql驱动`的`unicode`编码功能，在驱动外进行手动`encode` `decode`，传递给数据库直接是`str`，或者说是字节数组，还是可以得到正确的内容的，这就是所谓的错进错出了，但这时候因为数据库无法正确理解数据内容，也就无法正常排序查找了。
* 数据库连接字符集尽量使用与兼容中文的编码，例如`gbk`，`utf-8`，然后`python2`中输入给数据库的字符串使用`unicode`，不要使用`str`，让数据库驱动处理编码问题。
* 当`sqlalchemy`的`engine`的`url`有`charset`设置的时候，数据库驱动会默认使用`use_unicode=True`，这时候查询到的`orm`中的`String`类型，会以`unicode`返回，这点要注意，但是如果没有设置`charset`，在`python2`下`pymysql`中会将`charset`默认设置为`latin1`，然后`use_unicode`会被设置为`False`，这时会导致返回的`String`类型的数据为`python2`中的`str`，所以为了防止混乱，在`engine`创建的时候设置`charset`是一个比较好的选择
* `mysql`驱动`pymysql`对输入`str`跟`unicode`的处理差别：`pymysql/cursors.py Cursor._escape_args` 函数会进行参数编码检测，如果输入的参数是`unicode`类型的，则会调用`encode`使用连接编码进行编码
