---
layout: post
title: "C++值类型回顾"
subtitle:
author: "Peinan"
header-style: text
category: posts
tags:
  - C++
---

今天做项目的时候遇到一个关于函数签名的问题，顺便回顾一下C++值类型在函数中的使用（返回值、形参）。右值引用和移动语义的引入可以说是C++11的精华。

需求是我的服务端拥有一个统一的Send函数用来发送UDP包，该函数有两个参数，第一个为代表接收端的uint32_t类型的ID，第二个参数是一个json格式的数据包。问题就在第二个参数的传递方式上。

通常来说，传参有四种方式，传值、传（左值）引用、传右值引用、传常量（左值）引用，依次对应下列四行代码。

```cpp
void send(uint32_t peerId, json packet);
void send(uint32_t peerId, json& packet);
void send(uint32_t peerId, json&& packet);
void send(uint32_t peerId, const json& packet); // 比较特殊的是左值引用加上const后支持传递右值实参。

// 其他两种
void send(uint32_t peerId, const json packet); // 不常用，值类型传递自带拷贝，该const只对形参packet起作用，对外部实参无任何作用。
void send(uint32_t peerId, const json&& packet); // 类似上一个，需要对右值引用形参加const的场景不多。
```

针对这个项目，还有几个特殊的点。
+ 为了统计响应速度，我希望在每个UDP包中加上UNIX时间戳，贴时间戳的最好方式毫无疑问是在send函数中，因为我们调用该函数的地方可能多达二三十处，如果在外部定义，将会产生很多冗余代码。
+ 我希望能够利用移动语义和右值传递尽可能地减少json对象拷贝的次数。
+ 在下层代码中，存在构建json对象，再通过for循环调用send函数的广播情况。
+ 为了简化代码，我的json库包含了移动构造和拷贝构造。例如，我们可以简单地通过传递`{ 1 }`来构建一个仅包含数字1的json对象。
+ json包的构建往往分多层进行，例如负责发送对局状态的函数send_state会调用游戏实例TableInstance的capture方法来截取以json格式保存的游戏状态，而TableInstance的capture方法则会调用Player的capture方法来依次截取每个玩家的状态。这些capture函数的返回值均设置为值类型，如下式。（返回值为值类型的返回表达式被视为纯右值，可以绑定到上面除了左值引用的其它三个函数签名。）
  
```cpp
json TableInstance::capture() const {
  return { unordered_map<string, json> {
      { "player_1", player_1.capture() },
      { "player_2", player_2.capture() },
      { "player_3", player_3.capture() },
      { "player_4", player_4.capture() },
  } };
}
```

send的方式往往有三种，传左值、亡值或纯右值
```cpp
json data { something };
send(peerId, data); // lvalue;
```

```cpp
json data { something };
send(peerId, std::move(data)); // xvalue;
```

```cpp
send(peerId, { something }); // prvalue;
```

以上几种签名方式可以说各有利弊。
1. 使用值类型传递。
  + 优点 - 当我们使用xvalue或prvalue传参的时候，无任何拷贝开销。
  + 优点 - 非const，可以在send内部修改、打时间戳。
  + 缺点 - 当我们使用lvalue传参的时候，需要拷贝。使用lvalue传参多用于需要for循环发包的情况，原义是为了使用局部变量避免多次调用下层函数如capture等。
2. 使用左值引用传递。
  + 优点 - 当我们使用lvalue或xvalue传参的时候，无任何拷贝开销。
  + 优点 - 非const，可以在send内部修改、打时间戳。
  + 缺点 - 不支持传递rvalue，这样一来我们每次调用send函数都必须显式定义一个json局部变量。
3. 使用右值引用传递。
  + 优点 - 当我们使用xvalue或prvalue传参的时候，无任何拷贝开销。
  + 优点 - 非const，可以在send内部修改、打时间戳。
  + 缺点 - 不支持传递lvalue。虽然我们可以使用`std::move`将左值转为xvalue，但是由于传递xvalue时调用移动构造函数会导致原对象重置，因此无法和for循环发送并存，一般不使用xvalue传递。也就是说为了支持for循环使用单个json对象，必须支持直接传递lvalue。
4. 使用常量引用传递（未加入时间戳时的方案）
  + 优点 - 完美支持全部三种传递方式。注 - 虽然常引用可以绑定右值引用，但当右值引用和常引用形参函数签名并存时，优先匹配到右值引用的函数签名，参考同时存在拷贝构造函数与移动构造函数时传递右值。
  + 缺点 - const，无法在send内部修改、打时间戳。

概括一下就是，左值引用形参不支持右值 - 右值引用形参不支持左值，能传亡值但会重置原对象 - 常引用形参没法打时间戳 - 值类型形参有拷贝开销。



暂定解决方案

定义两个send函数，分别对应传左值引用和传右值引用。缺点是我的下层代码需要send作为回调，如此我可能需要传递两个回调函数。

这个是有关形参类型的，下面讨论一下返回值的类型。以capture函数为例。
一般来说返回值类型有以下几种。

```cpp
json capture() const;
const json capture() const;
json& capture() const;
const json& capture() const;
json&& capture() const;
const json&& capture() const;
```

值类型返回值适合返回函数内部的局部变量，用于构造对象。返回值类型的表达式被视为纯右值表达式，可用于移动构造。

左值引用和常左值引用由于不宜返回函数内部的局部变量（造成悬挂引用），多用于类成员函数或对形参的链式调用，如重载流运算符（对引用参数）和链表节点自遍历、迭代器（对类成员）- 类似用引用形式`node.next().next().next()`代替指针形式的`node->next()->next()->next()`。

返回右值引用的函数 - 我用得比较少，返回右值引用和返回值类型效果大致相同，差异可以参考[Returning by value vs returning by rvalue](http://www.cplusplus.com/forum/general/219121/)，返回值类型更加安全，待补充。

另 - 模板形参为`T&&`时，如果需要引用类型推断（即无同名、形参为`T&`的函数）时，`T&&`代表万能引用而非普通的右值引用，此时`T&&`可以同时绑定左值和右值，详情请查阅万能引用、引用折叠与完美转发。

另 - [为何常引用要设计为同时支持绑定左值引用和绑定右值引用](https://www.zhihu.com/question/40238995)

---
