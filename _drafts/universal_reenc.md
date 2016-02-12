---
layout: post
title: Universal Re-Encryption (or Re-Randomization)
---

I recently stumbled across a paper entitled, "Universal Re-Encryption for Mixnets,"
by P. Golle et al. [1]. Universal re-encryption (or, perhaps better called
*univeral re-randomization*) is a primitive that allows ciphertext to be re-encrypted
without knowledge of the corresponding public key. The applicability of this technique
to mixnets is quite obvious: untrusted mix nodes can, for example, re-encrypt messages
from senders to receivers in order to further hide the relation between input and output
messages. This process is shown in the figure below.


- *Key Generation* $$\mathsf{UKG}$$: Output $$(PK, SK) = (y = g^x, x)$$ for $$x \in_U Z_q$$
- *Encryption* $$\mathsf{UE}$$: Input message $$m$$ and public key $$y$$, and output the ciphertext $$CT = [(\alpha_0, \beta_0), (\alpha_1, \beta_1)] = [(my^{k_0}, g^{k_0}), y^{k_1}, g^{k_1})]$, where $r = (k_0, k_1) \in Z_q^2$$.
- *Decryption* $$\mathsf{UD}$$: Input ciphertext $$CT = [(\alpha_0, \beta_0), (\alpha_1, \beta_1)]$$, compute and output $$m_0 = \alpha_0/\beta_0^x$$ if $$m_1 = \alpha_1/\beta_1^x$$. 
- *Re-Encryption* $\mathsf{UR_e}$: Input ciphertext $CT = [(\alpha_0, \beta_0), (\alpha_1, \beta_1)]$, compute and output $CT = [(\alpha_0', \beta_0'), (\alpha_1', \beta_1')] = [(\alpha_0\alpha_1^{k_0'}, \beta_0\beta_1^{k_0'}),(\alpha_1^{k_1'}, \beta_1^{k_1'})]$, where $r' = (k_0', k_1') \in Z_q^r$

Observe that the input to the re-encryption algorithm is \emph{only} the ciphertext CT and 
the group parameter $q$. Different ElGamal ciphertexts are free to re-use this parameter since
it only determines the size of the ElGamal elements and does not have any affect on the 
choice of elements within (all samples are done uniformly at random from $Z_q$). 

TODO: mixnet with re-encryption example figure

TODO: describe the protocol and then show the code, and then the output
https://crypto.stanford.edu/~pgolle/papers/univrenc.pdf

{% gist 8847a967d58ff4e8d344 %}

TODO: describe the extension in https://gnunet.org/sites/default/files/10.1.1.108.4976.pdf
RSA-URE signatures

TODO: describe the attacks (at least the relevant ones -- forge signatures by combining ciphertexts)

http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.79.5764&rep=rep1&type=pdf

TODO: describe pallier and give code
