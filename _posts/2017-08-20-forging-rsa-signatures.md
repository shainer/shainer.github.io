---
layout: post
title:  "Forging RSA signatures"
date:   2017-08-20 18:30:00 +0100
categories: crypto
---
As I vaguely promised some weeks ago, going back to solve the crypto challenges I had missed in the sets before the eigth one, I found yet another interesting problem to talk about: exploiting several weaknesses to [forge a RSA signature](http://cryptopals.com/sets/6/challenges/42).

## Preconditions

The following conditions are necessary for this attack to work:

1. RSA is used to generate and verify digital signatures;
1. The keypair generation is lazy, and sets the public exponent e to 3;
1. The signature verification is also a bit lazy; I'll describe how later.

#1 is nothing weird: digital signatures need some form of asymmetric encryption and RSA is the most popular choice.

#2 can happen in practice due to how keypair generation works: the two internal parameters for RSA, q and p, need to be primes, large enough to make the factorization of N = pq hard, and such that (p - 1) and (q - 1) are either coprimes or don't have a lot of common factors after 2. A common way to achieve this is to set e, the "exponent" part of the final public key, to some small prime like 3, and then derive p, q and d (the exponent of the private key) from that. Having a small value for e also makes encryption and decryption quite easy, since we work with "small" numbers.

The full standard to compute signatures with RSA is described in [RFC 2313](https://tools.ietf.org/html/rfc2313). In short, the message is first hashed (most common algorithms are supported), then the following bytes block is generated:

```
00 01 FF .. FF 00 ASN.1 HASH
```

where ASN.1 is a byte string identifying the hash algorithm used (see [RFC 3447](https://www.ietf.org/rfc/rfc3447.txt)), and there's as many FF bytes as needed to make the total size equal to the size in
bytes of N, the modulus of the RSA keys. Note that N is part of both the public and the private RSA key. The block is then converted to the corresponding integer and encrypted with the private key; optionally, the result is converted again to an hex string or similar representation.

Now we can see where mistake #3 can come from: if I verify a RSA signature using, for example, regular expressions, it's easy to check that there's one or more FF bytes in the padding zone, but not checking that there's exactly the number I expect. Furthermore, I might not check that there's nothing else in the signature after the hash. Note that this bites me even if I separately check that the
total signature length is as expected.

If all these conditions are there, the attacker is able, without any knowledge of the private key, to forge a RSA signature for pretty much any message, and have it accepted by the verifier.

## How the forgery works

If I am verifying a signature, I am decrypting with the public key, which means this operation is performed:

```
D = EncryptedSignature ** e mod N
```

if e is equal to 3, it's quite possible that ```EncryptedSignature ** 3``` ends up being smaller than N, therefore the modulo operation does not change the result. So, if we forge a block that satisfies only the conditions we know the system checks for, and also corresponds to a perfect cube, we can pass the cube root as a signature to such a verification system, and it will be accepted as valid. How does that happen?

Let's take a block with this format:

```
00 01 FF 00 ASN.1 HASH GARBAGE
```

Now let's take the sub-block composed of the last 00 byte, the ASN.1 code, and the hash. If SHA-256 is used for the hash, the total size is 52 bytes. We then convert this block into an integer, which we call D.

```
Block = '00 ASN.1 HASH'
D = int(Block)
N = 2 ** len(Block) - D
```

Now let's say that our RSA key has length 2048 bits; with the format above, there are going to be (2048 - 52 - 3) bits left on the right for garbage. Let's call this number X. The numeric block is going to be:

```
2 ^ (2048 - 15) - 2 ^ (X + len(Hash)) + D * 2^X + garbage
2 ^ (2048 - 15) - N * (2 ** X) + garbage
```

**Disclaimer**: this is where I get lost, sadly. I don't understand what the 15 bits subtracted from the key size represent, or why the two equations above are equivalent. If somebody reads this and wants to send me an email for a more complete explanation, they are welcome to do it!

From empirical evaluation, it seems that for the garbage number you should prefer higher values: the lowest cube roots might end up being encoded in something that does not quite contain the full hash at the end, possibly because we run out of bits to convert before that. This is why my code takes a shortcut and just sets it to the highest number possible given the allowed number of bits.

The final step is computing the cube root and converting the result to an integer (in my case, by rounding down). Here I ran into a limitation of Python: the suggested way to compute a cube root is to elevate the number to the power of (1.0 / 3.0), but this requires the base to be converted to a float, and that does not work for very large integers such as this one. I could have looked at some mathematical library like numpy, but I am reluctant to add too many dependencies; eventually I found a code snippet on the Internet that does the job with the decimal builtin module.

Aaand we are done!
