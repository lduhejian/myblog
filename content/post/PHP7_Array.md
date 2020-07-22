---
title: "PHP7 中的数组"
date: 2020-07-17
draft: true
tags: [PHP7, Array, 数组]
description: "PHP7 中的数组"
---
本文中的代码来自 php-7.1.0  
本文主要是熟悉和理解 陈雷老师的慕课  

需要 debug 的 array.php 文件内容如下：
```
<?php
$a = [];

$a[1] = "a";

$a[] = "b";

$a["k1"] = "v1";
$a["k2"] = "v2";

echo $a["k1"];

$a["k1"] = "c";

unset($a["k2"]);
``` 
观察点
1. 数组元素的增删改查过程梳理出来

2. Packed Array 到 Hash Array 的转换过程

3. 里面主要涉及的关键函数

-----------------------
新增：
TODO 观察过程
在 \_zend_hash_init 打断点查看为数组 a 分配的 HashTable 地址为 0x7ffff40583c0 
\_zend_hash_init 完成了对 HashTable 的初始化  

在 zend_execute 打断点
查看参数 op_array 的内容：last 表示 op_array 中有 15 个 opcode  






-------------------------
