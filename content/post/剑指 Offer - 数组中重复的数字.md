---
title: "剑指 Offer - 数组中重复的数字"
date: 2021-03-25
draft: false
tags: [golang, 数据结构]
description: "golang, 数据结构"
---
```
package main

import "fmt"

func duplicate(numbers []int) (int, bool) {
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

func main() {
	numbers := []int{2, 3, 1, 0, 2, 5, 3}
	val, ok := duplicate(numbers)
	if ok {
		fmt.Println(val)
	}
}
```