---
layout: post
title:  "The Chinese remainder theorem (with algorithm)"
date:   2017-10-22 16:30:00 +0100
categories: crypto math
---

Let me preface by saying that you could potentially write a dozen blog posts with all the
implications and mathematical connections that I saw involving the [Chinese remainder theorem](https://en.wikipedia.org/wiki/Chinese_remainder_theorem).
That being said, I am going to focus on a basic description and how to implement it.

Crypto enthusiasts will have understood that this post comes directly from set 8
of the crypto challenges. I believe all things considered I spent more time producing
a viable implementation of this theorem than on the rest of the challenge combined.

## The theorem

Let me write the following set of k equations:

```
x = a1 (mod n1)
...
x = ak (mod nk)
```

This is equivalent to saying that ```x mod ni = ai``` (for i=1...k). The notation above is
common in group theory, where you can define the group of integers modulo some number
n and then you state equivalences (or _congruence_) within that group.

So x is the unknown; instead of knowing x, we know the remainder of the division
of x by a group of numbers. If the numbers ni are pairwise coprimes (i.e. each one
is coprime with all the others) then the equations have exactly one solution. Such
solution will be modulo N, with N equal to the product of all the ni.

For some notes on the history and the reason it was named the Chinese theorem
refer to Wikipedia (or dozen other websites for math); it is quite interesting :)

## Proof

There are many ways to prove this theorem. Most of them are directly related to
the algorithms we are going to present below to compute the solution. I picked
the proof that I found more immediate to understand; it will be employed
in the Gauss algorithm.

Let's define a slightly simpler problem, where we have only two equations.

```
x = a1 (mod n1)
x = a2 (mod n2)
```

As above, let's define ```N = n1 * n2```.

Let's define ```p = n1^-1 (mod n2)``` and ```q = n2^-1 (mod n1)```. This is the
operation called **modular inverse**, where we find the inverse of a number in
the group of numbers mod N. If I say that p and n1 are inverse in mod n2, this
means that ```p * n1 = 1 (mod n2)```. Such inverse will only exist when n1 and
n2 are coprimes, and here they are by definition.

Now I claim that a solution y to the set of equations can be expressed as:

```
y = a1 * q * n2 + a2 * p * n1  (mod N)
```

this is a valid solution because ```y = a1 * q * n2 = a1 (mod n1)``` and
```y = a2 * p * n1 = a2 (mod n2)```. This follows from the definition of the
modular inverse telling me that ```p * n1 = 1 (mod n2)``` and
```q * n2 = 1 (mod n1)```.

This is easily extendible to a generic number of equations, where the final
construction of y becomes:

```
y = sum(ai * (N / ni) * invmod(N / ni, ni)
```

When building p and q before, we used only n1 or only n2; what that generalizes
to is the product of all moduli excluding the "current one": the ```N / ni```
above. The rest is unchanged.

So we have a solution: the next step is to prove it is the unique solution. Let's
assume a second solution z exists for the same set of equations. Then ```z = a1 (mod n1)```,
which implies that ```z - y``` is a multiple of n1, since the remainder of their division
by n1 is the same number. By the same reasoning, ```z - y``` is also a multiple of n2.
But since n1 and n2 are coprimes, then it would also be a multiple of N, or as it
is often written:

```
z = y (mod N)
```

z must be the same as y in the mod N group.

## Algorithm 1: Gauss algorithm

This is quite easy: it is a direct translation to code of the construction
explained above. The n and a parameters are lists with all the related factors
in order, and N is the product of the moduli.

```python
def ChineseRemainderGauss(n, N, a):
    result = 0

    for i in range(len(n)):
        ai = a[i]
        ni = n[i]
        bi = N // ni

        result += ai * bi * invmod(bi, ni)

    return result % N
```

The good thing about this algorithm is that the result is guaranteed to be
positive, given bi and ni both positive. This does not apply to the next
implementation.

For an implementation of ```invmod``` (finding the modular inverse), see next
section.

## Algorithm 2: Euclid

This is the _direct construction_ procedure described by [Wikipedia](https://en.wikipedia.org/wiki/Chinese_remainder_theorem#Existence_.28direct_construction.29).

The extended Euclidean algorithm is used to find two coefficients a and b such
that ```a * (N / ni) + b * ni = gcd(N / ni, ni) = 1```.

Then x is computed the following way:

```
x = sum(ai * b * (N / ni))  for i=1...k
```

Translated into code:

```python
def ChineseRemainderEuclid(n, N, a):
    result = 0

    for i in range(len(n)):
        ai = a[i]
        ni = n[i]

        _, _, si = ExtendedEuclid(ni, N // ni)
        result += ai * si * (N // ni)

    return LeastPositiveEquivalent(result, N)
```

As you can see for my specific application, I wanted only positive results; but
the si coefficients can be negative in a lot of cases, making the final sum
negative. What do I do in that case? What I did there is to multiply the result
by -1, then add N and take the remainder of the division by N, to wrap around
the modulus of the solution.

### Extended Euclidean algorithm

As explained above, the algorithm takes two numbers, x and y, and returns two
coefficients a and b such that:

```
a * x + b * y = gcd(a, b)
```

The implementation returns both the coefficients and the GCD itself.

Now if I take two positive integers x and y, I know I can express them as

```
x = qy + r
```

where q is the _quotient_ of the division (i.e. ```q = x // y``` where // denotes
the integer division) and r is the _remainder_ and is always strictly smaller
than y. If x is a multiple or y, of course r is going to be zero.

The GCD of two integers can be found by repeating this procedure until the
remainder is 0; more specifically:

```
x = q1 * y + r1
q1 = q2 * r + r2
...
```

The final r before getting to 0 is the GCD. Let's see this with an example:

```
gcd(102, 38)

102 = 2*38 + 26
38 = 1*26 + 12
26 = 2*12 + 2
12 = 6*2 + 0
```

so the GCD is 2. Now to find the coefficients we work backwards from the
second-to-last division, expressing the new remainder in terms of the other
parts:

```
2 = 26 - 2*12
2 = 26 - 2*(38 - 1*26) = 26 - 2*(38 - 1*(102 - 2*38))
2 = 3*102 - 8*38
```

3 and -8 are the coefficients in the Bezout identity. To compute them in
practice we do not work backward, but simply store them as we go, as they
can be derived from the main division equation.

```python
def ExtendedEuclid(x, y):
    x0, x1, y0, y1 = 1, 0, 0, 1

    while y > 0:
        q, x, y = math.floor(x / y), y, x % y
        x0, x1 = x1, x0 - q * x1
        y0, y1 = y1, y0 - q * y1

    return a, x0, y0  # gcd and the two coefficients
```

### Modular inverse

Ok so let's suppose I want to find

```
17*x = 1 (mod 43)
```

my unknown x is the **modular inverse** of 17 in mod 43. First we need to
verify that gcd(17, 43) is 1, otherwise the inverse does not exist. Once we
have done that, we compute the two Bezout coefficients as shown above. If we
work that out manually by retracing all the divisions, we get:

```
Forward:
43 = 17*2 + 9
17 = 9*1 + 8
9 = 8*1 + 1  # <-- my GCD is here, so it is 1

Backward:
1 = 9 - 8*1
8 = 17 - 9*1
9 = 43 - 17*2

Replacing the first "backward" equation with everything else:
1 = 43 - 17*2 - 17 + 43 - 17*2 = 2*43 - 17*5
```

So we have expressed this in terms of ```a*x + b*y = gcd(x, y)```: 2 and -5
are our Bezout coefficients. Let's rewrite the last equation in mod 43.

```
2*43 - 5*17 = 1 (mod 43)
```

But 2*43 is a multiple of 43 so it is irrelevant here; this leaves us with:

```
-5*17 = 1 (mod 43)
```

By comparing this with the starting equation in the unknown x, -5 is the inverse
we were looking for. However we need a positive result: in this case we can do
the same procedure we have done earlier, by changing -5 to 5, then adding 43 and
computing the mod 43 to remain in the group. This yields 38.

The code is quite simple:

```python
def invmod(a, m):
	g, x, y = ExtendedEuclid(a, m)
	if g != 1:
	    raise ValueError('modular inverse does not exist')
	else:
	    return x % m
```

## References

Several websites contributed to me getting to the bottom of this interesting
theorem and provided the numerical examples I use above:

- [Wikipedia page](https://en.wikipedia.org/wiki/Chinese_remainder_theorem)
- [Stanford university explanation](https://crypto.stanford.edu/pbc/notes/numbertheory/crt.html)
- [More details on the Gauss algorithm](https://www.di-mgt.com.au/crt.html)
- [Tutorial video on the modular inverse computation](https://www.youtube.com/watch?v=fz1vxq5ts5I)
