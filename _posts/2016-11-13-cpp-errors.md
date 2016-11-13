---
layout: post
title:  "C++ and its errors"
date:   2016-11-13 21:40:00 +0100
categories: c++ opensource 
---

So while I was developing [Cookbook](https://github.com/shainer/cookbook.git) the compiler threw me this error which
I don't remember ever seeing before:

```
class A is an unaccessible base for class B
```

where B is a class in my project that is derived from A. I suspected "unaccessible" meant there is some problem with
permissions, and indeed, I had accidentally used the default (private) inheritance for A, instead of the public
inheritance I intended. You can of course find this out in 5 seconds on StackOverflow, but it's a nice trivia to remember.

Since I am talking about C++, here are a few cool features I have discovered recently. Some of these features are part of C++17.

[std::optional](http://en.cppreference.com/w/cpp/utility/optional): "optional" return value, i.e. it may not
    exist, usually when some error occurred. Useful when you need to return variables of a type which has no clear
    "undefined" value in the context. In different languages, this is typically handled by having the function throw
    an exception in case of errors, but exceptions are avoided by many C++ programmers so this is a good alternative.

[std::unordered_map](http://en.cppreference.com/w/cpp/container/unordered_map): implementation of a hash map; it
    offers more or less the same API as std::map, but all common accesses have amortized constant time, instead of
    logarithmic. As the name suggests, the keys are not sorted in any specific order, which is expected from hash maps.

[std::tie](http://en.cppreference.com/w/cpp/utility/tuple/tie): utility to built a tuple of lvalue-references.
    Let's say we have a function returning a std::tuple of two strings and an integer:

```c++
string s1, s2;
int i;
std::tie(s1, s2, i) = function()
```

Instead of storing the return value in a dedicated std::tuple object, and then unpacking the values in variables,
    we can jump directly to setting the variables. If a member of the tuple is not needed, std::ignore can be used
    as a placeholder. It also works with std::pair.


[std::variant](http://en.cppreference.com/w/cpp/utility/variant): not to be confused with QVariant. It can be considered a C++ implementation of an union: it holds one value from a set of allowed types, or nothing.

[std::any](http://en.cppreference.com/w/cpp/utility/any): this is the equivalent of QVariant, i.e. it stores one value of any type (with some restriction which is documented in the reference).