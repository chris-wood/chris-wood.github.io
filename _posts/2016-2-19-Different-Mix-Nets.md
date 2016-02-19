---
layout: post
title: More Mix than Net
---

I recently stumbled across a paper entitled, "Universal Re-Encryption for mix nets,"
by P. Golle et al. [1] that I felt is worth exploring. Universal re-encryption (URE), or, perhaps better called
*univeral re-randomization*, is a primitive that allows some ciphertext to be re-encrypted
without knowledge of the corresponding public key. The applicability of this technique
to Chaum-like mix nets [2] is intriguing: untrusted mix nodes can, for
example, re-encrypt messages from senders to receivers to further
hide the relation between input and output messages. The figure below illustrates this process. $R$ is a receiver of some message $Y$ sent by $S$. $R$ initiates this transfer by issuing a query $X$. The mix node ensures that the forwarded query does not match the input query and that forwarded result does not match the input result. In other words, it does some operation so that $X' \not= X$ and $Y' \not= Y$.

![A mix net with one sender and one receiver showing the re-encryption.](/images/reencr.png){: .center-image }

Generally, a mix node receives messages from many nodes and then shuffles
and re-encrypts them as needed before transmitting them to their intended recipient (in this case, $S$). The shuffle
procedure step serves to hide the mapping from the mix node's outputs to its inputs. This helps keep receivers anonymous.

![A mix net with many receivers.](/images/mix net.png){: .center-image }

In the original mix net proposal, many mix nodes existed to transfer messages between senders and receivers. Each mix node in a mix net had a pubic key for message encryption. Senders wrapped messages in layers of concentric encryption to form an onion. Each layer included at least two pieces of information: (1) some encrypted payload or plaintext message and (2) information about where to forward the message after removing a layer of encryption. The sender creates the onion based on the path of mix nodes through which the message traverses. Upon receipt of a message, a mix node removes a layer of encryption (via decryption with its private key) and forwards the inner payload to the destination address at some future time. A mix node may batch messages to prevent timing correlation attacks.

After a recipient processes a message it typically generates a response. The recipient encrypts this response with a one-time ephemeral key provided by the sender. The sender must then transfer the encrypted response to the sender. To remain anonymous, it is important for the recipient to not know the address of the sender. Thus, to route the message, senders provide anonymous return addresses for recipients. These anonymous addresses are also onions containing a path of addresses and one-time keys. When a mix node receives a response it removes one layer of decryption to uncover the next-hop address and a key, re-encrypts the response payload with this key, and then forwards the result to the next hop.  This process continues until the sender receives the response, at which point it can remove each layer of encryption on the message to uncover the response.

The mechanics for creating messages and responses sent in a mix net are
not important. The takeaway is that it relies on layers of concentric
encryption and well-defined paths through which messages propagate between
senders and receivers. URE is interesting because it enables a completely
different type of mix net protocol that is not based on these properties.
It works as follows. Senders publish messages encrypted for recipients to
a public bulletin board. Messages are encrypted under the same "group information",
e.g., the same ElGamal group. Periodically, a mix node will retrieve all messages
from the bulletin board, re-encrypt each of them, and then place them back on the
board in random order. To the passive eavesdropper, this has the effect of both
shuffling the messages and their contents. Recipients check for messages by
attempting to decrypt each message and only keeping the ones that succeed.
Decryption will fail on messages that are not encrypted for them, i.e., that were
not encrypted with their public key.

The interesting aspect of this protocol is that mix nodes only need information about
the encryption group and not the public key of each recipient. This allows the
nodes to blindly re-encrypt each message.

In this post I will describe the cryptographic details of the URE scheme
presented in [1]. I will also present a simple Python implementation that you can
run to see it in action. I will also review the basics of ElGamal encryption as a
primer for the main material.

## ElGamal Encryption

Functionally, ElGamal encryption works by masking some plaintext element with a
random value, or ephemeral key, and then encrypting the value so that the recipient
can decrypt it and unmask the ciphertext. To understand the details, we need some notation. Let $G$ be some group of order $q$
with generator $g$. The public and private key pair for a user is $h = g^x$ and
$x$, respectively, where $x \in \{1,\dots,q\}$. To encrypt a message $m$, which is
assumed to be an element of $G$ (or there exists a mapping to one element), a sender
generates a random element $y \in \{1,\dots,q\}$, computes $s = h^y$ and
then $c_2 = ms$, and outputs $(c_1, c_2) = (g^y, mh^y)$. Decryption
then works by computing $m = c_2 / c_1^x =  mh^y / g^{xy}$. This equality
holds since $h^y = g^{xy}$. Simple enough.

## URE Description

A universal re-randomizable encryption (URE) scheme is a tuple of algorithms $(\mathsf{UKG}, \mathsf{UE}, \mathsf{UD}, \mathsf{UR_e})$
which are loosely described below.

- *Key Generation* $\mathsf{UKG}$: Generate the public and private keys for a user.
- *Encryption* $\mathsf{UE}$: Encrypt a message under a public key.
- *Decryption* $\mathsf{UD}$: Decrypt a message with a private key.
- *Re-Encryption* $\mathsf{UR_e}$: Re-encrypt or re-randomize some ciphertext.

The authors of [1] used the term "universal" to mean that the re-randomization does not
need knowledge of the public key under which the some ciphertext was encryption (only
the system parameters, e.g., the ElGamal group generator and prime). This enables the
re-randomization process to be *transparent* and thus something that any intermediate entity can do any number of times before the intended recipient (i.e., the one holding the private key) decrypts the result.

## URE Details

The URE scheme based on ElGamal from [1] is presented below.

- *Key Generation* $\mathsf{UKG}$: Output $(PK, SK) = (y = g^x, x)$ for $x \in_U Z_q$

- *Encryption* $\mathsf{UE}$: On input message $m$ and public key $y$, compute and output the ciphertext

$CT = [(\alpha_0, \beta_0), (\alpha_1, \beta_1)] = [(my^{k_0}, g^{k_0}), (y^{k_1}, g^{k_1})]$,

where $r = (k_0, k_1) \in Z_q^2$.

- *Decryption* $\mathsf{UD}$: On input ciphertext $CT = [(\alpha_0, \beta_0), (\alpha_1, \beta_1)]$, compute

$m_0 = \alpha_0/\beta_0^x$ and $m_1 = \alpha_1/\beta_1^x$,

and output $m_0$ if $m_1 = 1$.

- *Re-Encryption* $\mathsf{UR_e}$: On input ciphertext $CT = [(\alpha_0, \beta_0), (\alpha_1, \beta_1)]$, compute and output

$CT' = [(\alpha_0', \beta_0'), (\alpha_1', \beta_1')] = [(\alpha_0\alpha_1^{k_0'}, \beta_0\beta_1^{k_0'}),(\alpha_1^{k_1'}, \beta_1^{k_1'})]$,

where $r' = (k_0', k_1') \in Z_q^r$

Altogether, this scheme is quite straightforward. The only input is a generator $g$
and prime $q$ forming a group over $Z_q$. Encryption and decryption are similar to the original ElGamal scheme presented in [3]. The only difference is that the ciphertext
carries both an encrypted message and a type of signature. When the ciphertext
is re-randomized in the $\mathsf{UR_e}$ scheme, the encrypted message is re-blinded
and the signature is updated accordingly. Decryption only returns the original message
if the signature is valid. To show this, let's compute $m_0$.

$$
\begin{align}
m_0 & = \alpha_0/\beta_0^x \\
 & = (\alpha_0\alpha_1^{k_0'}) / (\beta_0\beta_1^{k_0'})^x \\
 & = (my^{k_0}y^{k_1k_0'}) / (g^{k_0}g^{k_1k_0'})^x \\
 & = (my^{k_0}y^{k_1k_0'}) / (g^{k_0x + k_1k_0'x}) \\
 & = (mg^{xk_0}g^{xk_1k_0'}) / (g^{k_0x + k_1k_0'x}) \\
 & = (mg^{xk_0 + xk_1k_0'}) / (g^{k_0x + k_1k_0'x}) \\
 & = m
\end{align}
$$

Now, to verify that $m_1 = 1$, let's compute that too.

$$
\begin{align}
m_1 & = \alpha_1/\beta_1^x \\

 & = \alpha_1^{k_1'} / (\beta_1^{k_1'})^x \\
 & = y^{k_1k_1'} / (g^{k_1k_1'})^x \\
 & = y^{k_1k_1'} / (g^{k_1k_1'x}) \\
 & = g^{k_1k_1'x} / (g^{k_1k_1'x}) \\
 & = 1
\end{align}
$$

If a message or the signature is tampered with then the verification, and decryption will fail. The security of this scheme (as presented) boils down to the Decisional Diffie-Hellman problem [4] over the group $Z_q$, much in the same way that the semantic security of the original ElGamal encryption scheme does. (I hope to talk about some of these hardness problems in a future post.)

## Code

An implementation of this procedure is shown below.

{% gist 8847a967d58ff4e8d344 %}

Observe that the input to the re-encryption algorithm is *only* the ciphertext CT and
the group parameter $q$. Different ElGamal ciphertexts are free to re-use this parameter since
it only determines the size of the ElGamal elements and does not have any affect on the
choice of elements within (all samples are done uniformly at random from $Z_q$.

# References

- [1] Golle, Philippe, et al. "Universal re-encryption for mix nets." Topics in Cryptology–CT-RSA 2004. Springer Berlin Heidelberg, 2004. 163-178.
- [2] Chaum, David L. "Untraceable electronic mail, return addresses, and digital pseudonyms." Communications of the ACM 24.2 (1981): 84-90.
- [3] ElGamal, Taher. "A public key cryptosystem and a signature scheme based on discrete logarithms." Advances in cryptology. Springer Berlin Heidelberg, 1984.
- [4] Boneh, Dan. "The decision diffie-hellman problem." Algorithmic number theory. Springer Berlin Heidelberg, 1998. 48-63.

<!--
```
$ python raw.py
```
```
Plaintext             d7081f2570ccf78075a18fd34605e27d2e14ace1a8ccaff60dc85d1ffcbeea21
```
```
Decrypted message #1: d7081f2570ccf78075a18fd34605e27d2e14ace1a8ccaff60dc85d1ffcbeea21
```
```
Decrypted message #2: d7081f2570ccf78075a18fd34605e27d2e14ace1a8ccaff60dc85d1ffcbeea21
```
-->
