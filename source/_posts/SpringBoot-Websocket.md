---
title: SpringBoot Websocket
date: 2018-01-18 19:24:58
tags: springboot
categories:
- IT
- java
- spring
---

<!-- toc -->

# 1 创建项目
[https://start.spring.io/](https://start.spring.io/) 使用线上工具可以很容易创建spring-boot项目（我创建的是maven格式的项目）
Group :  org.link
Artifact : demo-spring-boot-websocket

spring boot版本：1.5.9.RELEASE

[也可以查看官方文档4.3.10.RELEASE/spring-framework-reference/html/websocket.html](https://docs.spring.io/spring/docs/4.3.10.RELEASE/spring-framework-reference/html/websocket.html)

# 2 添加Websocket支持
修改pom.xml
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

# 3 配置Websocket
```java
package org.link.demospringbootwebsocket.websocket;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.socket.config.annotation.EnableWebSocket;
import org.springframework.web.socket.config.annotation.WebSocketConfigurer;
import org.springframework.web.socket.config.annotation.WebSocketHandlerRegistry;

@Configuration
@EnableWebSocket
public class WebsocketConfig implements WebSocketConfigurer {
	
	@Autowired
	private MyHandler myHandler;
	
	@Override
	public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {	    	
		registry.addHandler(myHandler, "/socket").setAllowedOrigins("*").withSockJS();
	}
}
```
1. MyHandler: 处理消息的类，下面会说明
2. "/socket" ： websocket的地址
3. .setAllowedOrigins("*")： 解决浏览器跨域的问题（CORS）,可以设置为某个具体的域名，例如http://wwww.linjia.site, 只有这个域名下的js才可以跨域访问
4. .withSockJS(): 为什么使用sockjs呢，spring官网多sockjs的描述
    The goal of SockJS is to let applications use a WebSocket API but fall back to non-WebSocket alternatives when necessary at runtime, i.e. without the need to change application code.
    SockJS的目标是让应用程序使用WebSocket API，但在运行时需要时，可以回退到非WebSocket替代方案，不需要更改代码。 就是说浏览器不支持 WebSocket，该库可以模拟对 WebSocket 的支持   
    
# 4 自定义TextWebSocketHandler
Spring Websocket 有两种Handler TextWebSocketHandler 和 BinaryWebSocketHandler，我们选TextWebSocketHandler
```java
package org.link.demospringbootwebsocket.websocket;

import org.springframework.stereotype.Component;
import org.springframework.web.socket.CloseStatus;
import org.springframework.web.socket.TextMessage;
import org.springframework.web.socket.WebSocketSession;
import org.springframework.web.socket.handler.TextWebSocketHandler;

@Component
public class MyHandler extends TextWebSocketHandler{

	@Override
	public void afterConnectionClosed(WebSocketSession session,
			CloseStatus status) throws Exception {
		super.afterConnectionClosed(session, status);
	}

	@Override
	protected void handleTextMessage(WebSocketSession session,
			TextMessage message) throws Exception {
		super.handleTextMessage(session, message);
		
		// 打印接收到的消息
		System.out.println(message.getPayload());
		
		// 返回消息
		TextMessage msg = new TextMessage("From server :"+message.getPayload());
		session.sendMessage(msg);
	}
	
}
```

# 5 修改启动端口
```bash
# 修改application.properties文件
server.port=8181
```

# 6 sockjs客户端

我们使用sockjs作为客户端和服务器通信，建立src\main\resources\static\index.html页面，在classpath路径下，有如下文件夹名称 /static (or /public or /resources or /META-INF/resources)，里面的文件将作为静态文件被访问
{% asset_img 01.jpg %}

```html
<!DOCTYPE html>
<html>
<head>
<title></title>
<script src="/js/sockjs.min.js"></script>
<script src="/js/jquery-3.2.1.min.js"></script>
<script type="text/javascript">
	var sock = new SockJS('http://127.0.0.1:8181/socket');
	sock.onopen = function() {
		console.log('open');
	};

	sock.onmessage = function(e) {
		console.log('message', e.data);
		display(e.data);
	};

	sock.onclose = function() {
		console.log('close');
	};

	sock.onclose = function() {
		console.log('close');
	};

	function display(msg) {
		$("#message").append("<pre>" + msg + "<pre>");
	}

	function sendMsg() {
		sock.send($("#saySomething").val());
	}
</script>
</head>
<body>
	<input type="text" id="saySomething" value="Hello World">Say Something<br>

	<button onclick="sendMsg()">play</button>
	<HR>
	<div id="message"></div>
</body>
</html>
```
1. 请注意new SockJS('http://127.0.0.1:8181/socket')是http，sockjs会自动升级为ws
2. sockjs接口很简单，相关文档可以看[sockjs-client](https://github.com/sockjs/sockjs-client)
3. 点击play按钮服务器接收到“Hello World”，然后返回一个“From server :Hello World”

# 7 使用Https
建议使用Nginx
```bash
server {
        listen       443 ssl;
        server_name  <domain>;
        ssl on;
        ssl_certificate <file>.crt;
        ssl_certificate_key <file>.key;

        location ^~ /socket {
                proxy_pass http://127.0.0.1:8181;
                proxy_set_header X-Scheme $scheme;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";
                proxy_set_header Host $http_host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_pass_header Server;
        }
}
```
1. 修改成new SockJS('https://《domain》/socket') ,sockjs会自动升级为wss