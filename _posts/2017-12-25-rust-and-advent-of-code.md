---
layout: post
title:  "Advent of Code and Rust"
date:   2017-12-25 22:00:00 +0100
categories: rust
---

The [Advent of Code](http://adventofcode.com/2017), which I did not know
was a thing until this year, is an excellent way to practice a new
language. The puzzles are not too easy that you get bored, but not too hard
that using a new language you are not very familiar with becomes frustrating.
So finally I had found the perfect excuse to practice Rust. And here is
what I learned.

## Borrow checking and all that jazz

The Rust Book warns you that fighting with the borrow checker is a common
occurrence for every beginner. What happens is that you do something that is
perfectly normal and logical in any other language, and the compiler gives you
obscure errors, plus suggestions that might or might not work. However, once you
learn the basic concepts, everything falls into place.

Let's say we have an object O of a non-primitive type. If we assign O to another
object, or pass it as a parameter to a function, the object is **moved**.
This is equivalent to what happens in C++ when you call `std::move`: the name O
does not point to the memory region it was pointing to before. Owernship of
the memory region has been passed to the new name. Any subsequent usage of O
in the code will cause the Rust compiler to throw a (very detailed) error.

How do you get around that? One way is to make a copy of the object, if that
is what you wanted.

But if you want to act on the same structure, you should **borrow** the object
instead of getting full ownership. This is done by acquiring a reference to
the object. If you need to modify the object in the process,
the reference must be mutable, and this is called a **mutable borrow**. There
are two main rules about borrows:

- they last until the current scope ends;
- if you borrow mutably, you cannot borrow the same object again in the scope; conversely
you can have as many non-mutable borrows as you want.

The latter implies that iterating over a data structure and trying to modify it in
the same loop is not going to end well: the iteration causes a non-mutable borrow
of the structure (to access its elements), so modifying the structure violates
the rules (it would require a mutable borrow).

The first rule however allows you some freedom. Consider this code snippet:

```rust
{
    let count = map.entry(key1).or_insert(0);
    *count += 1;
}

{
    let count = map.entry(key2).or_insert(0);
    *count += 1;
}
```

map is an object of type `std::collections::HashMap`. Given two map keys,
`key1` and `key2`, I want to increment their value by 1, if they exist, or
insert them with value 1. The `entry` method is convenient, because
I can lookup a pair, insert it with a default value if it's not there, and
then modify the value. But for this I need a mutable borrow of `map`. This
means that if I put everything in the same scope, Rust is going to complain that
I am trying to do two mutable borrows of `map`.

I don't really care about both instructions being in the same scope:
I am okay with `count` only living until I perform the increment. So I create
scopes in a way that the borrows of `map` lasts only for when they are needed.

How about lifetimes, the other much-whispered Rust feature? I have only
used them once, and this post is getting too long, so perhaps another time.

## Matching

You can read about matching in any good Rust tutorial. It's cool, having
used it in Haskell I was pleasanty surprised to see it got adopted more widely.

## Global variables

You can use global variables, but every usage (read or write) needs to be
wrapped in an `unsafe` block, which is a generic way of telling the compiler
to relax a few of its safety checks.

In a nutshell, try not to use global variables. I approve of that sentiment.

## Missing things

There are a few operations that I give for granted that are not yet available in
the stable Rust build:

- In a range iteration, you cannot control the step, it's always 1.
- Remove an element in a data structure given an identical element (you can
use `retain` through).
- Find the index of an element in a structure.

I have been told all this in the experimental builds though, so no big deal.

## Casting

I like how easy casting is. However, in Rust there is no implicit type
conversion. This means that if you want to write a mathematical expression
involving one f64 and a bunch of i64, with the result as f64, it does not
look so nice: every i64 needs to be explicitly converted to f64 when it's used.

## Final opinions

Well I like Rust. I imagine the move semantics and all the extra safety checks
can be difficult to master if you are a beginner, just learning how to do
for loops and the like. However, if you have used e.g. C++ and have run into
some of the issues Rust is supposed to prevent, you are more likely to understand
why rules are made in a certain way.
