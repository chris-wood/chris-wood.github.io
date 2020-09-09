---
layout: post
title: Clean Slate Crypto for Future Network Architectures
---

The Internet Protocol is old and carries a lot of baggage. Nearly every
protocol in almost every upper layer has IP at its heart: DNS, TLS, UDP, TCP, etc. 
The list goes on and on, as it should. Unlike IP, however, these protocols 
are subject to non-trivial evolution over time.
As a necessary step for backwards compatibility, the "design delta" between 
successive versions of these protocols is often small. For example, putting 
aside many details, the changes between 
TLS 1.2 and TLS 1.3 are mostly focused on the mechanics
of the key exchange and record protocol. But the principle remains the same:
a client and a server must derive a session key to encrypt traffic. The 
supported cryptographic algorithms to make this possible changed (for the better),
but the high-level design is, more or less, rigid. 

The story is much different for future network architectures (FIAs). In
FIAs such as CCN and NDN, there's no baggage. They can start
with a clean slate. And as part of the design process, the community would be wise
to make the most of state-of-the-art technology from academia to
industry. I am particularly interested in how we can use recent cryptographic
advances to provide security properties such as integrity, confidentiality,
privacy, and anonymity as *core network services* in CCN and other networks.

To that end, I surveyed the field to see what could be useful in ICN architectures.
I will present the outcomes from my preliminary research at the Dagstuhl seminar
on ICN security and privacy. This post identifies and motivates some technologies that
I think could be useful going forward. Some of them are extreme (read: borderline stupid
or infeasible). But I think they still offer a fresh perspective for the future of networks.

# Forward Secure Public-Key Encryption

I believe that forward secrecy is a necessary characteristic for transport protocols
in any communication network that aspire to be or resemble the current Internet. 
This property comes at a price. Current protocols require end-to-end sessions between
the client and server to establish forward-secure traffic keys. (Forget for a moment
that mcTLS [1] exists.) That tends to break the ICN model. However, not all is lost. 
There does exist forward-secure public key encryption schemes that do not require
any online interaction. The one by Canetti et al. [2] allows secret keys to be updated
at regular intervals. Forward secrecy here means that exposure of the private key
at a given time interval does not break the scheme, i.e., allow one to decrypt 
messages encrypted in the previous intervals. The most compelling part of this scheme
is that it does not require senders to cooperate with the receiver. It only requires
loose time synchronization, which is cheap and easy to do nowadays. Green and Miers
followed up on this scheme to create a "forward secure asynchronous messaging" scheme
[3]. Their construction allows decryption rights to specific messages, or tags, to be
revoked over time by modifying, or puncturing, the decrypting key. This prevents key
exposure from leaking messages that could have previously been decrypted prior
to the puncture procedure. 

These schemes seem directly applicable to ICN. If we view producers and consumers
loosely coupled senders and receivers, then we should be able to leverage either
one of these schemes to build forward-secure object encryption. In both schemes,
consumers must update their decryption keys. We do not need to trust routers to
re-encrypt anything, which is the beauty. Routers can remain transparent caches
that serve encrypted data.

# Randomizable Signatures

Many encryption schemes such as ElGamal, BGN, etc. are randomizable. This means that
ciphertexts may be "shuffled" such that they may still be decrypted by their intended
private keys. This is useful for mixnets wherein ciphertexts can be re-randomized at
random to further mask the input and output message correlation. However, in ICN,
these schemes break down since any modification to the ciphertext, i.e., an encrypted
packet payload, would invalidate its signature. Randomizable signatures like that of
Blazy et al. allow ciphertexts and their signatures to be modified in tandem such that
the signature on the re-randomized ciphertext remains valid. This could be an extremely
useful property were it efficient. Imagine if we had a trusted replica service that
would continually re-randomize encrypted packets. Two requests for the same content would
yield different results. Of course, this only solves half the problem: the requests
are still identical and correlation is possible. However, I still believe it's worth 
considering. 

# Encrypted DPI

End-to-end encryption renders many in-network processing elements, such as firewalls, intrusion
detection systems, etc. useless. Without being able to perform DPI, these utilities
can no longer operate effectively. BlindBox presented the first practical solution to
enabling DPI over encrypted packets [5]. Their system allows an untrusted middlebox to
perform keyword (token) searches inside encrypted packets quite efficiently without
sacrificing the security of the encrypted packets. That is, in the base protocol, a middlebox 
can learn if particular keyword was present in a packet payload, but nothing more. 
Enhancements to this protocol allow the same middlebox to recover the traffic key if
a certain keyword is detected. 

Regardless of the approach, the notion of encrypted DPI is compelling. I aspire towards 
an ICN architecture where all packet contents are encrypted and yet still routable. 
Encrypted DPI might be one piece of the puzzle necessary to enable this type of action. 
As a matter of comparison, imagine that name prefixes were keywords and endsystems gave
routers, or middleboxes, the ability to learn when these keywords were present in a packet. 
This functionality alone would be sufficient to implement routing on encrypted names. 
(Hmm, maybe that'll be a fun homework assignment.)

# References

- [1] Naylor, David, et al. "Multi-context TLS (mcTLS): Enabling secure in-network functionality in TLS." ACM SIGCOMM Computer Communication Review. Vol. 45. No. 4. ACM, 2015.
- [2] Canetti, Ran, Shai Halevi, and Jonathan Katz. "A forward-secure public-key encryption scheme." International Conference on the Theory and Applications of Cryptographic Techniques. Springer Berlin Heidelberg, 2003.
- [3] Green, Matthew D., and Ian Miers. "Forward Secure Asynchronous Messaging from Puncturable Encryption." 2015 IEEE Symposium on Security and Privacy. IEEE, 2015.
- [4] Blazy, Olivier, et al. "Signatures on randomizable ciphertexts." International Workshop on Public Key Cryptography. Springer Berlin Heidelberg, 2011.
- [5] Sherry, Justine, et al. "Blindbox: Deep packet inspection over encrypted traffic." ACM SIGCOMM Computer Communication Review. Vol. 45. No. 4. ACM, 2015.
