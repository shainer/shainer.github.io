---
layout: post
title:  "Introduction to ring signatures"
date:   2017-10-15 23:00:00 +0100
categories: crypto
---

Well finally I got around to reading [this article about confidential transactions](https://cryptoservices.github.io/cryptography/2017/07/21/Sigs.html). Recommended if you are interested in how some
security guarantees in cryptocurrency transactions can be enforced. However, thanks to that article I also learned something
about [ring signatures](https://en.wikipedia.org/wiki/Ring_signature), so in this post I will talk about that.

Ring signatures are a special type of **group signature**. In group signatures you have (you guessed it!) a group of signers;
each of them owns a keypair. Messages are signed by the group in a way that the receiver can verify that the signature
was generated by the group, but cannot uncover which signer specifically made the signature (a property named _signer ambiguity_).

However group signatures have a "weakness": the group needs centralized management. There must be a manager that allows new
members in the group, and can revoke their anonymity if they want to; this is because they must know the private keys of each member
and therefore they can also reveal it publicly without "endangering" themselves directly. **Ring signatures**, presented for the first time
in 2001, remove this need. Any user can build a set that includes themselves, and then produce a signature
using its own private key and the public keys of the other members. Signer ambiguity is respected, therefore the verifier
cannot tell which of the set members was the signature's author. However notice that since the user only needs the public key
of another member, he/she can use them in a ring without the permission (or even the knowledge) of the key owners.

The original paper is aptly named "[How to leak a secret](https://link.springer.com/content/pdf/10.1007%2F3-540-45682-1_32.pdf)",
and indeed it describes a scenario where somebody is leaking a secret, wants the recipient to be sure the origin of the
secret can be trusted (or better, that it belongs to a trustworthy group), but does not want to reveal its identity to the
recipient completely.

Note that after the original publication other schemes of ring signatures were developed; I am going to focus on the first one.
Indeed the original article I linked uses a different one.

The second desirable property of this ring signature scheme is that adding a new ring member is efficient: both the generation
and verification procedures only need to compute one extra modular multiplication and one symmetric encryption. Pretty good :)

## Details

### The setup

Each signer in the ring is associated with a public key ```Pi``` and the generation scheme is known. All of them use the same scheme
(let's assume RSA for simplicity). Note that the moduli N of each key could have different bit sizes; to work around this problem,
the paper finds a bit size b larger than any of the moduli's size, then changes the encryption functions so that their outputs are unchanged
for most inputs, and set equal to the input for a negligible number of inputs. This guarnatees that if the original encryption function
was infeasible to invert (as in RSA) given a random output, the new one will be too.

The message to sign is named ```m```, and we have ```r``` members in the ring. The generation function of each signer is named ```gi```; This
is the function that computes the public key given the private key (and internal parameters such as the N in RSA). In RSA, this is the modular exponentiation.

We also have:

- a symmetric encryption function ```E``` such that for any key k, ```E``` with k is a permutation over strings of b bits.
- a public collision-resistant hash function ```H``` that maps inputs to strings of the length of k, used then as keys for E.

Finally we need a family of keyed _combining functions_ C(k, v). These take as input k, a random initialization vector v and r
arbitrary values Y, each composed of b bits. Each function will use ```E(k)``` to produce outputs of b bits, such that for any
(k, v) pair we have that the function has the following properties:

- it is a permutation over all the Y values;
- when fixing all the Y values but one, and with the output z known, there is exactly one solution for the remaining value and it is easy to compute;
- given k, v and z it is infeasible for an attacker to solve the equation ```C(k, v, g1(x1)...gr(xr)) = z``` (given access to each g function),
provided it is also infeasible for them to invert the g functions themselves.

### Signature generation

**Step 1**: compute ```k = H(m)```

**Step 2**: select a _glue_ value k, a bit string of length b, at random.

**Step 3**: select random ```xi``` values, one for each ring member beside yourself, again bit strings of length b; then compute ```yi = gi(xi)```.
This means using the random ```xi``` as replacements for the private keys you don't know.

**Step 4**: to find your own y value, solve the combining function for v, i.e. find the remaining value such that ```C(k, v, Y) = v```. By definition there must be exactly one solution and it should be easy to find. Then compute the x value from y by inverting the function (i.e. by computing the private key of a RSA pair given the public key).

The signature is the set of public keys P, the glue v and the set of X values you computed, including your own.

### Signature verification

The verification is quite trivial and is also based on the solutions of the ring equation:

```
yi = gi(xi)  for each xi
k = h(m)
```

then you verify that the equation with C (called _ring equation_) is satisfied with the given parameters.

### Security

The anonymity is guaranteed by the fact that the ring equation, when k and v are fixed, has ```(2^b)^(r-1)``` solutions, and each of them
can be chosen with equal probability by the signing procedure, since it is based on random numbers.

The paper proves a theorem that any forging algorithm A, i.e. any algorithm that is able to forge a valid signature for a message m after
observing a non-huge quantity of signatures for different messages, can be turned into an algorithm B that is able to invert a function gi
for any random y. Since we assumed such a task must be computationally hard for g to be inverted (and it certainly is for common keypair
algorithms such as RSA), A should not exist.

I am thinking it would be cool to attempt this in practice. Pick a bad function g and make the ring members use it. Then build A;
in the proof, A is simply an oracle that can produce valid signatures for messages; we do not care how this is achieved internally, so for
our purposes it can just run the signature procedure. Then we build B from that following the explanation of the paper, and we verify
that it actually inverts g. This looks like a new crypto challenge in the making and I am already excited! The procedure described for building B was not 100% clear to me upon reading the paper, so it will require more careful study. I will let you know if I succeed in this adventure;
I could even propose this as a new challenge for the Matasano team to add to their website.

### Conclusion

One interesting improvement of the AOS ring signature scheme (the one used in the original article) is that it works with signers that use
different keypair generation functions. It uses something called Schnorr signature as the basis, and then "chains" multiple signatures together
into a ring to produce the final result. The idea is still to chain all the signatures together in a way that makes them depend on each other
and allows the verifier to repeat the same procedure to verify we end in a loop.
