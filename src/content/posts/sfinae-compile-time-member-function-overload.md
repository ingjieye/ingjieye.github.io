---
title: "SFINAE: Overloading Member Functions at Compile Time"
published: 2020-01-10
tags:
  - C++
  - Templates
abbrlink: sfinae-compile-time-member-function-overload
---

I ran into a problem recently while writing a template class: how does a member function get a default behavior for one specific `template<typename T>` parameter type?

Take five seconds to think about it before reading on.

I suspect the first thing that comes to mind for you is the same thing that came to mind for me — call `typeid(T)` inside the function to check the type of T, and branch on it:

```cpp
if (typeid(T) == typeid(std::string)) {
	// default behavior for string
}
```

Sorry, but no. It doesn't work, and things are nowhere near as simple as we'd like.

So what happens? Let's look at a complete example first.

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

Compile it:

```cpp
% clang++ SFINAE.cpp -std=c++11
SFINAE.cpp:14:23: error: invalid operands to binary expression ('ostream' (aka 'basic_ostream<char>') and 'const std::vector<char, std::allocator<char> >')
            std::cout << t << std::endl;
            ~~~~~~~~~ ^  ~
```

The compiler tells us it can't call `std::cout << t << std::endl` — `t` is a `std::vector<char>`.

By now you can probably see it: on line 14 of the example above, the compiler instantiated the template with `std::vector<char>`, and the instantiated `Printer` looks like this:

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

The `if` on line 8 can never be true, but the compiler isn't that clever. All it sees is that you're trying to stream a `std::vector<char>` into `std::cout`, and that of course doesn't work.

You might be thinking: if the compiler were smart enough to eliminate a branch that can never execute at compile time, wouldn't that solve it? Let's set that aside for now, and look at how to solve this under C++11 instead. In other words: how do we implement a "compile-time if"?

Another idea comes to mind —

how do we get something like "partial specialization" for a class member function?

Here's the correct answer up front. Don't close the tab yet; I'll walk through it.

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

Let's start with the title of this post. SFINAE stands for "Substitution failure is not an error".

## typename

[http://feihu.me/blog/2014/the-origin-and-usage-of-typename/][1]

## SFINAE

[https://zhuanlan.zhihu.com/p/21314708][2]

## immediate context

[https://codeday.me/en/qa/20190306/13897.html][3]

## std::enable\_if\_

[1]: http://feihu.me/blog/2014/the-origin-and-usage-of-typename/
[2]: https://zhuanlan.zhihu.com/p/21314708
[3]: https://codeday.me/en/qa/20190306/13897.html
