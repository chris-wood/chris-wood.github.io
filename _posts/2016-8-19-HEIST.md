---
layout: post
title: Beware of the Side Channel
---

Side channel attacks on widespread protocols such as TLS seem to happen with
uncomfortable regularity in recent years. We've seen compression-related attacks
in CRIME and BREACH, timing issues with Lucky13, and downgrade attacks on weak or
vulnerable key exchange mechanisms with LogJam and DROWN. As specification changes
and implementation changes are rolled out to counter these problems, it would seem
that the rate of vulnerability disclosure would decrease. Sadly, based on the results
from Blackhat USA this year, this is not the case. In this post, I'll talk about HEIST [1] --
a brilliant new attack on HTTPS that exploits the properties of TCP. 

# Preliminaries

At its core, HEIST exploits the flow control algorithm. TCP controls the rate at which it 
sends data by observing the packet acknowledgements sent by the receiver. If a sender
transmits 5 packets and all are acknowledged, then the sender concludes that there is little
to no congestion on the path to the receiver. It then increases the data window that controls
the maximum number of outstanding packets send to the receiver. This process continues to
additively increase the window size until such time that a packet acknowledgement does not 
arrive at the sender within some allotted timeout value. At this time, the window is decreased
by a multiplicative factor and the procedure tries to slowly build to the optimal point
of congestion. For more details about this and the TCP flow control variants, see [2]. 

The size of data that can possibly fit in a single TCP packet is called the Maximum
Segment Size (MSS). I'll denote one MSS as $$d$$. If a piece of data is larger than $$d$$, then the data is segmented
into blocks of size $$d$$. The sender then tries to send the entire piece of data in
TCP as quickly as possible using the flow control algorithm above. 

Assume that the initial window size for TCP is 2. If the data to be sent fits within is less
than $$2d$$, then, without any packet loss, the entire data can be transmitted in a single
round trip. If, however, the data is longer than $$2d$$, at least two round trips will be required
to transmit the data in total. This key piece of information is the basis of HEIST. 

# Timing Information

In Javascript, the Promise API allows code to be executed as data is received by the client. 
This is illustrated in the code snippet below (borrowed from the HEIST paper).

{% gist 9e391bf48e9fe28b4144b91f50ef483c %}

There are three timestamps to consider. $$T_1$$ is the time the initial request for data is sent, 
$$T_2$$ is the time the first byte of data is received, and $$T_3$$ is the time the entire data
is received. If $$T_3 \approx T_2$$, then it must be the case that the the data fit within a single
window. Otherwise, if $$T_3 > T_2$$ by some non-negligible amount, then the data must have required two
windows, or round trips, to transfer. 

# Attack 

So how can an attacker use this timing information to attack HTTPS? The answer is apparently simple. 
The adversary only needs to capabilities: 

1. It must be able to inject data into the request that will be *reflected* in the response of a request (or API call).
2. It must be able to observe the timing information of requests.

If the adversary can reflect data in a response, it can measure the size of the actual data (non-reflected bytes) carried 
in a response. It does this by measuring the time taken to receive the response. To explain how, let's start with a
simple example. The attacker starts small by only inserting a single byte into the reflected payload. If $$T_2 \approx T_3$$,
then the data fit within a single window. So, the attacker increases the reflected payload size and repeats this
measurement. It does so until the data requires two RTTs to transfer. Then, knowing the value of $$d$$, the initial window
size, and the size of the reflected payload, the attacker can determine the amount of actual bytes in the response. 
(Taking into account header TCP and TLS headers, of course.) 

Using this approach, the attacker can learn the size of a response. This is all that's needed to launch an
attack like BREACH or CRIME. Specifically, the attacker carefully chooses his reflected payload contents such that
they match secret values in the response and are therefore compressed together. The attacker can then learn, byte-by-byte,
the values of these secrets by checking whether his guessed value decreased the response size or not. (This single bit
of information leaks whether or not his guess was correct.)

# Comments

The requirement to observe network packet sizes and timing information, which can be done via Javascript code,
means that HEIST can be carried out entirely in the browser. It does not need the attacker to be on the path
between the client and server. This is particularly troublesome for most, if not all, web applications. And it's
yet another lesson about where and how side channels can materialize in practice. The authors also point out
that this problem is worse for HTTP/2 since multiple resources can be requested in parallel over a single TCP
connection. This allows the attacker to reflect content through a resource *unrelated* to the target, which is
something that was necessary for HTTP/1. (If it's hard to reflect data for a particular resource, then it's hard to
carry out the HEIST attack.)

# References

- [1] https://www.blackhat.com/docs/us-16/materials/us-16-VanGoethem-HEIST-HTTP-Encrypted-Information-Can-Be-Stolen-Through-TCP-Windows-wp.pdf
- [2] http://www4.ncsu.edu/~rhee/export/bitcp/cubic-paper.pdf 

