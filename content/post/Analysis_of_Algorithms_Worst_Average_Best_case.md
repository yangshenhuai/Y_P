---
title: "算法分析(最坏，平均和最好情况)"
date: 2018-07-22T00:21:47+08:00
draft: false
tags: ["algorithms","geeksforgeeks"]
author: "Peter Yang"
---

翻译自[GeeksforGeeks](https://www.geeksforgeeks.org/analysis-of-algorithms-set-2-asymptotic-analysis/)


在[上篇文章](../analysis_of_algorithms_asymptotic_analysis/)我们讨论了线性分析克服了算法分析的问题，这篇文章，我们以线性分析为例看一下渐进分析

我们会有三种不同的情况去分析一个算法

1. 最坏情况
2. 平均情况
3. 最好情况

以下面的线性查找的实现为例

	#include <stdio.h>
	 
	// Linearly search x in arr[].  If x is present then return the index,
	// otherwise return -1
	int search(int arr[], int n, int x)
	{
	    int i;
	    for (i=0; i<n; i++)
	    {
	       if (arr[i] == x)
	         return i;
	    }
	    return -1;
	}
	 
	/* Driver program to test above functions*/
	int main()
	{
	    int arr[] = {1, 10, 30, 15};
	    int x = 30;
	    int n = sizeof(arr)/sizeof(arr[0]);
	    printf("%d is present at index %d", x, search(arr, n, x));
	 
	    getchar();
	    return 0;
	}

# 最坏情况(经常发生)
在最坏情况分析里， 我们计算一个算法的运行时间上限。我们必须清楚执行最多操作的情况。对线性查找来说，最坏的情况发生在要查找的元素(上面代码中的x)在数组中不存在。当 x 不存在时，`search()`会跟数组中的每个元素对比。所以线性查找最坏情况下的的时间复杂度为`Θ(n)`

# 平均情况（有时发生）
平均情况分析中，我们计算所有可能的时间并除以所有的input。我们必须知道（或者预测）所有case的分布。对线性查找来说，我们可以假设所有的情况是均匀分布的。所以我们把所有的情况相加并处以总数(n+1).以下就是平均情况的时间复杂度

![][1]

# 最好情况（虚假情况）
最好情况分析下，我们分析算法运行时间的下限。我们必须知道哪种操作会需要最少的操作。在线性查找中，最好情况是 x 出现在数组的第一个位置。
最好情况先所需操作的的数量是个常量（并不会依赖于数组长度 n ）.所以最好情况下的时间复杂度为 `Θ(1)`
多数情况下，我们分析算法时做最坏情况分析，通过最坏情况分析我们可以保证算法运行时间的上限，这是一个很有用的信息。
平均情况分析在实用中很难做到并且也很少去做，在平均分析中我们必须知道所有可能输入的数学分布。
最好情况分析并不能想最坏情况分析那样提供有用信息，你的算法可能需要一年才能跑完。

对很多算法来说，所有的情况都是渐进分析一样的，也就是说没有最坏和最好情况。 比如说，[归并排序](http://en.wikipedia.org/wiki/Merge_sort). 归并排序在所有情况下都需要`Θ(nLogn)`的操作。 大多数其它的排序刷法有最好和最坏情况。 比如说，快速排序中，最坏情况是输入数组已经是拍好序的数组最好的情况为支点(pivot)元素总是把数组分成两个一样长度的数组。对插入排序来数，最坏情况为数组是反序排序，最好情况是数组已经排好序了

# 参考

[MIT’s Video lecture 1 on Introduction to Algorithms.](http://www.youtube.com/watch?v=JPyuH4qXLZ0)


[1]: /img/avg_case_time.PNG