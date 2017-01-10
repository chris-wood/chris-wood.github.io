---
layout: post
title: Constant-Time Scrubbing
---

Constant-time code and crypto are two terms often used together. And
rightfully so. There are too many cases where implementations of core
crypto functions that are not constant time wound up leaking secret information,
including:

- RSA private key recovery [1] http://users.belgacom.net/dhem/papers/CG1998_1.pdf
- AES key recovery [2] https://cr.yp.to/antiforgery/cachetiming-20050414.pdf [3] https://cseweb.ucsd.edu/~kmowery/papers/aes-cache-timing.pdf
- Plaintext recovery with AES-CBC in TLS 1.2 [4] https://eprint.iacr.org/2015/1129.pdf 

The lesson is simple: the runtime of secret-dependent code must be independent
of the actual value of the secret. 

# A Horrible Use Case

The Web is full of references to example constant-time implementations
for a variety of standard operations that are useful when implementing 
various cryptographic algorithms. For example, the [Cryptography Coding Standard Rules](https://cryptocoding.net/index.php/Coding_rules)
includes C code for constant-time implementations of standard operations,
including element (byte, nibble, word) comparison, string (or byte array) 
comparison, modular exponentiation, etc. One operation that I could not
find was what I'm calling selective scrubbing. This procedure basically
works as follows. Assume we have the following code to perform AES decryption:

{% gist eba26aa3762d7f2914048253446e2ed3 %}

If the decryption fails, e.g., because the ciphertext tag is invalid, then
the function should return an error and no plaintext. However, imagine, for
the sake of argument, that the contents of the plaintext (`decrypted`) buffer
are not cleared before returning. In a purely software implementation of this routine
we would check the MAC before attempting to decrypt the ciphertext. 
(That's what many popular implementations do at least -- see 
the [Go](https://golang.org/src/crypto/cipher/gcm.go) implementation as an example.) However,
in accelerated variants, the decryption and MAC authentication steps may
be done in parallel, therefore leading to possible writes to the plaintext
buffer before the tag's validity is checked.

If an adversary is able to conduct chosen ciphertext attacks by injecting forged ciphertext and 
tag pairs for decryption, then it may be the case that some legitimate plaintext values
are contained in the buffer. Thus, it would be desirable to wipe the plaintext
buffer in the event of a decryption failure so as to not leak any information.
This wiping process, or scrubbing, adds extra time to the decryption routine. 
If scrubbing is done, then the ciphertext was invalid. Otherwise, the ciphertext
was valid. In a paranoid world, it may be possible to exploit this information and
turn the decryption function into an opaque "ciphertext validity oracle."

Let's imagine that this is a problem. I don't have a concrete attack with running code
to back this up. Nevertheless, I will press on under the assumption that this is a
timing oracle we should strive to prevent. 

The fix is clear: the plaintext buffer should be processed post-decryption regardless
of whether or not the tag is valid. If the tag is valid, the plaintext should
remain unchanged. Otherwise, the plaintext should be zeroed. To prevent the timing
oracle, we need to do this in constant time. 

(Before proceeding, let me be clear here and state that I'm not seriously advocating a
second pass over the plaintext to plug this timing side channel. That would be 
disasterous for performance. I needed an example to play around with constant time 
code, and this is what came up at the time. Alas, here we are.)

There are a variety of ways we might actually accomplish this task. 
The first way is to use a constant-time select function (see e.g. `ConstantTimeSelect`
from [Go](https://golang.org/pkg/crypto/subtle/)). For each slot in the output buffer, we can set its 
value to the already-stored value or 0 depending on the output of the tag comparison. 
This is shown below.

{% gist 7f959a93e7127692eb5c02fda14de4f7 %}

If we did not have this primitive available, however, not all hope is lost. Imagine we had a byte-mask
that was 0xFF whenever the validity flag was true and 0x00 otherwise. We could then AND each byte
in the plaintext with this mask to get the same result. But how do we compute this mask? 
Treat the validity flag as an unsigned integer where true is 0 and false is any other value.
This means that if there is any non-zero bit in the flag, then the mask should be 0x00. Otherwise,
it should be 0xFF. To generate this mask we can do the following: for each bit in the flag, 
OR this bit with every other bit on the left and on the right. In effect, if there is even a single
non-zero bit, then the result will be 0xFF since that bit will be OR'd in every spot. We then just
negate the value of the mask to get the desired result. If the flag is zero, the OR'd result will be 0x00,
and by negating this we get 0xFF, the intended result. The code for this is below.

{% gist 7e05755550a1ae4476e2ecb56fe974ec %}

# In Practice

It is clear that the driving example in the previous section is quite farfetched. In practice,
this sort of timing channel is not really a problem. So this may have been just an exercise in
futility. (It was, however, a nice excuse to familiarize myself with some of the tools available
to verify constant time implementations. I hope to explore them in a follow-up post.)
As previously stated, many software implementations will verify the MAC before attempting to do 
any plaintext decryption. If this is a constant-time operation, then there is no timing channel
or plaintext leakage. In hardware, such as in the AES-NI design, the plaintext buffers are 
zeroed out if the AAD step fails. 

XXX: test AES-NI zeroing out if AAD fails
https://github.com/kmcallister/aesni-examples
