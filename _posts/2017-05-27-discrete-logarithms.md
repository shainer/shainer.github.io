---
layout: post
title:  "Discrete logarithms: a guide"
date:   2017-05-27 18:00:00 +0100
categories: math crypto
---

I am working through the second challenge in the 8th cryptopals set, and I am already learning something new. Let's talk about discrete logarithms, what they are and how to compute them.

When preparing a Diffie Hellman key exchange, some parameters must be chosen first:

- P: a large prime, is the order of a finite **cyclic group**.
- G: the **generator** of the cyclic group.

G is a generator if for every element of the group, there is a x such that ```G^x mod P``` gives that element of the group.

The shared secret of Diffie Hellman is computed using **modular exponentiation** of the generator. There are algorithms to compute modular exponentiation efficiently, even for numbers that have hundreds of bits. However, the security of Diffie Hellman (among others) depends on the fact that the inverse operation, the **discrete logarithm**, cannot be computed efficiently in the general case, if the parameters are chosen with the right properties.

However, algorithms that are able to do better than the brute force still exist. Let's talk about two of them: **baby-step, giant-step** and **Pollard's kangaroo**.

## Baby-step, giant-step

Recall the formulation of the problem: we want to find x such that

```
G^x mod P = B
```

We can rewrite x as 

```
x = im + j
```

where m is sqrt(P) and i and j are two (unknown) coefficients between 0 and m. So applying some exponential properties the formulation becomes:

```
B(G^-m)^i = G^j
```

All operations happen inside the group, so modulo P. Now, we precompute ```G^j mod P``` for all values of j up to m. We store them in a data structure, for easy lookup later; the data structure maps these exponentials with the value of j. A natural choice here is a hash map.

One of the values of j will be the one that makes the above equation true. But we need to find i too. So we brute force it: for all possible i between 0 and m, we compute ```B(G^-m)^i mod P```. If the result is in the data structure we built initially, we retrieve the corresponding j. We have found both coefficients, and therefore x.

### Complexity

This is still a brute force algorithm, however, instead of trying all possible x between 0 and P (the trivial brute force), we only try up to m, the square root of P. This reduces the number of operations we perform in the worst case. So in terms of complexity analysis it does not mean much. We also "pay" for this speedup by using more memory to store m key-value pairs.

### Implementation

I did mention I wanted some practice with Rust, so [here is a Rust implementation of baby-step giant-step algorithm](https://github.com/shainer/baby-step). The code contains efficient implementations of modular inverse (to compute ```G^-m```), and of modular exponentiation. I have used both several times in the Matasano solutions, but here I took the time to examine, understand and explain them in the comments.

## Pollard's kangaroo algorithm

This algorithm is used when the discrete logarithm is known to lie in a subrange [a, b] of the group. Of course in the worst case you can set the subrange to [0, P-1], i.e. all elements in the group, but in that case more efficient alternatives exist.

The basic idea is to generate two pseudorandom sequences of elements in the range, and then looking for collisions. The first sequence starts from an element of known discrete logarithm, the second from the element whose logarithm we want to find (B).

- Define a deterministic function F from elements of the input group to S, a set of integers.
- Choose an integer N and compute a sequences of N integers like this:

```
y0 = G^b mod P
y := y * (G^F(y) mod P) mod P
```

In another variable, usually called the _distance_ D, you store the sum of all the F(y) you computed for the sequence. Also note that for the final element of the sequence, this property holds:

```
yFin = G^(b+D) mod P
```

This sequence is the **tame Kangaroo**. It starts from y0, an element whose discrete logarithm is b, the end of our range. From there, we take N jumps to other elements.

Now we define the **wild kangaroo**. The new sequence has the same definition, only we start from B:

```
y0 = B
y := y * (G^F(y) mod P)
```

Again we keep track of the "distance" travelled in D'. If the next element of the sequence collides with an element we have seen before, then the discrete logarithm is equal to

```
b + D - D'
```

We stop when we have travelled more than ```b - a + D```. This algorithm does not guarantee that a solution is always found: it is possible to exceed the limit without colliding with the tame kangaroo, even if a discrete logarithm exists.

F controls the size of the jumps you make at each iteration. Bigger jumps give you a better computation time for large numbers, but also increase the probability that you won't collide. To reduce the risk, N is chosen so that larger outputs of F correspond to a larger N: this causes the first sequence to have more elements.

### Implementation

None of now. Or at least not public: I baked a Python implementation in the solution of the latest challenge I am working on. When I have time I am going to take it out, translate it to Rust, and make it public, but after spending time on the baby-step giant-step algorithm, I don't feel like it :-)

## Others

There are other algorithms that are better than brute force, but none of them run in polynomial time. I have not studied or implemented them so I am not going to talk about them, but [Wikipedia has a list](https://en.wikipedia.org/wiki/Discrete_logarithm#Algorithms) if you want to learn more!
