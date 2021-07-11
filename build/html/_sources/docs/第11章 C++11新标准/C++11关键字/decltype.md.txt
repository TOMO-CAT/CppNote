# C++11关键字：decltype

## 简介

C++11引入的关键字decltype允许我们从表达式的类型推断出要定义的变量类型，编译器会分析表达式并得到的类型，却不实际计算该表达式的值。

## decltype处理顶层const和引用

> Tips：引用从来都是作为其所指对象的同义词出现的，只有用在decltype处是一个例外。

decltype处理顶层const和引用的方式与auto稍有不同，如果decltype使用的表达式是一个变量，那么decltype会返回该变量的类型（包括顶层const和引用在内）：

```c++
const int ci = 0;
const int &cj = ci;

decltype(ci) x = 0;  // x的类型是const int
decltype(cj) y = x;  // y的类型是const int&, y绑定到x
decltype(cj) z;      // 错误: z是一个引用, 必须初始化
```

如果decltype使用的表达式不是一个变量，则decltype返回表达式结果对应的类型（后面会提到有一些表达式返回左值，会向decltype返回一个引用类型）。前面提到decltype作用于一个变量时得到的结果就是该变量的类型，但是如果我们给变量加上一层或多层括号，编译器就会把它当做一个表达式得到引用类型：

> Tips：变量是一种可以作为赋值语句左值的特殊表达式，所以`decltype((variable))`的结果永远都是引用，而`decltype(variable)`的结果只有当`variable`本身是一个引用时才是引用。

```c++
int i = 42;

decltype(i) a;    // 正确: a是一个未初始化的int
decltype((i)) b;  // 错误: b的类型是int&, 必须初始化
```

## decltype与左值右值

如果表达式的求值结果是左值，那么关键字decltype作用于该表达式（不是变量）会得到一个引用类型。举个例子：

```c++
int *pi;

int a;
decltype(*pi) x = a;  // 解引用符生成左值, 因此decltype(*pi)结果是int&, 引用类型必须初始化
decltype(&pi) y;      // 取地址符生成右值, 因此decltype(&p)的结果是int**
```

