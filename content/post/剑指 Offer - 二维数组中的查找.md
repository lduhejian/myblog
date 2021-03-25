---
title: "剑指 Offer - 二维数组中的查找"
date: 2021-03-25
draft: false
tags: [golang, 数据结构]
description: "golang, 数据结构, 剑指 Offer - 二维数组中的查找"
---
```
package main

import "fmt"

func find(matrix [][]int, row int, column int, val int) bool {
	isFind := false
	var i, j int
	for j = 0; j < column; j++ {
		if matrix[0][j] == val {
			return true
		} else if matrix[0][j] > val {
			break
		}
	}
	j--
	for i = 1; i < row && j >= 0; i++ {
		if matrix[i][j] == val {
			return true
		} else if matrix[i][j] > val {
			j--
		}
	}
	return isFind
}

func main() {
	matrix := [][]int{
		{1, 2, 8, 9},
		{2, 4, 9, 12},
		{4, 7, 10, 13},
		{6, 8, 11, 15}}
	isFind := find(matrix, 4, 4, 0)
	fmt.Println(isFind)
}
```  