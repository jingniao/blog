---
title: 部署hexo
---

## 安装环境
### 安装hexo
``` bash
yum update
yum install epel-release
yum install nodejs git npm
npm install -g hexo-cli
```
### 拿到博客的内容
``` bash
git clone git@git.oschina.net:zaita/jingniao.github.io.git
cd jingniao.github.io
git clone git@git.oschina.net:zaita/mytheme.git themes/yilia
npm install
```
### 新建文章

``` bash
hexo new post new_post
```
编辑souces下的post文件夹下的新建的文件，然后可以进行编辑了  
一般需要更改的地方是文章内容里的title标签，默认是文件名，中文的话需要更改，文件名是url地址  
关于md文件的编写，一点一点熟悉吧，这个不着急。 
关于文章属性还是有一些有意思的东西，以后再看吧。  
### 生成静态内容并部署
完成文章编写的话
``` bash
git status
git add
git commit -m ""
git push
#以上是将写的内容同步到osc，源码要先同步
hexo generate
hexo deploy
#注意部署的话，需要先将github的公钥配置好
```