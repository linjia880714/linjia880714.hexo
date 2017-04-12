---
title: Nginx开启ssi
date: 2017-04-12 15:33:05
tags: Nginx
categories:
- IT
- Nginx
---

| 参数 | 默认值 | 解释 |
| --- | --- | --- |
| ssi | off | 开关：on开，off关 |
| ssi_silent_errors | off | 开启后在处理SSI文件出错时不输出错误提示:<br>”[an error occurred while processing the directive] |
| ssi_types | text/html | 启用ssi的格式 |

ssi，ssi_silent_errors和ssi_types，可以放在http,server和location的作用域下

详情请看 [ngx_http_ssi_module](http://nginx.org/en/docs/http/ngx_http_ssi_module.html)

```
server {
        listen       83;
        location / {
                 ssi on;
                 ssi_silent_errors on;
                 root   "D:\workspace\project\Link\xproject-html";
         expires 0;
        }
 }
```