---
title: hessian
date: 2018-07-22 17:56:37
tags:
---

<!-- toc -->

## 环境
JDK: 1.8

hessian-4.0.38.jar

## 分析

### 接口定义
```java
// 接口定义
package com.demo.demospringbase.hessian;

public interface HelloService {
    
    public void sayHi(Persion p);

}

// 对象定义
package com.demo.demospringbase.hessian;

import java.io.Serializable;

public class Persion implements Serializable{

    private String name;

    public Persion(String name) {
        super();
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
    
}

```

### 客户端调用
```java
package com.demo.demospringbase.hessian;

import java.net.MalformedURLException;
import com.caucho.hessian.client.HessianProxyFactory;

public class HessianClient {
    public static void main(String[] args) throws MalformedURLException {
        String url = "http://127.0.0.1:8080/hello";

        HessianProxyFactory factory = new HessianProxyFactory();
        factory.setOverloadEnabled(true);
        HelloService helloService = (HelloService) factory.create(HelloService.class, url);
        helloService.sayHi(new Persion("link"));
    }
}
```

### 源码研读
我们都是知道hessian是基于http传输的，我们现在来研究下`helloService.sayHi(new Persion("link"))`，这个过程到底发送了什么

这里只研究客户端怎么调用发送了什么

```java
// HessianProxyFactory.java

public Object create(Class<?> api, URL url, ClassLoader loader)
{
if (api == null)
  throw new NullPointerException("api must not be null for HessianProxyFactory.create()");
InvocationHandler handler = null;

handler = new HessianProxy(url, this, api);

// 使用jdc动态代理
return Proxy.newProxyInstance(loader,
                              new Class[] { api,
                                            HessianRemoteObject.class },
                              handler);
}
```
`helloService.sayHi(new Persion("link"))`调用的是HessianProxyFactory使用jdk动态代理生成的代理对象,所以我们接下看要看`HessianProxy.invoke`方法



```java
// HessianProxy.java

public Object invoke(Object proxy, Method method, Object []args) throws Throwable
{
    // 省略

    // 发送请求，请看下面的sendRequest方法
    conn = sendRequest(mangleName, args);

    // 省略
    // 处理返回的inputstream
}

protected HessianConnection sendRequest(String methodName, Object []args)
    throws IOException
  {
     
    // 省略

    AbstractHessianOutput out = _factory.getHessianOutput(os);
    // 生成发送给服务端的字节数组，这个使我们接下来重点研究的
    out.call(methodName, args);
    out.flush();
    conn.sendRequest();

    // 省略

  }
```

使用实际例子研究out.call发送放了什么

调用`helloService.sayHi(new Persion("link"))`发送了78个byte,我们一个个字节看
99, 2, 0, 109, 0, 13, 115, 97, 121, 72, 105, 95, 80, 101, 114, 115, 105, 111, 110, 77, 116, 0, 39, 99, 111, 109, 46, 100, 101, 109, 111, 46, 100, 101, 109, 111, 115, 112, 114, 105, 110, 103, 98, 97, 115, 101, 46, 104, 101, 115, 115, 105, 97, 110, 46, 80, 101, 114, 115, 105, 111, 110, 83, 0, 4, 110, 97, 109, 101, 83, 0, 4, 108, 105, 110, 107, 122, 122


#### 调用哪个方法
| 字节 | 值 | 解释 |
|--- |--- |--- | 
|99 | c | call|
|2 |2 | major主版本| 
|0 |0 | minor子版本| 
|109 | m | method | 
|0, 13 | 13 | 表示接下来13个字节就是方法名 | 
|115, 97, 121, 72, 105, <br>95, 80, 101, 114, 115,<br> 105, 111, 110 | sayHi_Persion | 调用方法的名称 | 
```java
// HessianOutput.java

public void startCall(String method, int length) throws IOException
{
    os.write('c');
    os.write(_version);
    os.write(0);

    os.write('m');
    int len = method.length();
    os.write(len >> 8);
    os.write(len);
    printString(method, 0, len);
}

// 使用utf-8编码
public void printString(String v, int offset, int length)throws IOException
{
    for (int i = 0; i < length; i++) {
      char ch = v.charAt(i + offset);

      if (ch < 0x80)
        os.write(ch);
      else if (ch < 0x800) {
        os.write(0xc0 + ((ch >> 6) & 0x1f));
        os.write(0x80 + (ch & 0x3f));
      }
      else {
        os.write(0xe0 + ((ch >> 12) & 0xf));
        os.write(0x80 + ((ch >> 6) & 0x3f));
        os.write(0x80 + (ch & 0x3f));
      }
    }
}
```

#### 参数是什么

Persion这个对象最后用的是`UnsafeSerializer`这个进行序列化的，决定使用哪个序列化器`SerializerFactory`的`getSerializer(Class cl)`方法
```java
// HessianOutput.java

public void writeObject(Object object) throws IOException                                                                              
{                                                                                                 
  if (object == null) {                                                                           
    writeNull();                                                                                  
    return;                                                                                       
  }                                                                                               
                                                                                                  
  Serializer serializer;                                                                          
  
  // 获取相应的序列化器                                                                                             
  serializer = _serializerFactory.getSerializer(object.getClass());                               
                                                                                                  
  serializer.writeObject(object, this);                                                           
}                                                                                                 
```


注意：UnsafeSerializer只是使用sun.misc.Unsafe的API去获取对象的值，最后序列化Sting，int，long，double这些还是使用HessianOutput提供的方法（可以看下面的HessianOutput方法列表）


```java
// UnsafeSerializer.java

public void writeObject(Object obj, AbstractHessianOutput out) throws IOException
    {
    if (out.addRef(obj)) {
      return;
    }

    Class<?> cl = obj.getClass();

    // 第一部分 Map头部（HessianOutput的方法）
    int ref = out.writeObjectBegin(cl.getName());

    if (ref >= 0) {
      writeInstance(obj, out);
    }
    else if (ref == -1) {
      // hessian2使用的
      writeDefinition20(out);
      out.writeObjectBegin(cl.getName());
      writeInstance(obj, out);
    }
    else {
      // 第二部分 Map值
      writeObject10(obj, out);
    }
}
```

##### Map的头部
| 字节 | 值 | 解释 |
|--- |--- |--- | 
|77, 116 | Mt| 表示这是个map的结构，{key：value} |
|0, 39 | 39 | 表示接下来的39个字节就是类名 |
|99, 111, 109, 46, 100, 101, 109, 111, 46, 100, <br>101, 109, 111, 115, 112, 114, 105, 110, 103, 98, <br> 97, 115, 101, 46, 104, 101, 115, 115, 105, 97, <br> 110, 46, 80, 101, 114, 115, 105, 111, 110 | com.demo.demospringbase.hessian.Persion| 类名 |
```java
// HessianOutput.java

public void writeMapBegin(String type) throws IOException
{
    os.write('M');
    os.write('t');
    printLenString(type);
}
```

##### Map的值
| 字节 | 值 | 解释 |
|--- |--- |--- | 
|83 |S |表示是一个String类型 |
|0, 4 |4 |表示接下来的4个字节就是filed的name |
|110, 97, 109, 101 |name |filed的name |
|83 |S |表示是一个String类型 |
|0, 4 |4 |表示接下来的4个字节就是filed的value |
|108, 105, 110, 107 |link |filed的vallue |
|122 |z |表示这是个map的结构结束 | 
```java
// UnsafeSerializer.java

protected void writeObject10(Object obj, AbstractHessianOutput out) throws IOException
    {
    for (int i = 0; i < _fields.length; i++) {
      Field field = _fields[i];

      out.writeString(field.getName());

      _fieldSerializers[i].serialize(out, obj);
    }

    // 第三部分 尾部（HessianOutput的方法）
    out.writeMapEnd();
}
```

#### 结束标志
| 字节 | 值 | 解释 |
|--- |--- |--- | 
|122 |z |示这次通讯的结束标志 | 

看下HessianOutput提供的方法，已经囊括了所有类型的序列方式
{% asset_img 01.png %}

hessian有很多序列化器，决定使用哪个序列化器`SerializerFactory`的`getSerializer(Class cl)`
{% asset_img 02.png %}

## 总结

本教程只是简单的介绍hessian序列化的过程，知道从哪里入手后一步步debug，就可以分析很复杂的调用

这次调用的的所有字节就结束了。由于为了演示字节数组代表的含义，所以方法比较简单。读者可以试一下参数有List，Enum

从协议可以看出，协议没有任何校验手段。hessian是靠http传输保证数据的有效性的
