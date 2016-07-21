---
title: matplotlib简单画图并输出中文
mathjax: false
date: 2016-04-30 16:29:42
tags: [python,matplotlib,中文]
categories: python
---
## 前言 
因为个人非常想要学习机器学习，但是python关于机器学习到一些相关库都不熟悉，那就需要了解python相关到库，numpy是基础，matplotlib是显示相关的库，也得知道，所以开始学习matplotlib画图了。 
本篇文章主要是简单的使用matplotlib画出一副简单的图片，然后添加一个中文标题  
## 问题原因 
如果要在画的图中需要用中文进行标记的话，默认中文是不能正常显示的，目前中文毕竟不算强势语言，在编程界更弱势，因为中文而引起的编程上的问题一直在困扰这我们，编码，字体等问题你可以解决它，但无法无视它。 
<!-- more --> 
## 问题描述
### 环境
ubuntu 16.04 64bit
anaconda 4.0 python 3.5.2
matplotlib 1.5.1
### 最终代码 
以下程序代码（仅片段） 
```python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib as mpl
zhfont = mpl.font_manager.FontProperties(
    fname='/usr/share/fonts/opentype/noto/NotoSansCJK.ttc')
x2 = np.linspace(0.0, 2.0)
y2 = np.cos(2 * np.pi * x2)
plt.plot(x2, y2)
plt.title(u'一个简单的函数 f = sin(2*pi*x)', fontproperties=zhfont)
plt.show()
```
效果： 
!['matplotlib画图使用中文标签'](/image/matplotlib-zh-title-setfonts.png)
### 程序说明：  
在查找这个中文显示的时候，出现三种方法：
1  直接修改matplotlibrc文件里的配置  
2  在程序中设置matplotlib的rcParams属性里font.sans-serif成一个已知的中文字体名  
3  第三种就是上面的方法，直接指定渲染用的字体路径  
### 遇到的问题
从表面来看第二种应该是最好的办法，因为不需要修改配置文件，第三种还得查看字体路径，好处是可以制定项目自带的字体而不是系统中已有的字体。第一种跟第二种是一样的效果，不过是修改配置文件，属于环境配置的范畴了。  
但是前两种都有一个前置条件：系统中已经有的中文字体，并且matplotlib可以自动识别的字体。而经过实验，matplotlib对ttc格式的字体文件用上面的代码可以正常载入，但是使用第二第一种方法的时候，是不会自动载入ttc格式的字体的，只默认支持ttf格式的字体，这就造成了ubuntu 16.04中虽然有默认的中文字体（自带谷歌版本的思源黑体），但是因为都是ttc格式的，所以使用前两种方法制定字体名的时候无法载入  
### 吸取的教训
* 不同操作系统环境下可能会造成不同的效果
* 相同的代码随着时间的推移可能因为软件版本的问题出现新的问题  
* 网上的教程里的方法，不能钻牛角尖
