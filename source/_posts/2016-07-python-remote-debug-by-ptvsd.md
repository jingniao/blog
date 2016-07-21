---
title: 在vscode使用ptvsd进行远程python调试
mathjax: false
date: 2016-07-21 21:58:58
tags: [python,vscode,remote-debug]
categories: python
---
# 远程调试python代码
## 普通方法
随着python的学习的进步，经常需要进行代码调试，在使用print打印出变量这种方法不是很灵花的情况下，使用pdb进行断点调试已经是比较熟练了：
```python
import pdb
pdb.settrace()
```
<!--more-->
这种方法的好处是能够能够根据情况进行断点执行命令，虽然对有些命令的支持不是很好，但是单步执行，函数跳入之类的都是没有问题的。在使用django开发的时候经常使用这种方法来进行调试。
## 使用ide的远程调试
pydev跟pycharm都提供了远程调试，但是说实话，做python的时间也算不太短了，pycharm一直使用的是社区版的，没有远程调试，pydev的远程调试没有使用成功。倒是今天接触的ptvsd使用成功了。
## ptvsd
这个原来是在vs上的python工具，远程调试也支持，现在vscode也开源不短时间了，vscode的python插件支持ptvsd进行远程调试。  
### 设置vscode调试
在python代码里，点击debug的设置按钮会生成当前工作区的一个debug设置配置文件，里面有几个debug配置，如果要使用远程debug，其他几个可以不要的，将下面的配置文件放到原来配置文件同级别的地方：
!['vscode设置debug'](/image/vscode-debug.png)
```json
        {
            "name": "remote",
            "type": "python",
            "request": "attach",
            "localRoot": "${workspaceRoot}",
            "remoteRoot": "/root/temp",
            "port": 3000,
            "secret": "my_secret",
            "host":"192.168.136.129"
        }
```
remoteRoot是远程代码所在的位置
host 是远程地址

然后在代码开头添加几行代码：
```python
import ptvsd
ptvsd.enable_attach("my_secret", address=('0.0.0.0', 3000))
```
保持本地代码于远程代码一致，然后ssh到远程主机，执行python脚本。
我在vscode的python组件的github库里只看到这两句，实际上只有它的时候，会造成，执行debug的时候，没有停止，vscode连接不到远程，而是一路执行到程序结尾。
[JamesPan的博客-python远程调试](https://blog.jamespan.me/2016/06/30/remote-debug-python-with-vscode/)  就是这样，但是这位老兄的代码是在一个循环里执行一些代码，然后每次循环sleep 1秒，相比开始的时候也遇到了这个问题。
后来我加了几行代码，就可以正常调试了：
```python
ptvsd.wait_for_attach() 
time.sleep(1) 
```
wait_for_attach 这行代码的作用是，将远程进程卡住，等到vscode开始连接远程python代码的时候才会继续执行
import time
wait_for_attach后，vscode执行remote debug后，能收到远程代码执行的结果，输出也重定向到了本地，但是在vscode添加的断点就没有起作用。在sleep之后，发现正常停在了断点处。不清楚是ptvsd和vscode本身的缺陷还是其他的什么问题。
## 出现的问题
大家看上面的操作大概就看出来了，远程调试每次开始调试的时候都需要到远程执行python脚本，而执行脚本完成之后，再点击vscode的debug按钮就连接不到远程debug（运行python的节点）了。这样看并没有比pdb方式方便太多的感觉。
## 适用范围
适用于纯后台python程序的调试，比pdb方便的地方在虽然每次都需要执行脚本，但是可以比较方便的设置断点来执行。  
还有另外一种情况就是查看远程python的后端程序输出（也可以重定向啊……）
至于上面仁兄那位的文章里介绍的保持远程于本地一致的pythong环境，然后可以调试第三方引用库，还没搞出来。这样看来，这个ptvsd对于vscode还是比较简陋的了，希望以后工作中，什么场景中会使用到这种远程调试。继续努力吧。
## 我理想中的python远程调试
跟sftp，ssh进行结合，然后，保持本地代码于远程一致，然后，其他的就像在本地debug一样方便，那就逆天了啊。