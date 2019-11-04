---
layout: post
title: "LeetCode - Interval Problem - Count Map"
subtitle:
author: "Peinan"
header-style: text
category: leetcode
tags:
  - Progress
---

[Problem 1](https://leetcode.com/problems/my-calendar-i/)

Solution1 - Using Binary Search O(nlogn) Time Complexity

```cpp
class MyCalendar {
public:
	set<pair<int, int>> intervals;
	MyCalendar() {
		intervals.insert({ INT_MIN, INT_MIN + 1 });
		intervals.insert({ INT_MAX - 1,INT_MAX });
	}

	bool book(int s, int e) {
		auto index = intervals.lower_bound({ s,e });
		if (index->first < e || (--index)->second > s)
			return false;
		intervals.insert({ s, e });
		return true;
	}
};
```

Solution2 - Using Sorted Set O(n^2) Time Complexity

```cpp
class MyCalendar {
public:
	map<int, int> count;

	bool book(int s, int e) {
		++count[s];
		--count[e];
		int sum = 0;
		for (pair<int, int> p : count) {
			sum += p.second;
			if (sum > 1) {
				--count[s];
				++count[e];
				return false;
			}
		}
		return true;
	}
};
```

---

[Problem 2](https://leetcode.com/problems/my-calendar-ii/)

Solution

```cpp
class MyCalendarTwo {
public:
	map<int, int> count;

	bool book(int s, int e) {
		++count[s];
		--count[e];
		int sum = 0;
		for (pair<int, int> p : count) {
			sum += p.second;
			if (sum > 2) {
				--count[s];
				++count[e];
				return false;
			}
		}
		return true;
	}
};
```

---

[Problem 3](https://leetcode.com/problems/my-calendar-iii/)

Solution

```cpp
class MyCalendarThree {
public:
	map<int, int> count;

	int book(int s, int e) {
		++count[s];
		--count[e];
		int result = 0;
		int sum = 0;
		for (pair<int, int> p : count)
			result = max(result, sum += p.second);
		return result;
	}
};
```

---



---