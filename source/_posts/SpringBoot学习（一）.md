---
title: SpringBoot学习 ( 一 )
date: 2017-08-02 14:45:14
tags: springboot
categories:
- IT
- java
- spring
---

<!-- toc -->

[Spring Boot 文档](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)


`版本：`1.5.6.RELEASE

# 1 创建项目
[https://start.spring.io/](https://start.spring.io/) 使用线上工具可以很容易创建spring-boot项目（我创建的是maven格式的项目）
Group :  com.example
Artifact : springBootTest

----

# 2 创建Http Server
## 2.1 修改pom.xml，加入依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

## 2.2 创建Controller
创建文件 com/example/springBootTest/controller/IndexController.java
```java
package com.example.springBootTest.controller;

import com.example.springBootTest.Params;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class IndexController {

    @RequestMapping("/")
    String home() {
        return "Hello World";
    }
}

```

## 2.3 运行项目
### 2.3.1 在IDE中(Eclipse,IntelliJ)
在SpringBootTestApplication右键 -> Run, localhost:8080访问
![](SpringBoot学习（一）/01.png)

### 2.3.2 编译成jar运行
```bash
# cd到工程路径
$ ls
mvnw			pom.xml			src
mvnw.cmd		springBootTest.iml	target

# 编译打包
$ mvn clean package -Dmaven.test.skip

# 运行
$ java -jar target/springBootTest-0.0.1-SNAPSHOT.jar

# 运行(debug)
$ java -Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=8000,suspend=n -jar target/springBootTest-0.0.1-SNAPSHOT.jar

```
编译打包后会生成2个jar 
springBootTest-0.0.1-SNAPSHOT.jar 集成所有依赖的jar
springBootTest-0.0.1-SNAPSHOT.jar.original 只要用户自己写的class
可以使用下面命令查看两个包的差异
```bash
#查看jar的目录结构
$ jar tvf springBootTest-0.0.1-SNAPSHOT.jar 

#解压jar包
$ jar xvf springBootTest-0.0.1-SNAPSHOT.jar 
```

### 2.3.3 使用maven运行
```bash
$ mvn spring-boot:run

# 可能需要设置maven的运行参数
$ export MAVEN_OPTS=-Xmx1024m -XX:MaxPermSize=128M
```

### 2.3.4 打成war包
1. 修改SpringBootApplication
```java

// 注意下和原本的区别
// 1. extends SpringBootServletInitializer
// 2. 多了configure方法
@SpringBootApplication
public class SpringBootTestApplication extends SpringBootServletInitializer {

	protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
		return application.sources(SpringBootTestApplication.class);
	}

	public static void main(String[] args) {
		SpringApplication.run(SpringBootTestApplication.class, args);
	}
}
```

2. 修改pom.xml
```xml
<!-- jar改成war -->
<packaging>war</packaging>

<!-- 增加依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-tomcat</artifactId>
    <scope>provided</scope>
</dependency>

```

3. 编译
```bash
$ mvn clean package -Dmaven.test.skip

# 在target目录下可以看到springBootTest-0.0.1-SNAPSHOT.war文件
```

----
# 3 修改启动的端口
```java
import org.springframework.boot.context.embedded.*;
import org.springframework.stereotype.Component;

@Component
public class CustomizationBean implements EmbeddedServletContainerCustomizer {
    
    public void customize(ConfigurableEmbeddedServletContainer container) {
        container.setPort(9000);
    }

}
```