---
layout: post
title: "LeetCode - Sieve of Eratosthenes"
subtitle:
author: "Peinan"
header-style: text
category: leetcode
tags:
  - Algorithms
---

An algorithm to find all prime numbers from 1 to n, with O(n) time complexity.

Brute force solution costs O(sqrt(n)) for each number from 1 to n.

```cpp
int countPrimes(int n) {
    vector<bool> tmp(max(2, n), true);
    tmp[0] = tmp[1] = false;
    int t = sqrt(n);
    for (int i = 2; i <= t; ++i) {
        if (tmp[i]) {
            for (int x = i * i; x < n; x += i) {
                tmp[x] = false;
            }
        }
    }
    int count = 0;
    for (int i = 0; i < n; ++i)
        count += tmp[i];
    return count;
}
```

---
