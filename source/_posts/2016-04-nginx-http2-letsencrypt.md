title: let's Encrypt的使用以及nginx的http2的配置
mathjax: false
date: 2016-04-08 21:57:23
tags: [nginx,http2,letsencrypt]
categories: 运维
---
本文大量参考了Jerry Qu的博客
[Let's Encrypt，免费好用的 HTTPS 证书](https://imququ.com/post/letsencrypt-certificate.html)

##  创建工作目录  
```bash
mkdir ssl
cd ssl/
```
## 创建本地密钥  
这里集中不同的密钥，注意区分
用户密钥
```bash
openssl genrsa 4096 > account.key
```
域名密钥
```bash
openssl genrsa 4096 > git.letus.club.key
```
生成csr证书请求文件
交互模式生成
```bash
openssl req -new -sha256 -key git.letus.club.key -out git.letus.club.csr
```
按照提示信息填写相关信息：默认不进行填写也可以
生成的csr文件的开头是
```
-----BEGIN CERTIFICATE REQUEST-----
```
## 域名配置验证服务 
在进行证书签名的时候证书机构需要知道你拥有这个域名的所有权，类似于谷歌站长的验证方式  
### 配置nginx的验证目录  
```ini
server {
    listen       80;
    server_name  git.letus.club;

    location / {
        rewrite ^/(.*)$ https://git.letus.club/$1 permanent;
    }
    location ^~ /.well-know/acme-challenge/ {
        alias /var/challenges/;
        try_files $uri =404;
    }
}
```
### 下载脚本，获取验证过的网站证书  
```bash
wget https://raw.githubusercontent.com/diafygi/acme-tiny/master/acme_tiny.py
```
验证并获取证书文件
```bash
python acme_tiny.py --account-key ./account.key --csr ./git.letus.club.csr --acme-dir /root/challenges/ > ./signed.crt
```
执行脚本的时候python库可能缺少，需要安装
```bash
yum install yum install python-argparse
```
上面的acme_tiny.py脚本输出
```bash
Parsing account key...
Parsing CSR...
Registering account...
Already registered!
Verifying git.letus.club...
git.letus.club verified!
Signing certificate...
Certificate signed!
```
这个过程需要验证服务是否可用  
所以保证通过指定域名能够访问文件  
nginx里的alias指向的地址跟指定的acme-dir地址应该一致，并且保证有相关权限
生成一个crt文件，文件开头是： 
```bash
-----BEGIN CERTIFICATE-----
```
获取上级证书并组成证书链
```bash
wget -O - https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem > intermediate.pem
cat signed.crt intermediate.pem > chained.pem
```
到这里就配置成功了证书文件了
## nginx的http2配置   
一个完成的配置文件如下
```ini
server {
    listen       443 ssl http2;
    server_name git.letus.club
    ssl on;

    ssl_certificate /root/ssl/chained.pem;
    ssl_certificate_key /root/ssl/git.letus.club.key;
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
    
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```
然后重启nginx
访问git.letus.club网址查看证书：
!["网站证书"](/image/letsencrypt_cert.png)

