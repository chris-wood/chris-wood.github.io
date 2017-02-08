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
can efficiently verify if desired. This provides extra assurance that certificates offered by TLS
servers are legitimate. It also allows server operators to detect when their identity has been
hijacked, modified, or otherwise tampered with based on the log contents. To do this, several pieces
are required.

1. Monitor:
2. Auditor:
3. Log server

# References

- [1] Merkle trees
