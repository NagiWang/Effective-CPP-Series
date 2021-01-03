# Effective C++

- [Effective C++](#effective-c)
  - [视 C++ 语言为一个联邦](#视-c-语言为一个联邦)
    - [多重范式 ( Multiparadigm Progamming )](#多重范式--multiparadigm-progamming-)
    - [子语言 ( Sublanguage )](#子语言--sublanguage-)
  - [尽量以 `const`、`enum`、`inline` 替换 `#define`](#尽量以-constenuminline-替换-define)
    - [常量定义](#常量定义)
    - [`class` 专属常量定义](#class-专属常量定义)
    - [宏函数](#宏函数)
  - [尽可能使用 `const`](#尽可能使用-const)

---

## 视 C++ 语言为一个联邦

### 多重范式 ( Multiparadigm Progamming )

- 过程形式 ( Procedural )
- 面向对象形式 ( Object-Oriented )
- 函数形式 ( Functional )
- 泛型形式 ( Generic )
- 元编程形式 ( Metaptogramming )

### 子语言 ( Sublanguage )

- C
- Object-Oriented C++
- Template C++
- STL

---

## 尽量以 `const`、`enum`、`inline` 替换 `#define`

### 常量定义

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

### `class` 专属常量定义

```cpp
class GamePlayer {
private:
    static const int NumTurns = 5;  // in-class 初值设定
    int scores[NumTurns];
};
```

上述代码在不支持 `in-class 初值设定` 的编译器上无法使用，可用如下方法替换

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

在不支持 `static 整数型 class 常量` 完成  `in-class 初值设定` 的编译器上，可通过 `the enum hack` 方式来进行实现

```cpp
class GamePlayer {
private:
    enum { NumTurns = 5 };  // the enum hack
    int scores[NumTurns];
};
```

`the enum hack` 的行为更像是 `#define` 而不像是 `const` ，但有时可能正是我们需要的。因为，取一个 `const` 的地址是合法的，而取一个`enum` 的地址并不合法。如果不想某个 `pointer` 或者 `reference` 指向某个整数常量，便可使用 `enum` 来实现。

### 宏函数

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

---

## 尽可能使用 `const`

- `const` 出现在 `*` 左边表示指针本身是常量，出现在 `*` 右边表示被指物是常量，写在类型之前还是之后意义相同。

- 对于 `iterator` 

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

  