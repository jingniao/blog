---
title: 小python程序——hexo用户统计
mathjax: false
date: 2016-07-24 21:42:17
tags: [python,hexo,openshift]
categories: [python]
---
# 前言
hexo挂着也有一段时间了，刚刚折腾的时候，也是有不少问题，现在再去整这些东西就轻车熟路了吧，之前还开过disqus评论，但是感觉没什么用，所以就关掉了，现在这个主题，作者已经不怎么更新了，所以有问题或者想要加什么东西就需要自己琢磨了。
# 最近的折腾
## html5的进击
周六的时候，虽然去漫展转了三个钟头，也是累的不轻，但是回来之后，还是继续周五晚上的活。当前这个主题在firefox的安卓下的js弹出有问题，点击左上角出来个人介绍之类的东西，再点击右边空白处，这个抽屉效果就会进去，但是在firefox的安卓浏览器上，弹不回去了。
<!--more-->
最后找到问题，是因为代码里监听的触摸事件是`webkitTransitionEnd`，这是chrome系的事件，在firefox中是不起作用的。在正式的html5标准里，firefox和chrome都支持了`transitionend`事件。换成这个，然后再将`-webkit-transform`和`-webkit-transition`相应的位置添加对应的`transform`和`transition`。虽然替换了之后firefox的安卓版效果跟chrome系的还是有点差别，主要是弹出来的时候过度动画没有效果。但是好歹是能出来进去了。  
从这里可以开出标准化的过程中也是有不少遗留的坑的。
## 用户访问统计
虽然没什么人在访问，评论也关闭了，但是还是想知道我的这个小的博客到底有没有人在访问，以及基于某种小心思，所以今天一天都耗在了这个上面。  
技术选型：
* flask
* openshift
* mysql

具体的思路：
1. 客户端使用jsonp向统计方发送一个请求。  
2. 统计方获取当前页面的view（6小时内的请求，同一个ip算同一次view）。如果没有这个页面的view，则设为1，记录该ip访问该页面时间。
3. 判断6小时内是否这个ip已经访问过这个页面
    1) 如果没有访问过，记录该ip访问这个页面的时间
    2) 如果已经访问过，超过6小时，更新访问时间，view + 1 返回，6小时以内，访问过，则直接返回上一步获取的view  

应该说是一个很简单的流程，这一天一直解决各种问题。遇到的问题列表
* openshift平台获取真实ip，request的`remote_addr`获取到的是内网ip。需要获取`header`里的`x-forwarded-for`
* mysql操作参数，python安装mysql连接包`mysql-python`有点小问题。但是也耗费了不少时间
* jsonp的使用，flask返回什么样的值，对于我这种只返回过json的值，jsonp应该返回什么值就有点抓瞎了。
### 项目地址
虽然只是一个很简单的小程序，但是还是提交到git了。
(http://git.oschina.net/zaita/statistics)[http://git.oschina.net/zaita/statistics]
# 感想
应该说还是写得程序太少，今天遇到的问题在没有动手实际操作过之前，是不知道的，只有多写些程序才能够发现问题。