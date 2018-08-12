---
title: "postgres 突然很慢"
date: 2019-10-07
draft: false
tags: [postgres]
description: "postgres 突然很慢. postgresql slow query"
---
今天使用 pg_dump 导数据的时候，很长时间都不能导出，明显不正常...  
看了一下线上程序...  
嗯... 队列相关的功能出了问题...  
尝试命令行执行队列，发现一直 processing...  
看了一下 laravel 日志，对写库的操作都非常的慢...  
嗯... 写库出了问题...  
可是怎么引起的呢?     
通过查询 pg_stat_activity 表  
发现绝大多数的 activity 来自一个特定 ip 对 \*_test 数据库的操作  
我想：可能是对 \*_test 数据库的操作影响了线上数据库的性能  
再一问，这个 ip 是公司一台测试服务器的 ip  
一查，服务器上有一个定时任务访问 \*_test 数据库  
定时任务停掉  
问题解决了 
