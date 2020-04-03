---
layout: post
title: "Cpp Class Memory"
subtitle:
author: "Peinan"
header-style: text
category: notes
tags:
 - C++
---

类、结构体大小计算 内存布局

大小计算原则如下
+ 空类不为0字节 取决于编译器实现 通常为1字节
+ 成员大小累加 注意对齐
+ 带基类的加上整个基类大小
+ 只要存在虚函数 需加上虚表指针 大小同一个指针变量
+ 只要存在虚继承 需加上虚基类指针 大小同一个指针变量
+ 一个类最多只有一个虚函数指针和一个虚基类指针 注意计算基类时也要考虑基类是否含有这两种指针

内存布局策略
+ 对于成员变量 声明在前的放在低地址 声明在后的放在高地址
+ 对于非虚继承 基类包含在子类内存单元中 且基类整体处于低地址
+ 对于多个非虚继承 基类包含在子类内存单元中 先声明的非虚基类位于低地址 后声明的非虚基类位于高地址
+ 对于虚继承 虚继承指针位于虚函数指针（可能无）后
+ 当类存在虚函数时 既需要包含虚函数指针时 虚函数指针位于类的最低地址
+ 类的静态变量和全部的函数（静态/非静态）均使用类外空间 故不计入类大小
+ 总览 低地址在上 高地址在下
  + 虚函数指针
  + 虚基类指针
  + 非虚基类对象按照申明顺序排放
  + 本类成员按照申明顺序排放
  + 虚基类对象 虚基类对象满足虚拟层级高的在低地址 即虚基类的虚基类在前

复杂类型举例
```cpp
class c1 {
	int c;

	virtual void cf() {};
};

class d1 : public virtual c1 {
	int d;

	virtual void df() {};
};

class e1 : public virtual c1 {
	int e;

	virtual void ef() {};
};

class f1 : public virtual d1, public virtual e1 {
	int f;

	virtual void ff() {};
};

class g1 : public virtual d1, public virtual e1 {
	int g;

	virtual void gf() {};
};

class h1 : public f1, public g1 {
	int h;

	void hf() {};
};
```

该例的布局预览 （使用Visual Studio工具）

```
class h1        size(60):
        +---
 0      | +--- (base class f1)
 0      | | {vfptr}
 4      | | {vbptr}
 8      | | f
        | +---
12      | +--- (base class g1)
12      | | {vfptr}
16      | | {vbptr}
20      | | g
        | +---
24      | h
        +---
        +--- (virtual base c1)
28      | {vfptr}
32      | c
        +---
        +--- (virtual base d1)
36      | {vfptr}
40      | {vbptr}
44      | d
        +---
        +--- (virtual base e1)
48      | {vfptr}
52      | {vbptr}
56      | e
        +---

h1::$vftable@f1@:
        | &h1_meta
        |  0
 0      | &f1::ff

h1::$vftable@g1@:
        | -12
 0      | &g1::gf

h1::$vbtable@f1@:
 0      | -4
 1      | 24 (h1d(f1+4)c1)
 2      | 32 (h1d(f1+4)d1)
 3      | 44 (h1d(f1+4)e1)

h1::$vbtable@g1@:
 0      | -4
 1      | 12 (h1d(g1+4)c1)
 2      | 20 (h1d(g1+4)d1)
 3      | 32 (h1d(g1+4)e1)

h1::$vftable@c1@:
        | -28
 0      | &c1::cf

h1::$vftable@d1@:
        | -36
 0      | &d1::df

h1::$vbtable@d1@:
 0      | -4
 1      | -12 (h1d(d1+4)c1)

h1::$vftable@e1@:
        | -48
 0      | &e1::ef

h1::$vbtable@e1@:
 0      | -4
 1      | -24 (h1d(e1+4)c1)
vbi:       class  offset o.vbptr  o.vbte fVtorDisp
              c1      28       4       4 0
              d1      36       4       8 0
              e1      48       4      12 0
```

注解
+ 虚基类c1
  + 虚函数指针 vfptr 用来匹配虚函数 cf
  + 成员 c
+ 虚基类d1
  + 虚函数指针 vfptr 用来匹配虚函数 df
  + 虚基类指针 vbptr 用来匹配虚基类 c1
  + 成员 d
+ 虚基类e1
  + 虚函数指针 vfptr 用来匹配虚函数 ef
  + 虚基类指针 vbptr 用来匹配虚基类 c1
  + 成员 e
+ 基类f1
  + 虚函数指针 vfptr 用来匹配虚函数 ff
  + 虚基类指针 vbptr 用来匹配虚基类 c1 d1 e1
  + 成员 f
+ 基类g1
  + 虚函数指针 vfptr 用来匹配虚函数 gf
  + 虚基类指针 vbptr 用来匹配虚基类 c1 d1 e1
  + 成员 g
+ 类h1
  + 非虚基类
    + 非虚基类 f1
    + 非虚基类 g1
  + 成员 h
  + 虚基类 c1
  + 虚基类 d1
  + 虚基类 e1

h1类总大小(32-bit) 60 bytes = 5 * vfptr(4 bytes) + 4 * vbptr(4 bytes) + 6 * int(4 bytes) 

---