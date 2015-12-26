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
are an artifact of the network protocols used for communication. 

In this post I will explain why confidentiality is a necessary though not
sufficient condition for privacy. This is particularly important for CCN as
it focuses on *data* as a first class entity in the network rather than
hosts and addresses. To begin, XXX



1) confidentiality is necessary for privacy: if data was unencrypted, privacy is impossible (proof by contradiction -- an unencrypted bank transaction)
2) not sufficient for privacy: if data is encrypted once for many people then a valid user (and eavesdropper) can learn what another user is consuming without decryption
3) (D)TLS with HTTP/2 solve this problem by using forward-secure encryption for all communication. There is no grey area. It's all or nothing. Maybe that's needed, maybe it's not. 



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
