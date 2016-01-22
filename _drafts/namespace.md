---
layout: post
title: The Names of Things
---

1. naming things is non-trivial and hard to do securely (most not realize there is an element of security related to it)
2. in ccn, names are used for routing based on LPM. some minimal prefix is necessary to route an interest to an authoritative producer. the rest can be encrypted using some semantically-secure scheme (plaintext reveals nothing of the ciphertext)
3. the prefix itself can be sufficiently revealing: /edu/uci/ics/cwood/files/pictures/file{1-5}. To reach my machine, one must send prefix up to cwood... it could be easy to deduce what the underlying plaintext is based on knowledge of the prefix. 
4. this is not like IP where an address can tell you what type of content is being served -- a single machine can serve many different applications and content. in CCN, the name is more specific and therefore reveals more (it's kind of like shifting information from the payload to the address), and that has security implications. 
5. if we want to not reveal a lot in the name, then the prefix should be short or the prefix shouldn not be indicative of the content. what is the result? a prefix that identifies a *service* to generate some data, rather than a name that identifies a specific piece of data. 
6. there's a tradeoff though: less specific names mean that the target service must provide more content... more specific names reveal more about the server and the content that's generated. 
7. maybe we need to be looking into differential privacy and PIR -- we shouldn't ignore years of research in these fields.


