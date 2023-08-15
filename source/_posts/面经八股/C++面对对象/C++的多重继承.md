---
title: C++的多重继承
date: 2023-05-22 00:00:00
tags:
updated:
categories:
- 面经八股
- C++面对对象
keywords:
description: 派生类只有一个基类，称为单继承（Single Inheritance）。除此之外，C++也支持多继承（Multiple Inheritance）。
cover: https://s2.loli.net/2023/08/15/Q3J85MH76aoLvAc.jpg
---

# 多继承

派生类只有一个基类，称为单继承（Single Inheritance）。除此之外，C++也支持多继承，这是面向对象编程的一个重要特性之一。

多继承（Multiple Inheritance）指的是一个类可以从多个基类派生从而继承多个基类的属性和方法，即一个派生类可以有两个或多个直接基类。多继承的派生类继承了所有可继承的父类成员，这允许我们可以在一个类中组合不同的功能和行为，以便更好地模拟现实世界的对象关系和交互。

多继承的语法很简单，将多个基类用逗号隔开即可。例如已声明类 A、类 B 和类 C，那么可以这样来声明派生类 D:
``` C++
class D : public A, private B, protected C {
    /*...*/
};
```
类 D 是多继承形式的派生类，它分别以公有、私有、保护的方式继承类 A、类 B、类 C。类 D 根据不同的继承方式获取 A、B、C 中的成员，确定它们在派生类中的访问权限。

# 多继承下的构造函数

多继承形式下的构造函数和单继承形式基本相同，只要在派生类的构造函数中调用多个基类的构造函数。

以上面的 A、B、C、D 类为例，类 D 的构造函数写法为：

``` C++
class D : public A, private B, protected C {
    /*...*/
public:
    D(/*形参列表*/) A(/*实参列表*/), B(/*实参列表*/), C(/*实参列表*/) {
        /*...*/
    }
    /*...*/
};
```

基类构造函数的调用顺序和它们在派生类构造函数中出现的顺序无关，而是**和声明派生类时基类出现的顺序相同**。仍然以上面的 A、B、C、D 类为例，即使类 D 的构造函数写为下面的形式，基类构造函数的调用顺序依然为 A、B、C 。

``` C++
class D : public A, private B, protected C {
    /*...*/
public:
    D(/*形参列表*/) B(/*实参列表*/), C(/*实参列表*/), A(/*实参列表*/) {
        /*...*/
    }
    /*...*/
};
```

下面是一个多继承的实例：
``` C++
#include <iostream>
using namespace std;

class BaseA {
public:
	BaseA(int a, int b) : m_a(a), m_b(b) {
		cout << "BaseA constructor" << endl;
	}
	~BaseA() {
		cout << "BaseA destructor" << endl;
	}
protected:
	int m_a, m_b;
};

class BaseB {
public:
	BaseB(int c, int d) : m_c(c), m_d(d) {
		cout << "BaseB constructor" << endl;
	}
	~BaseB() {
		cout << "BaseB destructor" << endl;
	}
protected:
	int m_c, m_d;
};

class Derived : public BaseA, public BaseB {
public:
	Derived(int a, int b, int c, int d, int e) : BaseA(a, b), BaseB(c, d), m_e(e) {
		cout << "Derived constructor" << endl;
	}
	~Derived() {
		cout << "Derived destructor" << endl;
	}
	void show() {
		cout << m_a << " " << m_b << " " << m_c << " " << m_d << " " << m_e << endl;
	}
private:
	int m_e;
};

int main() {
	Derived obj(1, 2, 3, 4, 5);
	obj.show();
	return 0;
}
```
> 运行结果
> BaseA constructor
> BaseB constructor
> Derived constructor
> 1 2 3 4 5
> Derived destructor
> BaseB destructor
> BaseA destructor

从运行结果可看出，多继承形式下析构函数的执行顺序和构造函数的执行顺序也是相反的。

# 命名冲突

需要注意的是，多继承可能会引入一些复杂性和歧义，特别是当多个基类具有相同的成员名称时（命名冲突）。

> 多继承容易让代码逻辑复杂、思路混乱，一直备受争议，中小型项目中较少使用，后来的 Java、C#、PHP 等干脆取消了多继承。

当继承的多个基类中有同名的成员时，如果直接访问该成员，就会产生命名冲突，编译器不知道使用哪个基类的成员。这个时候需要在成员名字前加上类名和域解析符 `::`，以显式指明到底使用哪个类的成员，消除二义性。

修改上面的代码，为 BaseA 和 BaseB 类添加 show() 函数，并将 Derived 类的 show() 函数更名为 display()：
``` C++
#include <iostream>
using namespace std;

class BaseA {
public:
	BaseA(int a, int b) : m_a(a), m_b(b) {
		cout << "BaseA constructor" << endl;
	}
	~BaseA() {
		cout << "BaseA destructor" << endl;
	}
	void show() {
		cout << m_a << " " << m_b << " ";
	}
protected:
	int m_a, m_b;
};

class BaseB {
public:
	BaseB(int c, int d) : m_c(c), m_d(d) {
		cout << "BaseB constructor" << endl;
	}
	~BaseB() {
		cout << "BaseB destructor" << endl;
	}
	void show() {
		cout << m_c << " " << m_d << " ";
	}
protected:
	int m_c, m_d;
};

class Derived : public BaseA, public BaseB {
public:
	Derived(int a, int b, int c, int d, int e) : BaseA(a, b), BaseB(c, d), m_e(e) {
		cout << "Derived constructor" << endl;
	}
	~Derived() {
		cout << "Derived destructor" << endl;
	}
	void display() {
		// 否则： [Error] reference to 'show' is ambiguous
		BaseA::show();
		BaseB::show();
		cout << m_e << endl;
	}
private:
	int m_e;
};

int main() {
	Derived obj(1, 2, 3, 4, 5);
	obj.display();
	return 0;
}
```

以上，我们使用作用域解析运算符解决直接的命名冲突问题。

关于多重继承下的菱形继承问题和虚继承的解决方案，之后再写。

# 参考

[C++多继承（多重继承）详解](http://c.biancheng.net/view/2277.html)

[ChatGPT](https://chat.openai.com/)