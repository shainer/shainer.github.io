---
layout: post
title:  "Implementing the CRIME attack"
date:   2017-01-02 22:10:00 +0100
categories: crypto 
---

I have just completed [challenge 51](http://cryptopals.com/sets/7/challenges/51) of the Matasano crypto challenges,
which is modeled on [the CRIME attack](https://en.wikipedia.org/wiki/CRIME). Let's talk about how it works!

### The target

The target are secrets exchanged through HTTPs, SPDY, TLS or any protocol offering secure communication between
a server and a client, usually at the application layer. The protocol also needs to compress the requests
using a compression algorithm based on [DEFLATE](https://en.wikipedia.org/wiki/DEFLATE). Common such algorithms are gzip and zlib.

### The setup

Requirements:

1. The attacker is able to sniff the compressed and encrypted requests and responses flowing between the two endpoints;
1. The attacker is able to craft his own requests using the same compression procedure the client is using.
It is not necessary for the server to accept such requests, or even to receive them in the first place. If the encryption
algorithm never changes the size of the plaintext (such as AES in CTR mode), it can be ignored altogether; otherwise
we only need to know how much padding is added to the plaintext (e.g. AES in CBC mode requires the plaintext to be padded
to the block size). This can usually be found in the protocol specification. We do not need to know any encryption keys,
initialization vectors, or other secret encryption parameters.

In the challenge, these requirements are implemented by giving the attacker access to a **compression oracle**: this
oracle takes a request payload, adds the headers specified by the protocol, compresses and encrypts, and returns the length
of the final request to the caller.

The length of the request is all we need to know to uncover, under certain conditions, a secret included in the headers
of the request. In the challenge, this secret is a session ID included in all the communications between client and
server.

### The DEFLATE algorithm

We will exploit one characteristic of LZ77, one of the two algorithms that forms DEFLATE.

LZ77 scans the input, and when it encounters the same substring for the second time, it replaces the second occurrence with a reference to the first one, in terms of distance and length.

An important parameter is the _window size_: how far back does the algorithm look for repetitions to reference to.

### The attack

In the first part of the challenge, AES in CTR mode encrypts the request, so we don't have to deal with padding: the sizes of the compressed request and the final ciphertext are the same.

Let's guess the first byte of the secret session ID: we produce requests whose payload is our guess of the byte (we try all possibilities). If we got it right, what should happen is that DEFLATE detects the two identical bytes, and produces a shorter compressed version than when our byte is unique in the window.

To proceed, we repeat the same for the next byte, appending it after the bytes we have already discovered. Of course, we need to know the length of the secret to know when to stop guessing.

This can go wrong in several ways:

- the DEFLATE algorithm outputs bits, but the protocol operates over bytes and byte length, so even if there is a difference, we may be unable to detect it because it is less than 8 bits.
- the window size of DEFLATE is not long enough to conflate our repetitions. Common window sizes are 256 or 512 bytes, so this is unlikely unless we have a lot of headers between the payload and our secret;
- the session ID contains many repetitions, so we match against different bytes than what we think. This can be mitigated by returning all the bytes that give the shortest length, and then generating and trying all the possible combinations. This does not happen in the challenge (IMHO) because the session ID is encoded with base64, which gives a diverse set of characters with respect to what would be used in plain ASCII.

In the second part of the challenge, AES in CBC mode is used, so we have to deal with the ciphertext length being padded to multiples of the block size. This requires some extra work on our size to make sure the size signal is still meaningful, i.e. it crosses a block boundary. You can refer to [my code](https://github.com/shainer/matasano/blob/master/set7/compression_ratio.py) to see how that is accomplished by careful control of the payload size and composition.

### Prevention
How to protect against these attacks? By disabling compression.

Usually, clients are able to request the compression algorithm they want to use and the server must honor it; modern browsers will select no compression. Some protocols have stopped supporting compression altogether.

Note that there are not many other options: the fact that we are able to see the compressed request size is enough to start this attack, and there is no way of hiding that from an attacker reading the request.

### References
[This slide deck](https://docs.google.com/presentation/d/11eBmGiHbYcHR9gL5nDyZChu_-lCa2GizeuOfaLU2HOU/edit#slide=id.g1d134dff_1_236) does a good job of explaining CRIME, its internals, and some practical results.

A more detailed textual explanation, with references to papers, can be found in [this crypto.stackexchange.com thread](http://crypto.stackexchange.com/questions/38047/compression-ratio-side-channel).