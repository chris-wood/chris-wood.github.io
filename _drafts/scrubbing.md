---
layout: post
title: Constant-Time Scrubbing
---

Constant-time and crypto code are two terms often used together. And
rightfully so. There are too many cases where implementations of core
crypto functions that are not constnat time have leaked secret information,
including:

- RSA private key recovery [1] // google...
- AES key recovery [2] // djb
- Plaintext recovery with AES-CBC in TLS 1.x [3] // lucky13

The lesson is simple: all code that depends on a secret must run in 
a constant amount of time that is independent of the secret. Or,
put another way, the value of the secret should not change the running
time of the code. 

The Web is full of references to example constant-time implementations
for a variety of standard operations that are useful when implementing 
various cryptographic algorithms. Some of these common operations
are element (byte, nibble, word) comparison, string (or byte array) 
comparison, modular exponentiation, etc. One operation that I could not
find was what I'm calling selective scrubbing. This procedure basically
works as follows. Assume we have the following code:

%% snippet of code that performs decryption and then checks value

Now assume that for some reason the decryption succeeds but the MAC 
verification fails. In a purely software implementation of this routine
we would check the MAC before attempting to decrypt the ciphertext. 
(That's what many popular implementations do at least [golang].) However,
in accelerated variants, the decryption and MAC authentication steps may
be done in parallel, therefore leading to possible writes to the plaintext
buffer. 

If we do nothing else, the result of this function
will indicate an error but the output buffer may actually contain the
decrypted plaintext. This clearly violates the standard AEAD API contract, 
wherein a failed authentication should yield the empty string. So what's the fix?
The desired result is clear: if the tags don't line up, then the output
buffer should be zeroed. But we should do this in constant time$$^*$$. 
Failing to do so leaks information that the message was invalid. Maybe one 
could exploit that to tweak AAD inputs to get forge messages. Who knows. 
I don't have a concrete attack with running code. Nevertheless, I will
press on under the assumption that it should be constant time out of professional
paranoia.

There are a variety of ways we might actually implement this in constant
time. Also, regardless of the approach, it's clear that doing so will cause us to
make another full pass over the plaintext buffer. (Otherwise, it would not
be constant time!) That's not so great, but I'm willing to accept that hit for now.
The first way is to use a constant-time select function (see e.g. `ConstantTimeSelect`
from [golang]). For each slot in the output buffer, we can set its value to
the already-stored value or 0 depending on the output of the tag comparison. 

XXX: describe the other way from the email

$$^*$$ I'll discuss what's actually done in practice for, say, AES-GCM, below.

XXX: discuss go-lang implementation and test AES-NI zeroing out if AAD fails
https://golang.org/src/crypto/cipher/gcm.go
https://github.com/kmcallister/aesni-examples
