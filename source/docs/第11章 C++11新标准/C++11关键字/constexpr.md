# C++11关键字：constexpr

## 简介

C++11引入了常量表达式constexpr的概念，指的是值不会改变并且在**编译期间**就能得到计算结果的表达式。

```c++
const int i = 10;          // 常量表达式
const int j = i + 1;       // 常量表达式
const int k = size();      // 仅当size()是一个constexpr函数时才是常量表达式, 运行时才能获得具体值就不是常量表达式
```

在一个复杂系统中，我们很难分辨一个初始值是否是常量表达式，通过constexpr关键字声明一个变量，我们可以让编译器来验证变量的值是否是一个常量表达式。

## 字面值是常量表达式

算术类型、引用和指针都属于字面值类型，自定义类则不属于字面值类型，因此也无法被定义为constexpr。

> Tips：尽管指针和引用都能被定义成constexpr，但它们的初始值却受到严格限制。一个constexpr指针的初始值必须是nullptr、0或者是存储于某个固定地址中的对象。

## constexpr是对指针的限制

在constexpr声明中定义了一个指针，限定符constexpr仅对指针有效，与指针所指对象无关：

```c++
const int *pi1 = nullptr;      // 底层const: pi1是指向整型常量的普通指针
constexpr int *pi2 = nullptr;  // 顶层const: pi2是指向整型的常量指针
```

我们也可以让constexpr指针指向常量：

```c++
constexpr int i = 10;
constexpr const int *pi = &i;  // 顶层const + 底层const 
```

