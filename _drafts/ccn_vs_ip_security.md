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

To begin, we need a definition of security that is appropriate for the discussion
at hand. Historically, before security transformed from an art to a
science, it was characterized by confidentiality, integrity, and availability [cite].  
These terms were loosely defined and molded to fit the needs of the user.
For example, confidentiality is roughly stated as the property that sensitive data
is not disclosed to unauthorized persons [1]. This is hand-wavy at best.
If we are to assess the security of CCN or IP then we need sound definitions
for these terms against which to do this assessment. Fortunately, thanks to
the work of researchers over the last 35 years, we have such definitions at
our disposal.

During the 1980s, security (and cryptography in particular) went through a
major phase change: once fluid, moving, and ad-hoc definitions of security were re-cast
as concrete ideas with mathematical formulations. As pointed out by Koblitz
and Menezes [2], and as written by Katz and Lindell in their fresh book
on cryptography [3],

> One of the key intellectual contributions of modern cryptography has been the
> realization that formal definitions of security are *essential* prerequisites for
> the design, usage, or study of any cryptographic primitive or protocol.

The same standards must be upheld for the assessment of any networking protocol as well.
Especially when the assessment is made with respect to the security properties of the
protocol. To that end, let us now revisit the CIA terms introduced above (in addition to
privacy and anonymity). I will briefly describe what each term means here. The formal
definitions are deferred to subsequent posts wherein I will assess CCN and IP with respect
to each.

* Confidentiality: Data is only accessible by those which are authorized or have
permission to access the data. Put differently, data is confidential if it is not
inadvertently disclosed to any unauthorized party.
* Integrity: Data cannot be changed by unauthorized any unauthorized person.
* Availability: Data is available to authorized persons when needed.
* Privacy: Information leaked by a particular operation (e.g., sending an IP
packet to from one host to another or sending a CCN interest) is (strictly)
limited. For example, in the context of CCN, such leakage cannot be used to
ascertain the what data was requested or what operation was performed.
* Anonymity: Information leaked by a particular operation cannot be used to
ascertain the identity of some party involved in performing the operation. For
example,

# 6. References
[1] Gollmann, Dieter. "Computer security." Wiley Interdisciplinary Reviews: Computational Statistics 2.5 (2010): 544-554.
[2] Koblitz, Neal, and Alfred Menezes. "Another Look at Security Definitions." IACR Cryptology ePrint Archive 2011 (2011): 343.
[3] Katz, Jonathan, and Yehuda Lindell. "Introduction to modern cryptography." CRC Press, 2014.
