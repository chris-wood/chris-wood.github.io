---
layout: post
title: CCN vs IP: Confidentiality
---

confidentiality: can't see data
integrity: didn't change data
availability: can't make data unavailable
privacy: can't learn information about the user


in IP: confidentiality means encrypting the channel, and there are several ways
to do this: IPSec and (D)TLS. Draw pictures about how they work.
    vulnerabilities are in implementation details, key management, (certificate)
 trust, and crypto algorithms
in CCN: confidentiality means encrypting data
    group-based encr: vulnerabilities are in implementation details, key managem
ent, access control management,
    session encr: same problems as TLS.

OUTCOME: ???

integrity:
    IP:
        IPSec and (D)TLS provide signatures and AEAD algorithms for integrity --
 good.
        still has problems of bootstrapping that protocol
    CCN:
        provides digital signatures and easy ways to locate keys (i.e., certiica
te retrieval is the same)

OUTCOME: integrity is the same, IP has more details to be concerned with, both h
ave to deal with trust. but CCN has the benefit of using different trust models
for all traffic in the network, not just PKI, i.e., WoT and schematized trust

availabiity:
    - both suffer from DoS
    - CCN is worse in several regards:
        - every request must generate some response -> attacking a producer is e
asier (DNS-like authenticated DNE is possible but hard)
        - routers have state that can be exploited (interest flooding attacks ar
e easy)

OUTCOME: IP wins

Privacy: names! (make that a separate blog entry)
