---
layout: post
title: Why Bother with ICN?
---

# Some History

Many, many years of networking research and development have left us with the
ubiquitious TCP/IP networking model that (to my knowledge) supports the majority
of network applications in modern society. The fundamental design principal of this
model is based on layers (abstractions) starting with the physical device layer and
building towards the application layer. Each layer is responsible for providing
additional functionality to the upper layer(s). For example, in the OSI model [1], the link layer (2) is
responsible for moving frames between nodes on the same WAN or LAN and the transport
layer (4) is responsible for providing end-to-end communication services for applications [2].

### TODO: tree picture here

Everyone knows that building functionality on top of abstraction and well-defined interfaces
is good practice for constructing complex systems. (What is software if not an abstraction
of the underlying hardware on which it runs?) And after 4 decades of intense growth,
development, and financial investment, we (the community) seem to have converged on the
IP model and all of the upper-layer abstractions it carries for connectivity as the de facto
standard by which users, devices, and applications become connected. For better or for worse,
we will be dependent on IP for the forseeable future.

TODO: show why it's not that great (pressure points on the IP model -- DNS hacks, security bandages, CDNs pooey, etc.)

# Why Bother with ICN?

-- TODO: link CCN/NDN

I was recently asked about why one might bother investing in ICN and its related technologies,
e.g., CCN and NDN.

TODO: CDNs and content distribution, maybe IoT as well
TODO; I think one of the biggest selling points is that it lets us re-think the abstractions we've been stuck with for the past 40 years.



-layering is good... but did we find the right abstractions?

- Why should industry consider finding funding dollars for ICN?
    - ability to re-think the network stack (IP with NATs emerged as an end-to-end protocol when applications and protocols were built on endpoints, not connectivity)
    - remember: networks are only needed because we want to transfer data from one machine to another. if we didn't need to get data from somewhere else or interact with a service, we would not need a network
    - TCP/IP protocol stack evolved organically: Ethernet let to the need for ARP, to grow we needed IP, IP led to the need for DHCP, to make e2e more useful we needed transport (TCP/UDP), and then we needed other application layer protocols.
    - ICN lets us toss out this stack and redistribute the necessary functionally in a more natural way.
    - required network and transport functionality:
        - connectivity for error- and modification-free data retreival
        - security and privacy of communication


--- TODO: insert the image of the stack re-shuffling here

- What do you consider the gaps/opens in ICN research?
    - security and privacy
    - broadcast encryption
    - distributed trust models (a la Bitcoin and PGP)

# References

- [1] Briscoe, Neil. "Understanding the OSI 7-layer model." PC Network Advisor 120.2 (2000).
- [2] Kurose, J., and K. Ross. Computer Networking: A Top Down Approach, 4e. Vol. 1. 2012.
