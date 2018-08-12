---
title: "leetcode-搜索插入位置"
date: 2020-08-20
draft: false
tags: [leetcode, c]
description: "leetcode-搜索插入位置"
---
[点击查看 leetcode 题目地址](https://leetcode-cn.com/leetbook/read/array-and-string/cxqdh/)

我的实现：
```
int searchInsert(int* nums, int numsSize, int target){
    int rangeL = 0, rangeR = numsSize - 1;
    if (target > nums[numsSize - 1]) return numsSize;
	if (target == nums[numsSize - 1]) return numsSize - 1;
    if (target <= nums[0]) return 0;
    while (1)
    {
		if (rangeR == rangeL + 1) return rangeR;
        int middle = rangeL + (rangeR - rangeL) / 2;
        if (nums[middle] == target) return middle;
        else if (nums[middle] > target) rangeR = middle;
        else rangeL = middle;
    }
    return -1;
}
```
google 后看到了一种 O(n) 的容易理解的编码方式：
```
int searchInsert(int* nums, int numsSize, int target)
{
	for (int i = 0; i < numsSize; i ++)
	{
		if (nums[i] >= target) return i;
	}
	return numsSize;
}
```

参考：  
[leetcode 35. Search Insert Position搜索插入位置(二分查找)](https://blog.csdn.net/Shauna_Wu/article/details/78735968)  