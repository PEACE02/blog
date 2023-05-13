---
title: C++中的const
date: 2023-04-25 00:00:00
tags:
updated:
categories:
- 面经八股
- C++基础语法
keywords:
description: 在 C/C++ 中，const 是一个关键字，用于表示常量，可以用于修饰变量、函数、指针等。
cover: https://s2.loli.net/2023/04/25/hvrcNQKtWJskpHm.jpg
---

在 C/C++ 中，`const` 是一个关键字，用于表示常量。`const` 可以用于修饰变量、函数、指针等。其主要作用有以下几种：

# 修饰变量，表示变量为只读

当 `const` 修饰变量时，该变量将被视为只读变量，即不能被修改。对于确定不会被修改的变量，我们应该加上 `const`，这样可以保证变量的值不会被无意的修改，也可以使编译器在代码优化时更加智能。

如下，编译错误，只读变量不能被修改：
```C++
const int a = 10;
a = 20;	// [Error] assignment of read-only variable 'a'
```

**但是！**
这里的变量只读，其实只是**编译器层面**的保证，我们可以通过**指针在运行时**去间接修改这个变量的值。

对 `const int` 类型取指针，就是 `const int*` 类型的指针，将其强制转换为 `int*` 类型，就去掉了 `const` 限制，从而修改变量的值。在 C++ 中，将 `const` 类型的指针强制转换为非 `const` 类型的指针被称为类型强制转换（Type Casting），这种行为称为 `const_cast`。

关于 const_cast ： [C++几种类型转换的区别](https://www.wangjiapeng.com/2023/05/12/%E9%9D%A2%E7%BB%8F%E5%85%AB%E8%82%A1/c++%E5%9F%BA%E7%A1%80%E8%AF%AD%E6%B3%95/c++%E5%87%A0%E7%A7%8D%E7%B1%BB%E5%9E%8B%E8%BD%AC%E6%8D%A2%E7%9A%84%E5%8C%BA%E5%88%AB/)

虽然可以这样操作，但这违反了 `const` 的语义，可能会导致程序崩溃或者产生未定义行为（undefined behavior），**实际编程中不可这样操作**。

因为编译器可能会做一些优化，也就是在用到 const 变量的地方，编译器可能生成的代码就直接替换为常量的值，而不是访问一遍常量的指令。所以极大可能是：虽然修改了值，但是不起作用。

如下使用 `const_cast` 修改 `const` 变量的值却不会起作用：

```C++
const int a = 10;
const int* p = &a;
int* q = const_cast<int*>(p);
*q = 20;
std::cout << "a = " << a << std::endl;      //  a = 10
std::cout << "*p = " << *p << std::endl;    // *p = 20
std::cout << "*q = " << *q << std::endl;    // *q = 20
```

在上面例子中，将 `p` 声明为 `const int*` 类型，为指向只读变量 `a` 的指针。然后使用 `const_cast` 将 `p` 强制转换为 `int*` 类型的指针 `q`，从而去掉了 `const` 限制，接下来通过指针 `q` 间接修改了变量 a 的值。

但是程序输出 a 的值仍然是 10，因为编译器优化，实际运行代码为 `std::cout << "a = " << 10 << std::endl;` 。

那到底是变了还是没变？以下做了补充测试：
```C++
#include <iostream>
int main() {
	const int a = 10;
	const int* p = &a;
	int* q = const_cast<int*>(p);
	
	std::cout << &a << std::endl;               // 0x6ffdf4
	std::cout << p << std::endl;                // 0x6ffdf4
	std::cout << q << std::endl;                // 0x6ffdf4
	
	*q = 20;
	std::cout << "a = " << a << std::endl;      //  a = 10
	std::cout << "*p = " << *p << std::endl;    // *p = 20
	std::cout << "*q = " << *q << std::endl;    // *q = 20
	
	std::cout << &a << std::endl;               // 0x6ffdf4
	std::cout << p << std::endl;                // 0x6ffdf4
	std::cout << q << std::endl;                // 0x6ffdf4
	
	const int* u = &a;
	std::cout << "*(&a) = " << a << std::endl;  // *(&a) = 10
	std::cout << "*u = " << *u << std::endl;    // *u = 20
}
```
可以说明在内存中的值确实是被改变了，而在程序中直接使用 `a`、`*(&a)` 都会被编译器优化为 `a` 的常量初始定义 10。所以程序中如果直接使用 `a` ，相当于使用被定义时的初始常量；但是通过指针取值就会取到修改后的值。即对应了上述中提到的**编译时**和**运行时**。

总之，使用 `const_cast` 去掉 `const` 限制是不推荐的，这会破坏程序的正确性和稳定性。**我们应该遵循 const 的语义，尽量不修改只读变量的值。**


# 修饰函数参数，表示函数不会修改参数

当 `const` 修饰函数参数时，表示函数内部不会修改该参数的值，这样做可以使代码更加安全，避免在函数内部无意中修改传入的参数值。尤其是引用作为参数时（pass by reference），如果确定不会修改引用，那么一定要使用 `const` 引用。

用法如下：
```C++
void func(const int& a) {
	a = 20;     // [Error] assignment of read-only reference 'a'
    // 如果 pass by value，即 func(const int a)，尝试 a = 20
    // [Error] assignment of read-only parameter 'a'
}
int main() {
	const int a = 10; func(a);
}
```

# 修改函数返回值，表示函数返回值为只读

当 `const` 修饰函数返回值时，表示函数的返回值为只读，不能被修改。这样做可以使函数返回的值更加安全，避免被误修改。

但是，以 int 为例，实际测试分有以下情况：

返回类型为 `const int`，接收类型为 `int`： 
```C++
#include <iostream>
const int func() {return 10;}
int main() {
	int a = func(); // int a = 10;
	a = 20;
	std::cout << a <<  std::endl;	// 20
}
```
`a` 是非 `const` 变量，来接收 `const int func()` 的返回值，对 `a` 而言没有 `const` 的限制，只相当于一次常量赋值。此时 `const` 修饰函数返回值看起来好像是没有作用的（但不是）。

返回类型为 `const int&`，接收类型为 `int` 时同理，`const` 修饰函数返回类型也是好像不起作用，因为对调用者而言，使用的是另外的一个非 `const` 变量。 

而对返回类型为 `const int` 或 `const int&`，当尝试使用引用接收时，被要求使用 `const int&` 类型接收，此时 `const` 修饰的函数返回值被保护。
```C++
const int func() {
	return 10;
}
int main() {
	// int& a = func();	// [Error] invalid initialization of non-const reference of type 'int&' from an rvalue of type 'int'
	const int& a = func();
	a = 20;             // [Error] assignment of read-only reference 'a'
}
```

再如下面例子返回类型为 `const int*`，调用者被要求用相同类型接收，使返回值可以避免被误修改。
```C++
#include <iostream>

const int* func(int* a) {
	*a = 10;
	return a;
}
int main() {
	int a = 1;
	// int* p = func(&a);   // [Error] invalid conversion from 'const int*' to 'int*' [-fpermissive]
	const int* p = func(&a);
	std::cout << a << std::endl;    // 10
	*p = 20;                // [Error] assignment of read-only location '* p'
}
```

所以当 `const` 修饰函数返回值时，如果返回类型为值或引用，当调用者用普通变量接收，此时应该是拷贝赋值，所以对调用者来说，是可以随意修改的，但是修改的只是拷贝过来的数据，而不会对原数据修改。如果返回类型为指针，则必须以指针接收返回值，此时被要求必须以 `const` 修饰的指针，以保护返回数据的安全。

[指针和引用的区别](https://www.wangjiapeng.com/2023/05/05/%E9%9D%A2%E7%BB%8F%E5%85%AB%E8%82%A1/c++%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86/%E6%8C%87%E9%92%88%E5%92%8C%E5%BC%95%E7%94%A8%E7%9A%84%E5%8C%BA%E5%88%AB/)

这其实很容易理解，有一份数据被指定为不可修改的，即用 `const` 修饰，这并不妨碍你将这份数据拷贝一份之后，对拷贝过来的数据进行修改，因为这不会改变原数据。但是如果尝试通过引用或指针的方式，直接对原数据修改，此时 `const` 会起到作用。


# 修改指针或引用

在 C/C++ 中，`const` 可以用来修饰指针，用于声明指针本身为只读变量或者指向只读变量的指针。根据 `const` 关键字的位置和类型，可以将 const 指针分为以下三种情况：

## 1. **指向只读变量的指针（常量指针）**
这种情况下，const 关键字修饰的是指针指向的变量，而不是指针本身。因此，指针本身可以被修改（意思是指针可以指向新的变量），但是不能通过指针修改指针指向的变量。
```C++
int a = 10;
const int b = 20;
	
int* q;
q = &a;		// √，普通指针指向普通变量 
q = &b;		// [Error] invalid conversion from 'const int*' to 'int*' [-fpermissive]
	
const int* p;   // 和  int const* p; 一样 
p = &a;         // √，常量指针可以指向普通变量 
p = &b;         // √，常量指针本身可以修改，且可以指向只读变量 
*p = 30;        // [Error] assignment of read-only location '* p'
```
在上面的例子中，使用 `int*` 声明了一个普通指针 `q`，只能指向普通常量，不能指向只读常量；使用 `const int*` 声明了一个指向只读变量的指针 `p`，既可以指向普通变量，也可以指向只读变量，但是无法通过指针修改只读变量的值。

## 2. **只读指针（指针常量）**
这种情况下，const 关键字修饰的是指针本身，使指针本身成为只读变量。因此，指针本身不能被修改（即指针一旦初始化就不能指向其他变量），但是可以通过指针修改所指向的变量。
```C++
int a = 10;
int b = 20;
int* const p = &a;  // 声明一个只读指针，指向 a
*p = 30;            // √，通过只读指针修改指向变量的值
p = &b;             // [Error] assignment of read-only variable 'p'
```
在上面的例子中，使用 `int* const` 声明了一个只读指针 `p`，指向普通变量 `a`，可以通过只读指针 `p` 修改普通变量 `a` 的值，但是无法修改只读指针本身的值。

只读指针相当于被限制而不能修改本身值的普通指针，因此不能初始化指向只读变量。

## 3. **只读指针指向只读变量（指向常量的指针常量）**
这种情况下，const 关键字同时修饰指针本身和指针所指向的变量，使指针本身和指针所指向的变量都为只读变量。因此，指针本身不能被修改，也不能通过指针修改所指向的变量。
```C++
const int a = 10;
const int* const p = &a;    // 或 int const* const
*p = 20;            // [Error] assignment of read-only location '*(const int*)p'
p = nullptr;        // [Error] assignment of read-only variable 'p'
```

## **常量引用**
常量引用是指引用一个只读变量，因此不能通过常量引用修改只读变量的值。
```C++
const int a = 10;
// int& r = a;  // [Error] invalid initialization of reference of type 'int&' from expression of type 'const int'
const int& r = a;
r = 20;         // [Error] assignment of read-only reference 'r'
```

# 修饰成员函数，表示函数不会修改对象的状态
当 `const` 修饰成员函数时，表示该函数不会修改对象的状态（就是不会修改成员变量）。这样有个好处是，被 `const` 修饰的 const 对象就可以调用这些成员方法了，因为 const 对象不允许调用非 const 的成员方法。

因为实例化对象是被 `const` 修饰的，只有调用 `const` 修饰的成员函数时，才能保证自己的状态（成员变量）不被修改。
```C++
class A {
public:
	int func() const {
		m_value = 20;   // [Error] assignment of member 'A::my_value' in read-only object
		return m_value;
	}
private:
	int my_value = 10;
};
```

还要注意的是，`const` 修饰的成员函数不能调用非 const 的成员函数，同样是因为不能保证非 const 成员函数不修改对象状态，即使非 const 成员函数真的没有修改。
```C++
class A {
public:
	int func() const {
		no_const_func();  // [Error] passing 'const A' as 'this' argument of 'void A::no_const_func()' discards qualifiers [-fpermissive]
		return my_value;
	}
	void no_const_func() {}
private:
	int my_value = 10;
};
```

总之，`const` 关键字的作用是为了保证变量的安全性和代码的可读性。