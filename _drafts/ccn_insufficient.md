---
layout: post
title: Addressing some ICN Fallacies
---

Lately I find myself strattling the boundary between industry product development
and academic research for CCN. And lately, while on that line, I have encountered more
and more people who misunderstand what it is that CCN brings to the table.
I often hear claims similar to, "just build the app
in CCN," and, "just send an interest and get the data." Simple, right? Or how
about, "routers will only serve *verified content* in response to an interest."
If you're famililar with the field, you've probably heard these too. And to the best
of my knowledge, there has not been a single piece of work among the myriad of
papers published on information-centric networking that has justified these claims.

There are two fundamental problems with these statements. Firstly, CCN (and ICNs)
in general try far too hard to fit a round peg in a square hole. In a way, CCN
is functionally equivalent to the HTTP GET method *at the network layer*. That's
great, considering that a great deal of traffic uses GET to acquire resources
from a server or through some API. However, I claim it is incomplete. And because
it is incomplete, some people are stuck in the mindset that we must live with
a networking architecture wherein the only primitive available to us is a GET.

Secondly, ubiqituous caching (or even caching at the edge) of *every* type of
content object (modulo policies applied based on popularity) will not always serve
the user. The CCNx protocol specification [xx] stipulates that a router shall not
respond with a cache content object unless it has been verified. This requirement
is, in my opinion, missing its target, which is to prevent content poisoning attacks.

In this post I will elaborate upon why I consider these topics to be problems
for CCN and I will explain what I think should be done to solve them.

# GET Real

In CCN, users and applications (consumers) send requests and receive responses.
Functionally, this is no different from the standard HTTP GET method, which
is inspired by the RESTful architecture first introduced in [xx]. REST is now
ubiquitious, and for a good reason: it's core features have straightforward
semantics and they are simple to use. For example, a GET request is expressed
via a message which contains a resource URL and the GET command. The entity
which serves this GET simply returns the resource data in an HTTP response
(or an appropriate error code if it can't be found).

[xx] REST thesis


# Caching Rules [!?]







CCN is the equivalent of HTTP GET, but what about the other methods? HEAD, POST, PUT, DELETE, CONNECT, CONNECT, OPTIONS, TRACE
Are we so naive that we think we can completely replace mutable resources with immutable content (POSTs are for mutations..., GETS with actions are side effects).
Mutabiliy is often needed
CCN should make mutability explicit and try to incorporate a POST-like command -- maybe these could also carry an address (pseudonym as in Bitcoin)
what are the semantics
