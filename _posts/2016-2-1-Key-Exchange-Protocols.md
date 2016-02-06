---
layout: post
title: Comments on Key Exchange
comments: true
---

I recently sat through a paper presentation that described a key exchange protocol for vehicular
MANETs. The authors used a variant of group-based Diffie-Hellman (DH) [1] to derive a shared key
for a set of nodes. Every node generated a random element in the DH group, raised the
common generator to this element, and distributed the result to the rest of the group.
In the worst case, this is quadratic in the number of nodes in the group. One stressed
point was none of the key exchange messages were signed. Naturally, this raised the suspicion
of possible MitM attacks [2]. The "remedy" was to XOR the output of the DH protocol with a
pre-shared key that is installed on every node. This operated under the assumption that only legitimate
vehicles with this pre-shared key are able to compute this XOR operation and, therefore, derive the key.

This protocol (for two parties) can be sketched as follows (the specific details about how k_g
are derived are not available at present so I am writing it down as I interpreted it):

    Alice              Bob

     x1                x2
             g^x1
           -------->
             g^x2
           <-------

    y = (g^x2)^x1      y = (g^x1)^x2

      k_g = H(y) XOR k_p // k_p is the pre-shared key

The claim was that this deters MitM attacks since an adversary is not expected to
have the pre-shared key and can therefore not derive the group key (the
pre-shared key is never sent between nodes).

There are several problems with this solution. For starters, it forces groups to be static.
It is not possible to add members to the group that have not been previously given
the pre-shared key. Second, node compromise is fatal to the security of the entire group. If
a node is compromised and the pre-shared key is obtained or leaked then it is possible to
impersonate the victim or launch MitM attacks without prevention or detection. Group
communication basically halts once this occurs. (This is all the more
surprising given that the work was geared towards military applications.)
In a similar vein, revocation is not possible. Once a node is given the pre-shared key,
it can partake in any future communication by faithfully executing the key exchange
protocol. This is trivial since there is no node identification or authentication (ouch!).

In general, this protocol does not meet some of the most elementary requirements
for modern key exchange protocols. Using TLS 1.3 as an example, these include,
at a minimum:

1. Resistance to server-side DoS attacks
2. Support for (mutual) authentication
3. Support for session re-keying
4. Forward-secure session keys

The only properties that this protocol can claim to possess are (3) and (4) (if DH shares
are not kept after a re-key). Forward secrecy does not matter if there is no peer
authentication to prevent MitM attacks.

It is unclear if the group session keys derived reveal anything about the
application-layer protocol which uses them. However, in defense of the authors,
I doubt this is the case based on what I saw. Regardless, given its very rigid
nature, this protocol hardly seems useful in practice. To me, this is just
another reminder that "simple" and "efficient" shortcuts in cryptography are
either not free or don't prove to be overwhelmingly valuable.

## References

- [1] Bresson, Emmanuel, Olivier Chevassut, and David Pointcheval. "The Group Diffie-Hellman Problems." Selected areas in cryptography. Springer Berlin Heidelberg, 2003.
- [2] [Diffie–Hellman key exchange](https://en.wikipedia.org/wiki/Diffie–Hellman_key_exchange)
