---
title: hibernate-tools详细使用教程
date: 2011-01-05 14:11:55
tags: eclipse
categories:
- IT
- java
- IDE
---

使用hibernate-tool的版本是hibernatetools-Update-2010-09-08_14-25-39-H23
1.在eclipse下安装好hibernate-tool插件，新建一个hibernate.cxf.xml文件
![](hibernate-tools详细使用教程/01.jpg)

2.写好数据库连接配置
![](hibernate-tools详细使用教程/02.jpg)

3
![](hibernate-tools详细使用教程/03.png)

4
![](hibernate-tools详细使用教程/04.png)

5
![](hibernate-tools详细使用教程/05.png)

6
![](hibernate-tools详细使用教程/06.png)

7
![](hibernate-tools详细使用教程/07.png)

8
![](hibernate-tools详细使用教程/08.png)

9
![](hibernate-tools详细使用教程/09.png)

10
![](hibernate-tools详细使用教程/10.png)

11
![](hibernate-tools详细使用教程/12.png)

12
![](hibernate-tools详细使用教程/13.png)

13
![](hibernate-tools详细使用教程/14.png)

14.下图生成的entity有注释是因为使用的模板不是hibernate-tools提供的，是自己修改的
![](hibernate-tools详细使用教程/03.png)


hibernate生成注释模板下载：
{% asset_link hbmtemplates-201009201535.rar %}

 
hibernate生成中文注释乱码解决（eclipse设置为utf-8编码）：
修改hibernate-tool的源码
{% asset_link hibernate-tools.jar %}
只要把hibernate-tool的hibernate-tools.jar文件替换成上面的jar包就ok了，目前只修改了oracle的中文注释乱码