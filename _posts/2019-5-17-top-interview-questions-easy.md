---
layout: post
title: 从排序数组中删除重复项
categories: development
tags: [leetcode,算法]
---

## 题目链接

[从排序数组中删除重复项](https://leetcode-cn.com/explore/interview/card/top-interview-questions-easy/1/array/21/)

## 解题思路：

迭代数组，判断下一个元素是否与当前元素相同，如果相同则删除当前元素。

## 代码

### PHP版本

```PHP
class Solution {

    /**
     * @param Integer[] $nums
     * @return Integer
     */
    function removeDuplicates(&$nums) {
        $size = sizeof($nums);// 防止数组越界
        foreach($nums as $key => $item){
            if($key + 1 <= $size) {
                if($nums[$key] === $nums[$key + 1]){
                    unset($nums[$key]);
                }
            }
        }
        return count($nums);
    }
}
```

语言坑点：

1. PHP使用for循环时`unset`掉元素不重置下标，数组下标保持原状，所以有可能漏掉判断元素。
2. PHP判断是否相等时会将NULL 与 0相等，必须连类型一起判断。
