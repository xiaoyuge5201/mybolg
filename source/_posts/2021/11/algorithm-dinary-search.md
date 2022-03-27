---
title: 二分法查找题
comments: true
abbrlink: 47921
date: 2021-11-26 21:57:32
tags: algorithm
categories: 算法
translate_title: binary-search
---
### 1. 第一个错误的版本

### 1.1 题目描述
你是产品经理，目前正在带领一个团队开发新的产品。不幸的是，你的产品的最新版本没有通过质量检测。由于每个版本都是基于之前的版本开发的，所以错误的版本之后的所有版本都是错的。
假设你有 n 个版本 [1, 2, ..., n]，你想找出导致之后所有版本出错的第一个错误的版本。
你可以通过调用bool isBadVersion(version)接口来判断版本号 version 是否在单元测试中出错。实现一个函数来查找第一个错误的版本。你应该尽量减少对调用 API 的次数。
示例
```text
输入：n = 5, bad = 4
输出：4
解释：
    调用 isBadVersion(3) -> false 
    调用 isBadVersion(5) -> true 
    调用 isBadVersion(4) -> true
所以，4 是第一个错误的版本
```

### 1.2 解题思路
当一个版本为正确版本，则该版本之前的所有版本均为正确版本；当一个版本为错误版本，则该版本之后的所有版本均为错误版本。我们可以利用这个性质进行二分查找。

具体地，将左右边界分别初始化为 1和 n ，其中 n 是给定的版本数量。设定左右边界之后，每次我们都依据左右边界找到其中间的版本，检查其是否为正确版本。如果该版本为正确版本，那么第一个错误的版本必然位于该版本的右侧，我们缩紧左边界；否则第一个错误的版本必然位于该版本及该版本的左侧，我们缩紧右边界。
这样我们每判断一次都可以缩紧一次边界，而每次缩紧时两边界距离将变为原来的一半，因此我们至多只需要缩紧 O(logn) 次。


### 1.3 代码
```java
public int firstBadVersion(int n) {
    int left = 1, right = n;
    while (left < right){
        int mid = left + (right - left) / 2;    // 防止计算时溢出
        if (isBadVersion(mid)){
            // 答案在区间 [left, mid] 中
           right = mid; //如果中间版本是错误的版本，那么它之后的都是错误的;
        }else {
            // 答案在区间 [mid+1, right] 中
            left = mid + 1;
        }
    }
    //此时有 left == right,退出了while循环
    return left;
}
```
