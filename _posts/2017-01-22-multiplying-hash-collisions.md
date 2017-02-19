---
layout: post
title:  "Multiplying hash collisions"
date:   2017-01-22 18:30:00 +0100
categories: crypto
---
Turns out that for some hash functions, finding a collision is easy and fun, but multiplying your  collision into a full disaster is even better!

After much counting on my fingers and scribbling on paper, I completed challenge 52 and challenge 53 of the [Matasano crypto challenges](http://cryptopals.com/). This set is less oriented toward practical scenarios than the previous ones, however it does a good job of showing a series of more obscure problems you should care about when working with hash functions.

## Iterated hash functions

Iterated hash functions are used when you need to apply a hash function that works on inputs of size X (or less) to longer inputs.

This is like block cryptography: for instance, the AES algorithm works on an input the same size as the key, and then you use one of the known _modes_ (CBC, CTR, etc...) to encrypt plaintexts of any size.

A common way to do this in the hashing world is to apply the function iteratively to each block of the input, derive an _intermediate state_, and use this state as an input for the next stage. The first state is usually a block called _initialization vector_ or IV, which is taken as input. The last state we compute is the hash we want.

Such a construction is called a [Merkle-Damgard construction](https://en.wikipedia.org/wiki/Merkle%E2%80%93Damg%C3%A5rd_construction). The underlying function a one-way compression function: it takes two blocks as an input, the plaintext and the previous state (or IV), and returns one block of output.
It is proved that if the compression function is collision resistant, then the whole construction will also be collision resistant.

Many popular hashing functions, such as MD4 and SHA-1, are based on this construction.

Even with the guarantee of collision resistance, this construction has some bad properties; and since we are good crypto challengers, we are going to exploit them!

### Multicollisions

Problem number 1: the cost of generating a certain number of collisions scales sublinearly. So if it is "not that hard" to find one collision it won't be that hard to find many more.

Let's say that my underlying function is pretty bad; in the challenge I used AES (we always operate on individual blocks to the mode does not matter, I used ECB), with a state length of just 2 bytes. This means that before feeding a new state to the next stage we pad it to the correct length, and then we drop the excess bits on the way out.

We then find a 1-block collision starting from an initial state (let's call it H0). Given that the state is only 2 bytes, only the first 2 bytes of the input actually influence the output: we can brute force our way by trying all combinations and find two that hash to the same value. In real life, where sizes are bigger and brute force infeasible, this would be replaced by using a known method of finding a collision for the algorithm at hand.
Let's call the colliding blocks B1 and B2, and their hash is H1.


At this point my life is very easy: I can repeat the same at the next stage, with the previous state set to H1. I find two colliding blocks for this stage, B3 and B4, but now I have 4 collisions: B1+B3, B1+B4, B2+B3 and B2+B4. All of them have the same hash.

It follows that if I apply this for n stages, I can generate 2^n collisions at the same cost I incur for generating n. So the cost scales sublinearly.

[This](https://github.com/shainer/matasano/blob/master/set7/iterated_hash_multicollisions.py) is my implementation.

### Collisions in two functions
Sometimes two weak hash functions (F and G) are combined into a stronger one (H) by concatenating their outputs:

```
H(m) = F(m) || G(m)
```

If collisions in F cost (2^b1/2) and collisions in G cost (2^b2/2), collisions in H should cost (2^(b1+b2)/2).

But what happens instead? If F is the same function I attacked in the previous scenario, we know it is cheap to compute many collisions in F. I keep on doing so until I find at least two messages in my pool that collide in G too. Even if G is stronger, as the number of collisions increase this becomes more likely. I have a collision in H without too much work.

In my challenge, F is a MD construction based on AES with state length 2, and G is a MD construction based on Blowfish with state length 3. I had to produce 8192 (2^13) collisions in F before finding one in G. Not too bad: most of the time is spent producing the final list of collisions for F given the individual colliding blocks.

[This](https://github.com/shainer/matasano/blob/master/set7/concatenated_hash_multicollisions.py) is my implementation.

## Expandable messages
2nd problem: this construction is vulnerable to second preimage attacks. In this attack, I am given a message M and its hash H, and I need to find a second message M' that collides with M.

Turns out that this is easy to do with long messages.

First remark: if I find a collision into an intermediate state of M's hash, I only need to extend my collision with the remaining M blocks to have my M'. This actually depends on the algorithm and the padding used internally, so we may have to pick a specific intermediate state to collide with.

Assumption (as before): generating collisions between messages of one block size is not too difficult.

What we do is generating a series of _expandable messages_. If M has 2^k blocks, we generate a set of messages that we can compose together into messages of any size from k to k+2^k-1.

We start from the initial state of the hash function, H0, and generate a collision between a single message block (picked at random) and a message of 2^(k-1)+1 blocks. Then we take the hash of these blocks, H1, and we find a collision using that as "IV", between a single message block and a message of 2^(k-2)+1 blocks. We repeat that until we get to 2^0+1=2. We store each of these pairs of messages.

At this point we find a _bridge_ block; this is going to be a block that combined with the final hash of the construction above, collides with an intermediate state of the construction of M. The choice of the position of this intermediate state depends on the algorithm, or on the application: if I benefit from keeping a certain number of M blocks around in my M', that influences my choice. In my construction I had no such limitation so I picked a random one.

This block needs to be placed at the specific position of the intermediate state in the construction. So we use our expandable messages to fill up the blocks before the bridge. We have to pick one of the two available messages for each intermediate state, in order, until we get to the size we want. This can be done manually, or automatically with a backtracking algorithm.

Then we concatenate the bridge, and the last blocks of M until we get to the same size as M itself.

This new blocks, M', will collide with M.

[This](https://github.com/shainer/matasano/blob/master/set7/expandable_messages.py) is my implementation.

## Conclusions

I am close to the end of this set, and uncomfortably excited because the next one is about elliptic curve cryptography; this topic has been on my TODO list for ages as the next theoretical study to have fun with.

Stay tuned for more posts once I figure that out :-)
