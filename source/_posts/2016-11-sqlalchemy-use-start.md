---
title: sqlalchemy的使用（一）——初始化以及简单使用
mathjax: false
date: 2016-11-26 11:55:11
tags: [python,sqlalchemy,sql]
categories: python
---
# 前言
学习并在工作中使用python的时间并不长，`django`的使用只能说用到的功能比较熟悉而已。并且工作中使用的并没有使用太多插件。只是将`django`作为一个请求接受以及返回格式化数据的东西。  
之前使用到了数据库，使用的是django自带的orm框架，有点是简单，并且在django中使用方便。最近需要在django外的独立程序中使用数据库，虽然有方法使用django的orm，但是毕竟隔了一层，不太方便移植的特点，让我开始学习`sqlalchemy`这个大名鼎鼎的python数据库框架。以下就是我遇到的几个坑，应该是比较基础的，记录下。

<!-- more --> 

# 正文

## 类映射
这里的一个类映射到数据库中是一个表  
db.py:

```python
from sqlalchemy import Column, String, Integer
from sqlalchemy.ext.declarative import declarative_base
Base = declarative_base()

class Persion(Base):
    __tablename__ = 'persion'
    id = Column(Integer, autoincrement=True, primary_key=True) 
    name = Column(String(1024))
```

除了`String`,`Integer`，还有其他类型，类型数量很多
`Column`的的第一个参数是数据表中的列名，可以省略，当省略的时候，将这个类变量的名字作为数据表中的列名。
访问`Persion.id`访问的是`persion`表中的`id`列


## 数据表初始化，以及简单查询

### 初始化表

初始化放到另外一个文件: 
client.py: 
```python
from sqlalchemy.orm import sessionmaker
from sqlalchemy import create_engine
from db import Persion,Base

db_path = "sqlite:///database.db"
engine = create_engine(db_path)  #创建连接引擎

Base.metadata.reflect(engine)  # 将数据映射绑定到引擎
Base.metadata.create_all(engine)  # 创建所有映射表
```

如果有多个类似与`db.py`文件，需要一起进行初始化的话，只要引用的是同一个`Base`实例，运行`create_all()`时类已经被导入，则可以一起初始化。   
>上述例子中`db_path`是数据库连接地址，这里只是sqlite简单的例子，如果是其他的类似`mysql`，`postgresql`等例子，会包含数据库名，驱动名，用户，密码，数据库名，[官方说明](http://docs.sqlalchemy.org/en/latest/core/engines.html)
### 插入数据库
接着上节的`client.py`

```python
Session = sessionmaker()
Session.configure(bind=engine)
session = Session() #然后就可以使用session进行查询操作了

persion_obj = Persion(id=1,name='jingniao')
session.add(persion_obj)
session.commit()
```

### 数据库连接进行查询

```python
query = session.query(Persion)
result = query.all()

```
解释：    
`query`是一个`sqlalchemy.orm.query.Query` object，代表的是一个查询操作。上面的例子是返回`Persion`的所有例子，如果数据表里有内容的话，会返回所有内容，类似与`select * from persion;`，不加过滤条件。  
如果要加过滤条件的话，可以添加`filter`，filter返回的也是一个`Query`对象，也就是说可以跟多个`filter`，进行链式调用。
```python
query = session.query(Persion).filter(Persion.id==1)
```
`Query`除了进行`filter`进行查询过滤外还有其他方法：  
`all` 返回所有的查询结果，是一个List,列表项是查询的对象实例，例如上面的例子中是Persion的实例  
`first` 返回的是所查询结果集的第一个实例  
`update` 更新数据操作，参数类似于`query`方法的参数  
`delete` 删除操作，将查询到的数据集删除  

除了上面的`update`操作，对于`all`返回的对象列表中的值或者`first`返回的对象来说，直接更改实例的属性的值也可以达到同样的效果  
上述更改操作（`update`，或者更改对象属性），删除操作(`delete`)，添加记录(`add`)，都需要对`session`进行`commit`操作（事物提交）
```python
session.commit()
```
### 结果转化
在`django`的orm使用中，对结果集，有自带的values_list方法将结果转化成python的字典列表格式，方便手动输出json或者还有设置序列化直接输出json的。  
`sqlalchemy`中好像就没那么方便的感觉（可能还有其他方法比较简便的就不清楚了），下面是一种方法：  
在db.py文件中，定义Base的派生类前添加：  
```python
def to_dict(self):
    out = {}
    for c in self.__table__.columns:
        out[c.name] = getattr(self, c.name, None)  
        # c.name获取的是数据表中的列名，self中可能没有这个列名对应的属性
Base.to_dict = to_dict
```
然后在`client.py`中使用查询结果的时候，需要转换成字典列表的形式的时候可以对result进行操作：
```
result_list = [per.to_dict() for per in result]
```
当然也可以对`to_dict`函数添加参数用于过滤或者其他控制，这个就不说了。

>注意：这个函数只在映射类的属性名与表中的列名一致的情况下使用，如果不一致是无法获取数据的。

一个键如果是`Integer`,可以设置字段自增，类似于1 2 3 4，同时可以设置主键，在使用自增id的时候可能会用到
给Persion增加属性
```python
id = Column(Integer, autoincrement=True, primary_key=True)
```
其他Column的参数：  
`nullable`=`False` or `True`  是否非空，默认是`True`。在sql 中转换成`NOT NULL` or  `NULL`  
`default` 设置默认值，`sqlalchemy`没有将默认值约束写入到数据库中，因为默认值可以是表达式，需要在python中计算  
