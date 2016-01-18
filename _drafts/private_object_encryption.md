---
layout: post
title: The Impossibility of Private Object Encryption in Vanilla CCN
---

Caching and privacy are in a constant struggle in CCN. One of the many claimed
benefits of opportunistic caching in CCN is that untrusted routers can keep popular
content close to consumers to reduce upstream congestion, decrease bandwidth 
consumption, and improve the content retrieval latency. Router caches operate as
generic key-value stores; consumer requests serve as the keys into the cache and
the resultant content that is served are the values. As a key-value store, there 
are two obvious yet important properties to consider. If two consumers send a 
request for the same key, they get the same value (i.e., the same bits are sent in 
response). Routers do nothing to change the matching value before serving it to
consumers. Secondly, caching content is only useful if it can serve more than one
consumer. To make use of the cache, consumers have to send requests for the *same* 
key. 

From a privacy perspective, this is not so great. It means that the requests of 
two separate users can be easily correlated. Whether or not the response (content)
is encrypted does not matter: two consumers who issue the same request will get 
the same response and an eavesdropper can easily determine if two consumers request
the same content by comparing their requests (without looking at the response). 
Following this train of thought, it is not hard to show that in fact the notion
of "private object encryption" is not possible in vanilla CCN. 

To prove this, let's first define what we mean by privacy. In the context of TLS,
privacy in the presence of untrusted network elements and eavesdroppers means 
that (side channel attacks aside) it is not possible to correlate the 
traffic of two separate users (consumers) even if they happen to be sending and 
receiving the same bits over the wire. Technically, this means that all traffic
is protected by CCA-secure encryption wherein two identical plaintexts will
always yield two distinct ciphertexts (except with probability negligible in
the length of the amount of randomness added -- see [1] for more details). Were
traffic not private (i.e., not protected with CCA-secure encryption), then it 
would be possible to determine when two identical messages were sent over the 
wire by comparing the bits (ciphertexts).

Now, to continue the proof, let's make the following assumptions:

1. Routers are untrusted. This means that any action or computation performed by 
a router must be possible by an eavesdropping adversary. 
2. Interest (key) to Content Object (value) matching is based on exact
match. 
3. Requests are issued for *one* value. That is, requests are not issued
for sets of content objects or for statistical properties of values. 

To send a request for a given key K, a consumer computes X = Transform(K) 
and sends the result to the network. This maps the application-layer 
key to some network-layer representation that can be sent over the wire. 
In CCN, Transform is simply the identity function. To process a request
for a given transformed key X, a router processes the key and uses it 
to lookup the value in its key-value cache. Together, request-response
protocol for a given key K and value V is captured in the following:

```
V = Lookup(Process(Transform(K))) 
```

Let R be a cache that has two key-value pairs (X1,V1) and (X2,V2) stored. 
Let X1 and X2 be two requests for K1 that arrive at router R. The lookup for 
both keys will be done as follows:

```
X1 => Lookup(Process(Transform(K1))) = V1
X2 => Lookup(Process(Transform(K1))) = V1
```
Assume, for the sake of contradiction, that these two requests are private
and not correlatable. Clearly, both requests will yield the same value V1. 
In doing so, the requests are immediately correlatable. To remedy this,
the router may use some form of re-randomizable encryption before responding
with the value (see [2] for an example of ElGamal-based re-randomizable encryption).
However, we can easily show that these two requests cannot possibly be
private just by examining the requests. 

But first, there are several things to consider about this pair of requests:

- Since any consumer must be able to send requests to any router, Transform must be a public function.
- Since R is an untrusted router, Process must also be a public function. 
- Lookup is a general function that returns the value associated with the input key. In CCN,
an input key matches one in the cache if they are identical (e.g., they have the same name). 

The second point is the most important. It means that an eavesdropping adversary can 
compute the Process function for any request it observes on the wire. If a router is
to serve the same content for two separate requests, then the inputs to the Lookup
function must be identical (CCN matches requests to responses using exact match). Therefore,
if an adversary can compute the Process function, it can easily determine when any two
requests will yield the same response. Even if the requests are different, i.e., 
X1 != X2, it must be the case that Process(X1) = Process(X2) since X1 = Transform(K1)
and X2 = Transform(K1). Therefore, these requests can be correlated, which is
a contradiction.

# Takeaway 

This problem (or benefit, depending on who you talk to) with vanilla CCN is that all
requests must be issued for one piece of content. This means that if caching is to be
useful, then two consumers must send the same request. No matter what type of 
encryption is applied to the content object, privacy is not possible since requests
can be correlated. 

There are at least two ways to solve this problem. First, requests can be generalized to
sets of content objects to give some measure of k-anonymity. Second, we could look into
the work of private information retrieval (PIR) and differential privacy. 

# References

[1] - CCA-secure reference
[2] - mixnets reencryption
[3] PIR
[4] differential privacy
