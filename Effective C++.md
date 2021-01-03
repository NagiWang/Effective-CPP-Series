- [视 C++ 语言为一个联邦](#视-c-语言为一个联邦)
  - [多重范式 ( Multiparadigm Progamming )](#多重范式--multiparadigm-progamming-)
  - [子语言 ( Sublanguage )](#子语言--sublanguage-)
  - [Summary](#summary)
- [尽量以 `const`、`enum`、`inline` 替换 `#define`](#尽量以-constenuminline-替换-define)
  - [常量定义](#常量定义)
  - [`class` 专属常量定义](#class-专属常量定义)
  - [宏函数](#宏函数)
  - [Summary](#summary-1)
- [尽可能使用 `const`](#尽可能使用-const)
  - [`const` 出现在 `*` 左边表示指针本身是常量，出现在 `*` 右边表示被指物是常量，写在类型之前还是之后意义相同](#const-出现在--左边表示指针本身是常量出现在--右边表示被指物是常量写在类型之前还是之后意义相同)
  - [对于 **iterator**](#对于-iterator)
  - [令函数返回一个常量值，往往可以降低因客户错误而造成的意外，而又不至于放弃安全性和高效性。](#令函数返回一个常量值往往可以降低因客户错误而造成的意外而又不至于放弃安全性和高效性)
  - [`const` 成员函数](#const-成员函数)
    - [**bitwise constness** 流派](#bitwise-constness-流派)
    - [**logical constness** 流派](#logical-constness-流派)
  - [在 `const` 和 `non-const` 成员函数中避免重复](#在-const-和-non-const-成员函数中避免重复)
  - [Summary](#summary-2)
- [确定对象被使用前已先被初始化](#确定对象被使用前已先被初始化)
  - [内置类型的初始化](#内置类型的初始化)
  - [构造函数](#构造函数)

---

# 视 C++ 语言为一个联邦

## 多重范式 ( Multiparadigm Progamming )

- 过程形式 ( Procedural )
- 面向对象形式 ( Object-Oriented )
- 函数形式 ( Functional )
- 泛型形式 ( Generic )
- 元编程形式 ( Metaptogramming )

## 子语言 ( Sublanguage )

- C
- Object-Oriented C++
- Template C++
- STL

## Summary

- **C++** 高效编程守则视情况而变化，取决于你使用 **C++** 的哪一个部分

---

# 尽量以 `const`、`enum`、`inline` 替换 `#define`

## 常量定义

```cpp
#define ASPECT_RATIO 1.653
```

可替换为

```cpp
const double AspectRatio = 1.653;
```

或者 ( C++ 11 以上 )

```cpp
constexpr double AspectRatio = 1.653;
```

对于常量字符串需要 `const` 两次

```cpp
const char* const autherName = "Scott Meyers";
```

更好的一种方式是

```cpp
const std::string autherName("Scott Meyers");
```

## `class` 专属常量定义

```cpp
class GamePlayer {
private:
    static const int NumTurns = 5;  // in-class 初值设定
    int scores[NumTurns];
};
```

上述代码在不支持 **in-class 初值设定** 的编译器上无法使用，可用如下方法替换

```cpp
// CostEstimate.hpp
class CostEstimate {
private:
    static const double FudgeFactor;
};
```

```cpp
// CostEstimate.cpp
const double
    CostEstimate::FudgeFactor = 1.35;
```

在不支持 **static 整数型 class 常量** 完成  **in-class 初值设定** 的编译器上，可通过 **the enum hack** 方式来进行实现

```cpp
class GamePlayer {
private:
    enum { NumTurns = 5 };  // the enum hack
    int scores[NumTurns];
};
```

**the enum hack** 的行为更像是 `#define` 而不像是 `const` ，但有时可能正是我们需要的。因为，取一个 `const` 的地址是合法的，而取一个`enum` 的地址并不合法。如果不想某个 **pointer** 或者 **reference** 指向某个整数常量，便可使用 `enum` 来实现。

## 宏函数

```cpp
#define CALL_WITH_MAX(a, b) func((a) > (b) ? (a) : (b))
```

改用 `template inline` 函数改写

```cpp
// C++ 0x
template <typename T>
inline T callWithMax(const T &a, const T &b) {
    func(a > b ? a : b);
}
```

或者用 `lambda` 函数

```cpp
// C++ 1x
template <typename T>
constexpr auto callWithMax = [](const T &a, const T &b) {
    func(a > b ? a : b);
};
```

```cpp
// C++ 1x
constexpr auto callWithMax = [](const auto &a, const auto &b) {
    func(a > b ? a : b);
};
```

```cpp
// C++ 20
constexpr auto callWithMax = [] <typename T> (const T &a, const T &b) {
    func(a > b ? a : b);
};
```

## Summary

- 对于单纯常量，最好以 `const` 对象或 `enums` 替换 `#define`
- 对于形似函数的宏 (`macros`)，最好改用 `inline` 函数替换 `#define`

---

# 尽可能使用 `const`

## `const` 出现在 `*` 左边表示指针本身是常量，出现在 `*` 右边表示被指物是常量，写在类型之前还是之后意义相同

```cpp
const int* a;
int const* b;
```

## 对于 **iterator**

``` cpp
std::vector<int> vec{};

const std::vector<int>::iterator iter =
    vec.begin();  // iter 作用像个 T* const
*iter = 10;       // 没问题，改变的是 iter 所指物
++iter;           // 错误！ iter 是 const

std::vector<int>::iterator citer =
    vec.begin();  // citer 作用像个 const T*
*citer = 10;      // 错误！ *citer 是 const
+++citer;         // 没问题，改变 citer
```

## 令函数返回一个常量值，往往可以降低因客户错误而造成的意外，而又不至于放弃安全性和高效性。

```cpp
class Rational {};
const Rational operator* (const Rational &lhs, const Rational &rhs);
```

可避免出现如下情况

```cpp
Rational a, b, c;
...
(a * b) = c;
if (a * b = c) ...
```

## `const` 成员函数

1. 使  `class` 接口比较容易理解
2. 使操作 `const` 对象成为可能

```cpp
class TextBlock {
private:
    std::string text;
public:
    ...
    const char& operator[] (std::size_t position) const
    { return text[position]; }
    char& operator[] (std::size_t position)
    { return text[position]; }
};

TextBlock tb("Hello");
const TextBlock ctb("World");

std::cout << tb[0];   // 没问题，读一个 non-const TextBlock
tb[0] = 'x';          // 没问题，写一个 non-const TextBlock
std::cout << ctb[0];  // 没问题，读一个 const TextBlock
ctb[0] = 'x';         // 错误!!! 写一个 const TextBlock
```

### **bitwise constness** 流派

**bitwise constness** 流派认为，成员函数只要不更改对象内任何成员变量 (`static` 除外) 时才可以是 `const` 。也就是说它不更改对象内任何一个 **bit**。

```CPP
class CTextBlock {
private:
    char* pText;
public:
    ...
    char& operator[] (std::size_t position) const  // bitwise const 声明
    { return pText[position]; }                   // 不妥
};

const CTextBlock cctb("Hello");
char* pc = &cctb[0];  // 调用 const operator[] 取得一个指向 cctb 的指针
*pc = 'J';            // cctb 的内容现在变成了 “Jello”
```

### **logical constness** 流派

**logical constness** 流派主张，一个 `const` 成员函数可以修改它所处理的对象内的某些 **bits**，但只有客户端侦测不出来的情况下才能如此。

```cpp
class CTextBlock {
private:
    char* pText;
    // 使用 mutable 修饰的成员变量可能总是会被修改，即便是在 const 函数内
    mutable std::size_t testLength;
    mutable bool lengthIsValid;
public:
    ...
    std::size_t length() const {
        if ( !lengthIsVaild ) {
            textLegnth = std::strlen(pText); 
            lengthIsValid = true;
        }
        return textLength;
    }
};
```

## 在 `const` 和 `non-const` 成员函数中避免重复

```cpp
class TextBlock {
private:
    std::string text;
public:
    ...
    const char& operator[] (std::size_t position) const {
        ...  // 边界检验      (bounds checking)
        ...  // 志记数据访问   (log access data)
        ...  // 检验数据完整性 (verify data integrity)
        ...  // 等等
        return text[position];
    }
    char& operator[] (std::size_t position) {
        return const_cast<char&>(                 // 移除 operator[] 返回值的 const 
            static_cast<const TextBlock&>(*this)  // 为 *this 加上 const
                [position]                        // 调用 const operator[]
        );
    }
};
```

## Summary

- 将某些东西声明为 `const` 可帮助编译器侦测出错误用法。`const` 可被施加于任何作用域内的**对象**、**函数参数**、**函数返回类型**、**成员函数本体**
- 编译器强制实行 **bitwise constness**，但你编写程序时应该使用**概念上的常量性 (conceptual constness)**
- 当 `const` 和 `non-const` 成员函数有着实质等价的实现时，令  `non-const` 版本调用 `const` 版本可避免代码重复

---

# 确定对象被使用前已先被初始化

**C++** 并不保证变量所有情况下都会被初始化，读取未初始化的值可能会导致不明确的行为。最佳处理办法是：**永远在使用对象之前将其初始化**，与**资源获取即初始化 RAII (Resource Acquisition Is Initialization)** 异曲同工。

## 内置类型的初始化

```cpp
int x = 0;                              // 对 int 进行手工初始化
const char* text = "A C-style string";  // 对指针进行手工初始化

double d;
std::cin >> d;                          // 以读取 input stream 的方式进行初始化
```

## 构造函数

确保每一个构造函数都将对象的每一个成员初始化。

```cpp

```

