---
layout: post
title: Private Service Discovery
---

I was recently using Wireshark [1] to debug a protocol issue with a CCN application
and stumbled across some interesting traffic coming from my machine. Every 30
seconds my box would broadcast a packet to the LAN (255.255.255.255) and get a
response back (from my own machine). This is part of the Dropbox LAN sync Discovery 
Protocol [2]. The contents of this packet are shown below. 

XXX: screen shot here

Inside the broadcast UDP packet is a JSON object which contains the following fields:
"host_int", "version", "displayname", "port", and "namespaces." The last one was 
particularly troubling. According to [3], namespaces are unique identifiers for the
roots or mount points of directories owned by a user. Files are then identified by
a namespace and the relative path. (In actuality, files are described in pieces by a 
collection of SHA-256 fingerprints of each piece. This lets us use a Merkle tree to 
determine when two files are identical and, when they are not, efficiently synchronize 
their contents.) 

According to [2], these discovery requests are broadcast to the LAN in hopes that
another machine which happens to own the same namespaces can be used to more efficiently
synchronize data. If another machine on the LAN owns the same namespaces then it responds.
(Or other machines always respond and the requestor computes the namespace intersection
to determine matching machines. Both of these are approaches are equivalent so the details
are not important.) The machines then synchronize their respective namespaces. If this 
does not happen, the machine pulls its data from the cloud. This is illustrated below.

XXX: image from dropbox site (with reference)

By now the alarm should be ringing if you've closely inspected these discovery packets.
They contain no form of authentication, i.e., no signature or MAC. What does this mean?
Anyone can advertise any namespace they want and initiate a synchronization event with
a victim machine. Consider the following attack: A passive eavesdropper listening 
for these discovery packets can collect all of the observed namespaces and then, in one
fowl swoop, start to echo them back in forged responses. This will cause the victim(s)
to try and synchronize the claimed namespaces with the attacker machine. The attacker
has, in effect, tried to hijack the namespaces. 

Fortunately, Dropbox vetts ownership of namespaces by distributing certificates 
(and private keys) to each client. To syncrhonize between machines, the responder
runs a LAN sync server to which the requestor connects over over TLS. Moreover,
in the initial handshake, the requestor specifies the SNI (Server Name Indication)
of the target to be the namespace to be synchronized. The SNI allows the server
to use the correct certificate when completing the TLS handshake, since the server
might have control over more than namespace and thus carry multiple certificates.
The TLS handshake will therefore only complete if the server can verify ownership of
the namespace private key. And once the TLS session is created, the client and server
run their synchronization protocol. The details of which are not important here.
 
I find several deficiencies with this approach:

1. If the client and server use the same certificate and private key XXXX

-- describe the replay attack, using vulnerable encryption mode (CBC), and ability to conduct padding oracle attack
-- describe the fixes
-- discuss privacy, mention stanford paper in brief


# References

- [1] wireshark
- [2] lan SYNC reference (https://www.dropbox.com/s/rs13ex5l0hlihhm/Screenshot%202016-09-12%2011.04.11.png?dl=0)

4. fixes:
    - try to connect and then blacklist source IP
    - sign announcements
5. what about privacy?
6. describe the stanford paper in brief, and provide a pointer to it

