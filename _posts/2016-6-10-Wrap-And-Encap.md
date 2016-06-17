---
layout: post
title: Key Wrapping and Encapsulation
---

One of the synonoms for the word 'wrap' is 'encapsulate.' However, 
in cryptography, and especially with respect to keying material, these 
two words mean very different things. In this post I'll explain these
two terms and what they mean. (Mostly as a way to get it straight in my
mind as well.)

# Key Wrapping

Key wrapping is a cryptographic algorithm which aims to provide "privacy and
integrity protection for specialized data such as cryptographic keys, ... without
the use of nonces" [1,2]. Without knowing any more details, there are two terms
that should resonate with cryptography folks: privacy and integrity. Or, said differently,
confidentiality and integrity. Authenticated encryption with associated data (AEAD) is one 
class of algorithms that provide these properties. Popular examples of AEAD algorithms
include AES-GCM, ChaCha20+Poly1305, and more. However, existing AEAD algorithms require
the use of nonces to ensure randomness that's required for CCA-security. So, as Phil
Rogaway framed it, the key wrapping problem is about creating *deterministic 
authenticated encryption* (DAE) algorithms [1].

DAE algorithms accept a symmetric key, a header (i.e., the associated data), and a
message and returns a ciphertext. The inverse operation takes the same key, header, and
ciphertext to produce the plaintext message. The security of the DAE scheme is defined
similarly to AEAD schemes. Namely, the adversary is given access to either (a) an 
encryption and decryption oracle or (b) a fake encryption and decryption oracle that
return random bits and failures, respectively. If the adversary cannot distinguish
between which oracle it received, then the DAE scheme is secure. If the adversary
could in fact distinguish between these two cases, then some information about 
the plaintext is leaked by the ciphertext from the encryption oracle or forgeries
are possible. Thus, privacy and integrity are captured by this definition. 

Rogaway's DAE construction is based on what he calls a synthetic initialization vector (SIV).
I don't know the details of this construction yet myself, so I'll defer that to a future
post. (Possibly when I'm on the plane to Germany this Saturday.)

# Key Encapsulation

Encapsulation, unlike wrapping, is about creating *provably secure* hybrid encryption
schemes. That is, public-key encryption when the message cannot be mapped to an element
of the PK group, e.g., an ElGamal or RSA group element.
The traditional way to do this is as follows: First, generate a random 
symmetric key K and encrypt the message M with the data to produce some ciphertext C. 
Then encrypt K under the recipient's public key to obtain the encrypted key K'. 
Finally, transfer (C, K') to the recipient. If they possess the right private key, 
they can decrypt K and then C to obtain M. Easy enough.

Sadly, to my knowledge, there have been no security proofs for these schemes. Moreover,
most schemes, such as RSA-OEAP [3], require the input to be padded, though it is still
secure. Padding schemes such as those defined in PKCS #1 v1.5 are *not secure* [4]. 

Key encapsulation was introduced as a way to solve this problem. Instead of encrypting 
the random (and padded) symmetric key, an element of the group, which is 
*used to derive the symmetric key*, is encrypted. To give an example, here's how 
the RSA key encapsulate mechanism works (paraphrased from [5] for simplicity).

1. Generate a random integer $$z < N$$.
2. Encrypt $$z$$ under the recipient's public key $$(N, e)$$ to obtain $$z'$$.
3. Derive the encryption key (or KEK, in the case where a symmetric key is being encrypted
for transport) using a KDF function with $$z$$ as input.
4. Encrypt the data $$M$$ using the encryption key to obtain $$C$$.
5. Output $$(z',C)$$.

To inverse this process and recover the data $$M$$, the recipient of this tuple 
decrypts $$z$$, computes the encryption key, and then inverses the symmetric-key
encryption algorithm. Easy enough.

# "Wrapping" Up

Key wrapping is DAE for *any type of message*, but particularly keys, to a party
that has the keys necessary to decrypt the message ciphertexts, whereas key 
encapsulation is about securely *transmitting* symmetric keys to another party and
using them to encrypt *any data*. Key wrapping requires only symmetric-key encryption,
whereas key encapsulation requires public-key encryption. 

# References

- [1] Rogaway, P., and T. Shrimpton. "Deterministic Authenticated-Encryption." Advances in Cryptology–EUROCRYPT. Vol. 6. 2007.
- [2] Dworkin, M. Request for review of key wrap algorithms. Cryptology ePrint report 2004/340, 2004. Contents are excerpts from a draft standard of the Accredited Standards Committee, X9, entitled ANS X9.102 — Wrapping of Keys and Associated Data.
- [3] Fujisaki, Eiichiro, et al. "RSA-OAEP is secure under the RSA assumption." Advances in Cryptology—CRYPTO 2001. Springer Berlin Heidelberg, 2001.
- [4] Coron, Jean-Sébastien, et al. "New attacks on PKCS# 1 v1. 5 encryption." Advances in Cryptology—EUROCRYPT 2000. Springer Berlin Heidelberg, 2000.
- [5] https://lists.w3.org/Archives/Public/public-xmlsec/2009May/att-0032/Key_Encapsulation.pdf
