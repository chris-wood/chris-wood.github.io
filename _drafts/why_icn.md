---
layout: post
title: Why Bother with ICN?
---




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

