![bb](img/bb.png)

---
# Effective CPP Series

Effective C++ Series ( Effective C++, More Effective C++, Effective Modern C++, etc. ) 读书笔记，记录一些自己认为重要的东西。笔记并不完全与书上一致，有引入一些 **C++ 1x** *(C++ 11 ~ C++ 20)* 的新特性。

---

## [Effective C++](Effective%20C++.md)

- [让自己习惯 **C++**](./Effective%20C++.md#1-视-c-语言为一个联邦)
  - [1. 视 **C++** 语言为一个联邦](./Effective%20C++.md#1-视-c-语言为一个联邦)
  - [2. 尽量以 `const`、`enum`、`inline` 替换 `#define`](./Effective%20C++.md#2-尽量以-constenuminline-替换-define)
  - [3. 尽可能使用 `const`](./Effective%20C++.md#3-尽可能使用-const)
  - [4. 确定对象被使用前已先被初始化](./Effective%20C++.md#4-确定对象被使用前已先被初始化)
- [构造/析构/赋值运算](./Effective%20C++.md#5-了解-c-默默编写并调用哪些函数)
  - [5. 了解 **C++** 默默编写并调用哪些函数](./Effective%20C++.md#5-了解-c-默默编写并调用哪些函数)
  - [6. 若不想使用编译器自动生成的函数，应当明确拒绝](./Effective%20C++.md#6-若不想使用编译器自动生成的函数应当明确拒绝)
  - [7. 为多态基类声明 **virtual 析构函数**](./Effective%20C++.md#7-为多态基类声明-virtual-析构函数)
  - [8. 别让 **异常** 逃离 **析构函数**](./Effective%20C++.md#8-别让-异常-逃离-析构函数)
