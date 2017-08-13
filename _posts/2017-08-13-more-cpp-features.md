---
layout: post
title:  "Interesting C++ features, part 2"
date:   2017-08-13 11:00:00 +0100
categories: c++
---

I have developed in C++ a lot, both at work and outside. So I like to keep updated with the new features and utilities introduced by
new versions or available through common libraries. This post will describe a few new things I have discovered recently.

If you are wondering why this is part 2, I have decided [this post](https://shainer.github.io/c++/opensource/2016/11/13/cpp-errors.html) can be
considered as "part 1" of this series. I plan to write more about C++ in the future: I'll make all the posts numbered and under the
"c++" category.

## span

This is not part of the STL, but rather of the Guidelines Support Library, which is any library implementing [this set of guidelines](
https://github.com/isocpp/CppCoreGuidelines/blob/master/CppCoreGuidelines.md) by the standards committee.

Let's say that you want to pass an array to a function as a pointer. A basic example:

```cpp
int sum(int* data, int n) {
  // sum all elements in data.
}
```

Now in order for this code to work, we assume n represents the array size, and that it's
actually correct and does not make us access out-of-bounds memory.

A better way:

```cpp
int sum(const span<int>& data) {
  // sum all elements in data.
}
```

a ```span``` of the array (which might or might not contain all the elements) is built on-the-fly to
represent a view of the array, and passed to any function. All the information about size are contained
internally.

```array_view``` (which has a more descriptive name) works the same way, but unlike ```span``` it's a read-only
view of the original array. This is preferrable to ensure no function can write on what is a view of another
data structure, without having to enforce constness instead.

## Custom error codes

This was introduced in C++11, but I think it was overshadowed by other more revolutionary features, since you seldom find
trace of it.

There is another blog which describes quite in details [how to define your own error code space](https://akrzemi1.wordpress.com/2017/07/12/your-own-error-code/) and [how to write error conditions](https://akrzemi1.wordpress.com/2017/08/12/your-own-error-condition/). It's no use repeating
the entire content of those posts here, so I'll make a short summary.

The ```std::error_code``` is a generic interface (used in non-programming sense of the word) to express custom
error codes. Error codes are identified by a _number_ (minus 0, which always means success) and a _domain_ or _category_; the
latter specifies the types of errors we are dealing with, and is identified by a name.

At a high level, this machinery allows you to construct and use variables of type ```std::error_code``` from the enum values representing the
custom error codes you need. By subclassing ```std::error_category``` you define a custom category,
with some nice functionalities such as associating an error message to each code.

Moving forward, it's also possible to express complex groupings and conditions on your set of errors by using ```std::error_condition```.
Let's say that all of your errors fall into the following sub-categories: **internal errors** (something happens inside your program) and
**external errors** (due to e.g. networking). You can extend the logic of your category with a function that tells you which sub-category
a given code belongs to.

Personal opinion: this is not trivial to use, and requires more boilerplate than usual for the initial set up of codes and categories.
However once that part is done (likely in some utility library) it is incredibly useful: good error handling is
something many applications and libraries miss (and let's not talk about dealing with ``errno``...). I am happy to pay the cost to
have sets of well-defined error spaces and codes to deal with.

## CppCon

CppCon 2017 is around the corner! I won't be able to attend, but it's on my todo list and I am currently spending time watching
talks from previous editions on YouTube. I definitely recommend to keep an eye for this year's talks.
