---
layout: post
title: "LeetCode - Multiset and Sliding Window - Advanced STL Using"
subtitle:
author: "Peinan"
header-style: text
category: leetcode
tags:
  - Algorithms
---

[Problem Here](https://leetcode.com/problems/sliding-window-median/)

Solution

```cpp
class Solution {
public:
	vector<double> medianSlidingWindow(vector<int>& nums, int k) {
		vector<double> result;
		multiset<int> s(nums.begin(), nums.begin() + k);
		multiset<int>::iterator it = next(s.begin(), k / 2);
		for (int i = k;; ++i) {
			if (k & 1)
				result.push_back(*it);
			else
				result.push_back(((double)(*it) + *prev(it)) / 2);

			if (i == nums.size())
				break;

            s.insert(nums[i]);
            if (nums[i] < *it)
				--it;	
			if (nums[i - k] <= *it)
				++it;
			s.erase(s.lower_bound(nums[i - k]));
		}
		return result;
	}
};
```

---