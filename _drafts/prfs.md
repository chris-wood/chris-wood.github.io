---
layout: post
title: PRFs Galore!
---

# Overview

Cryptographic protocols are built on abstractions and approximations of ideal mathematical constructs.
For example, we often assume that we have a perfect source of randomness. But in practice, we settle
for pseudorandomness. That is, a source of randomness that is indistinguishable from true randomness
in the presence of a computationally restricted adversary. Another pure construct is a random function.
A random function is a function $$f : \{0,1\}^n \to \{0,1\}^n$$ such that for some $$x \in \{0,1\}^n$$,
$$f(x)$$ is seemingly chosen at random from the codomain $$\{0,1\}^n$$. There are $$^{2^n}$$ different
random functions over the domain $$\{0,1\}^n$$. Like randomness, we often settle for an approximation
of this ideal construct in the form of a pseudorandom function, or PRF. As the name suggests, a PRF
simply approximates a truly random function. And they're used everywhere in cryptography. However,
what I find more interesting are the different flavors of PRFs that exist, including: standard PRFs,
oblivious PRFs, verifiable PRFs, and, yes, even oblivious *and* verifiable PRFs. In this post I'll describe
and implement (simple) versions of these constructs.

# Standard PRFs

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

<!-- # Oblivious Verifiable PRFs -->



# Constrained PRFs

Standard PRFs allow anyone with the key $$k$$ to evaluate $$F(k, x)$$. A constrained PRF
is one in which it is possible to derive a child or sub-key $$k_S$$ from $$k$$ that is restricted,
or constrained, to a certain subset S of the domain. Thus, with $$k_S$$, one can evaluate
the PRF at any point $$x \in S$$ but at no point $$x \notin S$$. Constrained PRFs were
first put forward by Boneh and Waters in [x]. They present a number of useful PRFs that
can be built in the constrained model, including:

- Left-right PRFs: A left-right PRF is one where the input domain is two dimensional (hence, left-right).
Evaluating the PRF at one point in the first dimension requires the "left key," whereas evaluating the PRF
at a point in the other dimension requires the "right key." Constrained keys can be used to restrict
evaluation to either domain. Boneh and Waters use this type of construction to build an identity-based
non-interactive key exchange protocol.

- Circuit PRFs: A PRF in which evaluation is restricted to all points $$x$$ such that $$C(x) = 1$$ for some
circuit $$C$$. This is perhaps the most flexible of the bunch with many appealing uses.

The (circuit-based) constrained PRFs of Boneh and Waters are built on multilinear maps. A multilinear map is
a natural extension to bilinear maps that join or pair $$k$$ input elements to form a single element of
the target group, i.e., $$e(g^{a_1}, \dots, g^{a_k}) = g^{a_1 \dots a_k}$$. The construction assumes constraints
are expressed as circuits composed of AND and OR gates. The input elements are of length $$n$$ and the maximum
depth of the circuit is $$l$$.

Each input pair $$d_{i,0}, d_{i,1}$$ of gate $$i$$ is assigned
a random element from $$Z_q$$, where $$q$$ is a suitably large prime. These constitute private keys. The public
counterparts are $$D_{i,0} = g^{d_{i,0}}$$ and $$D_{i,1} = g^{d_{i,1}}$$, respectively. Given such a circuit,
the PRF is constructed as:

$$
F(k, x) = g_{n + l}^{u \prod_{i \in |n|}d_{i, x_i}},
$$

where $$u$$ is a random exponent part of the secret key $$k$$ and $$x_i$$ is the $$i$$-th bit
of $$x$$. Functionally, this equates to evaluating the circuit given the input $$x$$. Constraining
the PRF is done by specifying and constructing such a circuit to be evaluated. (I leave it as
a homework problem to myself to implement this scheme to understand it better.)

As a finale note, the authors note that even standard PRFs can be made constrained. In particular, we may
constrained the input domain to a single domain element $$\{x\}$$ with key $$k_{\{x\}}$$ by letting
$$k_{\{x\}} = F(k, x)$$ for any standard PRF $$F$$. Computing the PRF for $$k_{\{x\}}$$ at point $$x$$
simply involves revealing $$k_{\{x\}}$$.




XXX: discuss the related work below. (bit-fixing PRF in the random oracle model, in particular, based on the GGM construction!)
- include the image for completeness

---- relatedwork

https://eprint.iacr.org/2014/372

https://eprint.iacr.org/2014/416.pdf


- [x] https://eprint.iacr.org/2013/352.pdf

- Related work on delegatable and functional PRFs.
A. Kiayias, S. Papadopoulos, N. Triandopoulos, and T. Zacharias. Delegatable pseudorandom functions and applications. In Proceedings ACM CCS, 2013.
E. Boyle, S. Goldwasser, and I. Ivan. Functional signatures and pseudorandom functions. Cryptology ePrint Archive, Report 2013/401, 2013.

# Puncturable PRFs

A puncturable PRF is special type of constrained PRF in which evaluation of the PRF is
limited to all points $$x$$ that have not been "punctured" from the input domain space. Each derived
key is associated with an input domain for which the PRF cannot be evaluated. Thus, 
it's the opposite of a constrained PRF, which restricts evaluation of the input to any element
in the corresponding domain.

https://eprint.iacr.org/2014/521.pdf


# Delegated PRFs

One might wonder if constrained PRF keys can be continually constrained to smaller and smaller subsets
of the input domain. Or, in other words, whether or not the PRF could be delegatable. Chandran et al.
asked and answered this question in [x]. Interestingly, their construction is yields a verifiable PRF
as well. They coin the construction a verifiable constrained random function (VCRF). It accepts
any constraint that can be encoded in a polynomially-sized circuit. Delegation works by further
restricting existing constrained keys. For example, given a key $$k_f$$ that is constrained by some
function $$f$$, one may further constrain (or delegate) this key to another constraint $$f'$$,
yielding $$k_{f \land f'}$$.

XXX: continue me here

- [x] https://eprint.iacr.org/2014/522.pdf

# Privately Constrained PRFs

XXX

# Key Homomorphic PRFs

A key homomorphic PRF is one in which it is possible to efficiently compute $$F(k_1 \oplus k_s, x)$$
given $$F(k_1,x)$$ and $$F(k_2, x)$$, where $k_1 \not= k_2$ and $\oplus$ is some *group* operation
such as XOR. These PRFs were first introduced by Naor et al. in [y] as a way of building so-called
distributed PRFs. Imagine there are $$n$$ independent, non-colluding servers, where $$S_i$$
has key $$k_i$$. With a key homomorphic PRF, it is possible for a client to request $$F(k_i, x)$$
from each server $$S_i$$ and then combine the result into $$F(k_1 \oplus \dots \oplus k_n, x)$$.
Another fascinating application of this PRF is symmetric-key proxy re-encryption. Imagine
we have a key homomorphic PRF such that $$F(k \oplus k', x) = F(k_1, x) \otimes F(k_2, x)$$,
where both $$\oplus$$ and $$\otimes$$ are group operations. Moreover, imagine we encrypt
a message $$m$$ by computing the $$j$$-th block as $$c_j \gets m_j \otimes F(k, N + j)$$
for some initial counter value $$N$$. We may re-encrypt this ciphertext from key $$k$$
to key $$k'$$ by modifying each block as $$c_j \gets c_j \otimes F(-k \oplus k', N + j)$$.
Unfolding this computation we obtain

$$
c_j \otimes F(-k \oplus k', N + j) = m_j \otimes F(k, N + j) \otimes F(-k \oplus k', N + j) = m_j \otimes F(k', N + j),
$$

as desired. Boneh et al. [x] also show how to use this scheme to efficiently rotate keys
for encrypted data stored in the cloud. Clients simply send the cloud -- the proxy --
re-encryption tokens ($$-k \oplus k'$$) to apply to each existing ciphertext.

Constructing these PRFs is simple in the random oracle model. Let $$H$$ be a hash
function that maps inputs to a Diffie Hellman group $$G$$, and let $$\otimes$$ be the group
operation (multiplication). Then, the PRF is simply $$F(k, x) = H(x)^k$$.
Homomorphism follows easily. Given $$F(k_1, x)$$ and $$F(k_2, x)$$, we may combine
them as follows:

$$
H(x)^{k_1} \otimes H(x)^{k_2} = H(x)^{k_1 + k_2} = F(k_1 + k_2, x)
$$

Boneh et al. [x] gave a construction for key homomorphic PRFs in the standard model
based on the LWE problem. 

- [y] M. Naor, B. Pinkas, and O. Reingold. Distributed pseudo-random functions and kdcs. In EUROCRYPT 99, pages 327–346. Springer, 1999.
- [x] http://crypto.stanford.edu/~dabo/papers/homprf.pdf

