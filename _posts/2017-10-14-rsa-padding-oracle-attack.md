---
layout: post
title:  "RSA padding oracle attack"
date:   2017-10-14 15:30:00 +0100
categories: crypto matasano
---

My long series of posts on the Matasano crypto challenges and cryptography in general cannot be called complete without a dissertation
on challenges [47](http://cryptopals.com/sets/6/challenges/47) and [48](http://cryptopals.com/sets/6/challenges/48), dedicated to the PKCS1.5 padding oracle and how it is exploited to break RSA and recover a plaintext. I was fascinated by this attack and read the whole paper before coding the implementation, so this post will include a bit more details on why the attack works.

# The setup

Alice takes her secret message and applies the PKCS1.5 encoding, getting a byte string of length equal to the number of bytes in the modulus of the RSA pair. She then encrypts it with Bob's public key and sends it over the network. You, as the attacker, have access to the resulting ciphertext, and an oracle function on Bob's server: when invoked, the server will decrypt the message and return true if the first two bytes in the plaintext are equal to '\x00\x02'. This is a necessary but not sufficient condition for the plaintext to be a PKCS1.5-encoded message; however for our purposes we can pretend that the oracle returning true means this ciphertext decrypts to a message _conformant_ to PKCS1.5.

For a full description of the PKCS1.5 format, refer to the source paper or the implementation directly. The latter is a bit lazy, filling the padding portion of the message with the same byte repeated as many times as needed, rather than pseudorandom bytes.

# The attack

Well now you want to use the oracle output to recover the full plaintext. The idea is that the RSA ciphertexts are just numbers; by intelligently searching through the space of numbers, you will find another ciphertext that decrypts to the same plaintext. Once the algorithm completes you will be certain to have found such a number even without any verification.

In cryptography, this is called an **adaptive chosen-ciphertext attack**. Adaptive here means we choose the following ciphertext based on information derived from the previous one.

Let's go into more details. We want to decrypt a ciphertext c, i.e. find ```m = c^d mod n```. Due to a well-known property of RSA, if I decrypt
```cs^e``` instead of c (for some arbitrary s), the plaintext will be equal to ```ms```. So if I pick some random s, send ```cs^e``` to the oracle, and the response is "true", I know that ```ms``` is PKCS1.5 conformant, i.e. it starts with '\x00\x02'. Let's set ```B = 2^8(k-2)``` where k is the byte length of the modulus of the RSA pair (i.e. of the parameter n). Then it must be true that ```2B <= ms mod n <= 3B```.

This means that by choosing different s, we are able to derive a set of intervals that must contains the plaintext m we are looking for. Once we are down to one potential interval, it is possible to choose s such that the probability of ```cs^e``` decrypting to a conformant message is quite high. After sufficient iterations, we'll end up in an iteration with one interval of length 1, and we will have found our ciphertext.

The first set of interval is derived by setting s to 1; we know that m is contained one interval, ```[2B, 3B - 1]```, as explained above. This is however too big to be useful yet, so we need to proceed to the next step.

In the next step, we increase our choice to s until we find another cs^e that decrypts to a conformant plaintext. The search either starts from
the value of s we found at the previous iteration, or from ```n / 3B``` for the first one. This is because small values (beside 1) are
less likely to generate conformant plaintexts. If we have only one interval in our set, however, we are able to narrow down the search a bit,
since we know that the number we are looking for must lie there. So, if our m lies between two numbers a and b, there will be a number r such that:

```
2B <= ms - rn <= 3B - 1 (for some r)
2B + rn / s <= m <= 3B - 1 + rn / s
```

and this gives us the computation needed to find s in step 2c.

From the same formula above we can also explain step 3, i.e. how to update the list of intervals to use in the next iteration. We just need to
take all possible intervals created by every value of r for which the first equation remains true, and for all possible (a, b) intervals we
have in the current iterations. We also make sure not to add any interval that is contained in one we already have.

# Remarks

The merits of this attack compared to others of its kind are simply that the number of oracle calls we have to do is on average smaller than
what other attacks need. The paper has some notes on why this is true and how to approximate the number of iterations required to find the
solution. I won't go into that part as I found it quite technical and long to explain.

For the **implementation**, the two challenges are very similar; the only difference is that in the first one you can afford to be sloppy
and not implement certain parts (such as handling multiple intervals in one iteration, which never happens). I did the full implementation
at the beginning so I only had to change the parameters to make it work again.
