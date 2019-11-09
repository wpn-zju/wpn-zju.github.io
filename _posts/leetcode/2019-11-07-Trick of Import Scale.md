---
layout: post
title: "LeetCode - Don't always let your mind constrained by input scale"
subtitle:
author: "Peinan"
header-style: text
category: leetcode
tags:
  - Tips
---

[Problem](https://leetcode.com/problems/soup-servings/)

Actually this problem is not so hard if you consider a 2-dimensional dynamic programming method. For the memory structure and the induction is easy to derive.

```
dp[i][j] = (dp[i - 4][j] + dp[i - 3][j - 1] + dp[i - 2][j - 2] + dp[i - 1][j - 3]) / 4;	// I make every number divided by 25.
```

However, when I first looked into this problem, I was struggled by the input scale, 0 <= N <= 10^9. Normally, it's impossible to use O(n^2) solution on such a large input scale. But notice that /"Answers within 10^-6 of the true value will be accepted as correct./", it's easy to prove that with the n increasing, the result is become larger and larger, and finally converge to 1. In this problem, when n is larger than 200, result is larger than or equal to 0.99999, that means you can just return 1 when the input is larger than 200. And because 200 is suitable for an O(n^2) solution, we can solve this problem.

---