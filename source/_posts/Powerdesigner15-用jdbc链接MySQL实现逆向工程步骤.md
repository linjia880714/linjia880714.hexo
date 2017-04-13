---
title: Powerdesigner15-用jdbc链接MySQL实现逆向工程步骤
date: 2010-12-31 09:49:39
tags: Powerdesigner
categories:
- IT
- Other
---

<!-- toc -->

---

### 1.建立一个物理模型
![](Powerdesigner15-用jdbc链接MySQL实现逆向工程步骤/01.png)

### 2.配置一个jdbc数据库链接，以mysql为例

注意：
使用 jdbc 连接数据库，要在环境变量设置 classpath ，值为数据库jdbc的 jar 文件路径

![](Powerdesigner15-用jdbc链接MySQL实现逆向工程步骤/02.png)
![](Powerdesigner15-用jdbc链接MySQL实现逆向工程步骤/03.png)
![](Powerdesigner15-用jdbc链接MySQL实现逆向工程步骤/04.png)

### 3.把物理模型的表更新到刚才配置的数据库（mysql）
![](Powerdesigner15-用jdbc链接MySQL实现逆向工程步骤/05.png)
![](Powerdesigner15-用jdbc链接MySQL实现逆向工程步骤/06.png)
![](Powerdesigner15-用jdbc链接MySQL实现逆向工程步骤/07.png)
