---
layout: post
title: What is ICN with Perfect Forward Secrecy?
---

The IETF and related standards organizations converged towards the fundamental requirements
for transport security in the Internet. The Snowden revelations accelerated this decision as a way to curb mass surveillance. The growing consensus is that (perfect) forward secrecy is necessary to protect
communication [1] [2] [3]. Forward secrecy is a property of a communication session which states that the key used
to protect traffic for the session is derived from a non-deterministic algorithm. In practice, this means that each
party contributes a fresh Diffie-Hellman key share used to derive the session key. Secure session protocols like
TLS [2] build on this basic key exchange primitive to make session-based communication more efficient. But at the end
of the day, they all share the property of forward secrecy. Transport security must be, at a minimum, forward secure.

Unfortunately, the ICN community seems to be in a state of denial about this fundamental requirement. Many research
papers and applications depend on traffic which is not only non-forward secure but not encrypted at all. I am guilty
of writing papers like this myself. Indeed, if the world decides to deploy ICN in the Internet, then it absolutely
must be held to the same security requirements as modern TCP/IP protocols.

Last week, Mark Stapp and I put together a document entitled ["ICN Privacy Principles"](https://github.com/chris-wood/icn-privacy-principles)
to make these requirements explicit. In it, we outline six basic principles that we believe should drive the baseline
of communication in ICN. Without restating these principles verbatim, I will summarize the main theme of the document in two sentences:

1. All traffic must be encrypted by default using ephemeral keys (i.e., traffic must be forward secure).
2. An entity must properly authenticate an endpoint before sending any application data
to it. (This remains true for both consumers and producers.)

If you are a believer in the IETF's stance on privacy, e.g., that correlation is an attack on users and
forward secrecy is a necessity, then these principles should strike a chord with you. However, they raise some
fundamental questions about ICN as an architecture. The original ICN vision relied on in-network state
and intelligence based on cleartext names [4]. So if ICN messages (interests and content objects) contain
only some routable prefix (or none at all) and everything else protected by ephemeral keys, then
is this network truly "information-centric"?

In my opinion: no, it's not. Forward secrecy implies the existence of a session, which also implies that
communication happens between two well-defined endpoints. In IP, these endpoints are machines with specific
addresses. However, in ICN, these endpoints are a consumer and something like a service (or producer)
which can generate or produce data. Routable prefixes identify these services and the encrypted body
of an interest or content object contains the application data to be transferred. Make no mistake: this is not a step
backwards. Many applications today scale to resemble these types of services. A huge number of applications
use CDNs to push data close to consumers who then interact with these services to get data. This means that
service-centric networking, as opposed to information-centric networking, seems to more closely align with
modern web and mobile application models.

And is it not the point of the ICN research vision to develop a network architecture that better suits
modern applications?

# References

- [1] Specification for DNS over TLS, [https://tools.ietf.org/html/draft-ietf-dprive-dns-over-tls-09](https://tools.ietf.org/html/draft-ietf-dprive-dns-over-tls-09)
- [2] The Transport Layer Security (TLS) Protocol Version 1.3, [https://tools.ietf.org/html/draft-ietf-tls-tls13-11](https://tools.ietf.org/html/draft-ietf-tls-tls13-11)/
- [3] tcpcrypt, [http://www.tcpcrypt.org/](http://www.tcpcrypt.org/).
- [4] Jacobson, Van, et al. "Networking named content." Proceedings of the 5th international conference on Emerging networking experiments and technologies. ACM, 2009.
