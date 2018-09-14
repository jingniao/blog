---
title: django升级-从1.6到1.11
mathjax: false
date: 2018-09-14 21:46:10
tags:
categories:
---
# 升级套路
基本上是将Django大版本升级，这三条命令循环逐步升级到`<2`
```shell
pip install -U "Django<1.8"
python manage.py runserver
python manage.py migrate
```
# 中间更改项
## urls.py更改
`from django.conf.urls import patterns`
patterns  弃用
urlpatterns从 patterns的实例变为一个普通的数组或者元组
`url(r'^test$', 'test.views.snips', name='snip')`方式不不再可用，将url的第二个参数替换为views函数或者其他可直接引用的函数
## 配置的修改
settings.py中
zh-cn到zh-hans
TEMPLATES添加（还没有研究这个设置详情）
TEMPLATE_* （应该是被TEMPLATES替代了）

## 总结
这还是一个不那么老的项目，而且代码量很少，剩下的就难看了，也没有动力去整