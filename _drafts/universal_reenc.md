---
layout: post
title: Re-Randomization and Public Shuffling
---

I recently stumbled across a paper entitled, "Universal Re-Encryption for Mixnets,"
by P. Golle et al. [1]. Universal re-encryption (or, perhaps better called
*univeral re-randomization*) is a primitive that allows some ciphertext to be re-encrypted
without knowledge of the corresponding public key. The value of this technique
to Chaum-like mixnets [2] is quite obvious: untrusted mix nodes can, for 
example, re-encrypt messages from senders to receivers in order to further 
hide the relation between input and output messages. This process is shown in 
the figure below.

TODO: mixnet figure

In this post I'll present a basic implementation of an ElGamal re-randomizable 
encryption scheme. I'll then discuss two related encryption schemes -- Pallier 
and BGN -- how they are used in a similar problem called shuffling. That is, 
the process of *publicly* shuffling already encrypted messages.

# Re-Randomizable Encryption

A universal re-randomizable encryption scheme is a tuple of algorithms $(\mathsf{UKG], \mathsf{UE}, \mathsf{UD}, and \mathsf{UR_e})$
which are loosely described below.

- *Key Generation* $$\mathsf{UKG}$$: Generate the public and private keys for a user.
- *Encryption* $$\mathsf{UE}$$: Encrypt a message under a public key.
- *Decryption* $$\mathsf{UD}$$: Decrypt a message with a private key.
- *Re-Encryption* $\mathsf{UR_e}$: Re-encrypt or re-randomize some ciphertext. 

The term universal was introduced in [1] to mean that the re-randomization does not
need knowledge of the public key under which the some ciphertext was encryption (only
the system parameters, e.g., the ElGamal group generator and prime). This enables the
re-randomization process to be *transparent* and therefore something that can 
be done by any intermediate entity any number of times before the result is decrypted
by the intended recipient (i.e., the one holding the private key). 

## ElGamal

The universal re-randomization encryption scheme based on ElGamal from [1] is presented below.

- *Key Generation* $$\mathsf{UKG}$$: Output $$(PK, SK) = (y = g^x, x)$$ for $$x \in_U Z_q$$
- *Encryption* $$\mathsf{UE}$$: Input message $$m$$ and public key $$y$$, and output the ciphertext $$CT = [(\alpha_0, \beta_0), (\alpha_1, \beta_1)] = [(my^{k_0}, g^{k_0}), y^{k_1}, g^{k_1})]$, where $r = (k_0, k_1) \in Z_q^2$$.
- *Decryption* $$\mathsf{UD}$$: Input ciphertext $$CT = [(\alpha_0, \beta_0), (\alpha_1, \beta_1)]$$, compute $$m_0 = \alpha_0/\beta_0^x$$ and $$m_1 = \alpha_1/\beta_1^x$$, and output $m_0$ if $m_1 = 1$.
- *Re-Encryption* $\mathsf{UR_e}$: Input ciphertext $CT = [(\alpha_0, \beta_0), (\alpha_1, \beta_1)]$, compute and output $CT = [(\alpha_0', \beta_0'), (\alpha_1', \beta_1')] = [(\alpha_0\alpha_1^{k_0'}, \beta_0\beta_1^{k_0'}),(\alpha_1^{k_1'}, \beta_1^{k_1'})]$, where $r' = (k_0', k_1') \in Z_q^r$

Altogether, this scheme is quite straightforward. The only input is a generator $g$
and prime $q$ forming a group over $Z_q$. Encryption and decryption are very much
like the original ElGamal scheme presented in [3]. The only difference is that the ciphertext
carries both an encrypted message and an ElGamal signature. So, when the ciphertext
is re-randomized in the $\mathsf{UR_e}$ scheme, the encrypted message is re-blinded
and the signature is updated accordingly. 
Decryption only returns the original message if the signature is valid. If a message
or the signature is tampered then the verification, and decryption will fail.
Creating forgeries, on the other hand, is feasible (as I'll discuss below).
The security of this scheme (as presented) boils down to the Decisional 
Diffie-Hellman problem [4] over the group $Z_q$, much in the same way that the 
semantic security of the original ElGamal encryption scheme does. 

[3] ElGamal paper
[4] DDH 

The code for this procedure is shown below.

{% gist 8847a967d58ff4e8d344 %}

Observe that the input to the re-encryption algorithm is \emph{only} the ciphertext CT and 
the group parameter $q$. Different ElGamal ciphertexts are free to re-use this parameter since
it only determines the size of the ElGamal elements and does not have any affect on the 
choice of elements within (all samples are done uniformly at random from $Z_q$). 

TODO: show the code in action!

## A comment on forgeries

Several extensions on the URE scheme proposed above have been designed to build mixnets.
I'll focus on the work of Klonowski et. al [10 from IURE] -- you can see papers built
on their results by checking Google Scholar. They modify the URE scheme above to use an 
RSA signature instead of an ElGamal signature. And, just as before, the security of the 
scheme relies on the face that the signature remains valid even as the ciphertext is updated
by artbirary nodes. 

The formal description of this scheme, called RSA-URE (URE with RSA signatures), is below.  

TODO: MODIFY AS PER PAPER

"Uni- versal re-encryption of signatures and controlling anonymous information flow"
TODO: describe the extension in https://gnunet.org/sites/default/files/10.1.1.108.4976.pdf

- *Key Generation* $$\mathsf{UKG}$$: Output $$(PK, SK) = (y = g^x, x)$$ for $$x \in_U Z_q$$
- *Encryption* $$\mathsf{UE}$$: Input message $$m$$ and public key $$y$$, and output the ciphertext $$CT = [(\alpha_0, \beta_0), (\alpha_1, \beta_1)] = [(my^{k_0}, g^{k_0}), y^{k_1}, g^{k_1})]$, where $r = (k_0, k_1) \in Z_q^2$$.
- *Decryption* $$\mathsf{UD}$$: Input ciphertext $$CT = [(\alpha_0, \beta_0), (\alpha_1, \beta_1)]$$, compute $$m_0 = \alpha_0/\beta_0^x$$ and $$m_1 = \alpha_1/\beta_1^x$$, and output $m_0$ if $m_1 = 1$.
- *Re-Encryption* $\mathsf{UR_e}$: Input ciphertext $CT = [(\alpha_0, \beta_0), (\alpha_1, \beta_1)]$, compute and output $CT = [(\alpha_0', \beta_0'), (\alpha_1', \beta_1')] = [(\alpha_0\alpha_1^{k_0'}, \beta_0\beta_1^{k_0'}),(\alpha_1^{k_1'}, \beta_1^{k_1'})]$, where $r' = (k_0', k_1') \in Z_q^r$


TODO: describe the attacks (at least the relevant ones -- forge signatures by combining ciphertexts)

http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.79.5764&rep=rep1&type=pdf

# Shuffling vs Re-Randomization

Consider the problem where we have $l$ parties with messages $m_i$ who wish to generate
the list $(m_{\pi(1)}, \dots, m_{\pi(l)})$, where $\pi$ is a permutations over the
integer range $[1,\dots,l]$. Suppose further that they wish to do this without
revealing $\pi$. Something that computes this is basically a mixnet -- it hides the
sender (provider) of each message $m_i$ by shuffling them around and about. What we 
desire is for this mixing procedure to be truly public. Adida et al. [x] show that
any homomorphic encryption scheme can be used to build an inefficient shuffling schemes
by obfuscation. They also show how to construct efficient shuffling schemes based on 
the BGN and Pallier encryption schemes. I'll give an overview of the one based on
Pallier. 

[x] https://eprint.iacr.org/2005/394.pdf


## 


