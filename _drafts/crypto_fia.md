---
layout: post
title: Clean Slate Crypto for Future Network Architectures
---

The Internet Protocol is old and carries a lot of baggage. Nearly every
protocol in nearly every upper layer has IP at its heart: DNS, TLS, UDP, TCP, etc. 
The list goes on. Unlike IP, however, these protocols evolve over time.
As a necessary step for backwards compatibility, the "design delta" between 
successive versions of these protocols is often small. For example, the 
changes between TLS 1.2 and TLS 1.3 are mostly focused on the mechanics
of the key exchange and record protocol. But the principle remains the same:
a client and a server must derive a session key to encrypt traffic. The 
supported cryptographic algorithms to make this possible changed (for the better),
but the high-level design is, more or less, the same. 

The story is much different for future network architectures (FIAs). In
FIAs such as CCN and NDN, there's no baggage. They are fortunate enough
to start with a clean slate. And as part the design process, it is advantagous
to consider transferring state-of-the-art technology from academia to
industry. I am particularly interested in how we can use recent cryptographic
advances to provide security properties such as integrity, confidentiality,
privacy, and anonymity as *core network services* in CCN and other networks.
To drive this endeavor forward I was tasked with surveying the field for
a "crypto update" presentation at the Dagstuhl seminar on ICN security
and privacy. This post distills the major findings from that effort. I
plan on writing a more detailed report soon, time pending. 

# Integrity

TODO

# Anonymity

TODO

# Privacy

TODO

# Availability

TODO

