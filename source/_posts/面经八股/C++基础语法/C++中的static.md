---
title: C++中的static
date: 2023-05-13 00:00:00
tags:
updated:
categories:
- 面经八股
- C++基础语法
keywords:
description: 在 C++ 中，static 是一个非常重要的关键字，它可以用于变量、函数和类中。
cover: https://s2.loli.net/2023/05/13/ey1LHmFjvtJhKI4.jpg
---

在 C++ 中，static 是一个非常重要的关键字，它可以用于变量、函数和类中。

# static 修饰全局变量
static 修饰全局变量可以将变量的作用域限定在当前文件中，使得其他文件无法访问该变量。

同时，static 修饰的全局变量在程序启动时被初始化（可以简单理解为在执行 main 函数之前，会执行一个全局的初始化函数，在那里会执行全局变量的初始化），生命周期和程序一样长。
```C++
// a.cpp
static int a = 10;  // static 修饰全局变量
int main() {
    a++;    // 合法，可以在当前文件中访问 a
    return 0;
}

// b.cpp
extern int a;   // 声明 a
void foo() {
    a++;    // 非法，会链接错误，其他文件无法访问 a
}
```
为什么会是非法，链接错误：[extern 的作用-从链接角度理解](https://www.wangjiapeng.com/2023/05/14/%E9%9D%A2%E7%BB%8F%E5%85%AB%E8%82%A1/c++%E5%9F%BA%E7%A1%80%E8%AF%AD%E6%B3%95/extern%20%E7%9A%84%E4%BD%9C%E7%94%A8-%E4%BB%8E%E9%93%BE%E6%8E%A5%E8%A7%92%E5%BA%A6%E7%90%86%E8%A7%A3/)


# static 修饰局部变量
static 修饰局部变量可以使得变量在函数调用结束后不会被销毁，而是一直存在于内存中，下次调用该函数时可以继续使用。

同时，由于 static 修饰的局部变量的作用域仅限于函数内部，因此其他函数无法访问该变量。
```C++
void foo() {
    static int count = 0;   // static 修饰局部变量
    count++;
    cout << count << endl;
}

int main() {
    foo();  // 输出 1
    foo();  // 输出 2
    foo();  // 输出 3
    return 0;
}
```


# static 修饰函数
static 修饰函数可以将函数的作用域限定在当前文件中，使得其他文件无法访问该函数。

同时，由于 static 修饰的函数只能在当前文件中被调用，因此可以避免命名冲突和代码的重复定义。
```C++
// a.cpp
static void foo() {     // static 修饰函数
    cout << "Hello, world!" << endl;
}

int main() {
    foo();      // 合法，可以在当前文件中调用 foo() 函数
}

// b.cpp
extern void foo();      // 声明 foo()
void bar() {
    foo();      // 非法，会链接错误，找不到 foo() 函数，其他文件无法调用
}
```


# static 修饰类成员变量和函数
static 修饰类成员变量和函数可以使得它们在所有类对象中共享，且不需要创建对象就可以直接访问。
```C++
class MyClass {
public:
    static int count;       // static 修饰类成员变量
    static void foo() {     // static 修饰类成员函数
        cout << count << endl;
    }
};

// 访问
MyClass::count;
MyClass::foo();
```