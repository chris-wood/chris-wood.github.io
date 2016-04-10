---
layout: post
title: Confidentiality in Information Centric Networking
---

Stephen Farrell gave a quick presentation on confidentiality and authorization at the last 
ICNRG interim meeting in Buenos Aires [1]. His talk was premised by the following assumptions:

1. ICN does not rely upon any form of host-based networking.
2. We don't use any new form of crypto. 
3. All caches are untrusted.

His informal conclusion was that confidentiality requires authorization and authorization
requires confidentiality. (A bit of a chicken and egg problem.) In other words, either 
confidentiality and authorization in ICN are impossible under the given assumptions or 
one of the assumptions must be violated to succeed. In all likelihood, the prohibition
of host-based networking would be the one that's tossed aside first. 

In this post, I will explore this seemingly unavoidable paradox. 

# The Chicken and the Egg

TODO: walk through his logic

TODO: crypto requirements:
    - consumer can only get data if they're authorized
    - two approaches (protect the request or the response):
        1. present identity and proof to party providing content
        2. embed access control in the response
    - option 1:
        - in order to check if a consumer is authorized, a consumer must identify themselves and present a proof (typical proof: signature and certificate)
            - for all existing crypto: if they have the right secret key, then this is possible
            - how do they get the secret key? must be transferred securely
    - option 2:
        - if AC is embedded in data, then authorization is based on the consumer's secret key(s
        - q: how do they get their secret keys?
            - all existing primitives require coordination or communication with some key manager
            - to transfer the secret key, they must be encrypted
            - transport encryption (here) requires client authentication

TODO: we can use existing primitives, but we cannot get away from *service* centric networking
    - intuitively: someone must be grant access to consumers (and must authenticate the consumer) at some point whether as a means to police requests or encrypt data for that person
        - authenticating a consumer requires that they present a certificate or some form of identity
        - someone must vouch for their identity... that continues recursively.
        - the voucher is also a service and the consumer somehow proves their identity
    - assumptions: consumers have installed trust anchors

# References

- [1] https://down.dsg.cs.tcd.ie/misc/confauth.pdf
