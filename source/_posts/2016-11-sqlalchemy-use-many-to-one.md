---
title: sqlalchemy的使用（二），表间外键约束
mathjax: false
date: 2016-11-27 08:55:07
tags: [python,sqlalchemy,sql]
categories: python
---
# 前言
前篇文章简单介绍了使用sqlalchemy的时候怎么建立数据库，创建数据映射表，初始化，以及简单的插入，删除，更新，查询操作。当然，查询包含的东西还有很多，其中的细节还有很多，这个在使用过程中会根据你的需求慢慢侧重不同的方向。  
数据库使用中一个无法绕过的功能就是外键的使用，虽然很多人认为web应用中应该消除外键，减少依赖。在高并发的程序中，web应用确实可能为了性能增加冗余信息的方式消除外键，但是在普通的程序，数据库操作没有成为瓶颈的时候，外键能比较好的维护数据实体之间的关系，较少冗余。

<!-- more -->  

# 外键使用
## 一对多关系
接上篇的`Persion`，现在添加另外一个表，`Book`，一个`Persion`有多个`Book`，`Book`有一个外键auther_id，它指向Persion的id列
```python
class Book(Base):
    __tablename = 'book'
    id = Column(Integer, autoincrement=True, primary_key=True)
    name=Column(String(1024))
    auther_id = Column(Integer,ForeignKey('persion.id'))
```
然后在`Persion`表中添加反向映射关系，让`Persion`能查询关联到它的`Book`，也让`Book`能查询到它归属的`Persion  `
在`Persion`中添加一个类属性：
```python
books = relationship('Book',backref='auther',lazy="dynamic")
```


books并不是可以实际输入的列，在创建Persion的时候不需要设置，只要在Book表中的有外键指向Persion表，则就可以通过books查询Book对象。   
relationship，第一个参数是Book的类名，backref参数是给Book添加一个属性，在这个例子中Book.auther指向的是Persion，可以通过它来查询。   
lazy则是加载方式，默认是select，是在查询books归属的Persion的时候就自动查询并加载books，也就是一次查询所有数据，这个例子中，设为dynamic是生成一个Query对象，但是还没有进行数据查询，可以使用类似与：per_obj.books.query(xxx).all()类似的语句对books进行二次过滤查询。
>值得一提的是，上面例子中backref向Book注入的auther的lazy属性默认是select，也就是在加载Book的结果的时候，也会顺便查询它归属的Persion

### 注意点说明
*  一对多关系，外键在多的一方，relationship在一的一方。
*  lazy属性的设置需要根据需求来设置，如果每次查询Persion都需要获取Book，那默认值select是值得使用。
*  如果两个表是互相有外键字段指向对方，则relationship的lazy属性不能是默认值，需要是dynamic之类的动态加载，否则会造成查询的时候，循环引用的问题。也就是说外键关系是一个环的时候select是不合适的。
* 如果一个表有多个外键，指向不同的表的时候relationship除了第一个参数，映射类的名字外，还需要指定关联关系是怎么样的primaryjoin='Persion.id==Book.auther_id'，类似的这样的关系。
## 多对一关系
这种是外键关系跟relationship在同一个类定义中。  
```python
class Parent(Base):
    __tablename__ = 'parent'
    id = Column(Integer, primary_key=True)
    child_id = Column(Integer, ForeignKey('child.id'))
    child = relationship("Child", backref="parents")
class Child(Base):
    __tablename__ = 'child'
    id = Column(Integer, primary_key=True)
```
官方给的例子是个。
## 一对一
```python
from sqlalchemy.orm import backref
class Parent(Base):
    __tablename__ = 'parent'
    id = Column(Integer, primary_key=True)
    child_id = Column(Integer, ForeignKey('child.id'))
    child = relationship("Child", backref=backref(
        "parent", uselist=False))
class Child(Base):
    __tablename__ = 'child'
    id = Column(Integer, primary_key=True)
```
看上去是特殊的多对一，backref=backref("parent", uselist=False)是关键，将关系映射中，一的注入的属性原来是个列表的时候，uselist=False否认是一个列表。
## 多对多
```python
association_table = Table('association', Base.metadata,
    Column('left_id', Integer, ForeignKey('left.id')),
    Column('right_id', Integer, ForeignKey('right.id'))
)

class Parent(Base):
    __tablename__ = 'left'
    id = Column(Integer, primary_key=True)
    children = relationship(
        "Child",
        secondary=association_table,
        back_populates="parents")

class Child(Base):
    __tablename__ = 'right'
    id = Column(Integer, primary_key=True)
    parents = relationship(
        "Parent",
        secondary=association_table,
        back_populates="children")
```
关键在于：relationship的back_populates，借助中间表来定义多对多。

>说明：工作中使用的最多的是一对多关系，多对多关系的我试验过，但是没用到，另外两种是官方给的例子。
>工作中使用两个一对多，互相只想多方，如下面的例子：   

## 循环一对多关系
还是我自己写的Persion和Book关系，`一个人可能写过多本书，一本Book只有一个Persion写，N个人最喜欢1个书，每个人只能有一个最喜欢的`这个例子可能不大恰当，但是就是两个单向的一对多关系，是不能用多对多关系的，下面是我给出的例子

```python
class Persion(Base):
    __tablename__ = 'persion'
    id = Column(Integer, autoincrement=True, primary_key=True)
    name = Column(String(1024))
    like_id = Column(Integer, ForeignKey('book.id'))
    books = relationship('Book', backref='auther', lazy="dynamic",
                         primaryjoin='Book.auther_id==Persion.id')

class Book(Base):
    __tablename__ = 'book'
    id = Column(Integer, autoincrement=True, primary_key=True)
    likes = relationship('Persion', backref='like', lazy="dynamic",
                         primaryjoin='Persion.like_id==Book.id')
    name = Column(String(1024))
    auther_id = Column(Integer, ForeignKey('persion.id'))

```
主要是添加primaryjoin属性，说明关联的字段

在使用sqlalchemy的时候有很多属性，类似lazy，backref，primaryjoin这样的属性，备选项很多，需要多多查询官方文档。只有使用过过才会比较熟悉