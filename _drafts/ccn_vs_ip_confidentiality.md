---
layout: post
title: CCN vs IP: Confidentiality
---

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
