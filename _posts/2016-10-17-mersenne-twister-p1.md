---
layout: post
title:  "Mersenne Twister, part 1: a primer on PRNGs"
date:   2016-10-17 21:04:00 +0100
categories: crypto random matasano 
---

Another post sponsored by the [Matasano Crypto Challenges](https://cryptopals.com). Today I am looking back at older challenges, and at the interesting world of pseudo-random number generators, or PRNGs.

As this is a very broad topic, I am going to divide it in multiple posts. We start with a generic discussion on PRNGs, and then we'll look into how Mersenne Twister works, why it is widely used, and what attacks I have implemented as part of the challenges.

## A primer on PRNGs
When talking about random number generators for computer applications, we usually refer to pseudo-random number generators.

A good PRNG needs to have two main features:

- it can be _expanded forever_, i.e. it is always possible to generate a new random bit from it.
- there is no _efficient_ algorithm that can distinguish the output of a PRNG from that of a random generator, i.e. one that selects a number from a set with a _uniform probability distribution_.

The second is also referred to as _next-bit unpredictability_: no algorithm can predict the value of the next generated bit with uncertainty lower than 0.5 (which corresponds to guessing it).

So a PRNG is not truly random, but for cryptographic purposes, we care about an attacker not be able to detect a pattern in the generated sequence. If a pattern can be detected, of course, the attacker possesses information that can lead to an "informed prediction" (i.e., more reliable than guessing) of the next number(s) in the sequence.

### Definition

A RNG (pseudo-random or not) is composed of:

- S: finite state space
- O: finite output space
- I: input space
- s0 in S: seed
- g:S->O output function
- T:IxS->S transition function.

There are two main categories of PRNGs.

### Linear congruential generators

In a [LCG](https://en.wikipedia.org/wiki/Linear_congruential_generator), the next output of the sequence can be computed as:

```
X(n+1) = a*X(n)+b mod m
```

So once the parameters A, B and M are chosen, the whole sequence depends on X(0), the **seed**. The transition function has no input, instead the input is only used when producing the seed.

It follows that the seed is the only truly random part of the algorithm, since the rest is deterministic. Random seeds should be acquired from sources with high entropy; in practice, this includes hardware clocks (seeding with current timestamps), key presses, or mouse movements.

Another characteristic is **periodicity**: numbers in the sequence are repeated with a period that is less than or equal to the size of S. Good generators have a period as close to S as possible.

Since the transition function is deterministic, these generators are vulnerable to **state compromise attack**: if we are able to construct the current state of the generator, we can find all next (and in some case previous) numbers in the sequence, even if we don't know the seed. We'll see such an attack in one of the challenges.

### Entropy gathering generators
At each "round", the generators gets random data from one or more *input sources*, estimates their entropy and produces an output.

Here the transition function depends on the input, so even if we discover the seed, we cannot predict the next number of the sequence, and there is no periodicity.

However, these generators cannot be employed if many random numbers are needed in a short time, due to the need of querying high-entropy sources every time a new number is requested.

### Hybrid generators
A good example of this is ```/dev/random``` as implemented in OpenBSD.

## Security
Establishing the security of a PRNG for cryptographic purposes is both difficult and important. Nowadays, battery of statistical-inspired tests are used to test the entropy of the output of a PRNG.

One such battery of tests is the [Diehard tests](https://en.wikipedia.org/wiki/Diehard_tests).
