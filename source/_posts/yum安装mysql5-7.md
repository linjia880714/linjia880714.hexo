---
title: yum安装mysql5.7
date: 2018-09-18 23:31:04
tags: mysql
---

<!-- toc -->

## 添加mysql的yum仓库
```
wget https://repo.mysql.com/mysql80-community-release-el6-1.noarch.rpm

sudo rpm -Uvh mysql80-community-release-el6-1.noarch.rpm
```

## 安装yum-config-manager
```bash
yum install yum-utils 
```


## mysql57设置为可用
```bash
sudo yum-config-manager --disable mysql80-community

sudo yum-config-manager --enable mysql57-community

## 查看mysql是否已经设置为5.7
yum repolist enabled | grep mysql
```

## mysql57安装
```bash
sudo yum install mysql-community-server
```

## mysql第一次启动且获取密码
```bash
sudo service mysqld start

## 提取密码
```bash
sudo grep 'temporary password' /var/log/mysqld.log
```

## 修改mysql的root密码
MySQL默认安装“validate_password”插件，密码至少需要1个大写字母，1个小写字母，1个数字，1个特殊字符。密码至少需要8位

注意：需要重设下密码，不然其他操作会失败报错 `You must reset your password using ALTER USER statement before executing this statement.`
```bash
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!';
```