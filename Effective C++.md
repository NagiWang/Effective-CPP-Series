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
  - [3.1. `const` 出现在 `*` 左或右意义相同](#31-const-出现在--左或右意义相同)
  - [3.2. 对于 **iterator**](#32-对于-iterator)
  - [3.3. 令函数返回一个常量值](#33-令函数返回一个常量值)
  - [3.4. `const` 成员函数](#34-const-成员函数)
    - [3.4.1. **bitwise constness** 流派](#341-bitwise-constness-流派)
    - [3.4.2. **logical constness** 流派](#342-logical-constness-流派)
  - [3.5. 在 `const` 和 `non-const` 成员函数中避免重复](#35-在-const-和-non-const-成员函数中避免重复)
  - [3.6. Summary](#36-summary)
- [4. 确定对象被使用前已先被初始化](#4-确定对象被使用前已先被初始化)
  - [4.1. 内置类型的初始化](#41-内置类型的初始化)
  - [4.2. **构造函数**](#42-构造函数)
  - [4.3. 定义在不同编译单元内的 **non-local static** 对象](#43-定义在不同编译单元内的-non-local-static-对象)
    - [4.3.1. Singleton](#431-singleton)
  - [4.4. Summary](#44-summary)
- [5. 了解 **C++** 默默编写并调用哪些函数](#5-了解-c-默默编写并调用哪些函数)
  - [5.1. **empty class** 不再是 **empty class**](#51-empty-class-不再是-empty-class)
  - [5.2. **default 构造函数** 和 **析构函数**](#52-default-构造函数-和-析构函数)
  - [5.3. **copy 构造函数** 和 **copy assignment 操作符**](#53-copy-构造函数-和-copy-assignment-操作符)
  - [5.4. Summary](#54-summary)
- [6. 若不想使用编译器自动生成的函数，应当明确拒绝](#6-若不想使用编译器自动生成的函数应当明确拒绝)
  - [6.1. 声明为 `private`](#61-声明为-private)
  - [6.2. 声明为 `delete`](#62-声明为-delete)
  - [6.3. 继承自 **Uncopyable**](#63-继承自-uncopyable)
  - [6.4. Summary](#64-summary)
- [7. 为多态基类声明 **virtual 析构函数**](#7-为多态基类声明-virtual-析构函数)
  - [7.1. 多态基类](#71-多态基类)
  - [7.2. **抽象类 (abstract classes)**](#72-抽象类-abstract-classes)

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

- **C++** 高效编程守则视情况而变化，取决于你使用 **C++** 的哪一个部分。

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

- 对于单纯常量，最好以 `const` 对象或 `enums` 替换 `#define`。
- 对于形似函数的宏 (`macros`)，最好改用 `inline` 函数替换 `#define`。

---

# 3. 尽可能使用 `const`

## 3.1. `const` 出现在 `*` 左或右意义相同

`const` 出现在 `*` 左边表示指针本身是常量，出现在 `*` 右边表示被指物是常量，写在类型之前还是之后意义相同。

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

## 3.3. 令函数返回一个常量值

令函数返回一个常量值，往往可以降低因客户错误而造成的意外，而又不至于放弃安全性和高效性。

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

- 将某些东西声明为 `const` 可帮助编译器侦测出错误用法。`const` 可被施加于任何作用域内的**对象**、**函数参数**、**函数返回类型**、**成员函数本体**。
- 编译器强制实行 **bitwise constness**，但你编写程序时应该使用**概念上的常量性 (conceptual constness)**。
- 当 `const` 和 `non-const` 成员函数有着实质等价的实现时，令  `non-const` 版本调用 `const` 版本可避免代码重复。

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

## 4.2. **构造函数**

确保每一个**构造函数**都将对象的每一个成员初始化。首选使用 **成员初值列 (member initialization list)** 进行初始化操作，可避免某些不必要的复制操作。

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
        std::size_t disks = FileSystem::get().numDisks();  // 如此使用
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
- 构造函数最好使用**成员初值列 (member initialization list)**，而不要在构造函数本体里使用赋值操作；**初值列**中列出的成员变量初始化顺序为它们在 `class` 中的声明顺序。
- 为避免**定义在不同编译单元内的 non-local static 对象初始化次序问题** ，使用 **local static 对象**替换**non-local static 对象**，即 **Singleton** 模式。

---

# 5. 了解 **C++** 默默编写并调用哪些函数

## 5.1. **empty class** 不再是 **empty class**

```cpp
class Empty {};
```

**C++** 处理之后等价于你写了如下代码

```cpp
class Empty {
public:
    Empty() = default;                             // default 构造函数
    ~Empty() = default;                            // default 析构函数
    Empty(Empty&& rhs) = default;                  // default move 构造函数 C++ 1x
    Empty(const Empty& rhs) = default;             // default copy 构造函数
    Empty& operator=(Empty&& rhs) = default;       // default move assigniment 操作符 C++ 1x
    Empty& operator=(const Empty& rhs) = default;  // default copy assigniment 操作符
};
```

如果你没自己声明，**C++** 默默为你声明 **copy 构造函数**、**copy assignment 操作符**、**析构函数**。此外，如果你没有声明任何**构造函数**，**C++** 便会为你声明一个 **default 构造函数**。所有这些函数都是 `public` 且 `inline` 的。若你自己同时没声明 **copy 构造函数**、**copy assignment 操作符**、**析构函数** 三者，**C++** 还会为你声明 **move 构造函数** 和 **move assignment 操作符** *（留在 **Effective Modern C++** 中讨论）*。

## 5.2. **default 构造函数** 和 **析构函数**

**default 构造函数** 和 **析构函数** 主要是给编译器一个地方来放置*藏身幕后*的代码，像是调用 **base classes** 和 **non-static** 成员变量的**构造函数**和**析构函数**。注意编译器产出的**析构函数**是个 **non-virtual**，除非该 **class** 的 **base class** 自身声明了 **virtual 析构函数**。

##  5.3. **copy 构造函数** 和 **copy assignment 操作符**

**copy 构造函数** 和 **copy assignment 操作符**，编译器创建的版本也只是单纯的把来源对象中的每一个 **non-static** 成员变量拷贝到目标对象。

- 如果你打算在一个 *内含 reference 成员* 或 *内含 const 成员* 的 **class** 内支持**赋值操作**，你必须自己定义 **copy assignment 操作符**。
- 如果某个 **base classes** 将 **copy assignment 操作符** 声明为 `private` ，编译器将拒绝为 **derived classes** 生成一个 **copy assignment 操作符**。

## 5.4. Summary

- 编译器暗自会为 `class` 创建 **default 构造函数**、**copy 构造函数** 和 **copy assignment 操作符**，以及**析构函数** *(有时还包括 **move 构造函数** 和 **move assignment 操作符**)*。

---

# 6. 若不想使用编译器自动生成的函数，应当明确拒绝

通常如果你不希望 `class` 支持某一特定机能，只要不声明对应的函数即可。但这个策略对**copy 构造函数** 和 **copy assignment 操作符**并不起作用[条款 5](#5-了解-c-默默编写并调用哪些函数)已经指出。如果你不希望你的 `class` 被复制，以下几个方法可以避免此问题

## 6.1. 声明为 `private`

```cpp
// C++ 0x
class HomeForSale {
private:
    HomeForSale(const HomeForSale&);             // 复制构造函数
    HomeForSale& operator=(const HomeForSale&);  // 复制赋值运算符
public:
    ...
};
```

## 6.2. 声明为 `delete`

```cpp
// C++ 11 起
class HomeForSale {
public:
    HomeForSale(const HomeForSale&) = delete;             // 弃置复制构造函数
    HomeForSale& operator=(const HomeForSale&) = delete;  // 弃置复制赋值运算符
    ...
};
```

## 6.3. 继承自 **Uncopyable**

```cpp
// C++ 11 起
class Uncopyable {
protected:
    Uncopyable() = default;                             // 子类可调用 构造函数 和 析构函数
    ~Uncopyable() = default;                            // 由于 Uncopyable 不含数据成员，所以不必是 virtual
public:
    Uncopyable(const Uncopyable&) = delete;             // 复制构造函数
    Uncopyable& operator=(const Uncopyable&) = delete;  // 复制赋值运算符
};

class HomeForSale : private Uncopyable {                // private 继承
public:
    ...
};
```

如果想要在 **C++ 0x** 或 **C++ 1x** 中都能通过继承某个 **class** 达到阻止编译器生成 **copy 构造函数** 和 **copy assignment 操作符** 的目的的话，可转向使用 `boost::noncopyable`，其定义如下 *(简化版)*

```cpp
class noncopyable {
protected:
#if !defined(BOOST_NO_CXX11_DEFAULTED_FUNCTIONS) && !defined(BOOST_NO_CXX11_NON_PUBLIC_DEFAULTED_FUNCTIONS)
    BOOST_CONSTEXPR noncopyable() = default;  // 根据 C++ 版本不同，默认构造函数可能为 constexpr
    ~noncopyable() = default;
#else
    noncopyable() {}
    ~noncopyable() {}
#endif
#if !defined(BOOST_NO_CXX11_DELETED_FUNCTIONS)
    noncopyable(const noncopyable&) = delete;
    noncopyable& operator=(const noncopyable&) = delete;
#else
private:// emphasize the following members are private
    noncopyable(const noncopyable&);
    noncopyable& operator=(const noncopyable&);
#endif
};
```

使用方法也差不多

```cpp
#include <boost/core/noncopyable.hpp>
class HomeForSale: private boost::noncopyable {
public:
    ...
};
```

**Uncopyable** *(或  `boost::noncopyable`)* 具备以下几个特点

- 不一定以 `public` 继承它。
- 不含数据
- **析构函数** 不一定得是 `virtual`

## 6.4. Summary

- 为驳回编译器 *(暗自)* 提供的机能，可将相应的成员函数声明为 `private` 并且不予实现 *(C++ 1x 中通常通过 `delete` 方式)* 。
- 使用像 **Uncopyable** 这样的 **base class** 也可以。

---

# 7. 为多态基类声明 **virtual 析构函数**

## 7.1. 多态基类 (polymorphic base classes)

```cpp
class TimeKeeper {  // base class
public:
    TimeKeeper();
    ~TimeKeeper();  // 非 virtual 析构函数
    ...
};

class AtomicClock : public TimeKeeper { ... };  // drived class

TimeKeeper* ptk = new AtomicClock();
delete ptk;        // 未定义行为，可能错误！
```

上述代码中，删除 `ptk` 后会造成一个诡异的 **局部销毁** 的对象，这可能是资源泄露、败坏数据结构、在调试期上浪费许多时间的绝佳途径。

消除这一问题的做法很简单：给 **base class** 一个 **virtual 析构函数**

```cpp
class TimeKeeper {  // base class
public:
    TimeKeeper();
    virtual ~TimeKeeper();  // 非 virtual 析构函数
    ...
};

class AtomicClock : public TimeKeeper { ... };  // drived class

TimeKeeper* ptk = new AtomicClock();
delete ptk;        // 正确
```

但令所有 **class** 的 **析构函数** 都为 `virtual` 也并不是一个好主意。欲实现 **virtual 函数** ，对象必须携带某些信息来决定在运行期应该调用哪一个 **virtual 函数**。这份信息由一个 **vptr (virtual table pointer)** 指针指出。**vptr** 指向一个由函数指针组成的数组 **vtbl (virtual table)**。每个带有 **virtual 函数** 的 `class` 都有一个 **vtbl**，这意味着 **vtbl** 将会占用额外的存储空间。

- 只有当 `class` 内含有至少一个 **virtual 函数** 时才为它声明 **virtual 析构函数**。
- 不要企图继承自一些 **STL 容器** *(vector, list, set, unordered_map, etc.)*，因为他们没有声明 **virtual 析构函数**。

## 7.2. 抽象类 (abstract classes)

含有一个及以上的 **pure virtual 函数** 的 `class` 被称为 **抽象类**，它不能被 **实体化 (instantiated)**。
