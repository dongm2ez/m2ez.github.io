---
layout: post
title: 排序算法-冒泡排序
categories: development
tags: [python]
---

## 算法描述

冒泡排序是一种简单的排序算法。它重复地遍历排序的数列，一次比较两个元素，如果它们的顺序错误就交换彼此。遍历数列的工作是重复地进行直到没有再需要交换，既该数列已经排序完成。这个算法的名字由来是因为越小的元素会经由交换慢慢“浮”到数列的顶端。

## 算法复杂度

平均时间复杂度 | 最好情况 | 最坏情况 | 空间复杂度 | 排序方式 | 稳定性
---- | --- | --- | --- | --- | ---
O(n平方) | O(n) | O(n平方) | O(1) | In-place | 稳定

## 算法实现步骤

1. 比较相邻的元素。如果第一个比第二个大，就交换它们两个；
2. 对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对，这样在最后的元素应该会是最大的数；
3. 针对所有的元素重复以上的步骤，除了最后一个；
4. 重复步骤1~3，直到排序完成。

## 算法过程演示

![](https://img.m2ez.com/bubble%20sort.gif)


## 实现代码

```Python
def bubble_sort(lists):
    size = len(lists)
    for i in range(size - 1):
        for j in range(0, size - 1 - i):
            if lists[j] > lists[j + 1]:
                lists[j], lists[j + 1] = lists[j + 1], lists[j]

lists = [54, 26, 93, 77, 44, 31, 44, 55, 20]
print("原列表为：%s" % lists)
bubble_sort(lists)
print("新列表为：%s" % lists)
```
