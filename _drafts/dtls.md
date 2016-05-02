---
layout: post
title: DTLS with OpenSSL
---

TLDR: I'm not an OpenSSL expert. I learn it as I go. And implementing DTLS required learning a lot about 
the library. This post tries to distill that knowledge and present running code that others may benefit from. 

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

There are three important constructs to use when adding DTLS support to your application using OpenSSL: 
SSL_CTX, SSL, and BIO. The SSL context structure contains all of the information used to execute a DTLS handshake
and create a session. For example, the selected set of ciphers is stored here along with the entity
identity information (e.g., the certificate and private key). The DTLS handshake is performed *in the
context* of this structure. So, in order to create a session, you must first configure your context as 
needed to get the intended result. In my minimal example, this means that you must minimally provide
the set of supported ciphers, public key certificate, and private key (used to sign handshake messages).

Beyond this basic functionality, the SSL_CTX API provides a much richer set of information to the client
to do things like manage trusted key stores, manage multiple sessions (within the same context), and configure
extensions such as the TLS server name. I believe the book (BULLET PROOF SSL) goes into this additional
functionality in more detail. I haven't gone through this book yet, but it's definitely in my queue.

Let's now look at the SSL structure. In essence, this is the concrete instance of a DTLS 
session that's given to the client (or callee). The client can use this API to connect to
or accept a connection from a peer. It also exposes the basic read and 
write operations needed to use the session to send data to and receive data from the peer. 
Again, for the purposes of getting off the ground, this is all you need to know. Controlling
protocol features such as timeout durations and retransmission policies is a topic for another
discussion.

The final piece of the puzzle is the BIO structure. This bridges the gap between the SSL API
and the SSL_CTX backend. The BIO structure is an I/O stream abstraction. It can use
memory or the network to perform I/O operations. According to the documentation [1], "it 
can transparently handle SSL connections, unencrypted network connections and file I/O."
There are two types of BIOs: source/sink and filter BIOs. A source/sink BIO involves some 
producer and consumer of data, e.g., as with a socket or file BIO. (Here we are concerned
with socket BIOs to enable network connectivity.) Basically, the BIO performs the I/O for
a DTLS session. The SSL API layers on top of the BIO structure as its I/O functionality
and uses it to drive the actual protocol, e.g., completion of the handshake, sending
and receiving packets, and sending signalling information. Thus, as the bridge between
the SSL_CTX state and the SSL API, a BIO must be created and wired to an SSL instance.

To sum up, to go from zero to a configured and connected DTLS sesssion, you must:

1. Create and configure a DTLS SSL_CTX.
2. Create a new BIO from the SSL_CTX.
3. Create an SSL instance from the DTLS BIO instance.

# DTLS Setup Code

Having described the major pieces of the puzzle, I'll now describe some of the code. 

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

