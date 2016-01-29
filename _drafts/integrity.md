---
layout: post
title: Data Integrity and Trust for Networks
---

data integrity is a key concept in CCN -- the authenticity of every content object is rooted in a digital signature. 
consumers can get contnet that is signed or specify a content object with a hash restriction (only one will be returned and the hash will match)
from a simple view, CCN provides better "data integrity" than IP. 
but applications and users are the ones using data -- in IP, since IP doesn't provide integrity, upper-layer protocols do. IPSec or TLS.
but what do IPSec and TLS do? they create secure sessions over which content is retrieved and can be authenticated
so what do modern TCP/IP applications and CCN applications have in common: trust.
CCN simply makes the problem of trust more transparent to applications by pushing it down to a lower layer in the stack. 
at the end of the day, some trust model, whether it's a centralized PKI or distributed PGP-like scheme, is needed to enable or bootstrap authenticated data retrieval 
so, to look at the question again: does CCN fare better than IP when it comes to integrity. This is an incomplete question. perhaps a 
  better way to ask it would be: does CCN fare better than IP for providing applications with data integrity? And if that's the question
  then I would say no. Integrity is rooted in the same trust problem and CCN doesn't do anything new or novel here. 




