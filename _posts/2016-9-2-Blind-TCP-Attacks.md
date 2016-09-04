---
layout: post
title: Sequence Numbers Are Considered Harmful?
---

At the USENIX Security conference this past August, Cao et al. presented a paper
entitled, "Off-Path TCP Exploits: Global Rate Limit Considered Dangerous" [1]. The paper
shows how a recent addendum to the TCP protocol, which was faithfully implemented
in the Linux kernel from 3.6 onwards (circa 2012), leads to a pretty
significant off-path attack on an ongoing TCP connection. The attack allows one to
identify if two hosts are communicating, learn the ongoing sequence number in use, 
and then reset or inject data into an ongoing connection. Ouch. 

In this post I'll present a high-level overview of the attack and then discuss what 
I believe to be the core problem with the protocol. 

# Scenario

RFC 5961 adds mitigations against what are known as blind attacks. Blind attacks
can seek to reset an ongoing connection or inject data into a connection. There are
two ways to carry out the blind reset attack. First, an adversary can send a packet
with the SYN bit prematurely set. Pre-5961, the connection would be immediately
reset if the sequence number in the SYN packet was deemed valid by the receiver. 
Otherwise, the receiver would respond with an ACK packet. Post-5961,
the receiver will challenge this SYN packet by sending a response with the ACK bit set.
Upon receipt, a valid sender will then respond with a packet that carries the RST bit and 
the correct sequence number. Thus, if the original SYN packet was spoofed, i.e., its source
address was forged, then the connection will not be closed. 

The second type of blind reset attack uses the RST bit to reset a connection. If
a RST packet carries a valid (in-window) sequence number, then the connection is reset.
Post-5961, the connection is only reset if the sequence number exactly matches the
next expected sequence number of the receiver. If it is less than the expected number,
nothing happens. If it is greater than the expected number, the receiver responds
with another challenge ACK packet. 

To blindly inject data, an adversary can forge malicious data packets with
in-window sequence numbers. By virtue of the SACK mechanism of TCP, these would
be accepted without any further checks. The problem here was that the window
of valid packets was huge -- basically half of the sequence number space. Post-5961,
the in-window ACK space was trimmed down to cover the range between (a) the 
current *unacknowledged number* (UNA) minus the maximum window size (MAX.WND) and (b) the
next expected sequence number (NXT). Anything less than this range, i.e., less than 
the maximum window size, fell in yet another challenge ACK space. This is shown
pictorially below. 

![Post-5961 ACK Space [1]](/images/posts/tcp5961_ackspace.png)

# Challenge ACKs

The remedies proposed in RFC 5961 may seem intuitively correct. However, challenge ACKs
are not cheap to send. They require additional network bandwidth and network cycles. 
Therefore, RFC 5961 also suggests to throttle the rate at which they are sent. Specifically,
it recommends that implementations SHOULD throttle to "be conservative." The Linux kernel
implementation of this throttling mechanism works by using a shared counter (initialized to 100) for the
number of challenge ACKs sent across all TCP connections. This is the side channel used to 
exploit the off path attacks. 

(For those interested, check ```sysctl``` to learn more about the configured value on your Linux box.)

# The Attack

The attack proceeds in phases. Given an off-path attack with no knowledge about whether 
a server and victim client are engaged in an ongoing TCP connection, the goal is to determine (1) whether
the client and server are communicating, (2) the sequence number (window) of the stream, and (3)
the ACK number of the stream. This is all that's needed to reset a connection or 
inject data. The adversary is assumed to have an active TCP connection to the target server.
This connection is used to send non-spoofed packets to the server to modify the state of the
shared challenge ACK counter. Each step of this attack is shown pictorially at the end 
of this section. 

## Step 1: Connection test

To determine if two hosts are communicating, the adversary sends a forged SYN-ACK packet
to a server (receiver). If the spoofed client information is correct, the receiver will 
respond with a challenge ACK packet as described above. If the client information is not
correct -- because the two hosts are not engaged in an active connection -- then the server
will respond with a RST packet. The adversary does not need to intercept this packet 
that was sent. Afterward, the adversary sends 100 non-spoofed and in-window RST packets 
to the target server. If the adversary receives 100 challenge ACK responses, then it knows
that its previous SYN-ACK packet must *not* have triggered a challenge ACK, and therefore
the client information was not correct. Otherwise, if it receives less than 100 challenge
ACKs, the adversary has learned that the server and target client were communicating. 

## Step 2: Sequence number test

Once the client and server information is known, the adversary can then proceed to learn 
the sequence number. To do so, the adversary sends a RST packet to the server with a guessed
sequence number. If the sequence number is in-window (but not exact), then a challenge
ACK is sent. If it is out-of-window, the RST is dropped. The adversary then again floods the
server with 100 of its own in-window non-spoofed RST packets. If it receives 100 challenge
ACKs, then it knows that the guessed sequence number was incorrect. Otherwise, if it receives
less than 100, it knows that its guessed sequence number was in-window.

## Step 3: ACK number test

The ACK number test is similar to the sequence number test except that the adversary
first sends a spoofed ACK packet to the server instead of a RST packet. It then reads the
shared counter by the non-spoofed RST packets to learn if its guessed ACK number was correct.

![Off-Path Connection, Sequence Number, and ACK Number Inference[1]](/images/posts/tcp_offpath_inference.png)

# Mitigations

The authors claim that the shared counter side channel is the root of the vulnerability. 
Thus, to remove it, simply use per-connection throttling. However, as the number of connections
increases on a system without bound, the number of challenge ACKs generated also increases.
This could have disasterous consequences on a server. Therefore, they recommend adding noise
to the throttling mechanism. Basically, instead of using a constant of 100 for the shared counter
value, that this varies over time. This will effectively confuse the adversary
in their test searches. In fact, the 4.7 upstream Linux kernel will randomize the maximum number 
of challenge ACKs sent per second. It will also randomly throttle challenge ACKs on a per-socket basis.

In theory this would work. However, I don't think it's addressing the core problem. These off-path attacks
are possible because it's trivial for one to learn the connection information for a TCP stream. But what if
the attacker's job of searching the sequence space was not a trivial binary search? What if, for example,
the sequence number in a packet was computed using a one-way function based on a counter and secret that
only the two communicating parties knew? This would effectively make the search space exponentially huge
and render these off-path attacks useless. However, it would likely cause problems for middleboxes and proxies. 
Still, I think it's an idea worth considering. I'm working on some code that will explore this
right now. I hope to write about it soon.

# References

- [1] [Cao, Yue, et al. "Off-Path TCP Exploits: Global Rate Limit Considered Dangerous."](http://www.cs.ucr.edu/~zhiyunq/pub/sec16_TCP_pure_offpath.pdf)

