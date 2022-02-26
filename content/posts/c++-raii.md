---
title: 【c++系列】RAII
date: 2022-02-26
category: Programming language
tags: 
    - c++
---


## RAII

使用对象管理资源，指的是

- 资源取得时机便是初始化时机`(Resource Acquisition Is Initialization)`
- 运用析构函数确保资源被释放

>  `RAII`对象（资源对象）

### 小心copying行为

在资源管理类中，需要特别小心拷贝行为。

通常，管理一个`RAII对象`，有两种选择：

- 禁止复制，将copying操作声明为`private`
- 对底层资源使用`引用计数法(reference count)`，例如share_ptr
- 复制底部资源，这种情况，通常需要深拷贝`(deep copying)`
- 转移底部资源的拥有权，例如unique_ptr

### 获取原始资源的API

例如unique_ptr会提供获取原始指针的API

```c++
unique_ptr<Test> t(new Test());
// 获取指针
Test *tmp = t.get();
```

> RAII classes并不是为了封装某物而存在，而是为了确保一个特殊行为：资源释放一定会发生。
>
> 因此获取原始资源的API的设计虽然看起来有点别扭，但是在实践中，却是有必要的。

### 指针资源释放

在使用数组指针的时候，需要特别注意释放，需要使用`delete []`，见示例

```c++
std::string *arr = new std::string[100];
delete arr; // 错误，可能仅仅只会释放数组的第一个元素的对象
delete []arr; // 正确释放空间
```

如果不是`new []`，就不要使用`delete []`，否则也会产生未知的行为。

> 其实最好的方式就是使用`vector`之类的标准程序库提供的库。

### 独立语句初始化智能指针

一个特殊的case，可能引发内存泄漏。

```c++
// 获取优先级，可能会有抛异常的情况
int priority();
processWidget(share_ptr<Widget> pw, int priority);

// 以下代码可能在极端情况下，可能会有问题
processWidget(share_ptr<Widget>(new Widget), priority());
```

语句的执行，包含以下三个步骤

- 调用`priority()`，标记为`a`
- 执行`new Widget`，标记为`b`
- 调用`share_ptr`的构造函数，标记为`c`

三个步骤的执行顺序，若为`b->a->c`，且a步骤异常了，b步骤的指针就会遗失，导致指针未被释放。

最保险的做法就是：

```c++
share_ptr<Widget> pw(new Widget);
process(pw, priority());
```

