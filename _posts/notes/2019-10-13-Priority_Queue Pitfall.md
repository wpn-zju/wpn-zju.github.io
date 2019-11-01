---
layout: post
title: "C++ 优先队列的陷阱"
subtitle:
author: "Peinan"
header-style: text
category: notes
tags:
  - C++
  - Priority Queue
---

C++优先队列的元素不适用直接修改来达到重新排列的目的，必须要先pop再push来实现。

（目测应该是仅在pop和push函数调用时会调用heap重排）

---