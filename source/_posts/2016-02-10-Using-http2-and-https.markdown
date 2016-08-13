---
layout:     post
title:      "http2 + https 装逼指南＆Let's Encrypt for nginx使用方法"
subtitle:   "http2，Let's Encrypt"
date:       2016-02-09
author:     "J3n5en"
tags:
    - http2
    - Let's Encrypt
---
[ Let's Encrypt](https://letsencrypt.org/)在2015/12/3开放了公测，先来介绍一下Let's Encrypt,它是Mozilla、Facebook、Cisco等很多巨头为了推动Web加密而新建的一个项目。more : [https://letsencrypt.org/](https://letsencrypt.org/ ) 来看看我的成果图
![image](/img/post-img/80598e16-ce80-11e5-8aa6-a36c6c261730.png)
Ｃｏｏｏｏｏｏｌ～
---
先来签证吧。

由于nginx 默认还不支持nginx自动配置。ｓｏ。手动呗。。
```
git clone https://github.com/letsencrypt/letsencrypt
cd letsencrypt
＃letsencrypt-auto certonly --webroot -w /var/www/example/ -d www.j3n5en.com -d j3n5en.com -d blog.j3n5en.com
```
-w 后面跟的是到时letsencrypt验证的ｗｅｂ临时目录。

再打开一个终端，进入/var/www/example  运行python的SimpleHTTPServer

`python -m SimpleHTTPServer 80 #监听80端口`
然后再运行
```
letsencrypt-auto certonly --webroot -w /var/www/example/ -d www.j3n5en.com -d j3n5en.com -d blog.j3n5en.com
```
OK~不出错的话  证书和私钥都在`/etc/letsencrypt/live/$domain`里面.

接着我们配置nginx (确保nginx为最新版我现在是nginx/1.9.7)
```
server {
        listen 80;
        location / {
        return 301 https://$host$request_uri;
        }
}
# 301 重定向  ->  强制 https
server {
     listen 443 ssl http2;
     #这里开启了http2

     server_name j3n5en.com;
     ssl on;
     ssl_certificate /etc/letsencrypt/live/www.j3n5en.com/fullchain.pem;
     ssl_certificate_key /etc/letsencrypt/live/www.j3n5en.com/privkey.pem;
     #这里指向刚刚生成的证书和私钥
     #下面省略其他配置.
```
ok ~ 

http2 + https  装逼指南到此结束