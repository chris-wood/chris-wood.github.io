---
layout: post
title: Universal Re-Encryption Signatures
---

# Universal Re-Encryption Signatures

Several extensions on the URE scheme proposed above have been designed to build mixnets.
I'll focus on the work of Klonowski et. al [10 from IURE] which proposes a signature
scheme for URE. You can see papers built on their results by checking Google Scholar.
They modify the URE scheme above to use an RSA signature instead of an ElGamal
signature. And, just as before, the security of the scheme relies on the face that the
signature remains valid even as the ciphertext is updated by artbirary nodes.

The formal description of this scheme, called RSA-URE (URE with RSA signatures), is
as follows. It assumes the existence and knowledge of an RSA modulus $N = pq$ and
generator $g$ for $\mathbf{Z}_{N}^*$. The

- *Key Generation* $\mathsf{UKG}$:

asd

- *Encryption* $\mathsf{UE}$:

asd

- *Decryption* $\mathsf{UD}$:

asd

- *Re-Encryption* $\mathsf{UR_e}$:

asd


TODO: describe the attacks (at least the relevant ones -- forge signatures by combining ciphertexts)

http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.79.5764&rep=rep1&type=pdf


"Uni- versal re-encryption of signatures and controlling anonymous information flow"
TODO: describe the extension in https://gnunet.org/sites/default/files/10.1.1.108.4976.pdf
