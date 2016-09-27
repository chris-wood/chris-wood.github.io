---
layout: post
title: Unpredictable Sequence Numbers for TCP
---

TCP sequence prediction attacks are (were?) a thing of the past. Any attacker with knowledge
of the correct sequence number and the 4 tuple of a TCP packet could inject data into an ongoing
TCP stream. The TCP state machine accepts any packet whose sequence number is in-window and. Verification
of the data is deferred to an upper layer in the stack, e.g., by TLS (if used). But what if this verification
is not done? I wager that pretty bad things can happen. 

Fortunately, guessing sequence numbers isn't entirely trivial with the approach formalized in RFC 6528 [1]. 
In particular, this RFC states that the intiial sequence number (ISN) should be set to the summation of
a 4us timer and a keyed PRF computed over the 4 tuple. The key is secret to the entity generating the 
ISN. This, coupled with the fast pace at which the timer advances, makes guessing ISN a highly non-trivial task. 

The current [Linux implementation works](http://lxr.free-electrons.com/source/net/core/secure_seq.c#L89) as follows. The PRF secret is advanced once an epoch. (I'm not sure
what the granularity is, but let's just say it's fast.) The ISN is then generated as the MD5 *transform*
of a secret word appended to the 4 tuple. This is shown below.

{% gist d0a4a61c2679ed8712547ae7d248f18c %}

This makes me uncomfortable. MD5 is an *awful* PRF. Collisions are trivial
to generate but pre-images are still hard to find (as far as I know). But is that reason alone to use it?
I think not. If nothing else, a good fix for the kernel would be to change this from MD5 to something more
reasonable, such as SipHash [2]. 

In theory, by the PRF properties of SipHash, the predictability problem should vanish. But I'm not convinced.
The PRF is used to only compute the *initial sequence number,*  not all subsequent sequence numbers. This means
that if an adversary were to try and guess the sequence number, their probability of succeeding at every
hop is *not* $$2^{-64}$$. It will be much larger, especially as the length of data transferred over the connection
increases. I find this not ideal. I would prefer a protocol that wasn't vulnerable to such attacks. And this
is where tcpcrypt [3] comes in. tcpcrypt is an extension to the TCP handshake that tries to add opportunistic encryption. It does so by signalling
the willingness to engage in a post-ACK key exchange. If both parties support the protocol and agree, a key exchange
is completed and a forward-secure session key is derived. This is then used to encrypt all packets between the client
and server. 

Why bother discussing tcpcrypt? Well, it bakes in the possibility to derive a secret known only to the sender 
and receiver. We could use this secret to create "virtual sequence numbers" that are (a) only sensible to the 
endpoints and (b) not easily guessable since they would not increase linearly. Let me try to elaborate a little bit. 
Suppose both the sender and receiver agreed on some value $$x$$. Now, let $$F_x(i)$$ be a keyed PRF (e.g., a
MAC) that, when invoked with an integer $$i$$ returns the PRF computed over that value. The sequence number
$$i$$ could then be advanced locally as per usual, but the "virtual numbers", i.e., the values $$F_x(i)$$,
would be transmitted in packets. Both side would be responsible for pre-computing the sequence numbers
for their current window. This would allow each side to map $$F_x(i)$$ to $$i$$. Beyond this simple abstraction,
everything else in the protocol remains the same. 

To play around with this idea, I wrote some code to emulate a TCP sender and receiver using these PRF-based
windows. The full program is below. You can run it and play around with the idea. I'd like to experiment with
it in the tcpcrypt implementation, time pending. Computationally speaking, I think it would have very little
impact on the runtime of the protocol. Computing these PRF values in bulk is a relatively infrequent operation. 
The other overhead added by this technique would be the reverse-mapping table from the PRF values to the underlying
sequence numbers. This state is linear in the size of the TCP window. Typically the window is framed in terms of
bytes (since the sequence number is a byte offset, not packet counter). In my approach, this sequence number would
have to be a packet counter. So, on top of the computation and memory overhead, this is another drawback.

{% gist a3301ad50909fa2a8455d69666f8af87 %}

# References

- [1] [RFC 6528]{https://tools.ietf.org/html/rfc6528}
- [2] [SipHash]{https://131002.net/siphash/}
- [3] [tcpcrypt]{http://www.tcpcrypt.org/}

