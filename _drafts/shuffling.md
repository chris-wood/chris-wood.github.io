---
layout: post
title: Shuffle Up and Deal
---

Consider the problem where we have $l$ parties with messages $m_i$ who wish to generate
the list $(m_{\pi(1)}, \dots, m_{\pi(l)})$, where $\pi$ is a permutation over the
integer range $[1,\dots,l]$. Suppose further that they wish to do this without
revealing $\pi$. This problem is essentially what mixnets were designed to solve. They
send the sender or provider of a series of messages received in sequence by shuffling
them around and about before forwarding them along. In doing so, the mixnet hides
the mapping between the $i$-th input message and the $i$-th output message.
In addition to permuting messages before forwarding them to their recipients, mixnets
usually also perform some type of decryption or re-encryption so that the input
and output messages are more difficult to correlate. One ideal property for these mixnets 
is that they work on opaque ciphertexts. In other words, it would be nice if the permutation
and translation (decryption or re-encryption) steps operated over ciphertexts without
having to observe the message plaintexts or know about the recipient of each message.
In fact, this is not farfetched at all. Adida et al. [1] show that any homomorphic 
encryption scheme can be used to build an inefficient decryption and re-encryption shuffling 
schemes by obfuscation. They also show how to construct efficient shuffling schemes based on
the BGN and Paillier encryption schemes. In this post, I'll review the fundamental idea behind
their shuffle algorithm and then describe an implementation of their re-encryption shuffle
algorithm based on a generalized Paillier encryption scheme.

# Permutations with Homomorphic Encryption

Let $m = [m_1,\dots,m_l]$ be a vector of $l$ messages we wish to permute by some permutation $\pi$. 
An easy way to do this is to generate a permutation matrix $\mathbf{A}$ where the rows are permuted according
to $\pi$. The matrix-vector product $m' = \mathbf{A}m$ then returns $m$ permuted under $\pi$. 
Let's take a closer look at how $m'_i$ is computed. By the matrix-vector multiplication
algorithm, it holds that

$$
\begin{align}
m'_i = \sum_{i}^{l} \mathbf{A}_{j, i} m_i
\end{align}
$$

What if the entries of $\mathbf{A}$ and $m$ were encrypted under a multiplicative homomorphic
encryption scheme? The product of the encryptions of $\mathbf{A}_{j, i}$ and $m_i$ would be 
equal to the encryption of the product $\mathbf{A}_{j,i}m_i$. Multiplying by an encrypted
"1" is sort of like re-encrypting the input. Moreover, decryption does not reveal $\pi$ 
if the encryption scheme is semantically secure. Intuitively, this is because the encryption
reveals nothing about the plaintext, i.e., the inputs from $\mathbf{A}$. 

The authors of [1] use the additive and multiplicative properties of the BGN encryption
scheme to implement a decryption shuffle algorithm, i.e., one in which decrypts and then
outputs a permutation of its inputs. Personally, I find re-encryption shuffling algorithms
to be more intriguing since they do not require the messages to be decrypted. So, the remainder
of this post is dedicated to illustrating their re-encryption shuffle algorithm with
some running code. But first, we need to do some review.

# A Review of Paillier

The Paillier encryption system is a tuple of algorithms $(\mathsf{G}, \mathsf{E}, \mathsf{G})$ for 
key generation, encryption, and decryption defined as follows (adapted from [1]).

TODO: change this based on my notebook
- Key generation: On input $1^{\kappa}, choose two random safe primes $p = 2p' + 1$ and $q = 2q' + 1$,
where $p'$ and $q'$ are also prime $n = pq$ is a $\kappa$-bit integer. Output the public key
$pk = n$ and secret key $sk = p$. 
- Encryption: On input $pk$ an $m \in \mathbb{Z}_n$, randomly select $r \in \mathbb{Z}_n^*$ and compute
$c = (n + 1)^mr^n \mod n^2$. 
- Decryption: On input $sk$ and $c$, given $e \cong 1 \mod n$ and $e \cong 0 \mod \phi(n)$, output
$p = (c^e - 1) / n$. 

To show correctnessm, assume we encrypted a message $m$ with the
public key $pk$ and retrieved the ciphertext $c$. We would expect for the output of the decryption
algorithm given $c$ and the secret key $sk$ to yield $p$ where $p = m$. Let's walk through
the decryption computation.

$$
\begin{align}
    // TODO
\end{align}
$$

TODO: finish spec and then show the code

# Generalizing Paillier

TODO: spec and code

# The Re-Encryption Shuffle Algorithm

TODO: write it down and then give the code

# References 
- [1] https://eprint.iacr.org/2005/394.pdf
