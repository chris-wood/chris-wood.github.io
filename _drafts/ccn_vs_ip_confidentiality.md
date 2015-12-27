---
layout: post
title: CCN vs IP: Confidentiality
---

In recent years, confidentiality and privacy have become relatively intertwined in practice. 
As mentioned in the previous post, privacy is the property that the actions or operations
of a particular user are not visible to unauthorized parties. This can be cast as another
form of confidentiality. After all, if data is not confidential, then it is almost certainly
true that the purpose of said data can be determined. 

Privacy is generally considered a desirable property in order to prevent pervasive
monitoring attacks, which are "widespread widespread (and often covert)
surveillance through intrusive gathering of protocol artefacts,
including application content, or protocol metadata such as headers" [1]. 
Surveillance is possible because we rely on networks such as the Internet
to communicate with the outside world. Indeed, the actions performed on a device 
that is disconnected from the network are private since they are not subject
to surveillance. It then seems that the coupling of privacy and confidentiality
are an artifact of the network protocols used for communication which are 
inherently subject to possibly malicious eavesdroppers (Big Brother). 

In this post I will explain why confidentiality is a necessary though not
sufficient condition for privacy. This is particularly important for CCN as
it focuses on *data* as a first class entity in the network rather than
hosts and addresses. To begin, consider first why confidentiality is
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

CCN is differently. It's data-centric communication model allows for
confidentialty and privacy to be decoupled when needed. 

TODO: describe an example of confidentiality and then privacy

TODO: argue that CCN provides a better communication abstraction that allows
security properties to be decorate the data exchange, but it does not
introduce new solutions to these problems. 

# References

[1] Farrell, Stephen, and Hannes Tschofenig. "Pervasive monitoring is an attack." (2014). https://tools.ietf.org/html/rfc7258




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
