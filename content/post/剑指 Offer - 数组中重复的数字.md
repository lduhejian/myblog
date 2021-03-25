---
title: "剑指 Offer - 数组中重复的数字"
date: 2021-03-25
draft: false
tags: [golang, 数据结构]
description: "golang, 数据结构, 剑指 Offer - 数组中重复的数字"
---
```
package main

import "fmt"

//改变原数组，空间复杂度 O(1), 时间复杂度 O(n)
func duplicateI(numbers []int) (int, bool) {
	for k, v := range numbers {
		for v != k {
			if v == numbers[v] {
				return v, true
			}
			numbers[k], numbers[v] = numbers[v], numbers[k]
		}
	}
	return 0, false
}

//不改变原数组，空间复杂度 O(n), 时间复杂度 O(n)
func duplicateII(numbers []int) (int, bool) {
	appear := make(map[int]bool)
	for _, v := range numbers {
		if _, ok := appear[v]; ok {
			return v, true
		}
		appear[v] = true
	}
	return 0, false
}

func main() {
	numbers := []int{2, 3, 1, 0, 2, 5, 3}
	if val, ok := duplicateI(numbers); ok {
		fmt.Println(val)
	}
	if val, ok := duplicateII(numbers); ok {
		fmt.Println(val)
	}
}
```

参考：  
[数组中重复的数字 - github](https://github.com/DinghaoLI/Coding-Interviews-Golang/blob/master/051-%E6%95%B0%E7%BB%84%E4%B8%AD%E9%87%8D%E5%A4%8D%E7%9A%84%E6%95%B0%E5%AD%97/problem051.go)  