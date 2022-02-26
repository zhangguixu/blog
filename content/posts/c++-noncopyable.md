---
title: 【c++系列】noncopyable
date: 2022-02-26
category: Programming language
tags: 
    - c++
---

## 

### 什么情况下发生拷贝

```c++
Foo foo, foo2; 
Foo foo2 = foo;  // 等号拷贝
Foo foo3(foo);   // 构造拷贝
```

传值拷贝和以值返回的函数时，都会发生构造拷贝

```c++
Foo func(Foo foo) 
{
  Foo foo2;
  // do somthing    
  return foo2; 
}
```

### **什么时候需要不可拷贝**

类之间占用的内存空间不相等，不可拷贝，见

```c++
/// 测试1：等号拷贝 
void copy()
{    
  Matrix<double> m1(3,3);
  Matrix<double> m2(3,3);
  m1 = m2;  
}
```

## **如何实现不可拷贝类**

1. 声明拷贝函数为私有

```c++
class nonconpyable
{
  private:  // emphasize the following members are private    
		noncopyable( const noncopyable& );    
		noncopyable& operator=( const noncopyable& );
}
```

2. C++ 11 中为不可拷贝类提供了更简单的实现方法，使用 delete 关键字即可：

```c++
template<typename _T> 
class Matrix
{
  public: 
  	Matrix(const Matrix<_T>&) = delete;   
    Matrix<_T>& operator = (const Matrix<_T>&) = delete; 
}
```

## 参考资料

https://fzheng.me/2016/11/20/cpp_noncopyable_class/

