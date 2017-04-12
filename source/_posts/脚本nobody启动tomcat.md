---
title: 脚本nobody启动tomcat
date: 2017-04-12 17:06:01
tags: Linxu
categories:
- IT
- Linux
---

### 场景：
crontab配置和了个定时任务检查tomcat是否挂掉，如果挂掉就自动启动。但一直没有作用（刚开始没有把错误重定向到一个文件，一直没有找到问题）

tomcat的启动方式：sudo -u nobody /usr/local/tomcat/bin/startup.sh

### 错误：
"sudo: sorry, you must have a tty to run sudo”
看错误的意思是nobody需要一个终端才可以执行这条命令

### 解决：
修改/etc/sudoers
1.Defaults requiretty，修改为 #Defaults requiretty，注释掉表示不需要控制终端（不推荐）。
2.Defaults requiretty，修改为 Defaults:nobody !requiretty，表示仅 nobody 用户不需要控制终端。
