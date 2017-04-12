---
title: Mysql索引学习
date: 2011-10-20 18:01:35
tags: Mysql
categories:
- IT
- DB
- Mysql
---

使用索引可以提高查询效率，废话不多说，先来个例子。

```sql
CREATE TABLE `person` (  
  `id` int(11) NOT NULL AUTO_INCREMENT,  
  `name` varchar(20) DEFAULT NULL,  
  `birthday` datetime DEFAULT NULL,  
  `isMan` int(11) DEFAULT NULL,  
  `salary` double DEFAULT NULL,  
  `test` int(11) DEFAULT NULL,  
  PRIMARY KEY (`id`)  
)   
```

在插入一些数据。例如：

![](Mysql索引学习/78b8b525-6624-3dcf-9eca-57291fe4ab24.png)

运行：
EXPLAIN select name from person where name='linjia4'

观察结果：
![](Mysql索引学习/ec8aed91-8a7a-3d2b-bf5a-e1bee8b57bf8.png)

row=23, 是因为这个表总共有23条记录，即是说做了全表扫描


在搜索的“name”加上索引：
create index testtest on person (name)
再执行EXPLAIN select name from person where name='linjia4'
观察结果：

![](Mysql索引学习/eeb1ed93-e39a-3621-9d5d-7dc872b7951e.png)
rows=1, 即是所当查询语句执行时通过name的索引，直接定位到那条数据，不用做全表扫描
 
索引的效果是显而易见的，大大的提高了查询的速度。但索引也不能滥用，增加索引是要付出代价的。