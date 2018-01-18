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
