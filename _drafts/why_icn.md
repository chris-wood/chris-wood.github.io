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

- quite difficult to manage and secure - kc claffy  https://vimeo.com/channels/ndnvfaq
- not designed for security (often "tossed over the wall" - david clark - to application folks)
- doesn't always hit the right abstractions -> DNS as a workaround for name-to-content resolution, CDN DNS interception for content location, etc.

TODO: show why it's not that great (pressure points on the IP model -- DNS hacks, security bandages, CDNs pooey, etc.)

# Why Bother with ICN?

-- TODO: link CCN/NDN

I was recently asked about why one might bother investing in ICN and its related technologies,
e.g., CCN and NDN. I don't think go-to responses such as, "it enables better security" (really, how?), 
"it reduces network congestion," and "it supports better content distribution" give 
any justice to the years of research and development from the ICN (CCN and NDN 
combined) community. I think the reason is much more profound and "ground breaking." 
To explain, I'll break down my answer into two different parts: (1) application-driven 
network designs and (2) re-distribution of network-layer functionality. 

## Top-Down Network Design


https://vimeo.com/channels/ndnvfaq/118965175 david clark MIT
- enables the network to be application-driven
- applications drive innovation, patterns emerge, and those are pulled into the underlying system (abstracted)

- refer to Jeff Burke's talk at NDNCom (it was great!) -- applications define how the network 
is to be used and the network shouldn't fight it. The network should aid modern application use
cases. 
- CDN is a good example here (its implementation has to fight and workaround the IP+DNS combo)
- CCN enables opportunistic, native CDNs (modulo security properties that modern CDNs given you)







## Laying the Foundation

- what does the network need to support different application?
- security through application design patterns
- take what we know the internet is being used for today (and in the future) and build a better "fit" underneath that can support the use of the network and apps
- replace IP address as fundamental unit of communication -> move to named data as the unit of communication
- IP wasn't designed for security
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

# Open Research Problems

The obvious followup question to, "why should we bother with ICN?" is, "what are the problems yet 
to be solved?". My role in this space comes from that of security and privacy. I think these are
given inadequate treatment among researchers in this space who primarily focus on applications, 
caching, and routing. Don't get my wrong. These are noble and necessary topics of research. However,
if the techniques they develop are ultimately insecure then all would have been for nothing. 
Having said that, I'll now summarize what I believe are the most pressing open problems in 
this field, in no particular order. I'll also provide related security questions that need to be
addressed to make this viable. 

- Naming and application namespace design
  - How do applications get permission to publish and operate under a given name?
  - How are namespace collisions handled?
- Trust management
  - Can we break our dependence on a centralized or decentralized PKI?
  - How can we adopt a distributed trust model (e.g., similar to that in Bitcoin)?
- Encryption and privacy
  - How can we enable object encryption without sacrificing privacy (as it is defined today)?
  - Can we make better broadcast encryption schemes that can handle the foreseen IoT scale?
- Routing and routing hints (a la NDN's Links)
  - How do we create secure routing hints?
  - Can we decouple "application names" from "network names"?

# A Comment on Application-Driven Design

- One problem is that not everything is or requires HTTP to operate. Many protocols operate without
HTTP request/response functionality and I see many people try to pigeonhole their applications to 
meet this need. 

# References

- [1] Briscoe, Neil. "Understanding the OSI 7-layer model." PC Network Advisor 120.2 (2000).
- [2] Kurose, J., and K. Ross. Computer Networking: A Top Down Approach, 4e. Vol. 1. 2012.
