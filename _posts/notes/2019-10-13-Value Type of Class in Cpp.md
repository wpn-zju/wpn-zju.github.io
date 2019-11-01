---
layout: post
title: "C++的万恶之源 - 允许类以值类型出现"
subtitle:
author: "Peinan"
header-style: text
category: notes
tags:
  - C++
---

最近C++课老本已经吃得差不多了，感觉这个问题一定要写个post。

个人认为C++有一个很大的特点，Java/C#等语言可能学70%语法可以去做开发，而C++可能只需要30%（极端点说，只学C语言那部分语法也不是不能用，C++的关键词个数是C语言的两倍多）。

这是因为C++有很多的冗余语法，其中引用/右值引用/移动构造就是一个，还有各种花里胡哨的初始化方法（大括号/小括号），实际上完全可以用指针来代替

拿移动构造来说，移动构造的本意是避免值类型在赋值的时候拷贝的性能损耗，但如果我们用指针，就完全没有拷贝的问题。指针赋值即浅拷贝（深拷贝则写拷贝构造函数）。

```cpp
A b;
// 此处省略一万行
A a = move(b);	// 这里调用移动构造函数避免拷贝性能损耗
// 上面完全可以写成
A* b = new A();
// 此处省略一万行
A* a = b;
b = nullptr;
// 这样一来，不就能少一个语法了吗。
```

值类型的原罪就在拷贝，可以说是成也拷贝，败也拷贝。自带初始化这一点省去了Java和C#里必须要new的步骤，但是值类型作为参数、作为返回值、作为局部变量的生命周期真是搞得一大波人云里雾里。举个例子，如果你写了下面这个操蛋的函数

```cpp
A myFunc(A a){
	A temp = a;
	// 此处省略一万步
	return temp;
}
```

返回值、传入参数、中间变量全部都是值类型，那么恭喜你，这中间进行了“无数次”的拷贝

1. 参数传入时拷贝1次

2. 参数传出时拷贝1次

3. 临时变量赋值拷贝1次

如果只是这样的话其实除了性能低点也没啥问题（这里插一句LeetCode很多题把值类型改成指针或引用效率有质的飞跃），关键是有时候你函数体中间还有一些指针变量或者外部变量，比如下面这个样子

```cpp
class D {
public:
	vector<int>* p = nullptr;

	D() {
		p = new vector<int>();
	}
};

D myFunc(D d) {
	D temp = d;
	*temp.p = *d.p;
	return temp;
}

int main() {
	D d;
	delete(d.p);
	d.p = new vector<int>{ 1,2,3,4,5 };
	D d2 = myFunc(d);
	for (int& i : *d2.p)
		cout << i << ' ';
	return 0;
}
```

这段代码除了拷贝次数多一点，是没有functional问题的，正确输出12345后结束。

```cpp
class D {
public:
	vector<int>* p = nullptr;

	D(const D& that) : D() {
		
	}

	D() {
		p = new vector<int>();
	}
};

D myFunc(D d) {
	D temp = d;
	*temp.p = *d.p;
	return temp;
}

int main() {
	D d;
	delete(d.p);
	d.p = new vector<int>{ 1,2,3,4,5 };
	D d2 = myFunc(d);
	for (int& i : *d2.p)
		cout << i << ' ';
	return 0;
}
```

这里我们除了加了一个空的Copy Constructor以外啥都没干，然后你会发现什么都没输出。

以下是正确的写法

```cpp
class D {
public:
	vector<int>* p = nullptr;

	D(const D& that) : D() {
		*p = *that.p;
	}

	D() {
		p = new vector<int>();
	}
};

D myFunc(D d) {
	D temp = d;
	*temp.p = *d.p;
	return temp;
}

int main() {
	D d;
	delete(d.p);
	d.p = new vector<int>{ 1,2,3,4,5 };
	D d2 = myFunc(d);
	for (int& i : *d2.p)
		cout << i << ' ';
	return 0;
}
```

基于上一个正确的写法，那么如果写成下面这样呢

```cpp
class D {
public:
	vector<int>* p = nullptr;

	D(const D& that) : D() {
		*p = *that.p;
	}
	
	// Move Ctor不需要调用Default Ctor这里为了不出现空指针
	D(D&& that) : D() {

	}

	D() {
		p = new vector<int>();
	}
};

D myFunc(D d) {
	D temp = d;
	*temp.p = *d.p;
	return temp;
}

int main() {
	D d;
	delete(d.p);
	d.p = new vector<int>{ 1,2,3,4,5 };
	D d2 = myFunc(d);
	for (int& i : *d2.p)
		cout << i << ' ';
	return 0;
}
```

啥都没干，只是加了个空的移动构造函数而已。输出结果为空。

正确写法如下

```cpp
class D {
public:
	vector<int>* p = nullptr;

	D(const D& that) : D() {
		*p = *that.p;
	}

	D(D&& that) : D() {
		*p = *that.p;
	}

	D() {
		p = new vector<int>();
	}
};

D myFunc(D d) {
	D temp = d;
	*temp.p = *d.p;
	return temp;
}

int main() {
	D d;
	delete(d.p);
	d.p = new vector<int>{ 1,2,3,4,5 };
	D d2 = myFunc(d);
	for (int& i : *d2.p)
		cout << i << ' ';
	return 0;
}
```



.......

这里其实有一个C++非常坑的点。英文原文如下

When used as a function argument and when two overloads of the function are available, one taking rvalue reference parameter and the other taking lvalue reference to const parameter, an rvalue binds to the rvalue reference overload (thus, if both copy and move constructors are available, an rvalue argument invokes the move constructor, and likewise with copy and move assignment operators).

对于定义了移动构造函数的情况下，实参为右值匹配到移动构造函数，而非拷贝构造函数，最后得到的d2是空的，内存跟踪结果为d2和temp地址不同。

所以对于第四个空移动构造函数的情况，弹出时候调用的是这个空移动构造函数进行拷贝

对于未定义移动构造函数，但定义了拷贝构造函数的情况下，实参为右值匹配到拷贝构造函数

所以对于第二个空拷贝构造函数的情况，弹出时候调用的是这个空拷贝构造函数进行拷贝，最后得到的d2是空的，内存跟踪结果为d2和temp地址不同。

那么问题来了，为什么第一个啥都没写也是对的呢？

这就涉及到缺省定义的问题，当什么都没申明的时候，实际上是内置了默认的移动构造函数和拷贝构造函数。那么这种情况下等同于都定义的情况，调用的就是默认的那个移动构造函数。（因此对第一段程序进行内存跟踪可以发现，d2的地址和temp相同。）

但撇开这个问题，假如C++和Java一样不允许值类型的类出现，那也不会有这个问题，传入参数和返回值强制变成了指针。可参考以下C++代码

```cpp
class D {
public:
	vector<int>* p = nullptr;

	D() {
		p = new vector<int>();
	}
};

D* myFunc(D* d) {
	D* temp = d;
	*(temp->p) = *(d->p);
	return temp;
}

int main() {
	D* d1 = new D();
	delete(d->p);
	d1->p = new vector<int>{ 1,2,3,4,5 };
	D* d2 = myFunc(d1);
	for (int& i : *(d2->p))
		cout << i << ' ';
	return 0;
}
```

正常运行并输出12345，而且全部用指针完全没有拷贝的损耗问题，注意d1/d2/temp和参数d全部指向同一地址，如需深拷贝需要修改。

```
延申学习 - C++开发的“三/五法则”

C++11之前 三法则

对于带指针的类，原则上必须写

1. 默认构造函数、

2. 拷贝构造函数

3. 左值等于

C++11以及之后 五法则

对于带指针的类，原则上必须写

1. 默认构造函数

2. 拷贝构造函数

3. 移动构造函数	C++11加入

4. 左值等于

5. 右值等于	C++11加入

为什么要全部写上，其实就是不想让默认行为捣乱。（C++11也引入了=delete和=default来解决默认函数的行为问题，实际上和自己写也差不多，毕竟首先你要能想到。）
```

很多学C++的同学学了大几个月都不能理解，为什么他们申明的变量找不到了。很多人一直都没法理解堆内存和栈内存的问题，以值类型申明的变量，跳出作用域即被回收，这才是问题的关键，但同时也有C++允许类以值类型出现的锅。

在Java和C#中，所有的class本质上都是一个class指针，同时必须初始化。

如果可以的话真希望C++禁止类以值类型出现（然而这不可能，C++是兼容为王），强制使用指针类型，那样一来实际上就类似不带内存回收的Java和C#了

但这样对C++的效率其实没多大影响，因为本身Java和C#就是慢在内存回收，但是Memory Leak的问题也会加重（需要智能指针）。

有时候真的觉得，学C++所谓的难，实际上是学这些冗余语法的难，明明不是自己的style却必须都学一遍，没办法考试要考运行结果啊。这也许就是“兼容”两字的承重吧。

---