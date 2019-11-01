---
layout: post
title: "LeetCode - Fast and Slow Pointers in Linkedlist"
subtitle:
author: "Peinan"
header-style: text
category: leetcode
tags:
  - Algorithms
---

快慢指针的算法源于不用任何额外空间判定链表是否存在回环

LeetCode - 141 环形链表

给定一个链表，判断链表中是否有环。

为了表示给定链表中的环，我们使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。 如果 pos 是 -1，则在该链表中没有环。

要求：空间复杂度为O(1)

```csharp
public class Solution {
    public bool HasCycle(ListNode head)
    {
        ListNode p1 = head, p2 = head;
        do
        {        
            if (p1 == null || p2 == null || p2.next == null || p2.next.next == null)
                return false;
            p1 = p1.next;
            p2 = p2.next.next;

        } while (p1.val != p2.val);
        return true;
    }
}
```

LeetCode - 142 环形链表

给定一个链表，返回链表开始入环的第一个节点。 如果链表无环，则返回 null。

为了表示给定链表中的环，我们使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。 如果 pos 是 -1，则在该链表中没有环。

要求：空间复杂度为O(1)

```csharp
public class Solution {
    public ListNode DetectCycle(ListNode head)
    {
        ListNode p1 = head, p2 = head;
        do
        {
            if (p1 == null || p2 == null || p2.next == null || p2.next.next == null)
                return null;
            p1 = p1.next;
            p2 = p2.next.next;

        } while (p1 != p2);
        ListNode p3 = head;
        while (p1 != p3)
        {
            p1 = p1.next;
            p3 = p3.next;
        } 
        return p1;
    }
}
```

LeetCode - 287 寻找重复数

给定一个包含 n + 1 个整数的数组 nums，其数字都在 1 到 n 之间（包括 1 和 n），可知至少存在一个重复的整数。假设只有一个重复的整数，找出这个重复的数。

要求：时间复杂度为O(n) 空间复杂度为O(1)

```csharp
public class Solution {
    public int FindDuplicate(int[] nums)
    {
        int a = 0, b = 0;
        do
        {
            a = nums[a];
            b = nums[b];
            b = nums[b];
        } while (a != b);
        int c = 0;
        while (a != c)
        {
            a = nums[a];
            c = nums[c];
        }
        return a;
    }
}
```

LeetCode - 202 快乐数

编写一个算法来判断一个数是不是“快乐数”。

一个“快乐数”定义为：对于一个正整数，每一次将该数替换为它每个位置上的数字的平方和，然后重复这个过程直到这个数变为 1，也可能是无限循环但始终变不到 1。如果可以变为 1，那么这个数就是快乐数。

```csharp
public class Solution {
    public bool IsHappy(int n)
    {
        int a = n, b = n;
        do
        {
            if (a == 1 || b == 1 || HappyOut(a) == 1)
                return true;
            a = HappyOut(a);
            b = HappyOut(b);
            b = HappyOut(b);
        } while (a != b);
        return false;
    }
    
    public int HappyOut(int n)
    {
        int sum = 0;
        while (n > 0)
        {
            sum += (n % 10) * (n % 10);
            n /= 10;
        }
        return sum;
    }
}
```
