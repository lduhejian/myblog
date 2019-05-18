---
title: "file_get_contents 获取三方接口的返回"
date: 2019-05-18
draft: false
tags: [php, file_get_contents]
---
```
file_get_contents($site);
```
上面这行代码 file_get_contents 默认等待时间是 60, 如果三方接口一直没有响应，你的程序就会一直等在那里 60s
如果访问用户很多，就会有很多线程同时在那里 waiting ... 影响程序运行

为了防止三方接口出问题时，我们的程序能够做出处理。我们进行如下操作。

1. 将 file_get_contents 的等待时间设小一些，例如 2s
```
$ctx = stream_context_create(array('http'=>
    array(
        'timeout' => 2,
    )
));
file_get_contents('http://example.com/', false, $ctx);
```

2. 使用错误控制运算符 @，处理异常情况
```
$content = @file_get_contents($site);
if($content === FALSE) { // handle error here... }
```

参考:  
https://www.php.net/manual/en/function.file-get-contents.php
https://www.php.net/manual/zh/language.operators.errorcontrol.php
https://stackoverflow.com/questions/10236166/does-file-get-contents-have-a-timeout-setting/10236480#10236480