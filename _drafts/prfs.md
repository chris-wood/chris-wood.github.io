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

# Oblivious PRFs

OPRFs are a special type of PRF in which two parties work to compute $$y = F(k, x)$$,
where $$k$$ is a key owned by the sender and $$x$$ is an input owned by the receiver.
Thus, OPRFs are not functions, per-se; they're protocols. Moreover, they are essentially
a generalization of blind signature protocols pioneered by Chaum in YEAR [x].

One type of OPRF protocol was recently described by Hugo Krawczyk at RWC 2017 [4]. The
PRF itself computes $$s = F(k, m) = H(m)^{k}$$, where $$H$$ is a hash function that maps
its input to an element of a proper Diffie Hellman group and $$k$$ is a random exponent.
The protocol to compute this functionality runs as follows:

1. The client computes $$H(m)^r$$ for some random value $$r$$ and sends the result to the server.
2. The server computes $$(H(m)^r)^k = H(m)^{rk}$$ and returns it to the client.
3. The client computes $$(H(m)^{rk})^{r^{-1}}$$, yielding $$H(m)^k$$.

By running this protocol, the client gets a signature $$s$$ over a random element in the group $$m$$.

# Verifiable PRFs

Verifiable PRFs (VPRFs) are a type of PRF in which one computes $$y = F(k, x)$$
and a (NP-proof) *proof* that $$y$$ is the correct output given $$x$$. Moreover,
the proof reveals nothing about other values $$y' \not= y$$ and $$x' \not= x$$.
It is important to note that $$k$$ is typically a *private key*, rather than a
symmetric key as in the traditional PRF model. (This is intuitively necessary
for the proof generation.)

VPRFs were first introduced by Micali in 1999. They have since found traction in transparency-related
projects. Specifically, since they permit data structures storing PRF proofs to be audited
for correctness without revealing the (potentially sensitive) input to the PRF.
(The Key Transparency [x] and CONIKS [y] projects are two very relevant examples of
this strategy. The exact VPRF used in both works is described below.)

We now describe a simple VPRF computation as made popular by CONIKS [y] and introduced
in [3]. Let $$G$$ be a group of prime order $$q$$ and generator $$g$$. The PRF key
$$k$$ is chosen at random from $$(1,q)$$. The corresponding public key is published
as $$Y = g^k$$. Let $$H_1$$ and $$H_2$$ be two hash functions, where $$H_1$$ maps
arbitrary messages onto $$G$$ and $$H_2$$ maps arbitrary messages into the range
$$(1,q)$$. Given this setup, the VPRF computation is simply: $$VRF_k(m) = H_1(m)^k$$.

The proof is based on a non-interactive proof of discrete log equality. Specifically,
since the public key is $$Y = g^k$$ and the VPRF output is $$v = h^k$$, where $$h = H_1(m)$$
for some message $$m$$, it suffices to produce a proof that $$log(Y) = log(h^k)$$.
The authors of [y] suggest using the Fiat-Shamir heuristic from [14]. Specifically,
the prover derives (computes) the following values:

$$
r \gets (1,q) \\
s = H_2(g, h, Y, v, g^r, h^r) \\
t = r - sk (\mod q)
$$

The prover then transmits $$(v, s, t)$$ to the verifier, who checks that equality
holds for the following expression:

$$
s = H_2(g, h, Y, v, g^t \cdot Y^s, H_1(m)^t \cdot v^s)
$$

Observe that $$g^t \cdot Y^s = g^{r - sk} \cdot g^{ks} = g^r$$ and
$$H_1(m)^t \cdot v^s = h^{r - sk} \cdot h^{ks} = h^r$$, as desired.

http://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=814584
http://www.jbonneau.com/doc/MBBFF15-coniks.pdf
[3] http://fc13.ifca.ai/proc/5-1.pdf

- Micali, Silvio, Michael Rabin, and Salil Vadhan. "Verifiable random functions." Foundations of Computer Science, 1999. 40th Annual Symposium on. IEEE, 1999.

https://people.csail.mit.edu/silvio/Selected%20Scientific%20Papers/Pseudo%20Randomness/Verifiable_Random_Functions.pdf

# Oblivious Verifiable PRFs

# Puncturable PRFs

# Constrained and Delegated PRFs

https://eprint.iacr.org/2014/522.pdf

# Privately Constrained PRFs
