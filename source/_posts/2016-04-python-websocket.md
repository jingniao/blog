---
title: python使用websocket的几种方式
mathjax: false
date: 2016-04-10 09:06:30
tags: [websocket,django,flask,tornado,gevent,uwsgi]
categories: python
---
## 前言
websocket 是一种html5新的接口，以前服务器推送需要进行ajax等方式进行轮训，对服务器压力较高，随着新标准的推进，使用websocket在推送等方面已经是比较成熟了，并且各个浏览器对websocket的支持情况已经比较好了，只要不是太老古古董，对这些暂时不考虑。  
使用websocket的时候，前端使用是比较规范的，js支持ws协议，感觉上类似于一个轻度封装的socket协议，只是以前需要自己维护socket的连接，现在能够以比较标准的方法来进行。  
总的来说因为前端是js，所以后端对websocket支持最好的是socket.io，在搜索websocket相关的内容的时候感觉socket.io对这个的推广也是不少的，但是现在使用的是python，因为新学习python
事件不长，各个框架都在接触一点还是有好处了。  

## 常用框架：
*  uwsgi
*  flask
*  tornado
*  django


<!-- more --> 
下面几个都是一个回音壁程序，也就是接受前端js发过来的websocket信息，然后将websocket再原路返回
前端js：
```js
var s = new WebSocket("%s://%s/foobar/");
s.onopen = function() {}
s.onmessage = function(e) {}
s.onerror = function(e) {}
s.onclose = function(e) {}
s.send(value);
```
这几条就是常用的js使用websocket的代码，处理逻辑没有写，要看完整的看下面uwsgi的官方给的例子，我基本上是照搬的。连接回掉，获取信息回掉，错误回掉，关闭回掉，以及发送信息

### uwsgi
官方文档已经很好了，第一个成功执行的websocket程序就是uwsgi，然后才慢慢的前端不变，然后后端找其他的方案，官方给的例子也是简单易懂的，例子在[websockets_chat_async.py](https://github.com/unbit/uwsgi/blob/master/tests/websockets_chat_async.py)，从这个例子来看，只用uwsgi，需要维护太多的内容，html与python混在一起实在不太好看，所幸这个例子足够简单。  
###  flask-uwsgi-websocket
[flask-uwsgi-websocket](https://github.com/zeekay/flask-uwsgi-websocket)
```python
from flask import Flask, request, render_template
from flask.ext.uwsgi_websocket import GeventWebSocket
app = Flask(__name__)
ws = GeventWebSocket(app)
@app.route('/')
def index():
    return render_template('index.html')
@ws.route('/foobar')
def echo(wscon):
    msg = wscon.receive()
    if msg is not None:
        wssss.send(msg)
if __name__ == '__main__':
    app.run(gevent=100)
```
这里使用flask自带python容器进行执行python web
从上面代码可以看到，使用很简单，其余部分跟普通的flask都没有区别，只需要在需要更改websocket的url请求修饰符，修饰符的来源是：
```python
ws = GeventWebSocket(app)
```
很简单也很强大，前端库因为逻辑不需要更改，所以感觉挺好的，但是这个库[Flask-uWSGI-WebSocket](https://github.com/zeekay/flask-uwsgi-websocket)好像有个bug,在用这个库的时候，前端js持续接收到空的消息然后触发了onmessage回掉函数，在使用同样的前端js，其他后端库的时候没有这个问题。  

### geventwebsocket 
[geventwebsocket](https://bitbucket.org/noppo/gevent-websocket)
```python
from flask import Flask, request, render_template, abort
from geventwebsocket.handler import WebSocketHandler
from gevent.pywsgi import WSGIServer

app = Flask(__name__)

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/foobar')
def echo():
    if request.environ.get('wsgi.websocket'):
        ws = request.environ['wsgi.websocket']
        if ws is None:
            abort(404)
        else:
            while True:
                if not  ws.closed:
                    message = ws.receive()
                    ws.send(message)


if __name__ == '__main__':
    http_server = WSGIServer(('',5000), app, handler_class=WebSocketHandler)
    http_server.serve_forever()
```
这段代码同样使用了flask来进行模板，url之类的解析，不同之处是不再使用flask自带的容器，而是当作一个应用，被gevent里的一个uwsgiserver容器来调用。  
而与普通使用方法不同的是注入了handler_class这个类，替换成websocket类型的，具体实现还没有看，但是从逻辑上可以理解，原来的wsgiserver不理解websocket，所以换一个理解websocket的类来进行处理，  
所以在foobar的程序中才可以从request的环境变量里获取websocket连接，从这里来看，websockethandler也对websocket连接进行了维护工作，简化了很多工作。  

需要注意的是，这个库当前最新版本是0.9.5，网上搜到了一个教程，但是它的版本针对的是0.3.5版本的，这个库的维护者还进行了变更，其中有些api好像有了变化，需要注意。

###  tornado
[tornado_websocket文档](http://www.tornadoweb.org/en/stable/websocket.html)
文档已经很全面了，就不贴代码了
优点，回掉方式，在异步化之后，并发处理能力应该不错，因为是原生支持websocket而不像flask需要寻找第三方插件，所以可能更值得信赖
官方给的一个聊天室的例子就很好[tornado_chatroom](https://github.com/tornadoweb/tornado/blob/master/demos/websocket/chatdemo.py)，坑比前面两个flask的少。值得推荐  
### dwebsocket
[dwebsocket](https://github.com/duanhongyi/dwebsocket)  
官方的没什么坑，django的一个插件，处理websocket，在django这个同步阻塞的库里给了websocket的方法。确实值得推荐，暂时不知道是怎么绕过django的同步阻塞的，有时间了看看它的代码，反正好像代码量也不多的样子。  
```python
from django.shortcuts import render
from dwebsocket import require_websocket

def index(request):
    return render(request, 'index.html')

@require_websocket
def foobar(request):
    while True:
        message = request.websocket.wait()
        request.websocket.send("return: "+ message)
```
跟flask的第一个库一样，都是只需要添加一个修饰符就可以了。  
## 总结
折腾了一天多的时间。个人比较感兴趣的几个python的websocket的使用，算是搞通了，至少自己不管是照搬还是适配，总算是能用了。  
uwsgi感觉比较原始的控制html，控制返回，独自使用好像不太方便，实际上第一个flask的例子后端应该就是使用uwsgi的，从它的名字就可以看到。
Flask-uWSGI-WebSocket和dwebsocket的方式类似，只是一个用于flask，一个用于django，但是前者有个不大不小的bug，给人感觉不太成熟的样子，后者倒是感觉不错，对于django来说，挺不错的样子，不过django的1.9版本出来了一个通道功能，用于执行类似于websocket的后台长时间的功能，找时间了解下，不过这个功能好像还是插件式提供的语句在，好像在下一个django会合到主分支版本的样子，文档在：  
[Django Channels](https://channels.readthedocs.org/en/latest/)  
tornado是一个新兴的异步框架，了解的不多，感觉上跟flask和django不太一样。但是可以作为利器使用，新手就不多说了。
