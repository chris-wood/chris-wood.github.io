---
layout: post
title: Is CCN more secure than IP?
---

Security is an annoyingly broad term, especially when used to describe a networking architecture. 
Many papers and presentations often tout CCN and related information-centric networking designs 
as more "secure than IP." Such bold claims are meaningless for several reasons. 
First, the notion of security is completely undefined. Is CCN more confidential 
than IP? Does it provide better privacy and anonymity? Can it withstand common 
networking attacks such as DDoS? The answer is unclear. Second, there is generally 
no adversarial model in which to assess these supposedly superior security properties. 
How can we say that "X is more secure than Y" without having defined an appropriate 
way to measure the security of X and Y. In this series of posts I will attempt to 
answer these questions one at a time. 

To begin, we need a definition of security that is appropriate for the disucssion at hand. 
Historically, before security transformed from an art to a 
science, it was characterized by confidentiality, integrity, and availability [cite].  
These terms were loosely defined and molded to fit the needs of the user.
For example, confidentiality is roughly stated as the property that sensitive data
is not disclosed to unauthorized persons [cite]. This is hand-wavy at best. 
If we are to assess the security of CCN or IP then we need sound definitions
for these terms against which to do this assessment. Fortunately, thanks to 
the work of researchers over the last 35 years, we have such definitions at 
our disposal. 

During the 1908s, security (and cryptographic in particular) went through a 
major phase change: once fluid, moving, and ad-hoc definitions of security were re-cast
as concrete ideas with mathematical formulations. As pointed out by Koblitz 
and Menezes [cite1], and as written by Katz and Lindell in their fresh book
on cryptography [cite2],

> One of the key intellectual contributions of modern cryptography has been the 
> realization that formal definitions of security are *essential* prerequisites for
> the design, usage, or study of any cryptographic primitive or protocol. 

[1] https://eprint.iacr.org/2011/343.pdf
[2] Katz, Jonathan, and Yehuda Lindell. Introduction to modern cryptography. CRC Press, 2014.

The same can be said for the assessment of any networking protocol as well.
Especially when the assessment is made with respect to the security properties of the
protocol. To that end, let us now revisit the CIA terms introduced above. Let us 
also include the notions of privacy and anonymity. 

# Confidentiality

# Integrity

# Availability

# Privacy

# Anonymity







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
