---
title: 4-Hexo加入谷歌统计
date: 2017-07-12 16:01:19
tags: Hexo
categories:
- IT
- Hexo
---


使用默认的theme（landscape）
![](4-Hexo加入谷歌统计/01.png)

修改这个文件 themes -> landscape -> _config.yml （PS: 不要改成根目录的那个_config.yml）
添加如下google代码
```
google_analytics: UA-101268827-1
```
UA-101268827-1是从google统计获取的

查看页面源码看是否已经加上了
![](4-Hexo加入谷歌统计/02.png)