---
layout: post
title: Private Service Discovery
---

I was recently using Wireshark [1] to debug a protocol issue with a CCN application
and stumbled across some interesting traffic coming from my machine. Every 30
seconds my box would broadcast a packet to the LAN (255.255.255.255) and get a
response back (from my own machine). This is part of the Dropbox LAN sync Discovery 
Protocol [2]. The contents of this packet are shown below. 

XXX: screen shot here

Inside the broadcast UDP packet is a JSON object which contains the following fields:
"host_int", "version", "displayname", "port", and "namespaces." The last one was 
particularly troubling. According to [3], namespaces are unique identifiers for the
roots or mount points of directories owned by a user. Files are then identified by
a namespace and the relative path. (In actuality, files are described in pieces by a 
collection of SHA-256 fingerprints of each piece. This lets us use a Merkle tree to 
determine when two files are identical and, when they are not, efficiently synchronize 
their contents.) 

According to [2], these discovery requests are broadcast to the LAN in hopes that
another machine which happens to own the same namespaces can be used to more efficiently
synchronize data. If another machine on the LAN owns the same namespaces then it responds.
(Or other machines always respond and the requestor computes the namespace intersection
to determine matching machines. Both of these are approaches are equivalent so the details
are not important.) The machines then synchronize their respective namespaces. If this 
does not happen, the machine pulls its data from the cloud. This is illustrated below.

XXX: image from dropbox site (with reference)

# Hijacking Namespaces

By now the alarm should be ringing if you've closely inspected these discovery packets.
They contain no form of authentication, i.e., no signature or MAC. What does this mean?
Anyone can advertise any namespace they want and initiate a synchronization event with
a victim machine. Consider the following attack: A passive eavesdropper listening 
for these discovery packets can collect all of the observed namespaces and then, in one
fowl swoop, start to echo them back in forged responses. This will cause the victim(s)
to try and synchronize the claimed namespaces with the attacker machine. The attacker
has, in effect, tried to hijack the namespaces. 

Fortunately, Dropbox vetts ownership of namespaces by distributing certificates 
(and private keys) to each client. To syncrhonize between machines, the responder
runs a LAN sync server to which the requestor connects over over TLS. Moreover,
in the initial handshake, the requestor specifies the SNI (Server Name Indication)
of the target to be the namespace to be synchronized. The SNI allows the server
to use the correct certificate when completing the TLS handshake, since the server
might have control over more than namespace and thus carry multiple certificates.
The TLS handshake will therefore only complete if the server can verify ownership of
the namespace private key. And once the TLS session is created, the client and server
run their synchronization protocol. The details of which are not important here.
 
This reduces the security of the synchronization protocol to TLS between a client
and server using the same certificate. While TLS may be the most important
protocol in use today, it's certainly not free of design and implementation problems. 
Everything from complicated timing attacks on the MAC computations (Lucky13) 
to often simple compression-based padding oracle attacks (CRIME and BREACH). 
Ivan Ristic put together a comprehensive list of the history of SSL/TLS [here](https://www.feistyduck.com/ssl-tls-and-pki-history/).
The attacks I've listed only scratch the surface.

# Validated Advertisements

If you're like me, you're not comfortable with something as brittle as TLS being the only thing
standing between a local attacker and your sensitive data. We need defense in depth, and a good place
to start is with the root of the problem: advertisements are *not* authenticated. And beyond the 
computation cost, I don't quite understand why. Recall that each namespace has an associated certificate 
with it. That means that each namespace advertisement could have a signature associated with it. The 
verification key would be implicitly identified by the namespace identifier. The receiver of an advertisement
would then verify the signature of each matching namespace in the message. If does not need to verify
namespaces with which it has no chance of synchronizing. 

This would give us a better security cushion than what's currently done. But there's an obvious
performance penalty here. A client with $$n$$ namespaces must perform, in the worst case, $$n$$
signature verifications in response to a discovery. An attacker can easily make this happen
by using garbage data for the namespace signature in each namespace advertised by the client. 

This may seem problematic at first. But before jumping to conclusions, what exactly is the overhead?
I put together some code to profile the signature verification costs using ECDSA. (I just chose this
because it's known to perform well, or at least comparatively better than RSA and DSA signatures.)
You can run it to check the numbers on your machine. The total verification time for up to 100 namespaces
is plotted below (in microseconds).

XXX: plot for verification time

My laptop only advertises 31 namespaces every 30 seconds, which, according to my data, would
require 0.063s to verify to completion. That's not too bad. But if your Dropbox space is huge,
you own many different folders, or if you are involved in many shared folders, this can grow
without bound. As the data indicates, this can become therefore prohibitvely expensive. And if
there were ever a list of rules for deciding how much cryptography should be used to solve a problem,
near the tope of that list it should read, "never do an unbounded number of cryptographic operations."
So it makes sense that Dropbox doesn't use signatures to verify namespace ownership. Doing so would
create a negative incentive to become involved in more shared folders or make better use of their 
service since it would come with an added computational cost. (Of course, one could always disable
LAN sync protocol in the first place. But let's not be crazy! ;))

# An Enhancement with Privacy 

-- discuss their approach: prefix-encrypt advertisements and allow for 0-RTT resumption, but still doens't solve the problem of a client attempting every ciphertext until it gets one it wants, but still very clever
-- discuss privacy, mention stanford paper in brief

# References

- [1] wireshark
- [2] lan SYNC reference (https://www.dropbox.com/s/rs13ex5l0hlihhm/Screenshot%202016-09-12%2011.04.11.png?dl=0)

