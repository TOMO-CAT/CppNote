# C++11关键字：noexcept

## 简介

> Tips：`noexcept`说明要么出现在该函数的所有声明语句和定义语句中，要么一次也不出现。

对于用户以及编译器而言，预先知道某个函数不会抛出异常是有利的：

* 有利于简化调用该函数的代码
* 可以执行某些特殊的优化操作，这些优化操作通常并不适用于可能抛出异常的代码（比如`vector`的`push_back()`操作）

在C++11新标准中，我们可以通过`noexcept`说明来指定某个函数不会抛出异常：

```c++
// recoup不会抛出异常
void recoup(int) noexcept;
// 等价写法
void recoup(int) noexcept(true);

// alloc可能抛出异常
void alloc(int);
// 等价写法
void alloc(int) noexcept(false);
```

## 违反异常说明

需要注意的是编译器不会再编译时检查`noexcept`说明：即使一个函数在说明`noexcept`同时又含有`throw`语句或者调用了可能抛出异常的其他函数，它也可能顺利编译通过。

```c++
// 尽管该函数明显违反了noexcept说明, 它也可能顺利编译通过
void f() noexcept {
    throw exception();
}
```

一旦一个`noexcept`函数抛出了异常，程序就会调用`terminate`以确保遵守不在运行时抛出异常的承诺，因此`noexcept`可以用在两种情况下：

* 我们确认该函数不会抛出异常
* 我们根本不知道该如何处理异常

## noexcept运算符

> Tips：与`sizeof()`类似，`noexcept()`也不会求其运算对象的值。

`noexcept`说明符的实参常常与`noexcept`运算符混合使用，用于判断给定的表达式是否会抛出异常。

```c++
// e是一个表达式, 当e调用的所有函数都做了noexcept说明时返回true
noexcept(e);
```

举个例子：

```c++
#include <iostream>
#include <exception>

void foo() noexcept {}
void foo2() {}
void bar() noexcept {
    throw std::exception();
}
void bar2() {
    throw std::exception();
}

int main() {
    std::cout << "noexcept(foo()): " << noexcept(foo()) << std::endl;
    std::cout << "noexcept(foo2()): " << noexcept(foo2()) << std::endl;
    std::cout << "noexcept(bar()): " << noexcept(bar()) << std::endl;
    std::cout << "noexcept(bar2()): " << noexcept(bar2()) << std::endl;

    return 0;
}

// 输出:
noexcept(foo()): 1
noexcept(foo2()): 0
noexcept(bar()): 1
noexcept(bar2()): 0
```

## noexcept与移动操作

由于移动操作通常是“窃取”资源而不分配资源，因此移动操作不会抛出任何异常。当编写不抛出异常的移动构造函数和移动赋值运算符时，我们必须在类头文件的声明和定义中都指定为`noexcept`来通知标准库我们的移动操作不会抛出异常，防止标准库为了处理抛出异常的可能性而做一些浪费性能的额外工作。

比如标准库`vector`承诺如果我们调用`push_back()`时发生异常，则`vector`自身不会发生改变。假设`push_back()`时触发了`vector`扩容，此时`vector`会将元素从旧的堆空间复制到新申请的堆空间，考虑移动构造函数和拷贝构造函数：

* 移动构造函数：假设移动构造函数未声明成`noexcept`的且移动部分而非全部元素后抛出了异常，此时使用旧空间中移后源对象的值是不安全的而新空间中未构造的元素还不存在，这种情况下不能满足`vecotr`自身不变的要求
* 拷贝构造函数：假设`vector`使用拷贝构造函数且在拷贝部分元素后发生了异常，虽然新空间中未构造的元素还不存在但旧空间的元素保持不变，`vector`可以释放新分配（但还未成功构造的）内存并返回

> Tips：为了避免潜在的问题，诸如`push_back()`等的标准库函数除非知道元素类型的移动构造函数不会抛出异常，否则在重新分配内存拷贝元素的过程中，它就必须使用拷贝构造函数而不是移动构造函数（这会造成一定的性能浪费）。如果希望在这些情况下对我们自定义类型对象进行移动而不是拷贝，就必须显式通过`noexcept`声明告诉标准库我们的移动构造函数是异常安全的。


