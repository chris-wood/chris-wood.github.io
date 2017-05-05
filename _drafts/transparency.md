---
layout: post
title: Trust, But Verify
---

Made famous by President Ronald Reagan, the phrase “trust, but verify” highlights the limits to which we as humans trust others. In the most dire of circumstances, it is not enough to take someone by their word. For reasons beyond sanity, we must verify what is said against what is actually true. And to avoid any source of fraud or malice, verification requires transparency. In recent years, this has become a cornerstone for much of Internet security. It is the driving goal behind project such as [certificate transparency](certificate transparency) and [key transparency](key transparency). The motivation for such endeavors should be clear: we simply place too much trust in services such as CAs to back critical infrastructure components such as certificate issuance and public key binding. Certificate and key transparency both emerged as a system for enabling transparent verification of information contained in PKIs and key servers, respectively. This is an interesting paradigm shift with a great deal of momentum, driven in large part by the Googles of the world. Thus, given its rising importance, I set out to learn more about the technology underneath. And it all starts with a concept known as verifiable data structures.

# Merkle Trees and Verifiable Data Structures

Verifiable data structures are at the basis for most modern transparency systems. 
And Merkle trees give these data structures legs. A Merkle tree is a simple data
structure that produces an immutable identifier for a finite set of elements. It's 
constructed by creating a binary tree out of the set, where each leaf node corresponds
to an individual element and each non-leaf node corresponds to the hash of the children
nodes. Building this tree from the leaves up to the root yields a single hash that is
the identifier of the set. To add an element to this set, one simply appends another
leaf to the tree and re-computes all of the dependent parent nodes. As before, this percolates
up to the leaf and produces a new, distinct identifier. Verifying that an element is
part of this set is easy, too. One simply produces the element in question, the 
so-called "co-path" of hashes up the tree, and the identifier (root hash). The co-path 
is the minimal set of nodes that are needed to re-compute the root hash and verify its
correctness. A Merkle tree over 8 elements with a co-path highlighted is shown below.

XXX: merkle tree with highlighted co-path

The concept of a verifiable log emerges as a natural application of Merkle trees. 
A verifiable log is an append-only log which accumulates entries (elements of a set)
and provides a proof of its correctness. Here, correctness covers problems such as
inadvertant or malicious modification or removal of log entries. The log is built by
continually adding elements to a Merkle tree and producing an identifier that is
signed. Verifying the log involves recomputing the Merkle tree hash, checking it for
equality, and verifying the signature. Thus, as a consequence, any user can enumerate
every item in the log.

A verifiable map is a similar data structure. It is a tree with $$2^{256}$$ (possibly empty) 
nodes. A key-value pair is added to the map by first computing a 256-bit cryptographic
hash digest of the key. The resultant value is interpreted as a 32-byte leaf identifier. 
The value is then associated with this leaf, and the parent nodes are updated as needed.
To avoid $$2^{256}$$ work for every insertion, the authors of [x] make the observation 
that empty leaves contain empty values. Thus, the hash of these leaves (and nodes) is
the hash of the empty string. Depending on the level of the inner leaf, the desired hash
value can therefore be efficiently computed, as shown below.

XXX: sparse merkle tree of 8 nodes with one entry, the others computed as iterative hashes of 0 strings

[x] https://github.com/google/trillian/blob/master/docs/VerifiableDataStructures.pdf
[x] http://sump2.links.org/files/RevocationTransparency.pdf

XXX: code for the verifiable map

The final verifiable data structure we consider is a verifiable log-backed map. The concept
is simple: a verifiable log records all of the in-order operations made to the verifiable map.
By parsing the log and performing the same elements on the map, one can reconstruct the map's
state. This data structure has the benefit of verifiable key-value pairs, but also of consistency
guarantees provided by the underlying log.

XXX: code for the verifiable log-based map

# Certificate Transparency

The Certificate Transparency project is built on the concept of these verifiable data structures, but
at a (to-be) massive scale. The goal of the project is to build a transparent PKI that clients
can efficiently verify, if so desired. This provides extra assurance that certificates offered by TLS
servers are legitimate and are not being offered up by a MITM attacker. 
It also allows server operators to detect when their identity has been
hijacked, modified, or otherwise tampered with based on the log contents. 

Before describing the CT ecosystem, first recall how standard PKIs operate. Certificate Authorities (CAs)
distribute (for profit) certificates to domain owners after vetting their identity. Domain owners then use
these certificates that chain up to a trusted CA root to establish TLS connections with clients. The general
MITM attack on this model is for adversary to intercept traffic between a victim client and server to offer
up a fake and fraudulently-obtained certificate to the client. Success requires that the client trust this
fake certificate, so it must come from a (possibly compromised) CA. This is not much of a stretch. Once complete,
the MITM can then establish a TLS session with the real server and marshall data back and forth unbeknownst to
either party. 

One (shaky) goal of CT is to detect such attacks *after they've already occurred*. To do this, it relies on a "simple"
system of checks and balances backed by verifiable data structures. In general, there exists a verifiable log
of certificates that can be audited for malicious behavior. For example, if a CA was duped and inadvertantly issued
a certificate to an attacker, the rightful domain owner can, in theory, detect this certificate in the log and take
corrective action. Collectively, this policy is realized with the following major pieces [CT-website]:

1. Monitor: An entity that watches out for suspicious behavior or certificates in logs. It is expected that most monitors will be operated by CAs, since it is within their best interest to track misissuance of certificates under the auspices of their identity.
2. Auditor: An entity that periodically checks the health of "the" log. Since there are several monitors and logs in flight, CT suggests the use of a gossip protocol to ensure that everyone is inspecting the same log. If an auditor detects a problem with a log, it's reported. 
3. Log server: An entity that collects issued certificates in an append-only, verifiable log. 

The system of checks and balances between these three entities is shown in the figure below.

XXX: figure of CT elements

CT is [far from perfect](https://mailarchive.ietf.org/arch/msg/trans/gO_DFW3v9FmBCOek_hifZ6KL368). 
Among its problems is the fact that logs and monitors are equally likely to
be compromised as today's PKI elements. (So really, what's the point?)

XXX: richard's comments here

# Key Transparency

XXX

# Trillian

XXX

# References

- [1] Merkle trees
