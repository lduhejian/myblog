---
title: " Micro-Benchmarks"
date: 2020-07-27
draft: false
tags: [PHP, Micro-Benchmarks]
description: "php, you may find yourself wondering which approach is the fastest, 函数性能比较"
---
很多时候你想知道哪个函数运行的更快。例如： str_replace() 和 preg_replace() 哪个运行更快？ 你可以写一个 micro-benchmark 测试程序来回答这类问题。

下面的代码是一个库文件 (ubm.php) 用来支持 micro-benchmark 测试程序的，他能够具体告诉你哪个函数运行更快。

ubm.php:
```
<?php 
register_shutdown_function('micro_benchmark_summary');
$ubm_timing = array();

function micro_benchmark($label, $impl_func, $iterations = 1) {
    global $ubm_timing;
    print "benchmarking `$label'...";
    flush();
    $start = current_usercpu_rusage();
    call_user_func($impl_func, $iterations);
    $ubm_timing[$label] = current_usercpu_rusage() - $start;
    print "<br />\n";
    return $ubm_timing[$label];
}

function micro_benchmark_summary() {
    global $ubm_timing;
    if (empty($ubm_timing)) {
        return;
    }
    arsort($ubm_timing);
    reset($ubm_timing);
    $slowest = current($ubm_timing);
    end($ubm_timing);
    print "<h2>And the winner is: ";
    print key($ubm_timing) . "</h2>\n";
    print "<table border=1>\n <tr>\n <td>&nbsp;</td>\n";
        foreach ($ubm_timing as $label => $usercpu) {
        print " <th>$label</th>\n";
    }
    print " </tr>\n";
    $ubm_timing_copy = $ubm_timing;
    foreach ($ubm_timing_copy as $label => $usercpu) {
        print " <tr>\n <td><b>$label</b><br />";
        printf("%.3fs</td>\n", $usercpu);
        foreach ($ubm_timing as $label2 => $usercpu2) {
            $percent = (($usercpu2 / $usercpu) - 1) * 100;
            if ($percent > 0) {
                printf("<td>%.3fs<br />%.1f%% slower",
                $usercpu2, $percent);
            } elseif ($percent < 0) {
                printf("<td>%.3fs<br />%.1f%% faster",
                $usercpu2, -$percent);
            } else {
                print "<td>&nbsp;";
            }
            print "</td>\n";
        }
        print " </tr>\n";
    }
    print "</table>\n";
}

function current_usercpu_rusage() {
     $ru = getrusage();
     return $ru['ru_utime.tv_sec'] + ($ru['ru_utime.tv_usec'] / 1000000.0);
}
```

>注意：ubm.php 使用 getrusage（）函数来计算消耗的CPU周期。getrusage（）中测量值的精准度取决于您的系统设置，但通常是1/100秒（在 FreeBSD 上是1/1000秒），因此在两个函数性能相差很小的时候，如果只测试一次可能得到错误的结果，因此需要多测试几次。

下面是 str_replace() 和 preg_replace() 进行比较的具体代码：
```
<?php
require 'ubm.php';
$str = "This string is not modified";
$loops = 1000000;
micro_benchmark('str_replace', 'bm_str_replace', $loops);
micro_benchmark('preg_replace', 'bm_preg_replace', $loops);
function bm_str_replace($loops) {
    global $str;
    for ($i = 0; $i < $loops; $i++) {
        str_replace("is not", "has been", $str);
    }
}
function bm_preg_replace($loops) {
    global $str;
    for ($i = 0; $i < $loops; $i++) {
        preg_replace("/is not/", "has been", $str);
    }
}
```

![](/uploads/Micro-Bechmarks/a4757bc1-13a3-406d-904c-438f9033eadd.png)

上图是执行结果。The percentages in each cell tell 
you how much faster or slower the previous test was compared to the test to the left.

通过上图结果我们知道  str_replace() 比  preg_replace() 快 20%。

这种测试方法更适合于有很少最好没有 IO 操作的函数，当存在 IO 操作时，测试结果就可能是错误的。其他的诸如读写磁盘和数据库操作也会影响测试结果。

我们最好多测试几次，来确保结果的准确性。如果几次测试结果存在不一致，那么可能测试函数中存在有不适合使用 micro-benchmark 的操作(例如 IO 操作), 或者系统中有影响 micro-benchmark 的服务。

>不要丢掉 micro-benchmarks!你可能利用他们来测试新的 php 版本中某个函数的优化效果。


##### 以上翻译自[《PHP 5 Power Programming》](https://ptgmedia.pearsoncmg.com/images/013147149X/downloads/013147149X_book.pdf)14.9.1 Micro-Benchmarks 节。
  
[文中代码](https://#)