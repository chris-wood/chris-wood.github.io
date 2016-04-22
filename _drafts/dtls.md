---
layout: post
title: DTLS with OpenSSL
---

TLDR: I'm not an OpenSSL expert. I learn it as I go. And implementing DTLS required learning a lot about 
the library. This post tries to distill that knowledge in text and present running code that others may benefit from. 

# Pain Points

I personally find the set of available cryptographic libraries (modulo the [Python cryptography](LINK) library)
hard to use. The documentation is either outdated, obscure, arcane, or not in existence. I'm particularly 
frustrated with the state of OpenSSL. The APIs for encryption, hashing, signing (and verifying), and using
cryptographic protocols such as TLS and DTLS are neither clear nor intuitive. This bceame a problem when I needed
to implement DTLS support in our PARC's CCNx software. Since we already depend on OpenSSL and write everything
in C, there wasn't much wiggle room to try other libraries. So I embarked on the adventure of trying to get
a simple DTLS client and server working. This post describes what I learned on this journey and it ends with
a basic client and server. 

# Transport Security in OpenSSL

- context and SSL differences
- discussion of the dependency (ssl depends on ctx)

# The BIO I/O Interface

- what it's used for
- memory BIO vs generic descriptor BIO
- BIOs for contexts instead of SSL (why?) -- the other way around would seem more natural to me

# DTLS Setup Code

TODO: dtls.c

# DTLS Server

dlts_server.c

# DTLS Client

dtls_client.c

# Lessons Learned

My difficulty in finishing this task stemmed from my lack of knowledge about OpenSSL. Once you understand how
the pieces fit together, the code has a (relatively) logical flow and seems intuitive (at least to me). The problem
is, again, that the documentation and online resources were not useful references for getting started. I learned 
from several different websites, blog posts, and code snippets on my way towards the final result. Maybe the 
documentation and code samples will see some improvement in the future. But for now, the code does what I need. 

