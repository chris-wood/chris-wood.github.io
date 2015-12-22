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
way to measure the security of X and Y?

We need a definition of security that is appropriate for the discussion
at hand. Fortunately, thanks to the work of researchers over the last 35 years,
we either have or can craft such definitions.
During the 1980s, security (and cryptography in particular) went through a
major phase change: once fluid, moving, and ad-hoc definitions of security were re-cast
as concrete ideas with mathematical formulations. As pointed out by Koblitz
and Menezes [2], and as written by Katz and Lindell in their fresh book
on cryptography [3],

> One of the key intellectual contributions of modern cryptography has been the
> realization that formal definitions of security are *essential* prerequisites for
> the design, usage, or study of any cryptographic primitive or protocol.

The same standards must be upheld for the assessment of any network protocol as well.
Especially when the assessment is made with respect to the security properties of the
protocol. In this series of posts I hope to present concrete definitions of security
with respect to the following properties (which I vaguely define here for brevity):

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
ascertain the identity of some party involved in performing the operation.

I will then assess how CCN fares against IP in each category. Hopefully, by
writing this down, many of the misconceptions about CCN with respect to its supposedly
"better security" properties will vanish.

# 6. References
[1] Gollmann, Dieter. "Computer security." Wiley Interdisciplinary Reviews: Computational Statistics 2.5 (2010): 544-554.
[2] Koblitz, Neal, and Alfred Menezes. "Another Look at Security Definitions." IACR Cryptology ePrint Archive 2011 (2011): 343.
[3] Katz, Jonathan, and Yehuda Lindell. "Introduction to modern cryptography." CRC Press, 2014.
