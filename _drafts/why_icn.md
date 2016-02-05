---
layout: post
title: Why Bother with ICN?
---

Many years of networking research and development have left us with the
ubiquitous TCP/IP networking model that (to my knowledge) supports the majority
of network applications in modern society. The fundamental design principal of this
model is based on layers (abstractions) starting with the physical device layer and
building towards the application layer. Each layer is responsible for providing
some unique set of functionality to the upper layer(s) based on the lower layer(s).
For example, in the OSI model [1], the link layer (2) is responsible for moving
frames between nodes on the same WAN or LAN and the transport layer (4) is responsible
for providing end-to-end communication services for applications [2].

Everyone knows that building functionality on top of abstractions and well-defined interfaces
is good practice for constructing complex systems. (What is software if not an abstraction
of the underlying hardware on which it runs?) And after four decades of intense growth,
development, and financial investment, we (the community) seem to have settled on the
IP model and all of the upper-layer abstractions it carries for connectivity as the de facto
standard by which users, devices, and applications become connected. Whether you like it
or not, we will be dependent on IP for the foreseeable future.

To some, this is problematic for a variety of reasons. Firstly, the Internet is difficult to manage
and secure at scale. Protocols and systems like DHCP, DNS, BGP, ICMP, etc. all exist because
we live in a world where volatile hosts can come and go as they please in the network (which,
in my opinion, should rightfully be the case). These technologies enable hosts to connect to
and use the Internet. Securing these end-hosts is yet another problem and has often been treated as an afterthought.
In fact, security is often tossed over the wall to application folks who are not best equipped to
adequately handle them. (This is not to say that networking people are security experts --
what I mean is that security is not something that we should so willingly defer to someone else.)
This is why many protocols like TCP and DNS are pushing for secure-by-default options manifested
in TCPcrypt [3] and DNS-over-TLS [4]. The pressures placed on these technologies is only exacerbated
as the traffic type, volume, and sources shifts based on emerging mobile and content-driven applications.

Another problem with the TCP/IP model seems to be that, in some cases, it did not hit the right abstraction
necessary for today's applications. After all, the DNS exists because humans can't be expected to
remember the IP address of the "best" machine which can serve, for example, Netflix content. It's a workaround
for the name-to-content resolution strategy that's needed when fetching content from the network.
The interaction between DNS and CDNs (which I try to summarize [here](http://chris-wood.github.io/CCN-vs-CDN/)) is an indication that the
mechanisms by which consumers fetch static content is not correct. There are several layers of hooks and
indirection needed to re-route a DNS query for a Netflix resource to a local CDN. This is needed because the
only available abstraction is an IP address from which to fetch content.

Today's networking technologies sedimented in place as applications were built on top of them and
money was poured into the necessary infrastructure to support them. However, applications have since
changed while the underlying technologies remained static. It's hard to shape concrete after it
dries. And given everything we have observed in recent years, it may be time to break out
the jackhammer and start from scratch (at least conceptually).

# What Does ICN Bring to the Table?

Information Centric Networking (ICN) is a breath of fresh air. To
some, it's a solution to the problems suffered by the
TCP/IP model. (I'll explain why below.) I was recently asked about why one
might bother investing in ICN and its related technologies,
e.g., CCN [5] and NDN [6]. I don't think off-the-cuff responses such as, "it enables better security,"
"it reduces network congestion," and "it supports better content distribution" give
justice to the years of research and development from the ICN community.
I think the reason is much more profound. To explain, I'll break down my reasoning
into two parts: (1) application-driven network designs and (2) re-distribution of
network-layer functionality.

## Top-Down Network Design

Consider some of the modern applications that many people (yourself probably included) use on a
daily basis: Flickr, Facebook, Netflix, Spotify, and Twitch. Each
of these massive systems have grown organically out of simple use cases. Flickr seeks to let
people share pictures with one another more easily. Facebook wants to help you "engage" in a social
life with others online. Spotify and Netflix want to stream entertainment in the form of audio and
video to you through any and every device you own. And finally, Twitch wants to let people stream their
(computer game) videos to the public. I will make no attempt to try to explain how these systems work,
but I will say that they share a common characteristic: they all deliver some static (immutable) content
(with some identifier) to consumers. (They differ greatly what that content is and how it is consumed, but bear with me.)
Granted, some content may have a degree of temporality (e.g., someone's home Facebook feed). But this
content is usually composed of immutable data underneath -- it's the bindings from identifiers to content
that changes.

What do these systems do to deliver content to their users as fast and efficiently as possible?
Generally speaking, they rely on a variety of standard protocols and technologies to make this
happen. This includes, in no particular order, TCP/UDP/QUIC, (D)TLS, HTTP, DNS, and CDNs.
Taken as a whole, these key pieces enable data to be easily transferred from one end-host to another.
By easy I mean that the API to obtain data is simple: an HTTP GET request.
The DNS is used to re-route these requests to an appropriate CDN node. Moreover, with the advent
of DNS-over-TLS, this will now require multiple round trips to resolve a name to an
address. Afterwards, the client device typically creates a (D)TLS session with the CDN node and then
uses TCP or UDP to send and receive data. (QUIC is used over UDP and is its own secure transport
protocol [7]). It would be hard to argue that his workflow is *not* incredibly common among modern
applications. Would you also agree that it isn't necessarily the most efficient? (Measuring
this is something I'd like to explore in a future post.)

One of the foundations of software engineering is the insatiable desire to recognize patterns
exhibited by software and to collate this functionality into a abstractions accessible
through some API(s). The TCP/IP network stack is a prime example of how these
types of abstractions can be layered upon one another to provide a simple interface
to complex machinery. Why then have we not found a better way to abstract the
basic pattern of fetching static content from a remote server?

This is what the ICN research community is doing: it's using application development to chisel
down into the network layer to see if maybe we can find a better set of abstractions for dealing
with *modern applications*. Currently, that abstraction is the form of named content rather than
addressable end-hosts in the network. The goal is to build on this abstraction to see if application
development can be simplified. Jeff Burke, a PI of the NDN project and professor at UCLA, gave a
great talk at NDNComm last year [8] about how applications can and should drive network
development. I highly recommend anyone who's interested in this topic to go back and listen to
or watch his presentation.

I'm not convinced that the great contribution of the ICN is the protocol itself. (To some,
it's nothing more than pushing a HTTP GET into the network). Rather, to me,
it's how we question the status quo of networking and security and seek out ways
in which how it could be done better.

## Retiling the Floor

What if we consider what the network provides from a bottom-up approach? Consider
the following layers in the TCP/IP stack and their roles:

- Layer 2 (link): hop-by-hop connectivity
- Layer 3 (network): end-to-end connectivity
- Layer 4 (transport): end-to-end communication services
- Layer 4+ (TLS): secure channels between two end points

As these layers grew, protocols and applications built on top became entrenched in
APIs that were host-centric. HTTP, for example, interacts with resources from a *specific server*.
It seems to me that the majority of HTTP requests are GETs, anyway, which are used,
as the name suggests, to obtain a specific resource from somewhere in the network. As this
use case increased and HTTP became the standard for web communication, the layers underneath
were almost set in stone.

But if we take a step back we can see that applications generally use HTTP to *get data*. To achieve
this, the client must be able to connect to and communicate with some host that has the data.
At a minimum, this means that the client must be able to talk to some adjacent machine
to perform communication so that this request can actually reach someone to answer it.
Thus, basic connectivity to other nodes in the network is a necessity. But
is organization of the rest of the stack a necessity? Beyond basic end-to-end connectivity,
the network stack must deal with security and privacy via TLS, DTLS, or QUIC, transporting data, and managing
session logic. (The need for security by default only recently became obvious in the network
stack. This is another reason why revisiting the underlying layers is important. The TCP/IP
model was not designed with security and privacy in mind.) These are necessary properties
of the layers between an application and the link layer in the stack, as summarized in the
following image.

[The traditional TCP/IP stack and the features it provides.](/images/posts/ccn_stack.png)

I would argue that it is not, however, mandatory for these features to be provided by the TCP/IP
abstraction. In other words, what if we could find a better way to accomplish these same things
without TCP or IP? What if we could rebuild these layers and their abstractions to achieve
the same functionality? This is another ambitious goal of ICN. It tries to reallocate the
functionality needed to *get data from the network* into sensible abstractions that better
suit today's applications.

[The proposed ICN stack and the features it provides.](/images/posts/ccn_stack_new.png)

ICN offers a compelling alternative to networking that might help us chip away at
the technical debt that has accrued as our dependence on the TCP/IP model grew.

# Open Research Problems

The obvious followup question to, "why should we bother with ICN?" is, "what are the problems yet
to be solved?". My role in this space lies in security and privacy. Unfortunately, I also think these are
given inadequate treatment among researchers who primarily focus on applications,
caching, and routing. Don't get me wrong. These are noble and necessary topics of research. However,
if the techniques they develop are ultimately insecure then all would have been for nothing.
Having said that, I'll now summarize what I believe are the most pressing open problems in
this field (in no particular order). I'll also provide related security questions that need to be
addressed in any viable solution. Maybe this can paint a picture of the future
of this vibrant area of research.

- *Naming and application namespace design*
  - How do applications get permission to publish and operate under a given name?
  - How are namespace collisions handled?
- *Trust management*
  - Can we break our dependence on a centralized or decentralized PKI?
  - How can we adopt distributed trust models (e.g., similar to that in Bitcoin)?
- *Encryption and privacy*
  - How can we enable object encryption without sacrificing privacy (as it is defined today)?
  - Can we build better broadcast encryption schemes that can handle the foreseen IoT scale?
- *Routing*
  - How do we create secure routing hints?
  - Can we decouple "application names" from "network names"?

# References

- [1] Briscoe, Neil. "Understanding the OSI 7-layer model." PC Network Advisor 120.2 (2000).
- [2] Kurose, J., and K. Ross. Computer Networking: A Top Down Approach, 4e. Vol. 1. 2012.
- [3] Mazieres, David, et al. "Cryptographic protection of TCP Streams (tcpcrypt)." (2014).
- [4] Zhu, Liang, et al. "Connection-oriented DNS to improve privacy and security." Security and Privacy (SP), 2015 IEEE Symposium on. IEEE, 2015.
- [5] [https://github.com/parc/ccnx_distillery](https://github.com/parc/ccnx_distillery)
- [6] [http://named-data.net](http://named-data.net)
- [7] [https://www.chromium.org/quic](https://www.chromium.org/quic)
- [8] [https://www.caida.org/workshops/ndn/1509/](https://www.caida.org/workshops/ndn/1509/)
