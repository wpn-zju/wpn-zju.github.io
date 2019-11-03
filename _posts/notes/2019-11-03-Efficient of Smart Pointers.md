---
layout: post
title: "Efficiency Smart Pointers vs Raw Pointers"
subtitle:
author: "Peinan"
header-style: text
category: notes
tags:
  - C++
---

One of our C++ class homework for practicing smart pointers. I implemented both smart pointer version and raw pointer version.

The basic problem is to write a binary tree data structure with \"next\" pointers.

For example,

![](/res/notes/esp_1.png)

Raw Pointer Version

Smart Pointer Version

Benchmark, with 1000 loop and 10 tests.

Raw Pointer Version

Time / Second

![](/res/notes/esp_2.png)

Smart Pointer Version

Time / Second

![](/res/notes/esp_3.png)

---