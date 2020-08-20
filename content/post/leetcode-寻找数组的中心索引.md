---
title: "leetcode-寻找数组的中心索引"
date: 2020-08-20
draft: false
tags: [leetcode, c]
description: "leetcode-寻找数组的中心索引"
---
[点击查看 leetcode 题目地址](https://leetcode-cn.com/leetbook/read/array-and-string/yf47s/)

最开始的实现：
```
int pivotIndex(int* nums, int numsSize) {
	int mIndex = 0;
	for (int i = 0; i < numsSize; i ++)
	{
		int left = 0, right = 0;
		for (int j = 0; j < mIndex; j ++)
		{
			left += nums[j];
		}
		for (int k = numsSize - 1; k > mIndex; k --)
		{
			right += nums[k];
		}
		if (left == right) return mIndex;
		mIndex ++;
	}
	return -1;
}
```

google 后看了别人写的之后:
```
int pivotIndex(int* nums, int numsSize) {
	short sum = 0, total = 0, i = 0;
	for (i = 0; i < numsSize; i ++)
	{
		sum += nums[i];
	}
	total = sum;
	for (i = 0; i < numsSize; i ++)
	{
		sum -= nums[i];
		if (total - nums[i] == sum + sum) return i;
	}
	return -1;
}
```

参考：  
[参考别人的实现在这里](https://github.com/youzouzou/leetcode/issues/53)  
