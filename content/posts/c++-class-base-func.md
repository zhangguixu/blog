---
title: 【c++系列】类的基本操作
date: 2022-02-26
category: Programming language
tags: 
    - c++
---

## 类基础操作

- Constructor 默认构造
  - Copy Contructor 构造复制
- Destructor 析构
- Assignment Operators 复制操作

示例，如下：

```c++
class Test
{
  public:
    Test(): code(1)
    {
      cout << "Test" << endl;
    }
  	// copy构造函数
    Test (const Test &rhs)
    {
      code = rhs.code;
      cout << "copy" << endl;
    }
   // copy assignment
    Test& operator=(const Test &rhs)
    {
      // 处理自我赋值
      if (this == &rhs)
      {
        return *this;
      }
      code = rhs.code;
      cout << "operator =" << endl;
      return *this;
    }
    ~Test()
    {
      cout << "~Test" << endl;
    }
    int code;
};

int main()
{
  Test t1;
  t1.code = 2;
  Test t2(t1); // copy
  cout << t2.code << endl;
  t2.code = 3;
  t1 = t2; // operator =
  cout << t1.code << endl;
  Test t3 = t1; // 注意！这里是调copy构造函数
}
```

当我们声明一个空类时，编译器会默认添加上这些`缺失`的基础操作函数，如果有了则编译器不会产生了。

- default copy constructor，会将来源对象的每一个`non-static`成功变量拷贝到目标对象中
- default desctructor，会成一个`non-virtual`析构函数

### 编译器无法生成copy assignment函数的case

当class中有以下两种情况：

- 引用类型的成员变量
- const的成员变量

编译器不会为class生成默认的`copy assginment`。因此当class不自行实现`copy assignement`函数，无法实现赋值拷贝。

```c++
class Test
{
  private:
  	const int i;
  	string &name;
}
```

> 编译器生成的copy constructor，会让多个对象的引用成员变量引用同一个变量

### noncopyable

希望每一个对象都是独一无二，禁止对象发生拷贝动作。

见[noncopyable](/blog/posts/c++-noncopyable/)

### Non-virtual Destructor

非`virtual`的析构函数，可能会导致的问题：

在继承中，需要将base class的析构函数声明为`virtual`属性，因为只有这样才可以正确的调用的析构函数。

```c++
class Base
{
  public:
  	Base()
    {
      cout << "Base" << endl;
    }
  	~Base()
    {
      cout << "~Base" << endl;
    }
}
class Test: public Base
{
  public:
  	Test()
    {
      cout << "Test" << endl;
    }
  	~Test()
    {
      cout << "~Test" << endl;
    }
}

{
  Base *bPtr = new Test(); // Base Test
  delete bPtr; // ~Base，无法正确的析构，析构函数virtual
}
```

> c++没有提供`禁止派生`的机制。

### Destructor的异常处理

析构函数绝对不要吐出任何异常，因为析构函数的异常无法被处理，可能会导致程序直接结束。

### operator=

1. 检测`自我赋值`，在某些情况下，自我赋值可能会导致程序异常，例如以下示例

```c++
class Test
{
  public:
    Test()
    {
      i = new int(1);
    }
    Test& operator=(const Test &rhs)
    {
      delete i;
      i = new int(*rhs.i);
      return *this;
    }
    ~Test()
    {
      delete i;
    }
  int *i;
};

{
  Test t;
  // 没有自我赋值检测，导致程序core dump
  t = t; 
}
```

2. 此外，还需要考虑另外一个问题`exception safety(异常安全性)`

> 这个问题所有申请资源的地方都需要考虑，例如构造函数

在以下的case，可以先保存原有的资源，等待新的资源申请成功之后，再删除旧的资源。

```c++
Widget& widget::operator=(const Widget& rhs)
{
  // 检测自我赋值
  if (this == &rhs)
  {
    return *this;
  }
  // 先记住原来的pb
  Bitmap *pOrig = pb; 
  // 令pb指向*pb的一个副本
  pb = new Bitmap(*rhs.pb); 
  delet pOrig;
  return *this;
}
```

### copy all part of an object

如果声明自己的copying函数，必须保证

- 将被拷对象的所有成员变量都做一份拷贝
- derived class的copying函数调用相应的base class函数

```c++
class Base
{
  public:
    Base(): j(1)
    {}
    int j;
};
class Test: public Base
{
  public:
    Test(): i(1)
    {}
    Test& operator=(const Test &rhs)
    {
      if (this == &rhs)
      {
        return *this;
      }
      // 对base class成分进行赋值动作
      // 由于编译器会默认生成operator=函数，因此即使自己没有声明这个函数，也可以调用
      Base::operator=(rhs);
      i = rhs.i;
      return *this;
    }
    int i;
};
```