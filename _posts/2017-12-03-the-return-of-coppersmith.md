---
layout: post
title:  "The return of Coppersmith's attack"
date:   2017-12-03 11:00:00 +0100
categories: crypto math
---

It was a close call between this and a post about elliptic curves. But in the
end I decided a post was going to help me summarize all I learned about
Coppersmith's attack in the past days. So here we go!

## What's up?

A group of researchers has found a vulnerability in how RSA keypairs are
generated in widely used cryptographic libraries. A lot of these libraries are
deployed on hardware that generates keys for smartcards or similar devices. This
vulnerability is easy to recognize given a few keys generated by the affected
software, and can be exploited to retrieve the private key of the pair in a
feasible computational time. Not a nice discovery :) but let's describe how
this is accomplished.

## The Coppersmith's attack

The first building block of this vulnerability is a well-known "total break"
attack against RSA. Total break means that we are able to recover the
private key of the pair, therefore we can then decrypt any cyphertext we
intercept.

With RSA, a ciphertext is computed as:

```
c = m^e mod N
```

The attacker wants to find m; what if they knew a part of m? What if

```
m = m0 + x0
```

with m0 known for some reason, and x0 the new unknown to break. There are
several ways to translate this to an equation that looks like:

```
f(x) = c - (m0 + x)^e  mod N
```

Now there are algorithms to find the root of a polynomial if such root
is small enough; let's call X the upper bound on our root. This will be our
solution. But here our polynomial is defined over "mod N", and there are no
simple algorithms for this case.

What we need to do is to build a second polynomial ```g(x)``` with the same roots as
f(x) but defined over the integer space Z. To do this we use Howgrave-Graham's
theorem that states that if g```g(x0) = 0 mod N``` (with ```|x0| <= X```) and
```||g(xX)|| < N/sqrt(n)``` then ```g(x0) = 0``` holds over the integer space
Z. In the third equation, n is the number of monomials that composes g(x).

Now we need to find the starting g(x0). Here is where lattices and the LLL
algorithm are useful. Let's describe what they are briefly.

### Lattices and LLL

If I take two vectors in 2D space, and they are not a linear combination of
each other, then such vectors can _generate_ the whole space by computing
different linear combinations. Now let's say that I am only allowed to compute
linear combinations with integer coefficients; instead of the whole 2D space
I can only generate a set of discrete points on the space: such new space
is called a _lattice_, and the two starting vectors are the _basis_ of
the lattice.

The LLL algorithm gets the basis of a lattice and returns the shortest vectors
that generate the same lattice. In particular there is a clear upper bound
on the first vector of the new basis. Exactly what we need!

### Putting it all together

The final part is easy: instead of generating one f(x) I generate multiple
ones until they form the basis for a lattice. I apply LLL on the result and
then take the first vector of the new basis (and its known upper bound) as
my g(x). Howgrave-Graham's theorem allow me to convert this g(x), still defined
in mod N, into a polynomial defined over the integer space.

There are a few more caveats on how the starting polynomial must be defined,
but this is the gist of the attack; once I have g(x) over the integers,
finding the roots is a solved problem. Mission accomplished!

## The new attack

So why did this attack deserve renewed attention recently? The researchers
performed statistical analysis on the RSA primes generated by common cryptographic
libraries and found some patterns that should not be there. Generating the very large
primes required for RSA to be secure, especially if you need to be fast, is
tricky, and there are a lot of conditions that you need to watch out for to
avoid accidentally making your pair easier to attack even without Coppersmith.

Such vulnerable libraries therefore use fomulas. In particular the libraries
examined by the paper set:

```
P (or Q) = k * M + (65537^a mod M)
```

where k and a are the only internal parameters. M is set once for all
the generations of pairs of a given bit size and is public. M is also quite large,
which means that k and a tend to be small. Therefore the resulting P has very
low entropy: two different P values will only differ by a relatively small amount
of bits (much smaller than the keysize), and the space of possible primes the
library can generate becomes smaller.

Now our "polynomial" has two roots, so the idea is to iterate through values of
one of them and use Coppersmith's method to compute the other. The researchers
tried setting a and computing k, but the required amount of attempts was
infeasibly large in the average case. So they got creative, by transforming the
equation as to use M', one of the small divisors of M, instead of M
itself. The way M is chosen makes sure small divisors always exist: it is actually
computed as the product of several small primes up to a given number. This makes
finding the corresponding k' and a' much easier. The optimal M' value in terms of
speed of attack, for every M supported by the library, was found by local
brute force search plus some heuristics; note that they only need to do this
once for every possible key size.

## Conclusion

No matter how many challenges I solve there's always more to cryptography that
meets the eye, even for a relatively simple and well-known algorithm such as RSA
One shortcut or simple failure in checking conditions can result in pretty bad
failures down the line; and it takes a beginner like me days of careful study
and scribbling on paper to even understand how such failures materialize.

## References

- [Overview of the attack and the paper](https://crocs.fi.muni.cz/public/papers/rsa_ccs17).
- [Implementation for SageMath by David Wong](https://github.com/mimoo/RSA-and-LLL-attacks).
- [David Wong's excellent explanatory video](https://www.youtube.com/watch?v=3cicTG3zeVQ).
- [Coppersmith's method](https://en.wikipedia.org/wiki/Coppersmith_method).
