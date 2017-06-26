---
layout: post
title: C++中的explicit关键字
date: 2017-06-19
category: C++语言
tag: C++类
author: Feng Yuan
---

* content
{: toc}



`explicit` 关键字主要用于修饰类的构造函数，限定此构造函数只能用于对应类的显示初始化，而不能进行隐式类型转换。隐式类型转换多发生在赋值，函数参数传递等过程中，对于类来说，如果没有明确定义类型转换 `X::operator T()`，那么非 `explicit` 构造函数缺省具备这样的转换能力。

例如，没有`explicit`关键字的情形，可以进行隐式类型转换：

```c++
class Base1{
public:
    //Constructors
    Base1(){}; //Default Constructor
    Base1(int x, int y, int z):a{x},b{y},c{z}{std::cout<<"Constructing..."<< endl;}
private:
    int a,b,c; //Members};

void fun(const Base1 & tmp){}

int main() {
    Base1 A{1,2,3};         //OK，显式初始化
    Base1 B=Base1{2,3,4};   //OK，拷贝初始化，显式调用构造函数
    Base1 C={3,4,5};        
    //OK，拷贝初始化，隐式转换，将{3,4,5}这个list转换成Base1类型，再拷贝给C
    fun({7,8,9});           //OK，函数参数传递，隐式类型转换
    return 0;
    }
```

如果使用`explicit`关键字声明了构造函数，只可以通过显式初始化：

```c++
class Base1{
public:
    //Constructors
    Base1(){}; //Default Constructor
    explicit Base1(int x, int y, int z):a{x},b{y},c{z}{
        std::cout<<"Constructing..." << endl;
        }
private:
    int a,b,c; //Members
};

void fun(const Base1 & tmp){}

int main() {
    Base1 A{1,2,3};         //OK，显式初始化
    Base1 B=Base1{2,3,4};   //OK，拷贝初始化，显式调用构造函数
    Base1 C={3,4,5};        
    //Error，拷贝初始化，隐式转换，将{3,4,5}这个list转换成Base1类型，这个转换无法进行
    fun({7,8,9});           //Error，函数参数传递，隐式类型转换
    return 0;
    }
```

通过构造函数进行的隐式类型转换，多用于单参数构造函数中，因为这种形式在语义上更加明确。上面的例子表明多参数构造函数同样具有这种功能，因此`explicit`关键字可以用于单参数或多参数构造函数中。一般情况下，构造函数都要使用`explicit`关键字声明，如果不声明，就要写明原因。