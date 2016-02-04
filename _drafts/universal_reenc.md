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

TODO: mixnet with re-encryption example figure

TODO: describe the protocol and then show the code, and then the output
https://crypto.stanford.edu/~pgolle/papers/univrenc.pdf

{% gist 8847a967d58ff4e8d344 %}

TODO: describe the extension in https://gnunet.org/sites/default/files/10.1.1.108.4976.pdf
RSA-URE signatures

TODO: describe the attacks (at least the relevant ones -- forge signatures by combining ciphertexts)

http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.79.5764&rep=rep1&type=pdf

TODO: describe pallier and give code
