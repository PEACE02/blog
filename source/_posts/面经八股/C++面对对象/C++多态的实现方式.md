---
title: C++多态的实现方式
date: 2023-05-16 00:00:00
tags:
updated:
categories:
- 面经八股
- C++面对对象
keywords:
description: C++中的多态是指同一个函数或者操作，在不同的对象上有不同的表现形式。
cover: https://s2.loli.net/2023/08/02/k1P2HcOM9UfDhAL.jpg
---

C++中的多态是指同一个函数或者操作，在不同的对象上有不同的表现形式。

**C++实现多态的方法主要包括虚函数、纯虚函数、模板函数。**

其中虚函数、纯虚函数实现的多态叫**动态多态**；模板函数、重载等实现的叫**静态多态**。

区分静态多态和动态多态的一个方法是：看所调用的具体方法是在编译期还是运行时决定，编译期是静态多态，运行时是动态多态。


## 虚函数、纯虚函数实现多态

在 C++ 中，可以使用虚函数来实现多态性。虚函数是指基类中声明的函数，在派生类中可以被重写（override）。

当我们使用基类指针或引用指向派生类对象时，通过虚函数的机制，可以调用到派生类重写的函数，从而实现多态。

C++多态必须满足两个条件：
1. **必须通过基类的指针或引用调用函数**
2. **被调用的函数是基类的虚函数，且派生类必须完成对该虚函数的重写**

举例说明：一个基类 Shape，两个派生类 Rectangle、Triangle 分别继承 Shape 。

``` C++
#include <iostream>
using namespace std;

class Shape {
public:
	virtual void printArea() = 0;
};

class Rectangle : public Shape {
	int width, height;
public:
	Rectangle(int w, int h) : width(w), height(h) {}
	void printArea() {
		cout << "Rectangle class area: " << width * height << endl;
	}
};

class Triangle : public Shape {
	int width, height;
public:
	Triangle(int w, int h) : width(w), height(h) {}
	void printArea () {
		cout << "Triangle class area: " << (width * height) / 2.0 << endl;
	}
};

int main() {
	Shape *shape = new Rectangle(4, 6);
	shape->printArea();
	delete shape;
	
	shape = new Triangle(4, 6);
	shape->printArea();
	delete shape;
	
	shape = nullptr;
	return 0;
}
```
执行程序后可以看到，我们使用基类指针调用了派生类重写的函数。


## 模板函数多态

模板函数可以根据传递参数的不同类型，自动生成相应类型的函数代码。模板函数可以实现多态。

举例说明：实现一个模板函数 getMax()，接收相同类型的两个参数，返回较大的一个。

``` C++
#include <iostream>
using namespace std;

template <class T>
T getMax(T x, T y) {
	return x > y ? x : y;
}

int main() {
	double a = 10.1, b = 10.2;
	cout << "int: " << getMax<int>(a, b) << endl;		// int 截断 
	cout << "double: " << getMax<double>(a, b) << endl;
	return 0;
}
```

在上面的例子中，编译器会生成两个 getMax() 的函数示例，一个是 int，一个是 double，这种调用的函数在编译期就能确定下来的叫静态多态。函数重载也是静态多态。