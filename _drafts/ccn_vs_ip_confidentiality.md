---
layout: post
title: Confidentiality and Privacy in CCN and TCP/IP
---

In recent years, confidentiality and privacy have become relatively intertwined in practice. 
As mentioned in the previous post, privacy is the property that the actions or operations
of a particular user are not visible to unauthorized parties. This can be cast as another
form of confidentiality. After all, if data is not confidential, then it is almost certainly
true that the purpose of said data can be determined. 

Privacy is generally considered a desirable property in order to deter pervasive
monitoring attacks, which are "widespread widespread (and often covert)
surveillance through intrusive gathering of protocol artefacts,
including application content, or protocol metadata such as headers" [1]. 
Surveillance is possible because we rely on networks such as the Internet
to communicate with the outside world. The actions performed on a device 
that is disconnected from the network are private since they are not subject
to surveillance. It then seems that the coupling of privacy and confidentiality
are an artifact of the network protocols used for communication which are 
inherently subject to possibly malicious eavesdroppers (Big Brother). 

In this post I will explain why confidentiality is a necessary though not
sufficient condition for privacy. I will also try to argue that, with respect
to networking, confidentiality is a *property of the data* and privacy is a 
*property of the users and the transfer of data*.

To begin, consider first why confidentiality is
needed for privacy. If the traffic between a web browser and banking service
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

Today, the Internet trend is to encrypt *all web traffic* using some form
of CCA-secure encryption algorithm backed by *forward secret* keys. CCA security,
or chosen-ciphertext attack security, roughly means that no two identical inputs
for an encryption algorithm will yield the same output (with negligible probability).
The latter property means that short-term or ephemeral session keys are safe
even in the event that the long-term secrets used in the key exchange protocol
are compromised. These two properties are realized by TLS, the standard
translate layer security protocol used to encrypt *all* TCP traffic which
encompasses much of the Internet. DTLS (datagram TLS) and QUIC are two UDP
variants which solve the same problem modulo some differentiating features. 
In fact, TLS 1.3 (the latest version) is heavily inspired by QUIC. 
The goals of these protocols, not the details, are important for this discussion.
At a high level, the mantra is to encrypt everything and everywhere. 
There is no grey area to encrypt only a subset of the data; it's all or nothing. 
In this way, confidentiality and privacy are intimately related in the 
current Internet. Most web applications to do not add additional layers
of encryption *on top of TLS* to protect content. 

Without getting into a political or philosophical discussion, I simply
propose that ubiquitious privacy-preserving communication mechanisms may 
not be appropriate for all forms of data. Said differently, it should be 
possible to decouple confidentiality and privacy and, when appropriate, 
effortlessly enable privacy-preserving technologies. The current TCP/IP
communication model almost necessitates that these two properties be
tightly coupled. Clients know *where* to retrieve data or interact
with services and simply perform all end-to-end actions within a secure
channel. The communication is inherently channel-based and therefore 
benefits from end-to-end encryption for both confidentiality and privacy.

CCN is different. It's data-centric communication model allows for
confidentialty and privacy to be decoupled when needed since data
is fixed to a *name* and not an *address*. To enable confidentiality,
content can be encrypted such that only authorized users can perform
decryption. This is typically done by encrypting the payload of a 
CCNx Content Object message [cite] -- everything else is left in the clear.
Since names explicitly identify a content object, this means that 
(consumer) privacy is not possible; Any eavedropper neighboring the
consumer can observe the name and learn what content the victim
consumer is after. 

Privacy in the presence of eavesdroppers implies that, for any link
susceptible to surveillance, a packet must not reveal information about its
contents. In TCP/IP, this is done by encrypting everything in the
TCP payload (in a TLS record). The only information leaked from a packet
is its size (which can be padded -- see Section 5.2.3 of [tls]) and
the source and destination address. In CCN, the equivalent property 
is that the name only reveals information about the location or service
which could have procured the content. I generally refer to this 
name prefix as the *minimal routable prefix,* as it is the maximal name
length that is needed to route an interest to some authoritative producer
who can procure the corresponding content. Everything after this 
prefix, including the payload in both interest and content objects,
must be protected with randomized encryption. CCNx-KE [ccnxke]
is a TLS-like key exchange protocol for CCN that enables private
communication based on secure channels. 

In TCP/IP, when a consumer needs to retrieve data, it must connect to a specific
endpoint and issue the request (HTTP GETs happen over TCP and TLS). 
If confidentiality is required, the result can be encrypted for the user
using their public key, for example. However, this requires proper
key management and distribution to be in place so that the server (producer)
can use the correct encryption key. Since client authentication is part
of the TLS protocol [tls], this protocol can be used to perform authentication,
authorization, and subsequent encryption. This effectively overloads TLS
to achieve both confidentiality and privacy. It may be the case that 
encryption happens at the application layer. This is common in CDN
applications wherein data is encrypted with a key that is separate from
that which is derived via TLS. For example, a client may connect to a CDN
node to retrieve encrypted media over a secure channel. The client then uses 
decryption keys retrieved from the data producer (e.g., Netflix) to 
decrypt the result. In this case, one form of encryption is used for
confidentiality (e.g., the CDN node cannot learn the contents of the
media file since they are encrypted) and another form of encryption is
used for privacy. This separation is cleaner in CCN since confidentiality
is achieved by encrypting the payload of a content object and privacy is
achieved by encrypting the information used to transport that payload, e.g.,
the name of an interest and its content object. 

Thus, data confidentiality mechanisms in CCN and TCP/IP applications are
identical; There is no profound difference between the two. However, privacy-preserving
techniques, while seemingly equivalent in practice (e.g., TLS vs CCNx-KE), 
have one important difference. In TCP/IP, communication happens between a
pair of end-hosts, be it a client and server or two similar peers in a P2P network.
This communication forms a channel between the two end-hosts. Thus, if data is to be
protected from eavesdroppers then either (a) every application must consiously
encrypt its data before issuing it to a transport protocol like TCP or UDP, or
(b) encrypt the channel so that *all* traffic flowing through it is encrypted
by default. Again, the momentum is on option (b)--encrypting the channel by default
using protocols like IPSec at the network layer, (D)TLS at the transport layer,
and HTTPS at the application layer in the protocol stack. The DPRIVE IETF [dprive]
working group is even pushing to encrypt all DNS traffic over TLS or DTLS depending
on the type of transport protocol used. 

In CCN, communication happens between a consumer and some entity which contains
or can procure the data. There may be more than one entity capable of returning the
desired content. Thus, if privacy is desired, a consumer must establish a secure channel
between one of these endpoints. But how does a consumer identify the correct endpoint when
all it has is the name for some content? How does it learn the correct minimal routable
prefix? I don't propose a solution here. Instead, I simply state that this problem
makes privacy *more difficult* in CCN. And its an artifact of merging DNS and TCP/IP
with name-based routing. This should be an intuitive observation since CCN is about
locating and transferring named data. It is not about locating services that can
produce data upon request. Privacy necessitates a secure channel between a consumer 
and some endpoint which can produce data. And CCN interests are based on data -- not
services. This underlying problem concerns me as people try to pitch CCN as a "more 
private" alternative to IP. 

# References

[1] Farrell, Stephen, and Hannes Tschofenig. "Pervasive monitoring is an attack." (2014). https://tools.ietf.org/html/rfc7258
[tls] https://tools.ietf.org/html/draft-ietf-tls-tls13-09#section-5.2.3
[ccnxke] TODO
[dprive] TODO



<!--
I think part of the mixture is due to current network protocols. 

As networking protocols, neither CCN nor IP solve the problem of data confidentiality. 
This task is (rightfully so) something that should be solved higher up in the stack. 
After all, whether or not data is disclosed to an unauthorized person is highly dependent
on *who* is unauthorized. And this task is application-agnostic. This is not to say
that "the network" should not aid users in providing some measure of confidentiality. 
I simply mean that the network cannot operate autonomously. 

IP stacks are decorated to provide this provide applications with confidentiality

Due to the channel-based communication model in IP, traditional TCP/IP stacks are
decorated to provide network- or transport-layer encryption via IPSec or (D)TLS. 




confidentiality: can't see data

*neither solution provides confidentiality out of the box*
... but, confidentiality solutions are built on top of the network protocol

in IP: confidentiality means encrypting the channel, and there are several ways
to do this: IPSec and (D)TLS. Draw pictures about how they work.
    vulnerabilities are in implementation details, key management, (certificate) trust, and crypto algorithms
in CCN: confidentiality means encrypting data
    group-based encr: vulnerabilities are in implementation details, key management, access control management,
    session encr: same problems as TLS.

OUTCOME: ???

-->
