---
title: "Studly Words in Laravel"
date: 2018-08-22T15:24:13+08:00
draft: false
tags: [laravel]
---
#### studly 函数

读 laravel framework 源码，发现了一个 studly 函数（在 Str.php 中）：

```
/**
  * Convert a value to studly caps case.
  *
  * @param  string  $value
  * @return string
  */
 public static function studly($value)
 {
     $key = $value;

     if (isset(static::$studlyCache[$key])) {
         return static::$studlyCache[$key];
     }

     $value = ucwords(str_replace(['-', '_'], ' ', $value));

     return static::$studlyCache[$key] = str_replace(' ', '', $value);
 }
```

model 的属性名通过下面这个关系映射出 mutator，例如：product_name => setProductNameAttribute：

```
'set'.Str::studly($attribute_name).'Attribute'
```

哈哈，从此再也不怕写错 mutator 了 :stuck_out_tongue_winking_eye:

#### camel 函数

我们把 studly 的结果的第一个字符变为小写，也就是我们的 camel 函数（驼峰命名，也在 Str.php 中）：

```
/**
  * Convert a value to camel case.
  *
  * @param  string  $value
  * @return string
  */
 public static function camel($value)
 {
     if (isset(static::$camelCache[$value])) {
         return static::$camelCache[$value];
     }

     return static::$camelCache[$value] = lcfirst(static::studly($value));
 }
```
