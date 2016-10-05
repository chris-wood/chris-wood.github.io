---
layout: post
title: The Linux Secure Sequence Generator
---

In [my last post](asd) I talked about TCP sequence number prediction attacks.
To recap, these are possible when an off-path adversary is able to correctly guess
the sequence number and connection tuple of an ongoing connection. The natural remedy
to this problem is to make the sequence numbers unpredictable for *every* packet
sent back and forth. I discussed this in my last post. However, the current remedy
is to only make the initial sequence number (ISN) unpredictable. The general approach
specified in RFC XXX is to set the ISN to be the output of a *keyed* PRF computed
over the connection tuple and some quickly evolving timer. The PRF key is generated
once for the system (see LINES XXX), but the "salt", or counter, changes over time
in a predictable fashion. 

In theory, this scheme is fine. A PRF $$F_k$$ is indistinguishable from truly random function $$f_r$$,
which have the property that, for a given input $$x$$, $$f_r(x)$$ is an element chosen uniformly
at random from the range. Thus, without knowledge of the key $$k$$, let alone the counter, an off-path 
adversary cannot compute the PRF and predict the ISN. 

But theory often differs from practice. Case in point: the Linux implementation uses MD5 with the 
secret-suffix method as its PRF. There's a wealth of information available to show that this
scheme is in fact *not* a secure MAC -- the primary reason being that MD5 is not collision 
resistant -- but is it okay as a PRF? Do the poor collision resistance properties of MD5 compromise
the security of the PRF? Take a look at the following ISN generation code extracted from version
4.7.4 of the Linux kernel [x].

{% gist XXX %}

XXX: explore this... talk with other security folks





XXX: extract MD5 code and compare cycles to SipHash


