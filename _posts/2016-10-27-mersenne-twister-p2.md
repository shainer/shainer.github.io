---
layout: post
title:  "Mersenne Twister part 2"
date:   2016-10-27 21:52:00 +0100
categories: crypto python matasano random 
---

Mersenne Twister is one of the most popularly used random number generators. It is even present in the C++ standard library ([http://www.cplusplus.com/reference/random/mt19937/](http://www.cplusplus.com/reference/random/mt19937/)). Let's see how it works!

The main parameters:

- w: word length. Usually 32 bits.
- n: _degree of recurrence_
- r


MT generates a sequence of numbers starting from a random seed. The period of this sequence, i.e. the number of unique outputs it is able to produce before it repeats itself, is 2^(wn - r) - 1. 

The most common implementation is MT19937; it has a period of 2^19937-1, more than all the particles in the known universe.

The parameters are chosen so that the period is a prime, making it a Mersenne prime (i.e. one less than a power of 2).

From the seed, an array of n numbers of w-bits is initialized. This is the initial state of the generator, which is updated as the sequence progresses.

In MT, n=624. It follows that it is sufficient to observe the output of the generator n times to reconstruct the internal state array. If we create a new MT random generator with this state "injected" into it, the MT generator will produce the same output sequence as the original. So we can clone the generator without knowing the seed. See next section for a Python program doing that.

It follows that while the MT algorithm is widely used for general-purpose random numbers, it is not considered secure enough for cryptographical uses.

### Cloning the MT generator

This is the code producing a new random number with MT:

```python
def randomNumber(self):
    if self.index >= self.N:
        self._twist()
    
    # This is what we need to retrieve later
    y = self.state[self.index]

    # Right shift by 11 bits
    y = y ^ y >> 11
    # Shift y left by 7 and take the bitwise and of 2636928640
    y = y ^ y << 7 & 2636928640
    # Shift y left by 15 and take the bitwise and of y and 4022730752
    y = y ^ y << 15 & 4022730752
    # Right shift by 18 bits
    y = y ^ y >> 18

    self.index += 1
    return int(y)
```

Let's take one operation, changing the variable names to make it more readable:
```
    y = x ^ (x << 7)
```

ignoring the mask for one second. Let's apply this operation to a random 32-bit number:

```
10101110101011001000001001011111 = x
01010110010000010010111110000000 = x << 7
11111000111011011010110111011111 = y
```

We need to reverse this. It's immediate to see that the right-most 7 bits of y (```1011111```) are the same as in x. So that's sorted. Then if we take the next batch of 7 bits (```1011011```) and XOR them with the previous batch of x (```1011111```), we get the second part of x (```0000100```). We can continue this, XOR'ing bits of y with the bits of x shifted of 7 positions to the right, until we reconstruct the whole number.

To also factor the mask into this, we then need to AND each partial result with the mask.

To reverse a right shift, we do the same, just in the opposite direction: this time we XOR bits of 7 with the bits of x shifted of 7 positions to the left (of course the shift itself can change).


```python
def Untemper(y):
    # Here we only need one shift since the first
    # 18 bits are already correct.
    y = y ^ (y >> 18)
    y = y ^ ((y << 15) & 4022730752)

    mask = 2636928640
    # We can do this in a loop that computes how
    # many iterations are needed to get the whole
    # number, but I am lazy.
    a = y << 7
    b = y ^ (a & mask)
    c = b << 7
    d = y ^ (c & mask)
    e = d << 7
    f = y ^ (e & mask)
    g = f << 7
    h = y ^ (g & mask)
    i = h << 7
    y = y ^ (i & mask)

    z = y >> 11
    x = y ^ z
    s = x >> 11
    y = y ^ s
    return y
```

Having this, we can take 624 outputs of the random number generator, and apply the Untemper function to retrieve the original state. At the end, we have the whole state array.

We can verify the cloning was successful by creating another generator with the "injected" state, instead of passing a seed. If we did things right, the generator will produce the exact same output sequence that we just observed.

If the original generator was running somewhere else, and we have a local clone, we can also use it to predict the next numbers that will be generated.

To protect against this, the challenges suggests to take each output of the MT generator and apply a cryptographical hash to it. This way, you would need to invert the hash before you are able to apply the untemper function.
