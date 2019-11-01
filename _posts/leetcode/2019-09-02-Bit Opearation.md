---
layout: post
title: "LeetCode - Bit Operation"
subtitle:
author: "Peinan"
header-style: text
category: leetcode
tags:
  - Algorithms
---

1. n的幂系列 - 最优解法

LeetCode - 231 2的幂

判断给定的整数是否为2的幂次方

```cpp
// 利用位运算快速求解
class Solution {
public:
    bool isPowerOfTwo(int n) {
          return n>0&&(n&n-1)==0;      
    }
};
```

LeetCode - 342 4的幂

判断给定的整数是否为4的幂次方

```cpp
// 基于2的幂再进一步判断
class Solution {
public:
    bool isPowerOfFour(int num) {
        return num > 0 && (num&num-1)==0 && (num&0x55555555)!= 0;
    }
};
```

LeetCode - 346 3的幂

判断给定的整数是否为3的幂次方

```cpp
// 利用3的幂因数只包含3的幂这一重要性质
class Solution {
public:
    bool isPowerOfThree(int n) {
        return n > 0 && 1162261467 % n == 0;
    }
};
```

2. 出现数字系列 - 最优解法

LeetCode - 136 只出现一次的数字

给定一个非空整数数组，除了某个元素只出现一次以外，其余每个元素均出现两次。找出那个只出现了一次的元素。

要求：时间复杂度为O(n) 空间复杂度为O(1)

```csharp
// 利用异或的性质 a^b^b = a
public int SingleNumber(int[] nums)
{
    int result = 0;
    foreach (int i in nums)
        result ^= i;
    return result;
}
```

LeetCode - 137 只出现一次的数字

给定一个非空整数数组，除了某个元素只出现一次以外，其余每个元素均出现了三次。找出那个只出现了一次的元素。

要求：时间复杂度为O(n) 空间复杂度为O(1)

```csharp
// 原问题转化为 => 利用位运算设计一种重复三次回到自身的计算
public int SingleNumber(int[] nums)
{
    int a = 0, b = 0, c = 0;
    foreach(int i in nums)
    {
        b = b | (a & i);
        a = a ^ i;
        c = a & b;
        b = b & ~c;
        a = a & ~c;
    }
    return a;
}
```

LeetCode - 260 只出现一次的数字

给定一个整数数组 nums，其中恰好有两个元素只出现一次，其余所有元素均出现两次。 找出只出现一次的那两个元素。

要求：时间复杂度为O(n) 空间复杂度为O(1)

```csharp
// 利用第一题的算法求出两个数字的异或值 并利用异或计算的特性进行分组处理 拆出两个特殊的数字后再用第一题的算法求解
    public int[] SingleNumber(int[] nums)
    {
        int a = 0;
        foreach (int i in nums)
            a ^= i;
        // assume x and y are two elements appear once, now a = x ^ y
        int index = 0;
        for (index = 0; index < 32; ++index)
        {
            uint temp = (uint)a;
            temp >>= index;
            temp <<= 31;
            temp >>= 31;
            if (temp == 1)
                break;
        }
        int b = 0, c = 0;
        foreach(int i in nums)
        {
            uint temp = (uint)i;
            temp >>= index;
            temp <<= 31;
            temp >>= 31;
            if (temp == 0)
                b ^= i;
            else
                c ^= i;
        }
        return new int[] { b, c };
    }
```

LeetCode - 645 错误的集合

集合 S 包含从1到 n 的整数。不幸的是，因为数据错误，导致集合里面某一个元素复制了成了集合里面的另外一个元素的值，导致集合丢失了一个整数并且有一个元素重复。

给定一个数组 nums 代表了集合 S 发生错误后的结果。你的任务是首先寻找到重复出现的整数，再找到丢失的整数，将它们以数组的形式返回。

```csharp
public class Solution {
    public int[] FindErrorNums(int[] nums)
    {
        int duplicatedNum = 0;
        for(int i = 0;i<nums.Length;++i)
            if (nums[Math.Abs(nums[i]) - 1] < 0)
                duplicatedNum = Math.Abs(nums[i]);
            else
                nums[Math.Abs(nums[i]) - 1] *= -1;
        int c = 0;
        for(int i = 0;i<nums.Length;++i)
            c ^= Math.Abs(nums[i]);
        c ^= duplicatedNum;
        for (int i = 1; i <= nums.Length; ++i)
            c ^= i;
        return new int[2] { duplicatedNum, c };
    }
}
```

LeetCode - 268 缺失数字

给定一个包含 0, 1, 2, ..., n 中 n 个数的序列，找出 0 .. n 中没有出现在序列中的那个数。

要求：时间复杂度为O(n) 空间复杂度为O(1)

```csharp
public class Solution {
    public int MissingNumber(int[] nums)
    {
        int k = 0;
        for (int i = 1; i < nums.Length + 1; ++i)
            k ^= i;
        foreach (int i in nums)
            k ^= i;
        return k;
    }
}
```

3. LeetCode - 318 最大单词长度乘积

给定一个字符串数组 words，找到 length(word[i]) * length(word[j]) 的最大值，并且这两个单词不含有公共字母。你可以认为每个单词只包含小写字母。如果不存在这样的两个单词，返回 0。

```csharp
public class Solution {
    public int MaxProduct(string[] words)
    {
        int max = 0;
        uint[] vec = new uint[words.Length];
        for(int i = 0;i<words.Length;++i)
        {
            uint tmp = 0;
            foreach(char c in words[i])
            {
                tmp |= 1u << c - 'a';
            }
            vec[i] = tmp;
        }
        for(int i = 0; i < vec.Length; ++i)
        {
            for(int j = i+1; j < vec.Length; ++j)
            {
                if ((vec[i] & vec[j]) == 0)
                    if (words[i].Length * words[j].Length > max)
                        max = words[i].Length * words[j].Length;
            }
        }
        return max;
    }
}
```

4. LeetCode - 477 汉明距离总和

两个整数的 汉明距离 指的是这两个数字的二进制数对应位不同的数量。

计算一个数组中，任意两个数之间汉明距离的总和。

```csharp
public class Solution {
    public int TotalHammingDistance(int[] nums)
    {
        int result = 0;
        for(int i = 0; i < 32; ++i)
        {
            int count0 = 0;
            int count1 = 0;
            foreach(int n in nums)
                if ((n & 1 << i) >> i == 0)
                    count0++;
                else
                    count1++;
            result += count0 * count1;
        }
        return result;
    }
}
```