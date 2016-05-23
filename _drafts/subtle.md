---
layout: post
title: Subleties in Crypto Code
---

TLDR: Constant-time operations are important to secure implementations
of cryptographic algorithms and implementations. In this post, I review 
the Go implementations for some simple constant-time operations.

# Side Channels

Writing cryptographic code is difficult for a variety of reasons. Not
only must the code be correct with respect to the relevant protocol
or algorithm, but the implementation must be free of vulnerabilities
that render the code ineffective. Several well-known vulnerabilities
exist; Buffer overflows are arguably the most infamous types that exploit
actual bugs in the code. Another often overlooked class of vulnerabilities
stem from side channels which emerge in the code (or physical implementation). 
Interestingly, it's difficult to classify the existence of side channels
as bugs. According to Wikipedia, a fairly standard definition for a bug is
"an error, flaw, failure or fault in a computer program or system that causes 
it to produce an incorrect or unexpected result, or to behave in unintended 
ways" [1]. Therefore, technically, a correct implementation of a cryptographic
algorithm such as AES with side channels is not a bug. This may be why attacks
based on side channels are often overlooked in practice. 

Several types of side channels exist. The more prominent ones include power and
timing channels. Power side channels are infamous for being exploited in
differential power analysis [2], which involves using power consumption measurements 
about hardware implementations of cryptographic algorithms to discover secret
keying material. Timing side channels are have been used to discover secret information
used in algorithms such as Diffie-Hellman, RSA, AES, and others [3,4. The core idea of
timing side channel attacks is to exploit differences in execution times for certain
algorithms (or parts thereof) to recover secret information, such as the private
exponent used in RSA or the secret key used in AES. I'll postpone the 
details of these attacks for another post. In the remainder of this post, I'll 
explore one type of solution to this class of attacks. 

# Constant Time Operations

The cure to timing side channels is simple: constant-time implementations that 
always run in the same amount of time. This means that the execution time *does
not* depend on the data. At the core of these implementations are byte-level
operations. Specifically, comparing bytes and unsigned integers for equality. 
This is a frequent step in many cryptographic algorithms and, as such, is 
imperative that it runs in constant time.

The Go cryptography library provides a number of constant-time functions
for byte-level operations. The list below enumerates the entire set [5]:

- func ConstantTimeByteEq(x, y uint8) int
- func ConstantTimeCompare(x, y []byte) int
- func ConstantTimeCopy(v int, x, y []byte)
- func ConstantTimeEq(x, y int32) int
- func ConstantTimeLessOrEq(x, y int) int
- func ConstantTimeSelect(v, x, y int) int

The implementation of these functions is pretty clever. Let's take a look 
at the first.

~~~
func ConstantTimeByteEq(x, y uint8) int {
    z := ^(x ^ y)
    z &= z >> 4
    z &= z >> 2
    z &= z >> 1
  
    return int(z)
}
~~~

Consider how this code runs when the two inputs are equal.
First, z will be initialized to 0xFF since (x ^ y) = 0 and the 
negation of 0 is 0xFF (2^8 - 1). The code then folds each bit
of z into itself using the AND operator. If any bit is zero,
the accumulated result transforms to zero. Otherwise, it's 
1, as desired. The integer comparison function is nearly identical.

Let's look at the comparison function next.

~~~
func ConstantTimeCompare(x, y []byte) int {
    if len(x) != len(y) {
        return 0
    }
        
    var v byte
        
    for i := 0; i < len(x); i++ {
        v |= x[i] ^ y[i]
    }
    return ConstantTimeByteEq(v, 0)
}
~~~

The code words as follows. If the length of the two input
slices is not equal (which can be determined in constant time),
then the function returns 0. This early escape does not impact the
constant time behavior of the code since the length can be determined
without examining the contents of either slice. If the slices are
of equal length, then each pair of elements are XORd and folded
into an accumulator value. If every element in the is equal, then
every XORd value will be 0 and thus the final accumulated result
will be 0. The final step is to compare the accumulator value to 0
using the previous byte equals function. It's therefore easy to
reason about the runtime of the code. 

To conclude, let's look at the less-than-or-equal function.

~~~
func ConstantTimeLessOrEq(x, y int) int {
    x32 := int32(x)
    y32 := int32(y)
    return int(((x32 - y32 - 1) >> 31) & 1)
}
~~~

The code returns 1 if x <= y, and 0 otherwise. There are two cases for
x and y to consider. If x <= y and both integers are positive, then the 
difference (x - y - 1) will be negative (the left-most bit will be 1). 
The sign bit is then masked with 0x1 and returned. In the event that
x and y are negative but x <= y, then the sign bit will still be negative.
(The difference between a large negative value and a smaller negative
value is still a negative value.) If x is negative, y is positive, and x <= y,
then (x - y - 1) will again be negative. (You can see a pattern emerging.
The -1 difference is needed to handle the edge case where x and y are 0.)
In other cases, when y > x, regardless of the parity of x or y, the 
difference (x - y - 1) will always be positive. The result of comparing the
sign bit after masking it with 1 will then always be 0. 

There are many other useful pieces of information to learn from the Go code. 
I've only discussed a small subset here which is important for proper 
cryptographic implementations. If you ever find yourself with time to kill
and a hankering to read good code, check out the Go code. It's well worth
the read. 

# References

[1] https://en.wikipedia.org/wiki/Software_bug
[2] http://web.mit.edu/6.857/OldStuff/Fall03/ref/kocher-DPATechInfo.pdf
[3] Kocher, Paul C. "Timing attacks on implementations of Diffie-Hellman, RSA, DSS, and other systems." Advances in Cryptology—CRYPTO’96. Springer Berlin Heidelberg, 1996.
[4] Bernstein, Daniel J. "Cache-timing attacks on AES." 2005.
[5] https://golang.org/pkg/crypto/subtle/
