---
layout: post
title: DTLS with OpenSSL
---

TLDR: I'm not an OpenSSL expert. I learn it as I go. And implementing DTLS
required learning more about the library. This post tries to distill that
knowledge and present running code that others may benefit from.

# Pain Points

I personally find the set of available cryptographic libraries (modulo modern ones like the [Python cryptography](https://cryptography.io/en/latest/) library
and that provided by [Go](https://golang.org/pkg/crypto/)) hard to use. The documentation is either outdated, obscure, or inconsistent. I'm particularly
frustrated with the state of OpenSSL. The APIs for encryption, hashing, signing (and verifying), and using
cryptographic protocols such as TLS and DTLS are neither clear nor intuitive. This became a problem when I needed
to implement DTLS support in PARC's libraries and CCNx forwarder. Since we already depend on OpenSSL, write everything
in C, and we were bound by licensing issues, there wasn't much wiggle room to try other libraries. So I embarked on
the adventure of trying to get a simple DTLS client and server working. This post describes what I
learned on this journey and it ends with a basic client and server.

# Transport Security in OpenSSL

There are three important constructs to use when adding DTLS support to your application using OpenSSL:
SSL_CTX, SSL, and BIO. The SSL context (SSL_CTX) structure contains all of the information used to execute a DTLS handshake
and create a session. For example, the selected set of ciphers is stored here along with the entity
identity information (e.g., the certificate and private key). You may think of the DTLS handshake being performed *in the
context* of this structure. So, in order to create a session, you must first configure your context as
needed to get the intended result. That may include, for example, adding trusted key stores so that your client
(or server) can verify signatures and certificates of your peer(s). In my minimal example, this means
that you must at least provide the set of supported ciphers, public key certificate, and private key
(used to sign handshake messages).

The SSL_CTX API provides a much richer set of information to the user for
things like managing trusted key stores, multiple sessions (within the same context), and configuring
extensions such as the TLS server name. I believe the [Bulletproof SSL and TLS](https://www.feistyduck.com/books/bulletproof-ssl-and-tls/)
goes into this additional functionality in more detail, but I haven't read it yet to verify.
(It's still in my queue.)

Okay, so that's all we need to know about the SSL_CTX structure.
Let's now look at the SSL structure. In essence, this is the concrete instance of a DTLS
session that's given to the user. The caller can use this API to connect to
or accept a connection from a peer. It also exposes the basic read and
write (I/O) operations needed to use the session to send data to and receive data from the peer.
Again, for the purposes of getting off the ground, this is all you need to know. Controlling
protocol features such as timeout durations and retransmission policies are topics for another
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
and receiving packets, and sending signaling information. Thus, as the bridge between
the SSL_CTX state and the SSL API, a BIO must be created and wired to an SSL instance.

To sum up, to go from zero to a configured and connected DTLS session, you must:

1. Create and configure a DTLS SSL_CTX.
2. Create a BIO from the SSL_CTX to use for I/O (sending and receiving packets).
3. Create a SSL instance from the DTLS BIO instance to start the DTLS connection
and then interact with peer(s).

# DTLS Setup Code

Having described the major pieces of the puzzle, I'll now walk through the code.
Before we begin, though, recall that the SSL context structure requires some credentials
(a certificate and private key) in order to work. So we need to create some key stores.
To do this, run the following commands:

~~~
openssl req -x509 -newkey rsa:2048 -days 3650 -nodes \
    -keyout client-key.pem -out client-cert.pem

openssl req -x509 -newkey rsa:2048 -days 3650 -nodes \
    -keyout server-key.pem -out server-cert.pem
~~~~

This will create a client and server certificate and private key file. We'll use
these to create the DTLS context.

For convenience, we'll wrap up the DTLS relevant
information into a struct that stores the context, BIO, and SSL instance:

~~~
typedef struct {
    SSL_CTX *ctx;
    SSL *ssl;
    BIO *bio;
} DTLSParams;
~~~

With this definition, take a look at the following code.

{% gist bdbadae0bdec7f17ab68631cafe6c614 %}

This function initializes the DTLS context in a DTLSParams structure. It has
the flow that I described in the previous section. Specifically, it creates
the context for DTLS (line 7), sets up the supported ciphers (line 15),
loads the certificate with the public key (line 32), and then loads and verifies
the corresponding private key (lines 40 and 48). (The peer verification callback
setup in line 23 is just a placeholder for now. I don't make use of it yet.)

That's it. Upon successful completion of this function, we now have a minimally
configured DTLS context (SSL_CTX) that we will use to create the DTLS session.
The remaining steps in getting set up differ somewhat between the client and server.
So to keep things simple, let's start with the server (shown below).

{% gist 1a34c0b34b25cb494299bca4f468c9f2 %}

This function is straightforward. It creates the BIO from the SSL_CTX, creates the SSL
instance from the BIO, and then toggles the "accept state" of the SSL instance.
This is done to indicate that it will be used as a server-side interface.

Now let's review the client configuration (shown below).

{% gist 5cdb70abcddbdbd099d20831b0c99bc4 %}

This is almost identical to the server configuration code except that the BIO
(network I/O) instance is given the server address to establish communication and
the client enables the "SSL_MODE_AUTO_RETRY" flag. The latter option ensures that
I/O only completes and returns after a handshake is complete (i.e., the client
automatically "retries" to read or write data if the handshake fails or session is lost) [2].

At this point, we have configured our DTLS context for both the client and server.
All that remains is to actually implement the client and server functionality.

# DTLS Server

The server code can vary wildly depending on what you want to do. For this post,
I created a simple echo server that will accept connections from clients, read
an incoming message, write it back to the peer, and then close the connection.
This requires the server to have a socket upon which to listen for and
accept connections. Once that is setup, the rest is easy:

{% gist c622b6cf96a7b5638a90ae5a6374d1a2 %}

Lines 1-13 create the listening socket and configure the DTLS context using
the code presented in the previous section. The remaining code performs the echo
loop described above. There are two important lines of code to take into account.
First, line 30 will set the file descriptor of the SSL instance to the socket of a newly
accepted peer. This allows the SSL I/O operations to use that new socket. The second
is line 34, wherein the DTLS handshake is completed (via `SSL_accept`). Only if
this succeeds, based on the pre-configured DTLS context, will the server then
jump to the echo block from line 37 to 50.

# DTLS Client

The DTLS client code is almost as simple. The client must create its DTLS context,
connect to the server, and then perform its write and read commands. Once complete,
the program terminates. This is shown below.

{% gist 6ead7f43347c2619d5a0716008b0410d %}

Line 13 is where the client connects to the server using the previously configured
address (from line 8). (IP_PORT is a preprocessor macro set to "127.0.0.1:4433".)
The rest of the code should be easy to read. The client opens a file, sends the
first 4KB to the server, and then prints out the result echoed back from the server.

# Summing Up

The complete code for this example can be found [here on Github](https://github.com/chris-wood/dtls-test).
Overall, implementing DTLS support didn't turn out to be so bad. The main difficulties
stemmed from my lack of experience with OpenSSL. However, once you understand how
the pieces fit together, the code has a (relatively) logical flow and seems intuitive.
The problem is, again, that the documentation and online resources were not incredibly
useful references when getting started. I learned from several different websites, blog
posts, and code snippets on my way towards the final result. Maybe the documentation and code
samples will see some improvement in the future. But for now, the code does what I need.

## references
- [1] https://www.openssl.org/docs/manmaster/crypto/bio.html
- [2] https://www.openssl.org/docs/manmaster/ssl/SSL_get_mode.html
