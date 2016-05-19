---
title: 使用git@osc的hook功能自动部署
mathjax: false
date: 2016-05-19 19:18:54
tags:
categories:
---
# 前言  
git@osc的hook功能能够在推代码的时候进行post http请求，所以，做了小程序，来让我的博客自动更新更新github page的项目的一个东西，应该说，项目是很简单的。  
项目分为两个部分，一部分是web部分，使用flask来接收来自git@osc的webhook，一部分做调用系统的hexo命令来进行更新操作。
<!-- more --> 
# 实现逻辑说明  
因为webhook的超时时间是5s，所以，flask项目的作用就是一个触发器，接收到请求后，通过redis来发送消息给另外一个守护进程。  
flask相关的核心代码：
```python
import redis
rd = redis.StrictRedis(host="localhost")
data = request.form['hook']
password = json.loads(data)['password']
rd.publish("hexo", "hexo")

```
另一个守护进程收到之后，进入到git@osc，执行三个操作：
```python
import commands
commands.getoutput("git pull")
commands.getoutput("hexo g")
commands.getoutput("hexo d")
```
# 总结  
整个流程很简单，只有两步。要学会使用编程解决一些小问题，这就是一个很好的例子，虽然代码很少，但是能很好的学习python
# 测试1  
