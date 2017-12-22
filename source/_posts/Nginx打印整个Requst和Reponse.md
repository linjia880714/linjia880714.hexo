---
title: Nginx打印整个Requst和Reponse
date: 2017-12-22 11:37:47
tags:
categories:
- IT
- Nginx
---
<!-- toc -->

## 应用场景
有时候和第三调接口，对方犯二，老是说是我方问题，所以把所有的request和response打印给对方看。

详细的输入输出在和第三方调试的时候用处极大

## 系统
此教程实在CentOS release 6.5 (Final)下实验的

## lua-nginx-module模块打印Response到日志文件
### 准备依赖
#### Lua
```bash
cd /usr/local/src/
wget http://luajit.org/download/LuaJIT-2.0.3.tar.gz
tar -xvf LuaJIT-2.0.3.tar.gz
cd LuaJIT-2.0.3
make && make install
```

#### ngx_devel_kit
```bash
cd /usr/local/src/
wget https://github.com/simpl/ngx_devel_kit/archive/v0.2.19.tar.gz
tar -xvf v0.2.19.tar.gz
```

#### lua-nginx-module
```bash
cd /usr/local/src/
wget https://github.com/chaoslawful/lua-nginx-module/archive/v0.9.6.tar.gz
tar -xvf v0.9.6.tar.gz
```

#### openssl
```bash
cd /usr/local/src/
wget https://www.openssl.org/source/openssl-1.0.2n.tar.gz
tar -xvf openssl-1.0.2n.tar.gz
```

#### zlib
```bash
cd /usr/local/src/
wget https://www.zlib.net/zlib-1.2.11.tar.gz
tar -xvf zlib-1.2.11.tar.gz 
cd zlib-1.2.11
./configure && make && make install

## 或是使用yum
 yum install zlib zlib-devel
```

#### pcre
```bash
cd /usr/local/src/
wget https://downloads.sourceforge.net/project/pcre/pcre/8.21/pcre-8.21.zip
unzip pcre-8.21.zip
cd pcre-8.21
./configure && make && make install

## 或是使用yum
yum install pcre-devel pcre 
```

### Nginx编译安装
```bash
## 设置Lua环境变量，以免Nginx启动找不相应模块
export LUAJIT_LIB=/usr/local/lib
export LUAJIT_INC=/usr/local/include/luajit-2.0

## Nginx编译安装
cd /usr/local/src/
wget https://nginx.org/download/nginx-1.8.1.tar.gz
tar -xvf nginx-1.8.1.tar.gz
cd nginx-1.8.1
./configure --prefix=/usr/local/nginx_debug --with-http_ssl_module --with-openssl=/usr/local/src/openssl-1.0.2n  --with-ipv6  --with-http_stub_status_module --with-http_ssl_module --with-http_flv_module --with-http_gzip_static_module --with-pcre --with-ld-opt="-Wl,-rpath,/usr/local/lib" --add-module=/usr/local/src/lua-nginx-module-0.9.6 --add-module=/usr/local/src/ngx_devel_kit-0.2.19 
make && make install
```

### 配置nginx.conf
配置有点长，只要搞懂了body_filter_by_lua，header_filter_by_lua就Ok了。我在网上找的都是打印Response Body的，没找到header，所以看文档自己写出来。[官方的文档](https://github.com/openresty/lua-nginx-module)很详细了，只要会一点Lua语法就很容易按照自己的需求输出
```bash
http {
	include       mime.types;
	default_type  application/octet-stream;

	log_format log_req_resp '$remote_addr - $remote_user [$time_local] '
				'"$request" $status $body_bytes_sent '
				'"$http_referer" "$http_user_agent" $request_time '
				'req_body:"$request_body" resp_body:"$resp_body" resp_header:"$resp_header"';

	server {
		lua_need_request_body on;
		set $resp_body "";
		body_filter_by_lua '
			local resp_body = string.sub(ngx.arg[1], 1, 1000)
			ngx.ctx.buffered = (ngx.ctx.buffered or "") .. resp_body
			if ngx.arg[2] then
			     ngx.var.resp_body = ngx.ctx.buffered
			end
		';

		set $resp_header "";
		header_filter_by_lua '
			local res = ""
			local h = ngx.resp.get_headers()
			for k, v in pairs(h) do
				local values = ""
				if type(v) == "table" then
					for index, str in pairs(v) do
						values = values .. str;
					end
				end 
				if  type(v) ~= "table" then
					values = v
				end 

			res = res .. k .. "=" .. values .. ";" 
			end
			ngx.var.resp_header = res
		';

		listen       443 ssl;
		access_log logs/access.log log_req_resp;
		
		location ^~ / {
			proxy_set_header Host $host;
			proxy_set_header X-Forwarded-For $remote_addr;
			proxy_set_header X-real-ip $remote_addr;
			proxy_pass http://127.0.0.1:9480;
		}
	}

}
```
代码一步步砍下来
1. 12行定义了变量 $resp_body， 
2. 13 ~ 19 通过Lua把Reponse Body赋值给 $resp_body
    body_filter_by_lua 会拦截Response ，这个Directive具体说明可以看[body_filter_by_lua](https://github.com/openresty/lua-nginx-module#body_filter_by_lua), 包裹在单引号的是Lua代码，以及模块给我们提供的一些[nginx-api-for-lua](https://github.com/openresty/lua-nginx-module#nginx-api-for-lua)
3. 21行定义了变量 $resp_header
4. 22~39 通过Lua把Reponse Header赋值给 $resp_body
	header_filter_by_lua 会拦截Response ，这个Directive具体说明可以看[header_filter_by_lua ](https://github.com/openresty/lua-nginx-module#header_filter_by_lua), 包裹在单引号的是Lua代码，以及模块给我们提供的一些[nginx-api-for-lua](https://github.com/openresty/lua-nginx-module#nginx-api-for-lua)

