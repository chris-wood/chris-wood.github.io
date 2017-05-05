---
layout: post
title: Trends in Protocol Security
---

2016 was a tremendous year for security protocols in the Internet. Private messaging took off,
transparency projects found traction, TLS 1.3 came closer to finalization, and people finally (?)
started taking NTP security seriously. All of these things
bode well for the population of Internet users at large. And as I track them moving forward,
I want to take some time and extract useful principles and properties of each that would serve
similar efforts in the future. In particular, I want to focus on the concepts of transparency
and implementation robustness.

*Transparency* is a property of some select information which requires that its users
be able to ascertain all relevant historical information (provenance data) since its
inception to the present time. A common approach to achieve transparency is via logging.
That is, you log the creation of an item and then any and all changes to the item over the
course of its lifetime. By inspecting the log, a user should be able to answer any query
they may have about the item.

In contrast to transparency, which is a system design property, *robustness*
is a decidedly different property of the system that can capture design and implementation
quality. A robust design is one which can stand up to failures. For example, a distributed
system protocol is robust if it can withstand interference or otherwise malicious behavior
induced by Byzantine [1] errors and faults. An implementation is robust if it operates
correctly in the presence of similar errors or faults.

From a security perspective, it's not hard to understand why these two properties are important
when designing and building systems. To substantiate that claim, I'll spend the rest of this
post describing a variety of protocols and systems that exhibit these properties. I hope to also
plug whatever lingering gaps of knowledge I have with respect to these systems.

# Certificate Transparency

# Key Transparency

# Network Time Robustness

As Adam Langley writes [x1], it's a accurate local time is a "widely accepted assumption" for many systems.
However, the original NTP protocol [x2] is 100% insecure. Anyone can tamper with the protocol communication
or messages to subvert a system's perception of real time. This can be exploited in subtle ways, such as bypassing
certificate validation logic for a truly-expired certificate. Put simply, after 15 years of work, secure NTP
remains somewhat elusive. Granted, recent works such as the Authenticated NTP protocol [x3] have shown how
to add encryption and authentication to the original messages, but this is only a (relatively speaking) small
step in the right direction. (By no means do I mean to belittle the work of Dowling et al. -- their iteration
of NTP uses a brilliant combination of key encapsulation and symmetric encryption to keep the server's stateless
while maintaining efficiency.) To build on this momentum, Langley and others developed and prototyped *Roughtime.*



[x1] https://www.imperialviolet.org/2016/09/19/roughtime.html
[x2] NTP
[ x3] https://www.usenix.org/system/files/conference/usenixsecurity16/sec16_paper_dowling.pdf

# GREASing TLS Version Negotiation

XXX

# References

- [1] Byzantine errors faults

#Trends: design transparency and implementation robustness
#CT, CONICKS, GREASE, RoughtTime
