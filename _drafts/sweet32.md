---
layout: post
title: The Bitter Sweet Reality of Block Cipher Collisions
---

I started reading Schneier et al.'s "Cryptography Engineering" last week. It's a great book
and a helpful reminder for many of the small details that make cryptography a difficult subject.
One of the running goals of the book is to talk about how two parties can communicate securely
with a security level of $$2^{128}$$. This means that, in the best case, any viable attack requires
at least $$2^{128}$$ units of work, which is well beyond practicality. 

One mathematical hurdle we struggle with is the birthday paradox. It says that given $$n$$ randomly
selected values, the probability that any two of them are equal is roughly $$n^{1/2}$$. This means that
for block ciphers with a block size of $$128$$ bits, there is a $$2^{-64}$$ probability that two ciphertexts
will collide (be equal). Depending on the mode in which this cipher is used, this can reveal quite a lot
of information. For example, if CBC mode is used, then a collision in two ciphertext blocks means that there
is a collision in the plaintext. Specifically, if blocks $$C_i = E_k(C_{i - 1} \oplus P_i)$$ and $$C_j = E_k(C_{j-1} \oplus P_j)$$
collide, then it follows that the inputs to these independent encryptions are identical. Therefore,
we know that $$C_{i-1} \oplus C_{i-1} = P_i \oplus P_j$$. This holds because $$E_k(\cdot)$$ is a 
PRP, or pseudorandom permutation. If we replaced $$E_k(\cdot)$$ with a PRF $$F_k(\cdot)$$, the same
would not be true. This is because it is possible for $$F_k(x) = F_k(y)$$ where $$x \not= y$$.

If the simpler CTR mode was used with this block cipher, then the collision reveals significantly
less information. (In fact, using CTR mode with a PRF is secure up to $$2^n$$ blocks, unlike CBC mode [1].)
Specifically, if $$C_i = P_i \oplus E_k(Nonce || i)$$ collides with 
$$C_j = P_j \oplus E_k(Nonce || j)$$, then by the permutation properties of $$E_k(\cdot)$$ it
follows that $$P_i = P_j$$ iff $$Nonce || i = Nonce || j$$, which is not possible since 
$$i \not= j$$ and $$E_k(\cdot)$$ is unique. Therefore, while it is not straightforward to
learn specific values of the plaintext, it is possible to distinguish the cipher in CTR mode from an 
ideal block cipher. To do so, one would encrypt $$2^{n/2}$$ blocks of identical plaintext
using CTR mode. If a collision occurs, which will happen with relatively high probability 
after that amount of encryptions with the same key, then the cipher is clearly a PRP in CTR
mode. 

Why is this important when most block ciphers we use have a 128-bt block size, and therefore
a $$2^{64}$$ security boundary? Well, not all block ciphers in use have such a large block size.
3DES, for example, has a 64-bit block size. This means that the CBC attack variant is possible
after collecting $$2^{32}$$ blocks of ciphertext encrypted with the same key. The recent
Sweet32 attack [2] examined this particular attack and its practicality on lightweight
ciphers such as 3DES, PRESENT [3], and HIGHT [4]. With web services that offer 3DES as a possible
cipher and without any bounds on session (and key) lifetime, this means that cleverly placed
code to record encrypted blocks can be used to carry out this type of leakage attack. 

The authors of the Sweet32 paper say that the proper mitigation strategy is to use a 128-bit
block cipher such as AES. It's standardized, optimized, and well-understood. However,
for legacy reasons, there are devices that simply cannot use this mode. In this case,
special care has to be taken to ensure that the amount of data encrypted with the same
key is less than the birthday bound ($$2^{32}$$ in the 3DES case). Re-keying is an important
part of session management [5] as it can significantly increase the security properties
of the encryption scheme in use. (This is intuitive since re-keying is basically refreshing
the birthday bound count back to zero.) 

So what can you take away from this attack? For starters, you should be using modern cryptographic
algorithms where possible. Second, you should be managing your secrets properly. One-time keys
are the best we can do from a security perspective. And since generating them is difficult, we should
limit the amount of times we use our ephemeral keys. And last but certainly not least, you should
be mindful of the ways in which you use cryptography, even if a certain approach is considered
best practice.

# References

- [1] M. Bellare, A. Desai, E. Jokipii, and P. Rogaway. A concrete security treatment of symmetric encryption. In 38th FOCS, pages 394–403. IEEE Computer Society Press, Oct. 1997.
- [2] [https://sweet32.info/SWEET32_CCS16.pdf](https://sweet32.info/SWEET32_CCS16.pdf)
- [3] A. Bogdanov, L. R. Knudsen, G. Leander, C. Paar, A. Poschmann, M. J. B. Robshaw, Y. Seurin, and C. Vikkelsoe. PRESENT: An ultra-lightweight block cipher. In P. Paillier and I. Verbauwhede, editors, CHES 2007, volume 4727 of LNCS, pages 450–466. Springer, Heidelberg, Sept. 2007.
- [4] D. Hong, J. Sung, S. Hong, J. Lim, S. Lee, B.-S. Koo, C. Lee, D. Chang, J. Lee, K. Jeong, H. Kim, J. Kim, and S. Chee. HIGHT: A new block cipher suitable for low-resource device. In L. Goubin and M. Matsui, editors, CHES 2006, volume 4249 of LNCS, pages 46–59. Springer, Heidelberg, Oct. 2006.
- [5] [https://cseweb.ucsd.edu/~mihir/papers/rekey.pdf](https://cseweb.ucsd.edu/~mihir/papers/rekey.pdf)
