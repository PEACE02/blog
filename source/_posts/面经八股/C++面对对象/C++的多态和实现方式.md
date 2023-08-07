---
title: C++的多态和实现方式
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

# 多态的概念与分类

多态（Polymorphisn）是面向对象程序设计（OOP）的一个重要特征。多态，字面意思为多种状态。在面对对象语言中，**一个接口，多种实现**，即为多态。

C++中的多态具体体现在编译和运行两个阶段。编译时是静态多态，在编译时就可以确定使用的接口；运行时多态是动态多态，具体引用的接口在运行时才能确定。

![C++多态的两种形式](https://s2.loli.net/2023/08/05/vY9num6MWeArEaN.png)

静态多态和动态多态的区别其实只是在什么时候将函数实现和函数调用关联起来。静态多态是指在编译期间就可以确定函数的调用地址，并产生代码，这就是静态的，也就是说地址是早绑定（静态绑定）的，静态多态往往也被叫做静态联编。动态多态则是指函数调用的地址不能在编译期间确定，需要在运行时确定，属于晚绑定（动态绑定），动态多态往往也被叫做动态联编。


# 多态的作用

为什么要使用多态？

封装可以使得代码模块化，继承可以扩展已存在的代码，他们的目的都是为了代码重用。而**多态的目的则是为了接口重用**。静态多态，将同一个接口进行不同的实现，根据传入的参数（个数或类型不同），调用不同的实现。动态多态，不论传递过来的哪个类的对象，函数都能够通过同一个接口调用到各自对象实现的方法。


# 静态多态

静态多态往往通过函数重载和模板（泛型编程）来实现。以模板函数为例，模板函数可以根据传递参数的不同类型，自动生成相应类型的函数代码，来实现多态。

例如：实现一个模板函数 getMax()，接收相同类型的两个参数，返回较大的一个。
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

在上面的例子中，编译器会生成两个 getMax() 的函数示例，一个是 int，一个是 double，这种调用的函数在编译期就能确定下来的就叫静态多态。


# 动态多态

动态多态通过 继承 + 虚函数（包含纯虚函数）实现的。虚函数是指基类中声明的函数，在派生类中可以被重写（override）。

当我们使用基类指针或引用指向派生类对象时，通过虚函数的机制，可以调用到派生类重写的函数，从而实现多态。

C++多态必须满足两个条件：
1. **必须通过基类的指针或引用调用函数**
2. **被调用的函数是基类的虚函数，且派生类必须完成对该虚函数的重写**

满足以上条件后，即使用基类指针或引用指向派生类对象时，调用虚函数，在程序运行期间（非编译期）才能判断实际所指向或引用对象的实际类型，根据其指向或引用的实际类型调用相应的方法，实现运行时函数地址的动态绑定。

如果没有使用虚函数，即没有利用C++多态性，则利用基类指针调用相应函数时，只会根据指针类型调用基类函数，而无法调用到派生类中被重写过的函数。因为没有多态性，函数调用的地址将是一定的，而固定的地址将始终调用到同一个函数。

举例说明动态多态的实现：
一个基类 Shape，两个派生类 Rectangle、Triangle 分别继承 Shape 。
``` C++
#include <iostream>
using namespace std;

class Shape {
public:
	virtual void printArea() = 0;       // 纯虚函数声明，没有函数体
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

执行程序后可以看到，我们使用基类指针调用了派生类重写的函数，这样就实现了运行时函数地址的动态绑定，即动态联编。

注意派生类重写的函数签名（函数名、参数列表、返回类型）必须和基类的虚函数保持一致，协变和析构函数除外。而访问限定符（private、protected、public）不属于函数签名的一部分，也可以不同。


# 相关问题

## 什么是协变？

协变（Covariance）是指在继承关系中，函数返回值类型可以在派生类中被改变为更具体的类型。

在C++中，如果一个类派生自另一个类，并且它的成员函数重写（override）了基类的虚函数，那么在派生类中可以改变虚函数的返回值类型，只要返回值类型是基类返回值类型的派生类即可。这种情况下，我们称函数返回值类型在派生类中发生了协变。

协变的特点是派生类中的函数返回值类型是基类函数返回值类型的子类型。这使得派生类的函数能够返回更具体的对象，而不会破坏基类指针或引用的类型安全性。

以下是一个简单的示例来说明协变：
``` C++
class Animal {
public:
    virtual Animal* clone() const {
        return new Animal(*this);
    }
};

class Dog : public Animal {
public:
    // 在派生类中，返回类型从 Animal* 改变为 Dog*
    Dog* clone() const override {
        return new Dog(*this);
    }
};

int main() {
    Animal* animalPtr = new Dog();
    Animal* clonedAnimalPtr = animalPtr->clone(); // 调用 Dog 类中的 clone 函数，返回 Dog* 类型
    delete animalPtr;
    delete clonedAnimalPtr;
    return 0;
}
```

在上述示例中，基类 Animal 中的虚函数 clone() 的返回值类型是 Animal*，而派生类 Dog 中重写（override）了该函数，并将其返回值类型改变为 Dog*，这就是协变。这使得在派生类中可以返回更具体的 Dog 类型对象，而不是只能返回基类 Animal 类型对象。

需要注意的是，协变只适用于指针和引用的返回值类型。C++中不支持对于函数参数类型和普通成员函数的返回值类型协变。


## 虚析构函数的作用

在C++中，虚析构函数的作用是允许通过基类指针删除派生类对象时，正确地调用派生类的析构函数。

虚析构函数使得在使用基类指针或引用删除派生类对象时，能够**根据对象的实际类型动态调用正确的析构函数**，从而正确释放派生类中的资源。

然而，虚析构函数仅在基类中声明为虚函数，并在基类中提供实现。当派生类的析构函数与基类的析构函数具有不同的实现时，并不算是析构函数的重写。

```C++
#include <iostream>
using namespace std;

class Base {
public:
    virtual ~Base() {   // 虚析构函数，基类提供实现
		cout << "Base destructor." << endl;
    }
};

class Derived : public Base {
public:
    ~Derived() {        // 派生类的析构函数，可以在此释放派生类中的资源
		cout << "Derived destructor." << endl;
    }                   
};

int main() {
    Base* basePtr = new Derived();
    delete basePtr;	   // 使用基类指针删除派生类对象，会调用正确的析构函数
    return 0;
}
/*
 Derived destructor.
 Base destructor.
*/
```

在上述示例中，Base 类的析构函数被定义为虚析构函数，它提供了正确的析构行为。Derived 类的析构函数提供了派生类的资源释放。

当使用基类指针 basePtr 删除 Derived 类对象时，会先调用 Derived 类的析构函数，再调用 Base 类的析构函数。这正是虚析构函数的作用，确保在删除派生类对象时能够正确地调用派生类的析构函数，释放派生类中的资源。

如果基类的析构函数不是虚函数，通过基类指针删除派生类对象时，只会根据指针类型调用基类的析构函数，这在编译期就已经确定，而不会先调用派生类的析构函数，可能发生内存泄漏。


## 哪些函数不能定义为虚函数？

1. **静态成员函数（Static member functions）**：静态成员函数是与类相关联的函数，而不是与类的实例对象相关联的。它们不属于任何特定对象，没有虚函数的必要性。虚函数的主要目的是实现运行时多态，而静态函数在编译时就能确定调用的具体函数。

2. **友元函数（Friend functions）**：友元函数是被声明在类外的，但具有访问类内私有或保护成员的特权。由于友元函数不是类的成员函数，所以不能是虚函数。

3. **全局函数（Global functions）**：不是类的成员函数，不能是虚函数。

4. **内联函数（Inline functions）**：内联函数是通过在调用处直接展开函数代码来优化性能的特殊类型函数。由于虚函数的调用涉及虚表查找等额外开销，所以虚函数不能是内联函数。

5. **构造函数（Constructor）**，包含拷贝构造函数等：虚函数的调用是在运行时根据对象的实际类型确定的，而构造函数是在创建对象时调用的，此时无法确定对象的实际类型，因为对象还没有被创建出来，其虚表指针 vptr 也不存在。


## 纯虚函数、接口类、抽象类

在上面例子中，`virtual void printArea() = 0;`，在成员函数（必须为虚函数）的形参列表后写 `=0`，则成员函数为纯虚函数。

**纯虚函数（Pure Virtual Function）是在 C++ 中定义一个没有函数体的虚函数。** 它在基类中声明，并且在基类中没有具体实现，只用来占位，告诉派生类需要提供该函数的实现。通过声明纯虚函数，基类可以强制派生类必须实现该函数，以使得派生类能够成为一个完整的具体类。

由于纯虚函数没有具体实现，因此包含纯虚函数的基类不能直接创建对象，也不能在基类中直接调用纯虚函数。需要将基类派生为具体的子类，并在派生类中提供纯虚函数的具体实现，才能实例化对象（派生类对象）并调用纯虚函数（派生类实现的）。

纯虚函数的主要用途是实现接口（Interface）和抽象类（Abstract Class）的概念。

**接口类，是只包含纯虚函数和常量数据成员的类**，用于描述类的行为规范而不涉及具体实现。

**抽象类，是包含纯虚函数和可能有实现的虚函数的类**，它允许派生类继承接口和通用行为，但不允许创建实例化对象。


---


# 参考

[C++多态的两种形式](https://cloud.tencent.com/developer/article/1347880) 

[C++多态实现方式](https://csguide.cn/cpp/object_oriented/polymorphism_in_cplusplus.html) 

[C++之多态](https://blog.csdn.net/qq_39412582/article/details/81628254) 