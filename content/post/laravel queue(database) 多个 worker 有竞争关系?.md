---
title: "laravel queue(database) 多个 worker 有竞争关系?"
date: 2019-08-17
draft: true
tags: [laravel, queue, database]
---
我有一个疑问：
laravel 队列在数据库驱动的情形下，会不会有竞争问题出现？
例如：jobs 表中的一条记录被读取执行，执行完成（但还未在数据库删除改记录）时，另一个 worker 有读取这条记录 ...

这种问题会出现吗？

让我们看下 laravel 的源码：