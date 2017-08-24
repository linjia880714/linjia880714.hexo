---
title: SpringBoot学习 ( 三 ) - Log
date: 2017-08-24 16:11:09
tags: springboot
categories:
- IT
- java
- spring
---

<!-- toc -->

----

[Spring Boot 文档](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)


`版本：`1.5.6.RELEASE

----

# 使用Log4j2
1. 修改pom.xml
```xml
<!-- 去掉spring-boot-starter-logging -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<!-- 加入spring-boot-starter-log4j2 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```

2. 在resource下创建文件log4j2.xml，spring会自动去加载在这个位置的这个文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT" follow="true">
            <PatternLayout>
                <pattern>%d %p %C{1.} [%t] %m%n</pattern>
            </PatternLayout>
        </Console>

        <RollingFile name="RollingFile" fileName="logs/eagle.log" filePattern="logs/eagle-%d{MM-dd-yyyy}.log" ignoreExceptions="false">
            <PatternLayout>
                <Pattern>%d %p %c{1.} [%t] %m%n</Pattern>
            </PatternLayout>
            <TimeBasedTriggeringPolicy />
        </RollingFile>
    </Appenders>

    <Loggers>
        <Logger name="org.springframework" level="warn" />
        <Root level="info">
            <AppenderRef ref="Console" />
            <AppenderRef ref="RollingFile" />
        </Root>
    </Loggers>
</Configuration>
```
配置文件中有个RollingFile，如果在Intellij IDEA右键运行的话，log文件是放在项目代码的根目录下（Mac系统，window没试过）


3. 指定配置文件，例如logging.xml（可选）
```bash
# 修改application.properties文件
logging.config = classpath:logging.xml
```