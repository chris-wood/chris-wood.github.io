---
layout: post
title: Extractors and Key Derivation
---

TLDR: Not all bit strings are created equal. Ideally, each bit of a random $$n$$-bit string has uniform
probability. However, each bit of a random group element which happens to be encoded in a $$n$$-bit string
certainly does not have uniform probability. Extractors serve to remove structure from otherwise structured
data and produce near-uniformly distributed strings for use in other cryptographic algorithms.

# Generating Random Bit Strings

Most cryptographic algorithms have some need for randomness. Block ciphers, stream ciphers, and PRFs all require
that their input keys are uniformly distributed bit strings. In some scenarios it may be appropriate to generate
these keys locally for use within a trusted environment, e.g., a secure enclave. However, the more common case
is when keys are derived via the execution of some protocol, such as TLS [1] or IKE [2]. And more often than not,
the exact key exchange mechanism is based on the legendary Diffie Hellman (DH) protocol. (The now-legacy RSA key exchange
algorithm is, or should be, only discussed for historical reasons, as it does not provide forward secrecy.)
The DH protocol is simple. Let $$g$$ be a generator for some finite and cyclic group $$G$$, and let $$\alpha$$ and $$\beta$$ be two
randomly generated elements in $$G$$ owned by Alice and Bob, or A and B. The protocol is based on the following
property: $$(g^\alpha)^{\beta} = (g^\beta)^{\alpha}$$. Executing this protocol yields a common value shared between
A and B alone. And for many, that's the end of the road. Use that shared secret *as a key* to start encrypting data
between A and B.  

[1] TLS
[2] IKE

In practice, this is only half the story. A key-derivation function (KDF) is often used to *extract* some suitable
amount of randomness needed to be used as a key. TLS, for example, uses the Hash-based KDF (HKDF) [3] scheme specified in
RFC XXXX [4] to transform the output of the DH protocol into a secret suitable for use as a block cipher key. It does
not matter if the DH output is encoded in a 128- or 256-bit string. Put simply, because the output is an element of some group,
each bit in the bit-wise representation of elements in said group is not uniformly distributed, the output cannot be used
as a key.

Cryptographic secrets, especially symmetric keys, must be uniformly distributed. Any knowledge of the key harms the
overally security of the key, if by no other reason than reducing the brute force attack complexity on algorithms using said key.
In the next section, I will explore the distribution of group elements used in common cryptographic algorithms
and protocols, such as ECDH algorithms based on secp2256r1 [???] and Curve25519 [???].

[4] HKDF
[3] https://eprint.iacr.org/2010/264.pdf

# Random Group Elements are Not Random Bit Strings

Many cryptographic protocols are built on groups. Common groups include $$Z_p^*$$, i.e., the finite field over the integers modulo some prime
$$p$$, and elements over elliptic curves such as secp256r1 and Curve25519. Consider what is the bit-wise distribution of elements in this group.
Take $$Z_{11}$$ as an example. $$Z_{11} = \{1, 2, 3, 4, 5, 6, 7, 8, 9, 10\}$$. In binary, these elements are encoded as follows:

$$
0000 \\
0001 \\
0010 \\
..\\
1000 \\
1001 \\
1010 \\
1011 \\
$$

There are $$10$$ elements in $$Z_{11}$$. They are encoded in $$4$$ bits. Distinguishing an encoding of random elements in this
group from random 4-bit strings is trivial: simply examine the most-significant bit. For a truly random bit string, the probability
that the MSB equals $$1$$ is exactly 0.5. However, across all elements in this group, the probability that the MSB equals $$1$$
is precisely $$4/10 = 0.4$$. The encoding of the algebraic structure leaks enough information to distinguish these elements
from truly random bit strings.

We can see exactly how each bit in an element over secp2556r1 and Curve25519 are distributed. The figure below shows the
bit-wise distribution of Curve25519-generated shared secrets without any randomness extraction.

![Curve25519 - no extraction](/images/extractor_plots/curve25519_raw.png)

The dip near the end should not be surprising. Consider how an element on Curve25519 is
defined. Per [curve2219], each private key is generated by (1) sampling a random 32-byte
string, (2) ANDing the MSB with 0xF8 (11111000) and the LSB with 0x7F (01111111), and (3)
ORing the LSB with 0x40 (0100000). Thus, multiplying any Curve25519 private key by a
corresponding public key will XXX...

The instructions on [site - https://cr.yp.to/ecdh.html] state that the output of the
Curve25519 DH should be hashed before used as a key. The figure below shows the bit-wise
distribution after running the output through HKDF.

![Curve25519 - extraction](/images/extractor_plots/curve25519_extract.png)

Collecting this for more samples would surely smooth the results. However, that is not needed to vanquish the glaring
difference between these two figures. When the shared secret is properly extracted into a random string, the probability
of the XXXXX LSB being $$0$$ returns to the expected value of approximately $$0.5$$.

It's worth noting that the secp256r1 DH algorithm output does not produce a group element with this same deficiency.
I repeated the same experiment for secp256r1. The unextracted distribution is below.

![secp256r1 - no extraction](/images/extractor_plots/p256_raw.png)

And the distribution with the extracted output is below.

![secp256r1 - extraction](/images/extractor_plots/p256_extract.png)

Barring statistical noise, these are essentially the same (graphically).
This is to be expected. Standard ECDH with secp256r1 is no different from ECDH with
any other curve (as far as I know). 

XXX: maybe the algorithm does extraction?

# Extractors and HKDF

As described by Shaltiei [x], randomess extractors are "algorithms that when given one sample from a weak random source, 
produce a sequence of truly random bits." Or, more precisely, extractors are "algorithms that map sources of sufficient min-entropy 
to outputs that are statistically close to uniform." They are a key ingredient to key exchange protocols. 
Beyond destroying any group structure that might be present in a shared secret, they have other uses in protocols:

- Key splitting: Extractors permit protocols to use two separate keys for reading and writing. 
This makes sequence counter (nonce) management easier and also increases the relative lifetime
of key reads and writes.
- Improved security: Security of Diffie-Hellman key exchange protocols (without extraction) relies on the 
Computational Diffie-Hellman problem. That is, given $$(g^a, g^b)$$, compute $$g^{ab}$$. This problem is
hard in many groups. A harder problem is the Decisional Diffie-Hellman problem, which is state as follows:
Given $$(g^a, g^b, g^{ab})$$ and $$(g^a, g^b, g^r)$$, where $$r$$ is a random element in the DH subgroup, 
distinguish between two tuples. By extracting a key from $$g^{ab}$$, security is improved from the CDH
to the DDH.

Extractors as used here are deterministic: they take no other input than the source of entropy, or the
key exchange shared secret. These are also often called computational extractors. 

XXX: continue with Hugo's paper
https://eprint.iacr.org/2010/264.pdf


- [x] https://cs.haifa.ac.il/~ronen/online_papers/ICALPinvited.pdf





Extractors are a beautiful part of cryptography. I wish I devoted more time
to reading about the subject.

XXX: statistical vs computational extractor, and a computational extractor
XXX: give the formal definition for extraction, and the HKDF algorithm, and an intuitive explanation!