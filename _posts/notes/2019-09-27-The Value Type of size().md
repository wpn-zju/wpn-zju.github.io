---
layout: post
title: "注意C++容器里的size()"
subtitle:
author: "Peinan"
header-style: text
category: notes
tags:
  - C++
---

所有C++容器的size()都是无符号类型，因此在有时循环要调用size() - 1作为判断条件的时候要注意对size()==0进行分流，否则直接size() - 1会报错。