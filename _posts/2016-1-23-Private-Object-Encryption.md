---
layout: post
title: Private Object Encryption in Vanilla CCN is an Oxymoron
---

TL;DR: Interests are matched to content objects in CCN using exact match. If
two consumers ask for the *same thing* then they get back *the same bits*.
No matter how the content (response) is encrypted, it is trivial for an adversary to determine
if two consumers ask for the same content by simply examining the interest (request) contents.

# Private Object Encryption in Vanilla CCN is an Oxymoron

One of the many claimed benefits of opportunistic caching in CCN is that untrusted routers
can keep popular content close to consumers to reduce upstream congestion, decrease bandwidth
consumption, and improve the content retrieval latency. Router caches operate as
generic key-value stores where consumer interests (requests) serve as the keys and
the resultant content that is served are the values. As a key-value store, there
are two obvious yet important properties to consider. Firstly, if two consumers send a
request for the same key K then they get the same value V (i.e., the same bits are sent in
response). Routers do nothing to change the matching value before serving it to
consumers. Secondly, caching content is only useful if it can serve more than one
consumer. To make use of the cache, multiple consumers have to send requests for the
*same* key.

From a privacy perspective, this is not so great. It means that the requests and
responses of two separate users can be easily correlated. Whether or not the response
(content) is encrypted does not matter; two consumers who issue the same request will get
the same response. An eavesdropper can easily determine if two consumers request
the same content by comparing their requests (without looking at the response).
This simple request and response protocol means that "private object encryption"
is an oxymoron in vanilla CCN. In this post I will elaborate on this idea further.

To begin, let me first define what I mean by privacy. In the context of TLS,
privacy in the presence of untrusted network elements and eavesdroppers means
that, side channel attacks aside, it is not possible to correlate the
traffic of two separate users (consumers) even if they happen to be sending and
receiving the same application data bits between each other. From a cryptographic
perspective, this means that all traffic is encrypted with some form of CCA-secure
encryption. Roughly speaking, a CCA-secure encryption scheme is one wherein
two identical plaintexts will
always yield two distinct ciphertexts (except with probability negligible in
the amount of randomness added -- see [1] for more details). An equivalent
definition is to say that the encryption is semantically secure. This means that
the ciphertext leaks nothing about the plaintext.
Were traffic not private (i.e., not protected with CCA-secure encryption), then it
would be possible to determine when two identical messages were sent over the
wire by comparing the bits (ciphertexts).

With a privacy goal in mind, let's now revisit the standard assumptions in vanilla CCN.

1. Routers are untrusted. This means that any action or computation performed by
a router must be possible by an eavesdropping adversary.
2. Interest (key) to content object (value) matching is based on exact
match. That is, two interests will only yield the same Content Object
if they are identical.
3. Requests are issued for *one* value. That is, requests are not issued
for sets of content objects or for statistical properties of values.

Consumers retrieve data from the network using the standard request-response
protocol described above. Let's formalize this to better explain the privacy problem.
To send a request (interest) for some content with the name (key) *K*,
a consumer computes *X = Transform(K)* and sends the result to the network.
The Transform function maps the
application-layer key to some network-layer representation that can be sent
over the wire. In CCN, Transform is simply the identity function and
keys are tuples of the form (Name, KeyIdRestriction, ContentObjectHashRestriction).
To handle a request for a given transformed key *X*, a router computes an index
*I = Process(X)* and uses *I* to lookup the value in its key-value store (cache).
The Process function maps a network-layer key to some index for the cache.
Again, this is just the identity function for untrusted routers.
If index *I* is not in the cache then *X* is forwarded upstream. Taken together,
this composition of functions to extract a value *V* associated with a key *K*
is as follows:

```
V = Lookup(Process(Transform(K)))
```

Now, let R be a cache that has two key-value pairs (I1,V1) and (I2,V2) stored.
Moreover, assume I1 and I2 are two indexes that are associated with some key K1.
These two indexes were created by processing two requests X1 and X2.
The lookup procedure for each of which is shown below:

```
X1 = Transform(K1) => I1 = Process(X1) => V1 = Lookup(I1)
X2 = Transform(K1) => X2 = Process(X2) => V2 = Lookup(I2)
```

Assume, for the sake of contradiction, that these two requests are private
and cannot be correlated. Clearly, if both requests are for the key K1, then
it must be the case that V1 = V2. Therefore, the results can be immediately correlated
by comparing the values. To remedy this,
the router could use some form of re-randomizable encryption before responding
with the value (see [2] for an example of ElGamal-based re-randomizable encryption).
This works to prevent correlation by looking at *only* the response.
However, an eavesdropper should be assumed to be able to see both the requests and
responses. Under this assumption, I'll now argue why these two requests cannot
possibly be private. Consider the following facts about this pair of requests:

- Since any consumer must be able to send requests to any router, Transform must be a public function.
- Since R is an untrusted router, Process must be a public function.
- Lookup is a general function that returns the value associated with the input key. In CCN,
an input key matches one in the cache if they are identical (e.g., they have the same name).

The second point is perhaps the more important of the three. It means that an eavesdropping adversary can
compute the Process function for any request it observes on the wire. If a router is
to serve the same content for two separate requests, then the inputs to the Lookup
function must be identical. Therefore,
if an adversary can compute the Process function, it can also determine when any two
requests will yield the same response. Even if the transformed requests are different, i.e.,
X1 != X2, it must be the case that Process(X1) = Process(X2) if both X1 and X2 were
derived from K1. Therefore, these requests can easily be correlated.

# Takeaway and Future Directions

The problem (or benefit... depending on who you talk to) with vanilla CCN is that all
requests must be issued for one piece of content. This means that if caching is to be
useful (i.e., the same content will serve two or more consumers), then two consumers
must send the same request. No matter what type of encryption is applied to the content
object, privacy is not possible since the requests can be easily correlated.

There are at least two ways to solve this problem. First, requests can be generalized to
sets of content objects to give some measure of k-anonymity [3]. Second, we could look into
the work of private information retrieval (PIR) [4] and differential privacy [5]. Both
of these technologies enable privacy in scenarios that bear a slight resemblance to the
request-response protocol of CCN.

# A Comment on Transform and Process

The Transform and Process abstractions are somewhat useless in most cases since
they return whatever was passed in as a parameter (i.e., *K = Process(Transform(K))*).
However, they become effective when we consider two peers engaged in a
session. In this case, the Transform function might return an encrypted key
and Process might preform the inverse decryption procedure. Of course, Transform
and Process would no longer be public functions in a session scenario.

# References

- [1] Katz, Jonathan, and Yehuda Lindell. Introduction to modern cryptography. CRC Press, 2014.
- [2] Golle, Philippe, et al. "Universal re-encryption for mixnets." Topics in Cryptology–CT-RSA 2004. Springer Berlin Heidelberg, 2004. 163-178.
- [3] Sweeney, Latanya. "k-anonymity: A model for protecting privacy." International Journal of Uncertainty, Fuzziness and Knowledge-Based Systems 10.05 (2002): 557-570.
- [4] Ostrovsky, Rafail, and William E. Skeith III. "A survey of single-database private information retrieval: Techniques and applications." Public Key Cryptography–PKC 2007. Springer Berlin Heidelberg, 2007. 393-411.
- [5] Dwork, Cynthia. "Differential privacy." Encyclopedia of Cryptography and Security. Springer US, 2011. 338-340.
