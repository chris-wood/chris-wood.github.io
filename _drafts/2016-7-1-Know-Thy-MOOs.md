---
layout: post
title: Know Thy MOOs
---

Block ciphers are symmetric-key encryption algorithms for encrypting 
discrete blocks or chunks of data. The size of the block determines how
much data can be encrypted in one invocation of the cipher. AES, for example,
has a block size of 128 bits with key sizes of 128, 192, and 256 bits. To
encrypt larger pieces of data we run these ciphers in so-called modes of operation (MOO).
There are a variety of MOOs, including electronic codebook (ECB), cipher block
chaining (CBC), cipher feedback (CFB), output feedback (OFB), and counter mode (CTR).
A single, isolated invocation of AES is the ECB mode [1]. Specifically, in this mode, 
each block is encrypted independently from the rest. Everyone knows (or should know) that 
this leaks a significant amount of information about the underlying plaintext. 
Thus, in practice, it's not incredibly useful, even though each block can be processed
in parallel for better performance. 

Naturally, the choice of what MOO to go with depends on the use case. If efficiency is
a requirement, then one might benefit from using a MOO that can be parallelized (for either
encryption or decryption). Other factors to consider include whether or not the message
should be padded [2], whether or not random access to plaintext data is needed [3], and
whether integrity in addition to confidentiality is important [4]. The CTR mode is widely
popular because it can be easily parallelized, does not fall prey to padding attacks like
the CBC MOO [2], and permits random read access. There are other MOOs that provide properties
similar to CTR. CFB, for example, which uses the output of a ciphertext block as input
to the subsequent ciphertext block, allows for random read access. As a consequence of
its linear encryption process, however, it is not as efficient. CBC is similar to CFB 
in that the output from a single block is carried over into subsequent blocks. However,
unlike CBC, the plaintext XOR step is performed after the key stream generation. In CBC,
the plaintext is XORd with the IV and then encrypted to create the initial ciphertext block.
This allows an attacker with knowledge of the IV and control of the plaintext to create
encrypted versions of any initial plaintext block, which is bad.

Rarely are these MOOs used in practice alone. AEAD constructions exist to add integrity
on top of these confidentiality MOOs without sacrificing the underlying features. 
AEAD algorithms include counter with CBC-MAC (CCM), Galois counter mode (GCM), ChaCha20+Poly1305, 
and the synthetic initialization vector (SIV) (the latter being a special type of AEAD that is
deterministic, i.e., it does not require an explicit IV). GCM is by far the most popular
AEAD construction used today since it is easily parallelizable, has hardware support,
and follows the encrypt-then-MAC design strategy for creating secure authenticated encryption
schemes. (MAC-then-encrypt is not secure against universal forgeries.) The general consensus 
is that if confidentiality and integrity is required then one should use one of these
AEAD schemes. 

Why then do companies like Netflix use CBC for encrypting media files? According to an
employee of Netflix, they use CBC in order to permit random access. Sometimes data chunks 
are transferred over a secure session and sometimes they are not. As users (randomly) seek 
throughout a movie they can be transferred stateless encryptions of the media without requiring
any intermedate chunks. However, CBC encryption is malleable. It would be perfectly reasonable
for an attacker to modify a single block of ciphertext, which would influence that same plaintext
block and the subsequent one (see [1] for the pipeline), and have the plaintext result be 
decryption okay. Therefore, it is not unreasonable to imagine an attacker that performs
such malleable modifications to censor or filter certain parts of the media. Without any integrity 
check, it's impossible to detect this particular problem. 

Now, I'm sure Netflix takes precautions against this type of attack, but it's interesting to
think about. What's more puzzling is why they don't just use an AEAD construction in the first
place. AES-GCM, for example, does permit this same type of random access in the underlying
CTR mode. However, integrity cannot be ascertained without possession of the entire ciphertext
block. This could be addressed by breaking down the media encryption into smaller blocks. For example,
instead of encrypting a media file with $$k = m \times n$$ plaintext blocks and transferring a single
GCM tag, one could encrypt $$m$$ blocks $$n$$ times and send $$n$$ tags. These tags could also be injected
in the stream of ciphertext blocks so that random access is still permissable after the collection of $$m$$
ciphertext blocks and a tag. 

Regardless of the approach here, there are options better suited than CBC for
encrypting large media files while still supporting random access. Maybe Netflix 
uses CBC because it's generally considered to be safe when used correctly (pad your
messages!). Or maybe they use CBC because switching to GCM is too much of a headache
and storing and transferring some extra bytes for MAC tags is too costly. I can only
speculate, but I think they can do better.

# References 

- [1] http://csrc.nist.gov/publications/nistpubs/800-38a/sp800-38a.pdf
- [2] Yau, Arnold KL, Kenneth G. Paterson, and Chris J. Mitchell. "Padding oracle attacks on CBC-mode encryption with secret and random IVs." International Workshop on Fast Software Encryption. Springer Berlin Heidelberg, 2005.
- [3] Martin, Luther. "XTS: A mode of AES for encrypting hard disks." IEEE Security & Privacy 3.8 (2010): 68-69.
- [4] Rogaway, Phillip. "Authenticated-encryption with associated-data." Proceedings of the 9th ACM conference on Computer and communications security. ACM, 2002.
