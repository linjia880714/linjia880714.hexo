---
title: Nginx自己签发证书配置https
date: 2017-04-12 15:46:33
tags: Nginx
categories:
- IT
- Nginx
---

<!-- toc -->

### 1.生成一个RSA密钥 
```bash
openssl genrsa -des3 -out sleepace.key 1024
```
执行时需要输入一个密码，请记住

---

### 2.拷贝一个不需要输入密码的密钥文件
```bash
openssl rsa -in sleepace.key -out sleepace_nopass.key
```
执行时需要输入第一步的密码

---

### 3.生成一个证书请求
```bash
openssl req -new -key sleepace.key -out sleepace.csr
```
需要输入公司的一些信息，如下图所示
{% asset_img 03.png %}

---

### 4.自己签发证书
```bash
openssl x509 -req -days 365 -in sleepace.csr -signkey sleepace.key -out sleepace.crt
```
执行时需要输入第一步的密码

### 5.配置Nginx
```
server {
        listen       443 ssl;
        server_name  www.xxx.com;
        
        ssl on;
        ssl_certificate /root/ssl/sleepace.crt;
        ssl_certificate_key /root/ssl/sleepace_nopass.key;
        # 若ssl_certificate_key使用sleepace_nopass.key，则每次启动Nginx服务器都要求输入key的密码。
        location / {
                proxy_pass http://127.0.0.1:9080;
        }
}

```