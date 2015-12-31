---
layout: post
title: CCN Privacy Pitfalls
---

In recent years, confidentiality and privacy have become relatively intertwined in practice.
To be clear, confidentiality is a *property of data* which guarantees that said data
is not disclosed to anyone without the owner's consent. Conversely, privacy is
a *property of users* which states that the actions or behaviors of a particular user
are not visible to unauthorized parties. In the context of networking, we are
concerned with the privacy of people using the network, where the observable
actions and behavior are the traffic that they send and receive. In this way,
privacy is more relevant to the *transfer of data* since, naturally, it is
data and metadata in transit that is subject to surveillance [1].
Surveillance is possible because we rely on networks such as the Internet
to communicate with the outside world. The actions performed on a device
that is disconnected from the network are private since they are not subject
to surveillance. It then seems that the coupling of privacy and confidentiality
are an artifact of the network protocols used for communication, which are
inherently subject to possibly malicious eavesdroppers (i.e., Big Brother).

In this post I will explain why confidentiality is a necessary though not
sufficient condition for privacy. To begin, first consider why confidentiality
is needed for privacy. If the traffic between a web browser and banking service
was not encrypted, then it would be trivial for an eavesdropper to
learn the details of a given session and, as a result, learn what the user
was doing in that session. In this context, privacy is not possible without
hiding the details of the traffic. However, encryption alone is not
sufficient. Suppose what could happen if clients hid the details of their
activity using some form of deterministic encryption, e.g., CPA-secure RSA,
using a *shared public key*. The properties of this encryption algorithm
mean that for any two identical plaintext inputs the output ciphertexts
will be identical. If two users hide requests for some shared resource
using encryption with the same public key, then these obfuscated requests
will be identical on the wire. To an eavesdropping who can only passively
observe packets in transit between clients and a server, it is easy to
deduce that some pair of clients asked for the same resource. How
does this knowledge relate to privacy? First, if these encrypted requests
correspond to some well-known resource, any adversary may guess the resource,
compute the encrypted form, and compare it to the observed form to verify
the guess. Secondly, the fact that some request matches that of another
client reveals distinctly more information than that of the request.

Today, the trend is to encrypt *all web traffic* using some form
of CCA-secure encryption algorithm backed by *forward secret* keys derived
from some key exchange protocol. CCA security,
or chosen-ciphertext attack security, roughly means that no two identical inputs
for an encryption algorithm will yield the same output (with negligible probability).
Forward secrecy means that short-term or ephemeral session keys are safe
even in the event that the long-term secrets used in the key exchange protocol
are compromised. These two properties are realized by TLS, the standard
translate layer security protocol used to encrypt TCP traffic which
encompasses much of the Internet. DTLS (datagram TLS) and QUIC are two UDP
variants which solve the same problem modulo some differentiating features
(e.g., secure source address tokens in QUIC). In fact, TLS 1.3
(the latest version) is heavily inspired by QUIC.
By using any of these protocols, confidentiality and privacy are intimately
related in the current Internet. Many web applications to do not add
additional layers of encryption *on top of TLS* to protect content.

It should be possible to decouple confidentiality and privacy and, when
appropriate, effortlessly enable privacy-preserving technologies. The current
TCP/IP communication model makes it easy to couple these two properties.
Clients know *where* to retrieve data or interact with services and
simply perform all end-to-end actions within a secure channel.
The communication is inherently channel-based and therefore
benefits from end-to-end encryption for both confidentiality and privacy.

CCN is different. It's data-centric communication model makes the separation
between confidentiality and privacy more clear since data is fixed
to a *name* and not an *address*. This means that data can be retrieved from
anywhere in the network. To make data confidential, content can
be encrypted such that only authorized users can perform
decryption. This is typically done by encrypting the payload of a
CCNx Content Object message -- everything else is left in the clear.
Since names explicitly identify a content object, this means that
(consumer) privacy is not possible; Any eavesdropper neighboring the
consumer can observe the name and learn what content the victim
consumer is after.

Enabling private communication in CCN is not as straightforward.
Privacy in the presence of eavesdroppers implies that, for any link
susceptible to surveillance, a packet must not reveal information about its
contents. In TCP/IP, this is done by encrypting everything in the
TCP payload (in a TLS record). The only information leaked from a packet
is its size (which can be padded -- see Section 5.2.3 of [2]) and
the source and destination address. In CCN, the equivalent property
is that the name only reveals information about the location or service
which could have procured the content. I generally refer to this
name prefix as the *minimal routable prefix* since it is the maximal name
length that is needed to route an interest to some authoritative producer
who can procure the corresponding content. Everything after this
prefix, including the payload in both interest and content objects,
must be protected with CCA-secure encryption. CCNx-KE [3] is a TLS-like
key exchange protocol for CCN that enables this type of private communication
based on secure channels. However, it does not solve the problem of
identifying the minimal routable prefix.

Thus, data confidentiality mechanisms in CCN and TCP/IP applications are, for all
intents and purposes, identical. Privacy is a much more disparate problem though.
Recall that, in TCP/IP, communication happens between a pair of end-hosts, be
it a client and server or two similar peers in a P2P network.
This communication forms a channel between the two end-hosts. Again, the industry
momentum is set on encrypting all traffic over these channels using TLS or some similar
protocol (TLS is in fact baked into HTTP2). Moreover, the DPRIVE IETF [4]
working group is even pushing to encrypt all DNS traffic over TLS or DTLS (depending
on the type of transport protocol used).

In CCN, communication happens between a consumer and some entity which contains
or can procure the data. There may be more than one entity capable of returning the
desired content, e.g., a caching router. Thus, if privacy is desired, a consumer
must establish a secure channel between one of these endpoints. But how does a
consumer identify the correct endpoint when all it has is the name for some
content? How does he or she learn the correct minimal routable prefix? The simple
answer is that, currently, there is no easy way. A consumer's application must
come with that information a priori. This is in stark contrast to the TCP/IP model
wherein consumers securely and privately obtain server addresses using DNS (with
DNSSEC [5] and DANE [6]). There is no guesswork. This means that, today, the problem
of private communication is distinctly harder in CCN than it is in TCP/IP.

The problem is to be expected since CCN is about locating and transferring named
data rather than locating services that can produce data upon request. Privacy,
in this context, necessitates a secure channel between a consumer and some endpoint
which can produce data. Moreover, CCN interests are based on data -- not services.
This underlying problem concerns me as people try to pitch CCN as a "more private"
alternative to IP. As an architecture which has claimed to try to replace IP (this
probably won't happen), it should not be more difficult to enable private communication.
CCN should at least have parity with IP in this respect.

In the next post I will explore the privacy issue in more depth. Namely, what is
the relationship between CCN names and data, services, and the privacy of those
who use them? How should namespaces be crafted to enable better privacy? How can
consumers learn the minimal routable prefix?

# References

- [1] S. Farrell and H. Tschofenig. "Pervasive monitoring is an attack." (2014). https://tools.ietf.org/html/rfc7258
- [2] E. Rescorla. "The Transport Layer Security (TLS) Protocol Version 1.3." https://tools.ietf.org/html/draft-ietf-tls-tls13-09
- [3] M. Mosko, E. Uzun, and C. Wood. "CCNx Key Exchange Protocol Version 1.0." https://github.com/PARC/ccnx-keyexchange-rfc/blob/master/draft-wood-icnrg-ccnxkeyexchange.txt
- [4] DNS PRIvate Exchange (dprive). https://datatracker.ietf.org/wg/dprive/charter/
- [5] G. Ateniese and S. Mangard. "A new approach to DNS security (DNSSEC)." Proceedings of the 8th ACM conference on Computer and Communications Security. ACM, 2001. https://web.cs.wpi.edu/~cshue/cs4404/papers/integrity_dnssec.pdf
- [6] P. Hoffman and J. Schlyter. "The DNS-Based Authentication of Named Entities (DANE) Transport Layer Security (TLS) Protocol: TLSA." https://tools.ietf.org/html/rfc6698
