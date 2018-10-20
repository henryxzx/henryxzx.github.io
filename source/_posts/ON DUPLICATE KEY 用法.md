---
title: Mysql ON DUPLICATE KEY 用法
date: 2018-10-20 00:36:28
tags: 'mysql'
categories: '数据库'
---

为保证数据表中的内容不会重复，通常会设定一个或多个列为Primary key

<!--more-->

首先，我想向表中插入一条数据(假设name为PK)

```mysql
INSERT INTO player(name, sex, weight)VALUES('jack', 'male', 50)
```

在插入这段数据之前，我们会先检查PK值是否已经存在以免发生错误，所以需要多写一段select语法

```mysql
SELECT * FROM player WHERE name = 'jack'
```

如果现在的功能需求是“**新增一条数据，如果已经存在就更新**”

就可以使用MySQL中的语法

```mysql
INSERT ... ON DUPLICATE KEY UPDATE
```

根据刚才的需求，改写sql语句

```mysql
INSERT into player(name, sex, weight)VALUES('jack', 'male', 50) ON DUPLICATE KEY UPDATE name = 'jack', sex = 'male', weight = '50'
```





