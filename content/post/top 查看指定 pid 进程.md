---
title: "top 查看指定 pid 进程"
date: 2019-05-23
draft: false
tags: [linux]
description: "linux top 查看指定 pid 进程 ps aux"
---
1. 通过关键字找到进程的 pid
```
ps aux | grep -i $keyword
```
2. 使用 top -p 查看进程状态
```
top -p $pid
```