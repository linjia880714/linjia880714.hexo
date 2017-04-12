---
title: 3-Hexo图片改成左对齐
date: 2017-04-12 16:43:18
tags: Hexo
categories:
- IT
- Hexo
---

修改文件主题文件themes/landscape/source/css/_partial/article.styl

landscape是Hexo的默认主题

```
  img, video
    max-width: 100%
    height: auto
    display: block
    margin: auto
```
改成
```
  img, video
    max-width: 100%
    height: auto
    display: block
    margin: auto
```
