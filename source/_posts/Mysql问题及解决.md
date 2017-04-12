---
title: Mysql问题及解决
date: 2017-04-12 15:25:20
tags: Mysql
categories:
- IT
- DB
- Mysql
---

<!-- toc -->

---

### LOAD DATA 发生错误：Lost connection to MySQL server during query
```bash
# 看下max_allowed_packet的大小
show VARIABLES like '%max_allowed_packet%';

# 设置max_allowed_packet=20M
set global max_allowed_packet = 50*1024*1024*10;
```