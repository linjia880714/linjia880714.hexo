---
title: Jenkins实践
date: 2018-02-24 17:41:20
tags:
---

<!-- toc -->

----

## 插件安装

| 插件名称  | 插件作用 | 
|---|---|
| Git plugin  | 从git数据源拉取代码 |
| Deploy to container Plugin  | 把war包发布到容器里 |

注意：安装完插件最好重启下jenkins，我安装完之后出现了一些奇怪的问题,重启了就好

----

Jenkins ver. 1.644

## 配置Jenkins基本信息
Jenkins > configuration
### JDK
点击“Add JDK”按钮添加JDK信息
{% asset_img 01.png %}
### Maven
点击“Add Maven”按钮添加maven信息
{% asset_img 02.png %}

----

## 发布一个maven项目到Tomcat
### 1. 创建项目
{% asset_img 03.png %}

### 2. 配置Git地址
{% asset_img 04.png %}

| 参数  | 解释   |
|---|---|
| Repository URL  | git仓库地址 |
| Credentials  | 由于我使用的是ssh访问git仓库，所以这里没有值，如果使用username和password登陆的话需要写入 |
| Branch SpecifierL  | 我使用的gitflow创建分支，如图所示我要发布分支feature/3.6.0_P2 |


### 3. 配置Build信息（Maven）
{% asset_img 05.png %}
| 参数  | 解释   |
|---|---|
| Root POM  | 指定pom.xml地址 |
| Goals and options  |  要执行的mvn命令，如图所示相当于执行mvn clean package -Dmaven.test.skip |

### 4. Deploy war包到Tomcat（Deploy to container Plugin）
{% asset_img 06.png %}
由于已经安装了Deploy to container Plugin，所以会有如上图的选项
{% asset_img 07.png %}

| 参数  | 解释   |
|---|---|
| WAR/EAR files  |  war包所在位置 |
| Context path | 访问前缀，例如想这样子访问http://172.14.1.100:11080/pro, 那就填pro |
| Containers Credentials | 访问tomcat的username和password（下面会讲如何设置），点击“Add”按钮添加 |
| Tomcat URL Tomcat URL |  tomcat的root路径 |


#### 设置Tomcat访问的username和password
修改 ${tomcat_dir}/conf/tomcat-users.xml, 然后重启
```xml
<?xml version='1.0' encoding='utf-8'?>
<tomcat-users>
  <role rolename="manager-gui"/>
  <role rolename="manager-script"/>
  <role rolename="manager-jmx"/>
  <role rolename="manager-status"/>
  <user username="tomcat" password="123456" roles="manager-gui,manager-script,manager-jmx,manager-status" />
</tomcat-users>
```
此例子设置
username：tomcat
password: 123456

如何验证设置成功，访问http://${ip}:${port}/，点击“manager app”按钮，弹出登录框，看是否能成功登陆

#### Deploy到tomcat可能会失败
要部署的tomcat抛出异常，如果部署失败不仅要看jenkins的错误信息，也要看tomcat的错误信息，再去定位问题
```
java.lang.OutOfMemoryError: PermGen space
```
因为war包太大（20M），所以java虚拟机会直接存到永久代（不一定要根据java启动的一些配置），永久代内存不够抛出异常

----

## 执行远程shell（SSH2 Easy Plugin）
使用的是SSH2 Easy Plugin，安装完之后记得重启，不然可能有些奇怪的问题
缺点是SSH2 Easy Plugin只能使用username和password登陆
### 基础配置
Jenkins > configuration
{% asset_img 08.png %}

| 	Server Group List 参数  | 解释 |
|---|---|
| Group Name  |  随意 |
| User Name | 登录名 |
| Password | 登陆密码 |

| 	Server List 参数  | 解释 |
|---|---|
| Server Group  |  这是个下拉框，所以必须上面的Server Group Add成功之后才能选 |
| Server Name | 随意 |
| Server IP | IP地址 |

### 配置job configuration
配置完基本设置之后配置job，在Build会有Remote Shell出现
{% asset_img 09.png %}
Target Server是个下拉框，根据Jenkins > configuration设置的参数
{% asset_img 10.png %}

----

## 可能发生的错误
