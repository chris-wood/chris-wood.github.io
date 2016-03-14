---
layout: post
title: A Python DSL for CCN
---

Most CCN applications I've seen or written so far go something like this:

1. Producers listen for interests with a particular prefix. Upon receipt
of such an interest, some callback function is invoked to generate or obtain
the data. The data is then put in the payload of a content object and sent
back to the network.
2. Consumers issue interests to the network for specific prefixes to get
data. Most of these calls are blocking.

This is a fairly common pattern so I decided to write a little framework 
to help hide the mundane details that go into structuring these applications.
The result -- [protoccn](link) -- is a Flask-like framework for creating 
producer and consumer CCN applications. In this post I'll outline some basic
examples of the framework to build a simple file server.

# Producer Framework

TODO

# Consumer Framework

TODO

# A Simple File Server

TODO

# References

- [1] Flask reference
