---
layout: post
title: Is CCN more secure than IP?
---

Security is an annoyingly broad term, especially when used to describe a networking architecture. Many presentations often tout CCN and related information-centric networking designs as more secure than IP. Such bold claims are meaningless for several reasons. First, the notion of security is completely undefined. Is CCN more confidential than IP? Does it provide better privacy and anonymity? Can it withstand common networking attacks such as DDoS? The answer is unclear. Second, there is generally no adversarial model in which to assess these supposedly superior security properties. How can we say that "X is more secure than Y" without having defined an appropriate way to measure the security of X and Y. In this post I will attempt to answer these questions. 

To begin, we need a definition of security. For classical reasons, I will start with the traditional CIA meaning of confidentiality, integrity, and availability. Moreover, I will only focus on the network layer itself. Most, if not all, supposeldy superior security properties of CCN applications rely on festures provided by the network.

confidentiality: can't see data
integrity: didn't change data
availability: can't make data unavailable

in IP: confidentiality means encrypting the channel, and there are several ways to do this: IPSec and (D)TLS. Draw pictures about how they work.
    vulnerabilities are in implementation details, key management, (certificate) trust, and crypto algorithms
in CCN: confidentiality means encrypting data
    group-based encr: vulnerabilities are in implementation details, key management, access control management, 
    session encr: same problems as TLS.

OUTCOME: ???

integrity: 
    IP:
        IPSec and (D)TLS provide signatures and AEAD algorithms for integrity -- good. 
        still has problems of bootstrapping that protocol
    CCN:
        provides digital signatures and easy ways to locate keys (i.e., certiicate retrieval is the same)

OUTCOME: integrity is the same, IP has more details to be concerned with, both have to deal with trust. but CCN has the benefit of using different trust models for all traffic in the network, not just PKI, i.e., WoT and schematized trust

availabiity:
    - both suffer from DoS
    - CCN is worse in several regards:
        - every request must generate some response -> attacking a producer is easier (DNS-like authenticated DNE is possible but hard)
        - routers have state that can be exploited (interest flooding attacks are easy)

OUTCOME: IP wins

Privacy: names! (make that a separate blog entry)
