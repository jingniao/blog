---
title: python学习——supervisor和redis,pgsql数据库连接
mathjax: false
date: 2016-04-06 00:21:35
tags: [supervisor,redis,postgresql]
categories: python
---

## 前言 
项目考虑使用守护进程，所以在看这方面的东西。管理使用supervisor来管理。很方便的使用supervisor
### supervisor使用
1，安装supervisor
```bash
yum install supervisor
```
2，配置supervisor，/etc/supervisord.d/下以ini为扩展名的都是配置：
```ini
[program:ddd]
command=/root/daemon.py
autorestart=true
```
command指向要执行的命令就可以了
<!-- more --> 

### redis数据库的链接以及消息队列
```python
import redis
rd = redis.StrictRedis(host=”localhost”)
rd.publish(uid,testnum)
#以上是向某个频道（channel）发布消息
channel = rd.pubsub()
channel.subscribe(uid)
msg = channel.parse_response()
#这个方法会阻塞执行，如果redis消息队列里没有消息，则会停在这里。
rd_fd = channel.connection._sock.fileno()
#获取频道的socket？类似于文件句柄的东西。这样就能用select或者epoll模型监听这个socket？从而判断是否有消息？这点存疑，以后用的时候在研究了
ready = select.select([rd_fd], [], [], 4.0)
#参数，三个列表，等待读取列表，等待写入列表，等待错误列表，最后是等待时间，这个函数会阻塞执行，这个是超时时间。
```
### postgresql数据库，使用python链接
```python
conn = psycopg2.connect(host=self.host, user=self.user, password=self.passwd) #链接到数据库
r = conn.cursor() #获取游标
r.execute(self.query[‘config’]) #进行sql语句的使用
r.close()
conn.commit() #提交事务
conn.close() #关闭游标以及数据库
```
