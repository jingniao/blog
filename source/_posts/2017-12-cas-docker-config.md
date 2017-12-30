---
title: cas配置——service配置修改
mathjax: false
date: 2017-12-30 22:11:21
tags: [cas,docker,overlay]
categories:
---

## 原因说明
从`hub.docker.com`上下载的`cas`官方镜像我没找到比较好的更改配置（例如`更改service配置`）的方法，因为默认采用`maven`的`overlay`进行构建的，原谅我这个不太熟悉`java`生态的的弱渣

所以还是从官方的构建脚本来自行构建，官方推送到`hub.docker.com`的镜像好像进行了一定的清理，跟直接使用`Dockerfile`构建的还是有点差别的

所以我fork了官方库，对版本进行了固定，然后为了绕开国内在进行maven构建时抽风似的网速影响，选择了docker的自动构建
<!-- more -->
## 配置
### 启动
```shell
docker run -it -p 8443:8443 jingniao/mycas:release-5.1.7 /bin/bash

#进入镜像中后，未指定目录一致为/cas-overlay 目录

./build.sh gencert 
# 生成证书，原来的证书keystore是空的，不进行生成会启动报错，一个容器只需要运行一次
./build.sh run # 重新进行war包构建以及启动java
```
然后就可以使用`https://cas.example.org:8443/cas/login` 来进行登陆了
默认账户密码：`casuser::Mellon`  
可以在`application.properties`或者`cas.properties`里修改添加或者删除配置：`cas.authn.accept.users`，为了安全，一般寻找配置里的内容将静态用户密码认证禁用  
### 更改cas服务器域名
请配置`hosts`或者`dns`，将`cas.example.org`指向docker主机所在的ip
如果需要更改`cas.example.org`为自定义的域名`xxx.xxx`，可以查看`build.sh`脚本里`gencert`生成的证书命令，然后在`cas.properties`里添加配置 `server.ssl.key-alias=xxx.xxx`


### 启用proxy模式服务

>这就需要修改`serivce配置`，而且如果需要`cas`的`proxy 代理模式`（`默认禁止`使用）需要修改`service`的配置

因为不了解`java`的生态，对于`overlay`之前也没有了解过，到底怎么进行配置修改没有头绪，我看到cas教程中有些是提示将配置文件跟`service`配置文件丢到`/etc/cas/config` `/etc/cas/services`文件夹里

但经过我实验，`/etc/cas/config`的`cas.properties`配置文件是可以其一定作用的，在启用`initFromJson` 配置文件后，放在`/etc/cas/services`的json服务配置文件没有起作用？不太清楚是我哪里配置有问题还是其他原因，这部分浪费了不少时间

所以我选择了下面的方法，
设置`service`配置
```shell
echo "cas.serviceRegistry.initFromJson=true" >> /cas-overlay/etc/cas/config/cas.properties
# 因为下面的build.sh启动脚本会将/cas-overlay/etc/cas/config/cas.properties复制到/etc/cas/config/cas.properties，如果不适用build.sh脚本启动而是直接运行war包，则需要修改/etc/cas/config里的配置
mkdir -p src/main/resources/services/
cat src/main/resources/services/test-1001.json
```
# 设置json内容，已经启用proxy
```json
{
  "@class" : "org.apereo.cas.services.RegexRegisteredService",
  "serviceId" : "^(https|imaps)://.*",
  "name" : "test",
  "id" : 1001,
  "description" : "test",
  "evaluationOrder" : 10,
  "allowedToProxy":true,
  "proxyPolicy" : {
    "@class" : "org.apereo.cas.services.RegexMatchingRegisteredServiceProxyPolicy",
    "pattern" : "^https?://.*"
  }
}
```
注意`evaluationOrder`是优先级，默认是有2个配置的，数值越小，优先级越高，如果定义了相同的`serviceId`，则会根据优先级高低进行选择
```shell
./build.sh run
```
进行overlay重新打包war，然后执行，会将src里目录的内容继承到cas.war文件里，达到修改配置的作用
如果有谁知道cas官方的docker镜像怎么修改service之类的配置，还请邮件通知我，感激不尽

到这里，cas server基本上可以使用了

到这里就可以允许https进行接入了


### 接入proxy模式应用
proxy请求端的https是可以访问的，浏览器可以访问http页面，但是在请求proxy的时候，cas server会对应用的https地址进行回掉。所以这里的应用域名对于cas server应该也是可以访问的。

另外需要注意一点的是应用的https证书对cas server来说应该是“合法”的，可以是真正公网权威证书机构签发的https证书

也可以是上面gencert 时生成的证书进行签发，然后将gencert是也会在/etc/cas/目录生成thekeystore和cas.cer，将cas.cer这个ca证书导入到cas server的jre环境里(跟以前12306需要导入根证书一样的道理)，这样对cas server来说，proxy请求用用的https证书就合法了`


## 更多内容
* 后端数据库，ldap之类的的接入
* 登陆界面的更改
* 推荐阅读 [悟空_](http://blog.csdn.net/u010475041/article/category/7156505) 的博客 以及他的 [github](https://github.com/kawhii/sso)，写了20篇的cas文章，感觉是个很厉害的人
