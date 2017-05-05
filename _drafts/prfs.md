---
layout: post
title: Profuse PRFs
---

Cryptographic protocols are built on abstractions and approximations of ideal mathematical constructs. 
For example, we often assume that we have a perfect source of randomness. But in practice, we settle
of pseudorandomness. That is, a source of randomness that is indistinguishable from true randomness
in the presence of a computationally restricted adversary. Another pure construct is a random function.
A random function is a function $$f : \{0,1\}^n \to \{0,1\}^n$$ such that for some $$x \in \{0,1\}^n$$,
$$f(x)$$ is seemingly chosen at random from the codomain $$\{0,1\}^n$$. There are $$^{2^n}$$ different
random functions over the domain $$\{0,1\}^n$$. Like randomness, we often settle for an approximation
of this ideal construct in the form of a pseudorandom function, or PRF. As the name suggests, a PRF
simply approximates a truly random function. And they're used everywhere in cryptography. However,
what I find more interesting are the different flavors of PRFs that exist, including: standard PRFs,
oblivious PRFs, verifiable PRFs, and, yes, even oblivious *and* verifiable PRFs. In this post I'll describe
and implement (simple) versions of these constructs. 

# Plain Old PRFs

A plain old PRF is an approximation of a truly random function. Concretely, this means that
given $$((x_1, f(x_1)), (x_2, f(x_2)),\dots,(x_i,f(x_i)))$$ for some PRF $$f$$ with domain and codomain $$\{0,1\}^n$$, 
it is infeasible for one predict $$f(x_{i+1})$$ for some $$x_{i+1}$$ with probability non-negligibly larger than $$2^{-n}$$ [1].
PRFs are the building blocks for message authentication codes (MACs), among other things. In practice, PRFs are 
typically constructed as keyed hash functions -- HMAC [1] being one of the more prominent examples. HMAC takes as input
a key $$k$$ and message $$x$$ to produce an output $$t$$. The security goal for HMAC, or any MAC for that matter, is 
that, without knowledge of the key, one is unable to predict $$t$$ given $$x$$. More formally, this means the MAC scheme
is secure against existential forgery attacks. This is precisely the behavior we would expect from a PRF. 

[1] standard PRF textbook definition
[2] HMAC

XXX: HMAC code in Go [https://golang.org/src/crypto/hmac/hmac.go]

# Tweakable PRFs

XXX: CMAC is OMAC1, submitted as a NIST standard

A tweakable PRF (TPRF) is similar to a tweakable block cipher [3] in that the tweak is a cheap way to individualize each 
invocation. (In that sense, a tweak has intent similar to a nonce, especially since the tweak can be public and
often changed quickly.) A TPRF is therefore one that accepts a tweak as input to individualize the output.
This is useful in cases where we want the PRF output to be different for a given message $$m$$ with the same
key $$k$$. The One-Key MAC (OMAC) construction [4] is one type of TPRF out in the wild. Unlike tweakble block ciphers, 
wherein the tweak must somehow affect each output block of ciphertext, the TPRF tweak need only affect the final
PRF output. This greatly simplifies the design, as illustrated below.

XXX: insert the figure here

[3 tweakable block cipher paper]
[4 omac]

AES-CMAC standard: https://tools.ietf.org/html/rfc4493#section-2.2 -> k1 and k2 may be viewed as tweaks which are derived based on the length of the input

XXX: write code for OMAC here

# Oblivious PRFs

# Verifiable PRFs

# Oblivious Verifiable PRFs

# Constrained PRFs

# Privately Constrained PRFs

# Puncturable PRFs
