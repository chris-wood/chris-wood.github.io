---
layout: post
title: ChaCha20 in Haskell
---

As ChaCha20 and Poly1305 make their way through the IETF standardization process, I thought
it would be a good idea to get some experience implementing (at least one of) these algorithms. 
I also wanted to play around with implementing some cryptographic algorithms in Haskell. 
To that end, I implemented ChaCha20 in Haskell. This was fairly easy to do following the 
RFC [1]. The main entry point is the encryption function `chaCha20Encrypt`. (In retrospect, 
this should be renamed to `chaCha20Mask` or something similar since it's the same function
used for both encryption and decryption.) It has the following type signature:

```
chaCha20Encrypt :: [Word32] -> Word32 -> [Word32] -> [Word8] -> [Word8]
```

That is, it takes the key, counter, nonce, and plaintext and generates the corresponding
ciphertext. I use Word32 types fo store each element of the key since that was the underlying
data type used in the quarter round functions. It would be easy to make all parameters Word8s. 

I don't have much to say beyond this. If you're interested, the code is below and [here](https://github.com/chris-wood/chacha20). I'll probably implement the Poly1305 algorithm next. I'd also like to benchmark the code
and use HUnit for the RFC test vectors. (Right now I have functions that simply execute
the function-under-test and print the result to the console. This is clearly not so great.)

Enjoy. 

{% gist 77d5f586908a6d87044f540fd8d4e43c %}

# References 

- [1] [https://tools.ietf.org/html/rfc7539](https://tools.ietf.org/html/rfc7539)


