---
layout: post
title: CCN vs CDN -- What is the Difference?
---

As a researcher and developer focusing on CCN and CCNx, one question I am often asked is, "What is
the difference between CCN and CDNs like Akamai and Limelight?" This is a valid question, and one that we (myself
included) as a community tend to dance around. And while a blog post probably isn't the place to address such
an important issue, something needs to be written down. This is my attempt to fill the void.

For those unfamiliar with Akamai, it is an enterprise content delivery network (CDN) used by media and 
e-commerce companies such as, or similar to, Netflix, Amazon, and Facebook for *bringing content
close to the consumer*. In this context, a consumer is any end-host that will use data provided or
served by CDNs on behalf of their original providers (e.g., Netflix). Examples of end-hosts include
personal computers, mobile phones, and gaming devices. One of the primary reasons that CDNs such as
Akamai exist is that consumers want low latency when fetching content, whether it's the latest 
episode of Parks and Recreation on Netflix or their favorite new song from iTunes. Before CDNs,
consumer requests for such content would have to complete a full round trip from the source to
the location (server) where the content is server. This adds considerable latency for just a single
packet, not to mention the fact that it usually requires multiple round-trip exchanges in order 
to retrieve content, especially if it is to be encrypted through an IPSec or TLS tunnel. Furthermore,
it forces *all packets* for *all content* served by the producer, e.g., Netflix, to go to a single
server. There are numerous problems with this arrangement, such as:

- Increased difficulty to provide QoS guarantees for all clients, which ultimately results from an increase in incoming traffic
- Decreased probability of availability due to network outages or (D)DoS attacks
- etc.

This type of scenario, wherein a single server is used to provide (serve) content to all possible consumers is depicted below.

![Traditional client-server model for content distribution.](/images/posts/ccn_vs_cdn-figure1.png)

CDNs came into play to remedy the many problems listed above. Put simply, they are glorified caches that
producers like Netflix pay to host content on their behalf. To distribute traffic loads, reduce the RTT
latency for retrieving content, and increase the overall user experience for clients, producers will
*proactively* insert content into CDN caches that are geographically distributed around the globe. Then,
when requesting content, consumers will fetch it from their nearest CDN server (cache), not the original
producer. This is shown below.

![Over-simplified view of a modern CDN network. Highlighted nodes are CDN servers (caches) which replicate content from the original server for clients in their local proximity.](/images/posts/ccn_vs_cdn-figure2.png)

The nodes near the consumers are commonly called *edge caches*, since they cache content close to consumers,
or, rather, at the "edge" of the network. This type of deployment is claimed to offer the following 
benefits to its paying subscribers (see [this document](http://www.akamai.com/dl/feature_sheets/fs_edgesuite_securecontentdelivery.pdf)):

- Security
- Increased performance
- Reduced (operating) cost and complexity
- Unlimited scalability

With the ability to launch a new node in virtually any geographical region in the world (pending those with
legal restriction), the last two properties are not difficult to observe. However, the first two properties 
require a bit more discussion. However, to understand how these work, it's important to understand
how content is delivered via the CDN network. For the rest of this post, I'll restrict frame my discussion
in the context of Akamai; other CDNs surely have similar designs with almost identical functionality. 

Consider what happens when a consumer (using a laptop) issues a request for the 
resource http://www.netflix.com/WiPlayer?movieid=123. The consumer's browser first
contacts its local DNS resolver to map the target www.netflix.com to a server's IP 
address so that it can subsequently ask for the content. Part of Akamai's service includes
a _Request Router_, which is a DNS-based proxy that redirects such DNS queries to appropriate
edge caches. The Request Router takes into account factors other than geographic location of
the consumer (which is identified by the query's source IP address); it considers the load
of existing edge cache servers as well, and probably some other proprietary selection 
criteria. After an appropriate edge cache is identified, the Request Router returns its corresponding
IP address to the client's local resolver, and the consumer's browser subsequently contacts
the edge cache for the content. To the browser, there is no difference between fetching 
content from this cache or from the original producer; the same protocols are used either way.
If the edge cache needs to contact the original producer for any reason, e.g., if the client's
identity needs to be verified and its access to the requested content needs to be authorized,
the edge cache server will contact the appropriate producer server to fetch this information
or perform these checks. For Akamai, these requests are often served by what's called the
HyperCache -- a "common HTTP caching infrastructure for operator, operator customer, and 
OTT content." There is one additional component called the Intercept Service that, based
on its name, intercepts and redirects HTTP traffic to support caching. 

When content security (confidentiality) is important, most CDNs use SSL/TLS to encrypt 
content while in transit from edge caches to the consumers. However, since secure information
*never* resides within the Akamai network, edge caches must contact the original producer 
for this content so that it can be securely relayed to the consumer. This is commonly done to
transfer content encryption keys used to access static media hosted in CDNs. For example,
a producer might encrypt sensitive content using a random symmetric key and then choose to host
this (possibly large) content in a CDN network to make it quickly available to consumers. 
If consumers then wish to access this content, they must obtain the appropriate decryption key
from the producer, either through the edge cache servers or by contacting the producer directly. 
This distributed arrangement is what's employed in many modern DRM technologies like Microsoft
Playready, which is used by Silverlight and, thus, Netflix.

Anyone with a background in computer science or a related field should understand that
caching is generally useful to reduce access times, and thus, CDNs provide increased
performance and QoS guarantees over the traditional single-server alternatives. The general
idea is to bring content closer to consumers for quicker access. Simple enough.

Okay, that's enough of CDNs for a while. Let's change gears and talk about CCN and related
information-centric networking architectures such as Named Data Networking (NDN). In contrast
to today's IP-based networks, content, rather than hosts or addresses, is named and transferred
throughout the network. For example, the example Netflix movie above might have the same name,
/www.netflix.com/WiPlayer?movieid=123. To retrieve this content, a consumer would issue a
request, or an interest, for the desired content that carries *its name*. The network would
then route this request to the producer or any entity that can "satisfy" the interest with
the expected content object. Since content is identified by name, not by location, this means
that it can be opportunistically cached anywhere at the network. In fact, in CCN, routers
are expected to have caches to store content so that they may satisfy future interests with 
their cached content instead of forwarding it upstream. To route interests, routers use what's called
a Forwarding Information Base (FIB), which is just a routing table that maps *names* to *interfaces*.
This routing table is populated using standard routing protocols, or they can be configured manually.
Upon receiving an interest, a router first checks to see if the content is contained in the cache --
by looking for a match with the same name -- and, if not, it looks up the next hop using 
longest-prefix matching on the name in the FIB. The astute reader might observe that this might
mean that multiple interests for the same missing content are forwarded by a single router in a short
time window, thereby leading upstream congestion in the network. To remove this problem, all
routers contains a Pending Interest Table (PIT), which is a table that stores (a) the names
of outstanding (pending) interests that have been forwarded but not yet satisfied, (b), the interfaces
upon which interests for this content were received at the router, and (c) some additional metadata. 
The metadata is not important for this discussion, but I encourage more interested readers to
read [this paper](LINK) for more details.

Okay, so the general idea is to name content, move it around the network *upon request from consumers*, 
and opportunistically cache it where needed. Simple enough. The following images illustrate this type
of communication model.

![Step 1](/images/posts/ccn_img1.png) 
![Step 2](/images/posts/ccn_img2.png) 
![Step 3](/images/posts/ccn_img3.png) 
![Step 4](/images/posts/ccn_img4.png) 

CCN's name-based retrieval process means that content is no longer accessed in a client-to-server fashion.
Content may be served from any router's cache. Consequently, it's important that each consumer is able
to verify the authenticity of the content it receives. To do this, all content is signed (there are
exceptions, but they're not important here) by its producer. This enables consumers to verify the
signature before actually using the content. I'll skip over the details of fetching, verifying, and 
actually trusting the public keys used to perform this signature verification. For now, it's just important
to know that there exists a standard mechanism by which content authenticity can be ascertained, and
it doesn't require online interaction with the producer. Also, since content is no longer delivered
in a client-to-server channel, standard techniques for securing sensitive content do not 
apply (in a straightforward fashion). Rather, a more natural way to protect content is to simply 
_encrypt it_, and then then distribute the keys in an offline or secure fashion. This is exactly
what we discussed with Microsoft PlayReady. In fact, there are numerous research papers exploring
application and infrastructure support for this type of encryption. 

We now have a sufficiently complete description for both CDNs and CCN. So, what's the difference? 
Of course, both CDN networks and CCN cache content, and that's the primary benefit of CCN, right?
Wrong. There is significantly more to CCN than meets the eye, and much more than modern CDNs provide.
For brevity, I'll simply enumerate them below.

* Active and intelligent forwarding strategies for routers.
* Publisher mobility is easily supported via CCN routing protocols. 
* Congestion control can be enforced *within the network*.
* The existing (and problematic) IP stack and accompanying can be completely replaced with a new set of layers.
* Existing APIs can be completely reworked to focus on *content*, not on addresses.
* Content security is not tied to the channel, but to the content itself. 

Put simply, CCN provides "native" CDN support for completely opportunistiy caching, i.e., content
popularity does not have to be predited beforehand, all while providing the benefits above that
are not provided by IP-based CDNs. 

I hope this helps others who ask themselves the same question. 

