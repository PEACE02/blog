---
title: C++几种类型转换的区别
date: 2023-05-12 00:00:00
tags:
updated:
categories:
- 面经八股
- C++基础语法
keywords:
description: 在 C 语言中，我们大多数是用(type_name) expression 这种方式来做强制类型转换，但是在 C++ 中，更推荐使用四个转换操作符来实现显式类型转换。
cover: https://s2.loli.net/2023/05/12/9HavYAe6wbpVc8s.jpg
---

在 C 语言中，我们大多数是用 (type_name) expression 这种方式来做强制类型转换，但是在 C++ 中，更推荐使用四个转换操作符来实现显式类型转换。

# static_cast
用法：`static_cast<new_type>(expression)`
其实 static_cast 和 C 语言 (type_name) expression 做强制类型转换基本是等价的。
主要用于以下场景：

## 基本类型之间的转换
将一个基本类型转换为另一个基本类型，例如将整数转换为浮点数或将字符转换为整数。
```C++
int a = 10;
double b = static_cast<double>(a);  // 将整数 a 转换为双精度浮点数 b
```

## 指针类型之间的转换
将一个指针类型转换为另一个指针类型，尤其是在类层次结构中从基类指针转换为派生类指针，这类转换不执行运行时类型检查，可能不安全，要自己保证指针确实可以相互转换。
```C++
class Base {};
class Derived : public Base {};

Base* base_ptr = new Derived();
// 将基类指针 base_ptr 转换为派生类指针 derived_ptr
Derived* derived_ptr = static_cast<Derived*>(base_ptr);
```

## 引用类型之间的转换
类似于指针类型之间的转换，可以将一个引用类型转换为另一个引用类型，在这种情况下，也应注意安全性。
```C++
Derived derived_obj;
Base& base_ref = derived_obj;
// 将基类引用 base_ref 转换为派生类引用 derived_ref
Derived& derived_ref = static_cast<Derived&>(base_ref);
```
static_cast 在编译时执行类型转换，在运行时不执行类型检查，所以在进行指针或引用类型转换时，需要自己保证合法性。如果想要运行时执行类型检查，可以使用 dynamic_cast 进行安全的向下类型转换。

---

# dynamic_cast
用法：`dynamic_cast<new_type>(expression)`
dynamic_cast 在 C++ 中主要应用于父子类层次结构中的安全类型转换。它在运行时执行类型检查，因此相比于 static_cast，它更安全。
主要用于以下场景：
## 向下类型转换
当需要将基类指针或引用转换为派生类指针或引用时，dynamic_cast 可以确保类型兼容性，如果转换失败，dynamic_cast 将返回空指针（对于指针类型）或抛出异常（对于引用类型）。
```C++
class Base { virtual void dummy() {} };
class Derived : public Base { int a; };

Base* base_ptr = new Derived();
// 将基类指针 base_ptr 转换为派生类指针 derived_ptr，如果类型兼容，则成功
Derived* derived_ptr = dynamic_cast<Derived*>(base_ptr);
```

## 用于多态类型检查
处理多态对象时，dynamic_cast 可以用来确认对象的实际类型，例如：
```C++
class Animal { public: virtual ~Animal() {} };
class Dog : public Animal { public: void bark() {/*...*/} };
class Cat : public Animal { public: void meow() {/*...*/} };

Animal* animal_ptr = /*...*/;
// 尝试将 Animal 指针转换为 Dog 指针
Dog* dog_ptr = dynamic_cast<Dog*>(animal_ptr);
if (dog_ptr != nullptr) dog_ptr->bark(); 
// 尝试将 Animal 指针转换为 Cat 指针
Cat* cat_ptr = dynamic_cast<Cat*>(animal_ptr);
if (cat_ptr != nullptr) cat_ptr->meow(); 
```
另外，要使 dynamic_cast 有效，基类至少需要一个虚拟函数，因为 dynamic_cast 只有在基类存在虚函数（虚函数表）的情况下，才有可能将基类指针转换为子类。

## dynamic_cast 底层原理
dynamic_cast 的底层原理依赖于运行时的类型信息（RTTI, Runtime Type Information）。C++ 编译器在编译时为支持多态的类生成 RTTI，它包含了类的类型信息和类层次结构。

我们知道当使用虚函数时，编译器会为每个类生成一个虚函数表（vtable），并在其中存储指向虚函数的指针。伴随虚函数表的还有 RTTI（运行时类型信息），这些辅助的信息可以用来帮助我们运行时识别对象的类型信息。

《深度探索C++对象模型》中有个例子：
```C++
class Point {
public:
    Point(float xval);
    virtual ~Point();

    float x() const;
    static int PointCount();

protected:
    virtual ostream& print(ostream& os) const;

    float _x;
    static int _point_count;
};
```
[](https://s2.loli.net/2023/05/13/l6AHhir1PtqDION.png)
首先，每个多态对象都有一个指向其 vtable 的指针，称为 vptr。 RTTI（就是上面图中的 type_info 结构）通常与 vtable 关联。 dynamic_cast 就是利用 RTTI 来执行运行时类型检查和安全类型转换。以下是 dynamic_cast 的工作原理的简化描述：
1. 首先，dynamic_cast 通过查询对象的 vptr 来获取去 RTTI（这也是为什么 dynamic_cast 要求对象有虚函数）；
2. 然后，dynamic_cast 比较请求的目标类型与从 RTTI 获得的实际类型，如果目标类型是实际类型或其基类，则转换成功；
3. 如果目标类型是派生类，dynamic_cast 会检查类层次结构，以确定转换是否合法。如果在类层次结构中找到了目标类型，则转换成功，否则转换失败；
4. 当转换成功时，dynamic_cast 返回转换后的指针或引用；
5. 如果转换失败，对于指针类型，dynamic_cast 返回空指针，对于引用类型，会抛出一个 std::bad_cast 异常。

因为 dynamic_cast 依赖于运行时类型信息，它的性能可能低于其他类型转换操作（如 static_cast），static_cast 是编译器静态转换，在编译时完成。

---

# const_cast
用法：`const_cast<new_type>(expression)`
new_type 必须是一个指针、引用或指向对象类型成员的指针。
主要用于以下场景：

## 修改 const 对象
当需要修改 const 对象时，可以使用 const_cast 来删除 const 属性。
```C++
const int a = 10;
int* mutable_ptr = const_cast<int*>(&a);    // 删除 const 属性，使 a 的值可修改
*mutable_ptr = 20;  // 修改 a 的值
```

## const 对象调用非 const 成员函数
当需要使用 const 对象调用非 const 成员函数时，可以使用 const_cast 删除对象的 const 属性。
```C++
class MyClass {
public:
    void non_const_function() {/*...*/}
};

const MyClass my_const_obj;
// 删除 const 属性，使可以调用非 const 成员函数
MyClass* mutable_obj_ptr = const_cast<MyClass*>(&my_const_obj);
mutable_obj_ptr->non_const_function();     // 调用非 const 成员函数
```
不过上述行为都是不安全的，可能导致未定义行为，应谨慎使用。

---

# reinterpret_cast
用法：`reinterpret_cast<new_type>(expression)`
reinterpret_cast 用于在不同类型之间进行低级别的转换。

首先从字面意思理解，interpret 是 “解释，诠释” 的意思，加上前缀"re"就是“重新解释”的意思。cast 在这里可以翻译成“转型”（在侯捷老师翻译的《深度探索 C++ 对象模型》、《Effective C++（第三版）》中，cast 都被翻译成了转型），这样整个词义即“重新解释的转型”。

它仅仅是重新解释底层 bit，也就是对指针所指向的那片 bits 换个类型做解释（大概相当于对同一本书换了一个人来读），而不进行任何类型检查。

因此，reinterpret_cast 也是不安全的，可能导致未定义行为，应谨慎使用。

reinterpret_cast 的典型应用场景：

## 指针类型之间的转换
在某些情况下，需要在不同指针类型之间进行转换，如将一个 int 指针转换为 char 指针，这在 C 语言中用的非常多，C 语言中就是直接使用 (type_name) expression 进行强制类型转换。
```C++
int a = 10;
int* int_ptr = &a;
char* char_ptr = reinterpret_cast<char*>(int_ptr);  // 将 int 指针转换为 char 指针
```
[C++类型转换之reinterpret_cast](https://zhuanlan.zhihu.com/p/33040213)