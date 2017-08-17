---
title: SpringBoot学习（二）
date: 2017-08-17 18:00:51
tags: springboot
categories:
- IT
- java
- spring
---

<!-- toc -->

[Spring Boot 文档](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)

`版本：`1.5.6.RELEASE

# Rest风格的URI
访问方式 Http://host/user/100
```java
import com.example.springBootTest.bean.User;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/user")
public class UserController {

    @RequestMapping("/{userId}")
    public User getUser(@PathVariable long userId)
    {
        User user = new User();
        user.setUserId(userId);
        return user;
    }

    @RequestMapping("/")
    public User getUser2(User user)
    {
        return user;
    }

}
```

# 访问静态资源
在classpath路径下，有如下文件夹名称 /static (or /public or /resources or /META-INF/resources) 
里面的文件将作为静态文件被访问, 建立如下文件
![](SpringBoot学习（二）/01.png)
Http://host/index.html
Http://host/user2/user.html
注意：为什么是user2而不是user，在上面我们建立了个Controller（UserController），如果使用user的话，优先匹配Controller，会报错

# 格式化输出Date
spring默认使用Jackson输出json，Jackson对于Date是输出成时间戳，要改成"yyyy-MM-dd hh:mm:ss"这种格式的字符串
```java
import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.DeserializationContext;
import com.fasterxml.jackson.databind.JsonDeserializer;
import com.fasterxml.jackson.databind.JsonSerializer;
import com.fasterxml.jackson.databind.SerializerProvider;
import com.fasterxml.jackson.databind.annotation.JsonDeserialize;
import org.springframework.boot.jackson.JsonComponent;

import java.io.IOException;
import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * Created by jialin on 17/8/2017.
 */
@JsonComponent
public class JsonConfig {

    private static final SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");

    public static class Serializer extends JsonSerializer<Date> {
        public void serialize(Date value, JsonGenerator gen, SerializerProvider serializers) throws IOException, JsonProcessingException {
            gen.writeString(format.format(value));
        }
    }

    public static class Deserializer extends JsonDeserializer<Date> {
        public Date deserialize(JsonParser p, DeserializationContext ctxt) throws IOException, JsonProcessingException {
            return null;
        }
    }

}

```
