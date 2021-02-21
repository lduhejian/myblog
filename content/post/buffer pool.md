---
title: "buffer pool"
date: 2021-02-21
draft: true
tags: [mysql]
description: "mysql buffer pool"
---
buffer pool 是在内存中存储数据和索引的，用于快速访问？

buffer pool 的结构是怎样的？链表，管理的单位是 page

变体的 LRU 


mysql 默认数值单位是

buffer pool 的大小
可以离线也可以在线设置

查看 innodb buffer pool 大小 
```
SELECT @@innodb_buffer_pool_size/1024/1024
```
@@innodb_buffer_pool_size 为什么和show engine innodb status 不一样？

https://dev.mysql.com/doc/refman/5.7/en/innodb-buffer-pool.html
https://dev.mysql.com/doc/refman/5.7/en/innodb-buffer-pool-resize.html