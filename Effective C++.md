- [1. 视 **C++** 语言为一个联邦](#1-视-c-语言为一个联邦)
  - [1.1. 多重范式 ( Multiparadigm Progamming )](#11-多重范式--multiparadigm-progamming-)
  - [1.2. 子语言 ( Sublanguage )](#12-子语言--sublanguage-)
  - [1.3. Summary](#13-summary)
- [2. 尽量以 `const`、`enum`、`inline` 替换 `#define`](#2-尽量以-constenuminline-替换-define)
  - [2.1. 常量定义](#21-常量定义)
  - [2.2. `class` 专属常量定义](#22-class-专属常量定义)
  - [2.3. 宏函数](#23-宏函数)
  - [2.4. Summary](#24-summary)
- [3. 尽可能使用 `const`](#3-尽可能使用-const)
  - [3.1. `const` 出现在 `*` 左边表示指针本身是常量，出现在 `*` 右边表示被指物是常量，写在类型之前还是之后意义相同](#31-const-出现在--左边表示指针本身是常量出现在--右边表示被指物是常量写在类型之前还是之后意义相同)
  - [3.2. 对于 **iterator**](#32-对于-iterator)
  - [3.3. 令函数返回一个常量值，往往可以降低因客户错误而造成的意外，而又不至于放弃安全性和高效性。](#33-令函数返回一个常量值往往可以降低因客户错误而造成的意外而又不至于放弃安全性和高效性)
  - [3.4. `const` 成员函数](#34-const-成员函数)
    - [3.4.1. **bitwise constness** 流派](#341-bitwise-constness-流派)
    - [3.4.2. **logical constness** 流派](#342-logical-constness-流派)
  - [3.5. 在 `const` 和 `non-const` 成员函数中避免重复](#35-在-const-和-non-const-成员函数中避免重复)
  - [3.6. Summary](#36-summary)
- [4. 确定对象被使用前已先被初始化](#4-确定对象被使用前已先被初始化)
  - [4.1. 内置类型的初始化](#41-内置类型的初始化)
  - [4.2. 构造函数](#42-构造函数)
  - [4.3. 定义在不同编译单元内的 **non-local static** 对象](#43-定义在不同编译单元内的-non-local-static-对象)
    - [4.3.1. Singleton](#431-singleton)
  - [4.4. Summary](#44-summary)

---

# 1. 视 **C++** 语言为一个联邦

## 1.1. 多重范式 ( Multiparadigm Progamming )

- 过程形式 ( Procedural )
- 面向对象形式 ( Object-Oriented )
- 函数形式 ( Functional )
- 泛型形式 ( Generic )
- 元编程形式 ( Metaptogramming )

## 1.2. 子语言 ( Sublanguage )

- C
- Object-Oriented C++
- Template C++
- STL

## 1.3. Summary

- **C++** 高效编程守则视情况而变化，取决于你使用 **C++** 的哪一个部分

---

# 2. 尽量以 `const`、`enum`、`inline` 替换 `#define`

## 2.1. 常量定义

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

## 2.2. `class` 专属常量定义

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

## 2.3. 宏函数

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

或者用 `lambda` 函数 (可能有一定的调用开销)

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

## 2.4. Summary

- 对于单纯常量，最好以 `const` 对象或 `enums` 替换 `#define`
- 对于形似函数的宏 (`macros`)，最好改用 `inline` 函数替换 `#define`

---

# 3. 尽可能使用 `const`

## 3.1. `const` 出现在 `*` 左边表示指针本身是常量，出现在 `*` 右边表示被指物是常量，写在类型之前还是之后意义相同

```cpp
const int* a;
int const* b;
```

## 3.2. 对于 **iterator**

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

## 3.3. 令函数返回一个常量值，往往可以降低因客户错误而造成的意外，而又不至于放弃安全性和高效性。

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

## 3.4. `const` 成员函数

- 使  `class` 接口比较容易理解
- 使操作 `const` 对象成为可能

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

### 3.4.1. **bitwise constness** 流派

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

### 3.4.2. **logical constness** 流派

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

## 3.5. 在 `const` 和 `non-const` 成员函数中避免重复

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

## 3.6. Summary

- 将某些东西声明为 `const` 可帮助编译器侦测出错误用法。`const` 可被施加于任何作用域内的**对象**、**函数参数**、**函数返回类型**、**成员函数本体**
- 编译器强制实行 **bitwise constness**，但你编写程序时应该使用**概念上的常量性 (conceptual constness)**
- 当 `const` 和 `non-const` 成员函数有着实质等价的实现时，令  `non-const` 版本调用 `const` 版本可避免代码重复

---

# 4. 确定对象被使用前已先被初始化

**C++** 并不保证变量所有情况下都会被初始化，读取未初始化的值可能会导致不明确的行为。最佳处理办法是：**永远在使用对象之前将其初始化**，与**资源获取即初始化 RAII (Resource Acquisition Is Initialization)** 异曲同工。

## 4.1. 内置类型的初始化

```cpp
int x = 0;                              // 对 int 进行手工初始化
const char* text = "A C-style string";  // 对指针进行手工初始化

double d;
std::cin >> d;                          // 以读取 input stream 的方式进行初始化
```

## 4.2. 构造函数

确保每一个构造函数都将对象的每一个成员初始化。首选使用 **成员初值列 (member initialization list)** 进行初始化操作，可避免某些不必要的复制操作。

```cpp
class PhoneNumber {};
class ABEntry {
private:
    std::string theName;
    std::string theAddress;
    std::list<PhoneNumber> thePhones;
    int numTimesConsulted;
public:
    ABEntry(const std::string& name, const std::string& address,
            const std::list<PhoneNumber>& phones)
        : theName{name},           // 调用 theName 的 default 构造函数
          theAddress{address},     // 为 theAddress 做类似的操作
          thePhones{phones},       // 为 thePhones 做类似的操作
          numTimesConsulted(0)     // 记得将 numTimesConsulted 显式初始化为 0
    { }                            // 初始化顺序以声明顺序为准
};
```

## 4.3. 定义在不同编译单元内的 **non-local static** 对象

**C++** 对 **定义在不同编译单元内的 non-local static 对象** 的初始化次序无明确定义，且无解。

```cpp
class FileSystem {
public:
    ...
    std::size_t numDisks() const;
    ...
};
extern FileSystem tfs;

class Directory {
public:
    Directory(auto... params) {
        ...
        std::size_t disks = tfs.numDisks(); // tfs 不一定在 Directory 初始化之前完成初始化
        ...
    }
};

Directory tempDir(params...);
```

### 4.3.1. Singleton

将 **non-local static** 对象转变为 **local static** 对象，可通过以下方式改写

```cpp
class FileSystem { ... };
FileSystem& tfs() {
    static FileSystem fs;  // local static 对象
    return fs;
}

class Directory {
public:
    Directory(auto... params) {
        ...
        std::size_t disks = tfs().numDisks();  // 调用包含 local static 对象的函数
        ...
    }
};

Directory& tempDir() {
    static Directory td;  // local static 对象
    return td;
}
```

但这并不是最好的方法，因为上述方法完全没有考虑如何避免创建多个 `FileSystem` 对象，这在 **Singleton** 模式中是致命的。我们可通过以下方法来避免

```cpp
struct FileSystem {
protected:
    FileSystem() { ... }        // 将构造函数隐藏
public:
    static FileSystem& get() {
        // thread-safe in C++11
        static FileSystem fs;   // 内部函数可访问构造函数
        return fs;
    }
    FileSystem(FileSystem&&) = delete;
    FileSystem(FileSystem const&) = delete;
    FileSystem& operator=(FileSystem&&) = delete;
    FileSystem& operator=(FileSystem const&) = delete;
};

class Directory {
public:
    Directory(auto... params) {
        ...
        std::size_t disks = FileSystem::get();  // 如此使用
        ...
    }
};

Directory& tempDir() {
    static Directory td;
    return td;
}
```

## 4.4. Summary

- 为内置类型对象进行手工初始化，因为 **C++** 不保证初始化它们
- 构造函数最好使用**成员初值列 (member initialization list)**，而不要在构造函数本体里使用赋值操作；**初值列**中列出的成员变量初始化顺序为它们在 `class` 中的声明顺序
- 为避免**定义在不同编译单元内的 non-local static 对象初始化次序问题** ，使用 **local static 对象**替换**non-local static 对象**，即 **Singleton** 模式

---
