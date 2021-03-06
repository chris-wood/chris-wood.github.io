---
layout: post
title: AES Key Wrapping
---

I previously wrote about the [difference between key wrapping and encapsulation](http://chris-wood.github.io/2016/06/10/Wrap-And-Encap.html). 
To learn more I set out to implement some of these algorithms. And to keep things simple, 
I started with the AES key wrapping algorithm detailed in [1]. This procedure can be used
to encrypt any number of 64-bit blocks of data. So if you want to wrap a 256-bit key DEK
with another key KEK, you break it up into four blocks and then pass it to the wrapping function.
The output of this procedure would be a vector of five 64-bit blocks. This basically wraps the 
DEK ciphertext and authenticator into a single vector. 

The wrapping algorithm itself is trivial to implement. Given a key KEK and $$n$$ input blocks, it works as follows:

- Initialize some *encryption* state $$A$$ with a fixed IV (0xA6A6A6A6A6A6A6A6) and copy the 
$$n$$ input blocks into the first row of a state *round* matrix of $$n \times 6n$$ elements. The
round matrix will have $$6n$$ rows and $$n$$ columns.
- Enter a loop with $$6n$$ iterations and perform the following:
    - For each iteration $$t$$, concatenate the current encryption state with the first element
    in the $$(t-1)$$th row of the round matrix and store the result in a temporary variable $$B$$.
    - Encrypt $$B$$ with KEK, XOR the 64 most-significant bits of the result with $$t$$ 
    (treated as an unsigned 64-bit integer), and save the result in $$A$$. This will be the $$t$$th 
    value of $$A$$.
    - Enter a loop with $$(n-1)$$ iterations and perform the following:
        - For each iteration $$i$$, copy the $$(i+1)$$st column from the $$(t-1)$$th row into the $$i$$th column in the $$t$$th row. 
        This shifts a column from the previous round to the left on slot.
    - Concatenate the $$(t-1)$$st value of $$A$$ and the first element of the $$(t-1)$$st row and encrypt the result. Save the
    result in a temporary variable $$B$$.
    - Set the $$n$$th column of the $$t$$th row to the 64 least-significant bits of $$B$$.
- Output a vector containing the the last value of $$A$$ and the last row of the round matrix. 
    
Basically, the algorithm works by iteratively folding the input blocks (as initial round values) into the current encryption 
state, encrypting the result, and the cascading that output to the next round values and slightly updating the 
encryption state again (with an XOR of the iteration count). Naturally, the unwrap procedure performs this procedure 
in reverse. The goal of which is to recover the original fixed IV. The probability of that failing
is roughly $$2^{-64}$$. This feels like an uncomfortably small margin given today's 
algorithms. (Especially with the terrible prospect of attacks that come with Grover's algorithm on symmetric key algorithms [2].)
 
To illustrate the simplicity of this algorithm, I implemented it in Python. The code is below. It includes all
test vectors from the official RFC [3]. You can run it yourself to verify its correctness. 

{% gist bbb8828540517e19189a48bf81c0a9b2 %}

And that's it. I'd like to implement the recent AES-GCM-SIV algorithm [4] next, which is basically a form of deterministic 
authenticated encryption. Maybe I can do that on the plane to Boston next week. 

# References
 
- [1] [http://csrc.nist.gov/groups/ST/toolkit/documents/kms/key-wrap.pdf](http://csrc.nist.gov/groups/ST/toolkit/documents/kms/key-wrap.pdf)
- [2] [https://people.cs.umass.edu/~strubell/doc/quantum_tutorial.pdf](https://people.cs.umass.edu/~strubell/doc/quantum_tutorial.pdf)
- [3] [https://tools.ietf.org/html/rfc3394](https://tools.ietf.org/html/rfc3394)
- [4] [https://eprint.iacr.org/2015/102.pdf](https://eprint.iacr.org/2015/102.pdf)
