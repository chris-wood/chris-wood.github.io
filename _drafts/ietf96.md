---
layout: post
title: IETF 96 Highlights
---

# Highlights

I was at the 96th IETF meeting in Berlin this week. Overall, it was very productive. 
Outside of my ICN obligations, my favorite sessions were the TLS, TCPINC, and LURK
working groups. The TLS protocol seems to be simplifying and converging towards a stable 
design. There are several independent and interoperable (to some extent) implementations.
The TCP ENO protocol is getting polished and should be going through last call
soon. Even better, tcpcrypt went through very little design changes (according to
Andrea Bittau). That was encouraging to hear. Time pending, I want to capitalize on its 
stability and try my own implementation for MirageOS. 

In contrast to the others,
LURK was fairly uneventful. There was not a great deal of traction on the mailing list
since the Buenos Aires meeting. Nor were there many other use cases for delegated private
key access outside of de facto CDN scenario. Most of the talks did not rely on any 
fancy crypto or new protocols. They basically consisted of a lazy-loading certificate
fetch RESTful protocol. Basically, the CDN edge servers would contact the origin 
if and when a new certificate was needed. The lifetime of this certificate is bounded,
of course. Also, importantly, these protocols do not use the origin as a signing oracle
for TLS connections. (That would be a nightmare and basically defeat the purpose of LURK.) 

I'm sad to say that I missed both of the incredibly popular BOFs on the topic -- QUIC
and PLUS. According to other attendees, there were great talks about how QUIC is being
used at places such as Google and Akamai. There were also talks about the actual implementation.
(This is something I'd love to implement if I had more time. Maybe on vacation in August?)
As far as I know, the PLUS (Path-Layer UDP Subtstrate) BOF focused on trying to scope
a shim layer on top of UDP that would enable transport protocols such as QUIC to be built.
I assume the expectation is that QUIC would adopt whatever comes from the PLUS WG (assuming
something does form) and that other similar transport protocols would sprout as a 
result. That's pretty exciting since it would bring transport protocols up to the
user space and allow people to easily experiment with different approaches. 

There was also a lot of talk about these UDP-based transport protocols in the plenary talk.
Someone from Apple made the comment that the use of UDP-based protocols was not for NAT and
firewall traversal; rather, it was to allow for traffic to de-multiplexed at above the network
layer (via ports). IP could have been designed to do this, but it wasn't, so we rely on UDP.
Phillip Hallam-Baker also made the comment that people tend ot be focusing too much on
middlebox support for network protocols. Basically, who gives a f*&k that middleboxes break
with end-to-end encryption; the goal of such protocols is to support applications, not 
middleboxes. Basically, he was pleading with the group for a change of perspective to focus
on how protocols can empower applications, not empower middleboxes or things beneath the applications.

# ICNRG Notes

The main purpose of the trip was to attend the ICNRG meeting. Marc Mosko and I gave updates
about the CCNx specification documents that were well received -- even though not many
folks actually read the drafts. I will need to circulate a plea for support on the mailing list 
soon after I return home. Then there were talks about ICN and IoT. (Personally, I don't find
this an interesting topic of research. But that's not to say it isn't important.) Finally,
we concluded with a discussion of the BOF proposal. It seems that people would be in favor of
a BOF. We simply (?!) need to hash out details surrounding security, IPR, and community consensus
before proceeding with the second proposal. 

I took notes during the session. If you're interested in reading them, you can find them
[on the ICNRG site here](LINK HERE). 

# Berlin Adventures

Of course, I wandered around town and did some sight seeing. Berlin is a beautiful city, 
though not as a medeival as Paris. (Of course, Paris didn't suffer from being blown
half to hell during the war.) Rather than write up my adventures here, the picture below
captures the highlights. (The Last Kremlin Flag was awesome!)

## TODO: picture here...

