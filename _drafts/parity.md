--
layout: post
title: What does security and privacy parity mean for CCN?
---

Anyone following the development of current IP transport protocols and 
associated application-layer protocols (i.e., HTTPS) should be aware
of the current security and privacy trends. After the Snowden fallout
and an increasing amount of distrust in federal agencies to respect the
privacy of the world's inhabitants, the need and demand for secure and 
private communication has never been higher. This, coupled with the 
emergence of efforts to create security solutions not backed by "trusted" 
third parties, such as Let's Encrypt, has created a political, cultural,
and industrial atmosphere of privacy paranoia with a tremendous amount
of momentum. In general, the movement seems to be to encrypt everything with
perfect forward secrecy. 

Why is this the case? The fear is that unencrypted traffic is subject to
(deep) packet inspection, and rightfully so. With tools like Wireshark it's
almost trivial for anyone with a little know-how to snoop on their neighbor's
packets in the local coffee shop. While this is a legitimate reason to
encrypt your traffic, the real reason is much more paramount. When traffic is
unencrypted your ISP is free to inspect anything and everything that they
carry on your behalf. They may extract information from your traffic and 
provide it to others for their personal benefit, among many other malicious
or evil behavior. 

IP traffic is particularly problematic because it carries (a) application data,
which can be sensitive information that one might not want to reveal to 
eavesdroppers, and (b) source and destination addresses. The latter "links" the
application data to the two entities located at the source and destination 
addresses. Linkability of data to clients or servers is a violation of privacy:
If one can link data to you then you must not be communicating privately. 

This is an important observation for IP traffic: communication happens between a 
pair of end-hosts, be it a client and server or two similar peers in a P2P network. 
This communication forms a channel between the two end-hosts. Thus, if data is to be
protected from eavesdroppers then either (a) every application must consiously 
encrypt its data before issuing it to a transport protocol like TCP or UDP, or
(b) encrypt the channel so that *all* traffic flowing through it is encrypted
by default. Again, the momentum is on option (b)--encrypting the channel by default
using protocols like IPSec at the network layer, (D)TLS at the transport layer,
and HTTPS at the application layer in the protocol stack. The DPRIVE IETF
working group is even pushing to encrypt all DNS traffic over TLS or DTLS depending
on the type of transport protocol used. 

Why am I bothering discussing any of this? I study security and privacy in CCN, a
network architecture in which the security model focuses on the data that is moved
over a network, rather than the medium through which it flows. If CCN is to have 
any hope of widespread adoption, then it must address the growing privacy 
paranoia that is sweeping the industry. However, it is unclear how CCN should address
the need for private communication. What does it mean to achieve privacy parity
with IP? 

Let's consider the IP definition for privacy. 

TODO: define it... and then say how we might get the same thing in CCN




The following table lists some plausible approaches.

|Approach|Implications|
|Encrypt all interest and content object exchanges using forward-secure encryption.|Opportunistic caching is no longer viable. Two interests for the same content will never yield the same name to result in a cache hit.|
|



