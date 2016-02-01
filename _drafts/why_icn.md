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

### TODO: internet tree picture here

Everyone knows that building functionality on top of abstraction and well-defined interfaces
is good practice for constructing complex systems. (What is software if not an abstraction
of the underlying hardware on which it runs?) And after 4 decades of intense growth,
development, and financial investment, we (the community) seem to have converged on the
IP model and all of the upper-layer abstractions it carries for connectivity as the de facto
standard by which users, devices, and applications become connected. Whether you like it
or not, we will be dependent on IP for the forseeable future. 

This is problematic for a variety of reasons. Firstly, the Internet is difficult to manage
and secure (at scale). Protocols and systems like DHCP, DNS, BGP, ICMP, etc. all exist because
we live in a world where volatile hosts can come and go as they please in the network (which,
in my opinion, should rightfully be the case). These technologies enable hosts to connect to
and use the Internet. Securing these endhosts is yet another problem -- an afterthought. 
Security is often tossed over the wall to application folks who are not best equipped to
adequately handle them. (This is not to say that networking people are security experts -- 
what I mean is that security is not something that should be so easily deferred to someone else.)
This is why many protocols like TCP and DNS are pushing for secure-by-default options manifested
in TCPcrypt [XX] and DNS-over-TLS [XX]. The pressures placed on these technologies is only exacerbated
as the traffic type, volume, and sources shifts based on emerging mobile and content-driven applications.

Another major problem with the TCP/IP model seems to be that it did not hit the right abstraction
necessary for today's applications. After all, the DNS exists because humans can't be expected to 
remember the IP address of a machine which can serve, for example, Netflix content. It's a workaround
for the name-to-content resolution strategy that's needed when fetching content from the network. 
The interaction between DNS and CDNs, which I briefly describe in CITE_BLOG_POST, is an indication that the 
mechanisms by which consumers fetch static content is not right. There are several layers of hooks and 
indirection needed to re-route a DNS query for a Netflix movie to a local CDN. This is needed because the 
only available abstraction is an IP address from which to fetch content. 

Today's networking technologies sedimented in place as applications were built on top of them and 
money was poured into the necessary infrastructure to support them. However, as applications changed, 
the underlying technologies remained static. It's hard to shape concrete after it dries. And given
everything we have observed in recent years, it may be time to break out the jackhammer and 
start from scratch (at least conceptually).

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

- describe goals of each of these applications
twitch.tv  
flickr
facebook
netflix
spotify

- describe how they rely on these protocols to do this
DNS
TCP
TLS
HTTP1/2
FTP

- describe the patterns that emerge (content delivery, DNS resolution and redirection, HTTP gets for mostly everything, etc)
- the standard pattern in software design is to recognize these patterns and the necessary abstractions they provide, and then
collate the functionality to provide this application
- this is what ICN research is doing: it's using application development (which is the most important since apps serve consumers and users)
to drive abstraction changes that are realized at the architectural level. Jeff Burke gave a great overview of how this is
happening at NDNComm last year. 
- a great contribution of ICN research is not necessarily on the protocol itself. Rather, it's on the
mentality of questioning the status quo of networking and rethinking how it could be done better.
- replace IP address as fundamental unit of communication -> move to named data as the unit of communication

## Laying the Foundation

- what does the network need to support different applications? (connectivity, end-to-end services, something like HTTP, and security)
- IP wasn't designed for security
- take what we know the internet is being used for today (and in the future) and build a better "fit" underneath that can support the use of the network and apps

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
