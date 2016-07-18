---
layout: post
title: Exploring Padding Oracles
---

Padding oracles are old news. Vaudenay introduced them to world back in 2002 as a trivial way
to attack the CBC mode of encryption [1]. Since then, however, they've come back to bite us on numerous
occasions. Perhaps the most famous example is the POODLE attack on TLS [2]. In the original padding
oracle attack on CBC-mode encryption, Vaudenay points to the RC5-CBC-PAD algorithm outlined in
RFC 2040 [3]. All you need to know is (1) that this padding scheme is deterministic and (2)
decryption implementations should ensure that the padding of a decrypted message is correct
according to this scheme. The padding oracle attack relies on the attacker learning if this padding
is valid or not. In the CBC mode, encryption works by XORing $$i$$th block of plaintext with
the $$(i-1)$$st ciphertext block and then encrypting the result. Decryption performs the opposite:
the $$i$$th block is decrypted, XORed against the $$(i-1)$$st ciphertext block, and then, most importantly,
the padding is checked. This is the most crucial step. Since the attacker has control over the ciphertext,
he can manipulate the $$(i-1)$$st block to try and learn some information about the plaintext. 

To give a concrete example, the attacker can learn the last byte of the last block of plaintext
by doing the following. For illustration purposes, let P[i] and C[i] refer to the $$i$$th plaintext
and ciphertext blocks, respectively. Also, let P[i][16] refer to the 16th (last) byte of the P[i]. 
Finally, assume there are $$n$$ blocks of ciphertext.
Now, the attacker modifies C[n-1] by XORing C[n-1][16] with 0x01 and b, where b is the plaintext
byte guess. If this guess is correct, the decryption algorithm will end up computing 
P[n][16] XOR C[n-1][16] XOR 0x01. If P[n][16] = C[n-1][16], then the result of this will be
0x01. If the padding value of 0x01 is correct, then the attacker learns the last byte of the plaintext 
block... after only 256 tries. That means the entire plaintext block can be recovered in 4096
decryption attempts, which is pretty significant. 

I won't go into details about how to generalize this attack since 
that's been done numerous times before (just Google it). Rather, I'd like to show some code that
shows it in action.

# Padding Oracle Code

To recreate this attack, I wrote some code that performs AES-CBC encryption using the PKCS7
padding scheme [4]. This padding algorithm works as follows. Assuming a block length of N bytes
and a message with B bytes, the algorithm either (a) appends B mod N bytes equal to B mod N if
the remainder is non-zero, or (b) N bytes equal to N. For example, if N = 16 and B = 20, then
the algorithm adds 12 bytes to the message, each equal to 12. If N = 16 and B = 16, then the algorithm
adds 16 bytes to the message, each equal to 16. To verify the padding of the message, one simple examples 
the last byte of the message and then checks that this value X is repeated X times at the end
of the messge. The Python code to do this is shown below. 

{% gist 87d0f34af234f36cdbf7f4630b4c09b0 %}

The next step was actually implementing the AES-CBC encryption. The actual PRF used for the block 
cipher *not important* here -- all that's required is that, given an IV, key, and block of plaintext,
a single ciphertext block is computed. The code to do this is below.

{% gist a65f4d816be8dec756fce6178f33b49d %}

As you can see, the code tosses an exception if the padding is incorrect after CBC decryption. 
This is just to help illustrate the simplicity of the attack. 

The last task is to actually write the attack code. What I've done is follow the recipe outlined
at the beginning of this post. Starting from the last block of ciphertext, and from the last byte of
that block, I search for the right value of plaintext iteratively until found. Then I move onto
the previous byte in the same (or previous) block. I'd rather let the code speak for itself -- 
as it should -- than explain the details here. So, if you're interested, take a peek at the
result below. 

{% gist b47ba7b6b91fe01092cc2567ed367475 %}

You can download and run the code [from here](https://github.com/chris-wood/fake-cbc-padding-oracle). 

# Summing Up

Padding oracles are no joke. Why people continue to use CBC is beyond me. However, even if people
insist on this outdated mode of operation, there are ways around this particular attack. One can,
for example, add an additional MAC to the ciphertext to prevent an adversary from tweaking specific
blocks of plaintext. This can be done, for example, to protect CBC-encrypted HTTP cookies. However, 
if this is the route you go down, you'd be better off just using a standard authenticated
enryption and decryption (AEAD) algorithm instead. It's time to move past CBC. 

# References

- [1] Vaudenay, Serge. "Security Flaws Induced by CBC Padding—Applications to SSL, IPSEC, WTLS..." International Conference on the Theory and Applications of Cryptographic Techniques. Springer Berlin Heidelberg, 2002.
- [2] Möller, Bodo, Thai Duong, and Krzysztof Kotowicz. "This POODLE bites: exploiting the SSL 3.0 fallback." PDF online (2014).
- [3] Baldwin, R., and R. Rivest. "RFC 2040: The RC5, RC5-CBC, RC5-CBC-Pad, and RC5-CTS Algorithms. October 30, 1996."
- [4] Kaliski, Burt. "Pkcs# 7: Cryptographic message syntax version 1.5." (1998).

