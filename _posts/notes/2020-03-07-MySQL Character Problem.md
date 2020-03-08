---
layout: post
title: "MySQL Character Problem"
subtitle:
author: "Peinan"
header-style: text
category: notes
tags:
  - MySQL
---

今天使用MySQL时发现无法使用jdbc插入中文等UTF8字符，尝试了修改MySQL设置中的编码选项，但仍无法显示。

解决方法：MySQL不仅本身有默认编码格式，而且每个database，每张table，每个字段，都有默认编码格式，虽然修改了全局的默认编码，但是由于之前的数据仍然使用的是非UTF格式，因此仍然不能显示。手动设置全部数据格式为UTF8MB4后可以正确显示UTF8字符。

另外，MySQL中的UTF8为UTF8MB4。

---