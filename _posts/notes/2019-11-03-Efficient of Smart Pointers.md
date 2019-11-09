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

```cpp
#include <iostream>
#include <memory>
#include <math.h>
#include <queue>

using namespace std;

class node {
public:
	int value;
	shared_ptr<node> right;
	shared_ptr<node> l_child;
	shared_ptr<node> r_child;
	node() {}
	node(int i) { value = i; }
};

class tree {
public:
	shared_ptr<node> root;
	int level;
	tree() { level = 0; }

	//Implement all member functions below
	tree(int n);//constructor for n-level tree
	//and initilize values as shown in the diagram; 0, 1, 2, ...
	//Note that root node is at level 1 and its value will be initialized to 0

	tree(const tree &T);//copy constructor
	~tree();//destructor
	tree(tree &&T); //move constructor

	tree(const initializer_list<int> &V);//The first number in V is tree level;
	//the rest are values from top to bottom and from left to right
	//For example, to create the tree with n=3 in the diagram,
	//tree T1 = {3, 0,1,2,3,4,5,6}; //where the first 3 is tree level, and the rest are values

	void operator= (const tree &R);//L-value operator=
	void operator= (tree &&R); //R-value operator=


	tree ThreeTimes(); //return a tree with all node value being three times
	friend ostream & operator<<(ostream &str, const tree &T);

	int sum(shared_ptr<node> p);//sum of node values in sub-tree rooted at p
	void delete_level(int i); // Delete nodes at level i. Some nodes at level(s) higher
	//than i will also be deleted accordingly. As described in class.  (Also see the
	//example in the main function.)

	shared_ptr<node> find(int i); //find the first node with value i and return
	//its address; if not found, return nullptr;
};

// In the homework, I use Level-Order Traversal to treverse the tree.
tree::tree(int n) : tree() {
	level = n;
	shared_ptr<node> prev = nullptr;
	queue<shared_ptr<node>> que;
	root = make_shared<node>(0);
	que.push(root);
	while (n--) {
		int size = que.size();
		while (size--) {
			shared_ptr<node> cur = que.front();
			if (prev)
				prev->right = cur;
			prev = cur;
			if (n) {
				que.push(cur->l_child = make_shared<node>(cur->value * 2 + 1));
				que.push(cur->r_child = make_shared<node>(cur->value * 2 + 2));
			}
			que.pop();
		}
	}
	prev->right = root;
}

tree::tree(const tree& that) : tree() {
	level = that.level;
	int n = level;
	shared_ptr<node> prev = nullptr;
	queue<shared_ptr<node>> que1;
	queue<shared_ptr<node>> que2;
	root = make_shared<node>(that.root->value);
	que1.push(root);
	que2.push(that.root);
	while (n--) {
		int size = que1.size();
		while (size--) {
			shared_ptr<node> cur1 = que1.front();
			shared_ptr<node> cur2 = que2.front();
			if (prev)
				prev->right = cur1;
			prev = cur1;
			if (n) {
				que1.push(cur1->l_child = make_shared<node>(cur2->l_child->value));
				que1.push(cur1->r_child = make_shared<node>(cur2->r_child->value));
				que2.push(cur2->l_child);
				que2.push(cur2->r_child);
			}
			que1.pop();
			que2.pop();
		}
	}
	prev->right = root;
}

tree::tree(tree&& that) {
	level = that.level;
	that.level = 0;
	root = that.root;
	that.root = nullptr;
}

void tree::operator=(const tree& that) {
	this->~tree();
	level = that.level;
	int n = level;
	shared_ptr<node> prev = nullptr;
	queue<shared_ptr<node>> que1;
	queue<shared_ptr<node>> que2;
	root = make_shared<node>(that.root->value);
	que1.push(root);
	que2.push(that.root);
	while (n--) {
		int size = que1.size();
		while (size--) {
			shared_ptr<node> cur1 = que1.front();
			shared_ptr<node> cur2 = que2.front();
			if (prev)
				prev->right = cur1;
			prev = cur1;
			if (n) {
				que1.push(cur1->l_child = make_shared<node>(cur2->l_child->value));
				que1.push(cur1->r_child = make_shared<node>(cur2->r_child->value));
				que2.push(cur2->l_child);
				que2.push(cur2->r_child);
			}
			que1.pop();
			que2.pop();
		}
	}
	prev->right = root;
}

void tree::operator=(tree&& that) {
	this->~tree();
	level = that.level;
	that.level = 0;
	root = that.root;
	that.root = nullptr;
}

tree::tree(const initializer_list<int>& V) : tree(*V.begin()) {
	// Replace value with values in the initializer list
	const int* p = V.begin();
	shared_ptr<node> prev = root;
	do {
		prev->value = p[prev->value + 1];
		prev = prev->right;
	} while (prev != root);
}

tree tree::ThreeTimes() {
	tree ret;
	ret.level = level;
	int n = level;
	shared_ptr<node> prev = nullptr;
	queue<shared_ptr<node>> que1;
	queue<shared_ptr<node>> que2;
	ret.root = make_shared<node>(root->value * 3);
	que1.push(ret.root);
	que2.push(root);
	while (n--) {
		int size = que1.size();
		while (size--) {
			shared_ptr<node> cur1 = que1.front();
			shared_ptr<node> cur2 = que2.front();
			if (prev)
				prev->right = cur1;
			prev = cur1;
			if (n) {
				que1.push(cur1->l_child = make_shared<node>(cur2->l_child->value * 3));
				que1.push(cur1->r_child = make_shared<node>(cur2->r_child->value * 3));
				que2.push(cur2->l_child);
				que2.push(cur2->r_child);
			}
			que1.pop();
			que2.pop();
		}
	}
	prev->right = ret.root;
	return ret;
}

tree::~tree() {
	if (!root)
		return;
	shared_ptr<node> cur = root;
	while (cur->right != root)
		cur = cur->right;
	cur->right.reset();
}

ostream& operator<<(ostream& os, const tree &obj) {
	shared_ptr<node> cur = obj.root;
	do {
		os << cur->value << ' ';
		cur = cur->right;
	} while (cur != obj.root);
	return os;
}

int tree::sum(shared_ptr<node> p) {
	int result = 0;
	queue<shared_ptr<node>> que;
	que.push(p);
	while (!que.empty()) {
		shared_ptr<node> cur = que.front();
		if (cur->l_child)
			que.push(cur->l_child);
		if (cur->r_child)
			que.push(cur->r_child);
		result += cur->value;
		que.pop();
	}
	return result;
}

void tree::delete_level(int i) {
	queue<shared_ptr<node>> que;
	que.push(root);
	int levelCount = 0;
	while (!que.empty()) {
		int size = que.size();
		++levelCount;
		while (size--) {
			shared_ptr<node> cur = que.front();
			if (levelCount == i - 1) {
				shared_ptr<node> l = cur->l_child;
				cur->l_child = l->l_child;
				l.reset();
				shared_ptr<node> r = cur->r_child;
				cur->r_child = r->l_child;
				r.reset();
			}
			else {
				if (cur->l_child)
					que.push(cur->l_child);
				if (cur->r_child)
					que.push(cur->r_child);
			}
			que.pop();
		}
	}
	shared_ptr<node> prev = nullptr;
	que.push(root);
	while (!que.empty()) {
		int size = que.size();
		while (size--) {
			shared_ptr<node> cur = que.front();
			if (prev)
				prev->right = cur;
			prev = cur;
			if (cur->l_child)
				que.push(cur->l_child);
			if (cur->r_child)
				que.push(cur->r_child);
			que.pop();
		}
	}
	prev->right = root;
}

shared_ptr<node> tree::find(int i) {
	shared_ptr<node> cur = root;
	while (cur->right != root)
		if (cur->value == i)
			return cur;
		else
			cur = cur->right;
	return cur->value == i ? cur : nullptr;
}
```

Smart Pointer Version

```cpp
#include <iostream>
#include <memory>
#include <math.h>
#include <queue>

using namespace std;

class node {
public:
	int value;
	node* right;
	node* l_child;
	node* r_child;
	node() { right = l_child = r_child = nullptr; }
	node(int i) { right = l_child = r_child = nullptr; value = i; }
};

class tree {
public:
	node* root;
	int level;

	tree() { root = nullptr; level = 0; }

	//Implement all member functions below
	tree(int n);//constructor for n-level tree
	//and initilize values as shown in the diagram; 0, 1, 2, ...
	//Note that root node is at level 1 and its value will be initialized to 0

	tree(const tree &T);//copy constructor
	~tree();//destructor
	tree(tree &&T); //move constructor

	tree(const initializer_list<int> &V);//The first number in V is tree level;
	//the rest are values from top to bottom and from left to right
	//For example, to create the tree with n=3 in the diagram,
	//tree T1 = {3, 0,1,2,3,4,5,6}; //where the first 3 is tree level, and the rest are values

	void operator= (const tree &R);//L-value operator=
	void operator= (tree &&R); //R-value operator=

	tree ThreeTimes(); //return a tree with all node value being three times
	friend ostream & operator<<(ostream &str, const tree &T);

	int sum(node* p);//sum of node values in sub-tree rooted at p
	void delete_level(int i); // Delete nodes at level i. Some nodes at level(s) higher
	//than i will also be deleted accordingly. As described in class.  (Also see the
	//example in the main function.)

	node* find(int i); //find the first node with value i and return
	//its address; if not found, return nullptr;

	void recursiveDelete(node* p);
};

// In the homework, I use Level-Order Traversal to treverse the tree.
tree::tree(int n) : tree() {
	level = n;
	node* prev = nullptr;
	queue<node*> que;
	root = new node(0);
	que.push(root);
	while (n--) {
		int size = que.size();
		while (size--) {
			node* cur = que.front();
			if (prev)
				prev->right = cur;
			prev = cur;
			if (n) {
				que.push(cur->l_child = new node(cur->value * 2 + 1));
				que.push(cur->r_child = new node(cur->value * 2 + 2));
			}
			que.pop();
		}
	}
	prev->right = root;
}

tree::tree(const tree& that) : tree() {
	level = that.level;
	int n = level;
	node* prev = nullptr;
	queue<node*> que1;
	queue<node*> que2;
	root = new node(that.root->value);
	que1.push(root);
	que2.push(that.root);
	while (n--) {
		int size = que1.size();
		while (size--) {
			node* cur1 = que1.front();
			node* cur2 = que2.front();
			if (prev)
				prev->right = cur1;
			prev = cur1;
			if (n) {
				que1.push(cur1->l_child = new node(cur2->l_child->value));
				que1.push(cur1->r_child = new node(cur2->r_child->value));
				que2.push(cur2->l_child);
				que2.push(cur2->r_child);
			}
			que1.pop();
			que2.pop();
		}
	}
	prev->right = root;
}

tree::tree(tree&& that) {
	level = that.level;
	that.level = 0;
	root = that.root;
	that.root = nullptr;
}

void tree::operator=(const tree& that) {
	this->~tree();
	level = that.level;
	int n = level;
	node* prev = nullptr;
	queue<node*> que1;
	queue<node*> que2;
	root = new node(that.root->value);
	que1.push(root);
	que2.push(that.root);
	while (n--) {
		int size = que1.size();
		while (size--) {
			node* cur1 = que1.front();
			node* cur2 = que2.front();
			if (prev)
				prev->right = cur1;
			prev = cur1;
			if (n) {
				que1.push(cur1->l_child = new node(cur2->l_child->value));
				que1.push(cur1->r_child = new node(cur2->r_child->value));
				que2.push(cur2->l_child);
				que2.push(cur2->r_child);
			}
			que1.pop();
			que2.pop();
		}
	}
	prev->right = root;
}

void tree::operator=(tree&& that) {
	this->~tree();
	level = that.level;
	that.level = 0;
	root = that.root;
	that.root = nullptr;
}

tree::tree(const initializer_list<int>& V) : tree(*V.begin()) {
	// Replace value with values in the initializer list
	const int* p = V.begin();
	node* prev = root;
	do {
		prev->value = p[prev->value + 1];
		prev = prev->right;
	} while (prev != root);
}

tree tree::ThreeTimes() {
	tree ret;
	ret.level = level;
	int n = level;
	node* prev = nullptr;
	queue<node*> que1;
	queue<node*> que2;
	ret.root = new node(root->value * 3);
	que1.push(ret.root);
	que2.push(root);
	while (n--) {
		int size = que1.size();
		while (size--) {
			node* cur1 = que1.front();
			node* cur2 = que2.front();
			if (prev)
				prev->right = cur1;
			prev = cur1;
			if (n) {
				que1.push(cur1->l_child = new node(cur2->l_child->value * 3));
				que1.push(cur1->r_child = new node(cur2->r_child->value * 3));
				que2.push(cur2->l_child);
				que2.push(cur2->r_child);
			}
			que1.pop();
			que2.pop();
		}
	}
	prev->right = ret.root;
	return ret;
}

tree::~tree() {
	if (!root)
		return;
	node* prev = nullptr;
	node* cur = root;
	do {
		prev = cur;
		cur = cur->right;
		delete(prev);
	} while (cur != root);
}

ostream& operator<<(ostream& os, const tree &obj) {
	node* cur = obj.root;
	do {
		os << cur->value << ' ';
		cur = cur->right;
	} while (cur != obj.root);
	return os;
}

int tree::sum(node* p) {
	int result = 0;
	queue<node*> que;
	que.push(p);
	while (!que.empty()) {
		node* cur = que.front();
		if (cur->l_child)
			que.push(cur->l_child);
		if (cur->r_child)
			que.push(cur->r_child);
		result += cur->value;
		que.pop();
	}
	return result;
}

void tree::recursiveDelete(node* p) {
	if (p->l_child)
		recursiveDelete(p->l_child);
	if (p->r_child)
		recursiveDelete(p->r_child);
	delete(p);
}

void tree::delete_level(int i) {
	queue<node*> que;
	que.push(root);
	int levelCount = 0;
	while (!que.empty()) {
		int size = que.size();
		++levelCount;
		while (size--) {
			node* cur = que.front();
			if (levelCount == i - 1) {
				node* l = cur->l_child;
				cur->l_child = l->l_child;
				if (l->r_child)
					recursiveDelete(l->r_child);
				delete(l);
				node* r = cur->r_child;
				cur->r_child = r->l_child;
				if (r->r_child)
					recursiveDelete(r->r_child);
				delete(r);
			}
			else {
				if (cur->l_child)
					que.push(cur->l_child);
				if (cur->r_child)
					que.push(cur->r_child);
			}
			que.pop();
		}
	}
	node* prev = nullptr;
	que.push(root);
	while (!que.empty()) {
		int size = que.size();
		while (size--) {
			node* cur = que.front();
			if (prev)
				prev->right = cur;
			prev = cur;
			if (cur->l_child)
				que.push(cur->l_child);
			if (cur->r_child)
				que.push(cur->r_child);
			que.pop();
		}
	}
	prev->right = root;
}

node* tree::find(int i) {
	node* cur = root;
	while (cur->right != root)
		if (cur->value == i)
			return cur;
		else
			cur = cur->right;
	return cur->value == i ? cur : nullptr;
}
```

Test Code

```cpp
#include <ctime>
int main() {
	int m = 10;
	while (m--) {
		int n = 10000;
		clock_t begin, end;
		begin = clock();
		while (n--) {
			tree T1(3);
			tree T2 = { 4, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24 };
			tree T3(T2);
			tree T4;
			T4 = T3;
			T4 = T3.ThreeTimes();
			T4.delete_level(3);
			T3.sum(T3.find(12));
		}
		end = clock();
		cout << (end - begin) / 1000.0 << endl;
	}

	getchar();
	getchar();
	return 0;
}
```

Benchmark, with 1000 loop and 10 tests.

Raw Pointer Version

Time / Second

![](/res/notes/esp_2.png)

Smart Pointer Version

Time / Second

![](/res/notes/esp_3.png)

---