---
title: Maven pom实用配置
date: 2017-03-30 20:04:54
tags: java
categories:
- IT
- java
- maven
---

<!-- toc -->

### 不要发布
当项目是很多的模块，有些模块（例如打成war包的模块）不需要deploy
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-deploy-plugin</artifactId>
    <configuration>
     <skip>true</skip>
    </configuration>
</plugin>
```

### 获取svn版本号
```xml
<scm>    
  <url>scm:svn:svn://203.195.156.249/services/sleepace-pro-api</url>
  <connection>scm:svn:svn://203.195.156.249/services/sleepace-pro-api</connection>
</scm>
<build>
 <finalName>sleepace-pro-api-${buildNumber}</finalName>
 <plugins>
  <plugin>
   <groupId>org.codehaus.mojo</groupId>
   <artifactId>buildnumber-maven-plugin</artifactId>
   <version>1.2</version>
   <executions>
    <execution>
     <phase>compile</phase>
     <goals>
      <goal>create</goal>
     </goals>
    </execution>
   </executions>
   <configuration>
    <doCheck>false</doCheck>
    <doUpdate>false</doUpdate>
    <username>linjia</username>
    <password>linjia</password>
    <items>
     <item>scmVersion</item>
    </items>
   </configuration>
  </plugin>
 </plugins>
</build>
```
scm配置svn得地址
username登录名
password登录密码

### 把source一起发布
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-source-plugin</artifactId>
    <version>2.1</version>
    <configuration>
        <attach>true</attach>
    </configuration>
    <executions>
        <execution>
            <phase>compile</phase>
            <goals>
                <goal>jar</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```


### 打成jar包，并制定main方法
```bash
mvn clean compile assembly:single
```
```xml
<plugin> 
  <groupId>org.apache.maven.plugins</groupId>  
  <artifactId>maven-assembly-plugin</artifactId>  
  <version>2.3</version>  
  <configuration> 
    <appendAssemblyId>false</appendAssemblyId>
    <descriptorRefs>   
      <descriptorRef>jar-with-dependencies</descriptorRef>
    </descriptorRefs>
    <archive>   
      <manifest>   
        <mainClass>com.medica.servicenetty.HealthServer</mainClass>    
      </manifest>  
    </archive>  
  </configuration>
</plugin>
```

### 把jar安装到本地
```xml
<dependency>
    <groupId>ch.ethz.ganymed</groupId>
    <artifactId>ganymed-ssh2</artifactId>
    <version>build210</version>
</dependency>
```
```bash
mvn install:install-file -Dfile=ganymed-ssh2-build210.jar -DgroupId=ch.ethz.ganymed -DartifactId=ganymed-ssh2 -Dversion=build210 -Dpackaging=jar
```

### mvn dependency:tree
排查包冲突超好用，列出所有依赖关系
```bash
mvn dependency:tree
```