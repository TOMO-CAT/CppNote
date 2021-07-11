# C++11关键字：override和final

## 场景

在传统C++中，经常容易发现意外重载虚函数的事情：

```c++
struct Base {
    virtual void foo();
};

struct SubClass: Base {
    void foo();
};
```

有下列三种非预期的场景：

* `SubClass::foo`可能是程序员加入的一个和基类虚函数恰好同名的成员函数，却被编译器当作重载虚函数
* `SubClass::foo`可能是程序员想重载虚函数，但是因为形参列表不同导致编译器认为这是一个新定义的成员函数
* 当基类的虚函数`Base::foo`被删除后，`SubClass::foo`就不再重载该虚函数而摇身一变成为一个普通的成员函数

## override

一旦类中的某个函数被声明为虚函数，那么在所有的派生类中它都是虚函数。一个派生类的函数如果覆盖了某个继承而来的虚函数，那么它的形参类型必须与基类函数完全一致。

> 派生类中如果定义了一个函数与基类中虚函数的名字相同但是形参列表不同，编译器会认为新定义的函数与基类中原有的函数是相互独立的。这会带来一个问题：如果我们本来希望派生类可以覆盖掉基类中的虚函数，但是一不小心把形参列表写错了，这可能与我们的本意不符。

C++11新标准提供了`override`关键字来显式地告知虚拟器进行重载，编译器将检查基类是否存在这样的虚函数，否则将无法通过编译。这样的好处是使得程序员的意图更加清晰（覆盖基类中的虚函数），如果我们使用`override`关键字标记了某个函数但是该函数没有覆盖已有的虚函数，此时编译器会报错。

```c++
struct Base {
    virtual void foo(int);
};

struct SubClass: Base {
    virtual void foo(int) override;    // 合法
    virtual void foo(float) override;  // 非法, 父类无此虚函数
};
```

## final

我们可以把类中的某个函数指定为`final`，之后任何尝试覆盖该函数的操作都会引发错误，用于防止类被继续继承或者终止虚函数继续重载。

```c++
struct Base {
    virtual void foo() final;
};

struct SubClass1 final: Base {  
};  // 合法

struct SubClass2: SubClass1 {
};  // 非法, SubClass1已final

struct SubClass3: Base {
    void foo();  // 非法, foo已final
};
```



