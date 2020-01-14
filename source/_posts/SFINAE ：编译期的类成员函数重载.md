---
title: SFINAE ：编译期的类成员函数重载
date: 2020-01-10 00:00:00
tags:
---

最近在写模板类的时候遇到一个问题：一个类成员函数如何针对某一特定的`template<typename T>` 模板参数类型，有默认的行为。

请你先想五秒钟，再接着往下看。

我想你应该和我一样，映入眼帘的第一个想法，是在这个函数里调用typeid(T)来判断T的类型，以此来让函数针对某一类型T有默认的行为：

```cpp
if (typeid(T) == typeid(std::string)) {
	// 针对 string 的默认行为
}
```

但是很抱歉，事与愿违，事情远远没有我们想象的那么简单。
发生了什么事呢？
让我们先来个看个完整的例子

```cpp
#include <string>
#include <vector>
#include <iostream>

template<typename T>
class Printer
{
public:
    Printer() { }

    void DoIt(const T& t)
    {
        if (typeid(T) == typeid(std::string)) {
            std::cout << t << std::endl;
        } else {
            std::cout << "I don't know how to print" << std::endl;
        }
    }
};

int main(int argc, char** argv)
{
    Printer<std::string> p;
    p.DoIt("WTF");

    Printer<std::vector<char>> p2;
    p2.DoIt({'W', 'T', 'F'});

    return 0;
}
```

编译它

```cpp
% clang++ SFINAE.cpp -std=c++11
SFINAE.cpp:14:23: error: invalid operands to binary expression ('ostream' (aka 'basic_ostream<char>') and 'const std::vector<char, std::allocator<char> >')
            std::cout << t << std::endl;
            ~~~~~~~~~ ^  ~
```

可以看到编译器给出了没办法调用`std::cout<< t << std::endl`的错误，t是一个`std::vector<char>`类型。
看到这里你应该明白了，在上述例子的第14行中，编译器对模板使用了`std::vector<char>`类型进行实例化，实例化后的Printer类变成了这个样子：

```cpp
class Printer
{
public:
    Printer() { }

    void DoIt(const std::vector<char>& t)
    {
        if (typeid(std::vector<char>) == typeid(std::string)) {
            std::cout << t << std::endl;
        } else {
            std::cout << "I don't know how to print" << std::endl;
        }
    }
};
```

虽然第8行的if永远都不会成立，但是编译器可没有那么聪明。他只看到了你要对 `std::vector<char>`类型调用std::cout进行输出，这当然是不行的。
你可能会想，要是编译器足够聪明，可以把这段永远不会执行的if分支在编译期给消灭掉，不就得了吗？对此我们暂时按下不表。先来看看在C++11标准下，要如何解决这个问题。也就是说，如何实现一个 “编译期的if”。

另一个想法映入眼帘，

我们要如何实现针对类成员函数的“偏特化”。

先把正确答案贴上，别急着关闭网页，我慢慢跟你解释。

```cpp
class Printer
{
public:
    Printer() {
    }

    template <typename U = T>
    void
    DoIt(const T& t, typename std::enable_if<std::is_same<U, std::string>::value, void>::type * = nullptr)
    {
        std::cout << t << std::endl;
    }

    template <typename U = T>
    void
    DoIt(const T& t, typename std::enable_if<!std::is_same<U, std::string>::value, void>::type * = nullptr)
    {
        std::cout << "I don't know how to print" << std::endl;
    }
};
```

先从本文的标题说起。SFINAE是「Substitution failure is not an error」的缩写。
## typename
[http://feihu.me/blog/2014/the-origin-and-usage-of-typename/][1]

## SFINAE
[https://zhuanlan.zhihu.com/p/21314708][2]

## immediate context
[https://codeday.me/en/qa/20190306/13897.html][3]

## std::enable\_if\_

[1]:	http://feihu.me/blog/2014/the-origin-and-usage-of-typename/
[2]:	https://zhuanlan.zhihu.com/p/21314708
[3]:	https://codeday.me/en/qa/20190306/13897.html