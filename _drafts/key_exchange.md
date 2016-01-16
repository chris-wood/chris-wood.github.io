---
layout: post
title: Comments on Key Exchange
---

I sat through a paper presentation yesterday about a key exchange protocol for vehicular
MANETs. The authors used a variant of group-based Diffie Hellman (DH) to derive a shared key
for a set of nodes. Every node generated a random element in the DH group, raised the 
common generator to this element, and distributed the result to the rest of the group. 
In the worst case, this is quadratic in the number of nodes in the group. One stressed
point was none of the key exchange messages were signed. Naturally, this raised the question
of possible MitM attacks. The "remedy" was to XOR the output of the DH protocol with a
pre-shared key that is installed on every node. The protocol for two parties can be sketched
as follows (the specific details about how k_g are derived are not available at present so
I am writing it down as I interpreted it):

~~~
Alice              Bob

 x1                x2
         g^x1
       --------> 
         g^x2
       <-------

y = (g^x2)^x1      y = (g^x1)^x2

      k_g = H(y) XOR k_p // k_p is the pre-shared key
~~~

The claim was that this deter MitM attacks since an adversary is not 
expected to have the pre-shared key and can therefore 
not derive the group key (the pre-shared key is never sent over the air). 

Besides the fact that MANET key exchange is a solved problem [CITE EXAMPLES], there are
several problems with this approach. Firstly, it enforces node groups to be fixed or static. 
It is not possible to add members to the group that have not been previously given 
the pre-shared key. Second, compromise is fatal to the security of the entire scheme. If
a node is compromised and the pre-shared key is leaked then it is possible to launch MitM 
attacks without prevention or detection. Group communication basically halts once this occurs.
(Which is all the more surprising given that the work was geared towards military applications.)
In a similar vein, revocation is not possible. Once a node is given the pre-shared key, 
it can partake in any future communication by simply observing and deriving the key. The 
scheme is not forward secure. Recall that there is no node identification or authentication, 
so this is trivial.  

Overlooking these problems, the scheme actually suffers from a much more serious design flaw. 
The pre-shared key is repeatedly used to derive the group shared key. 

TODO: describe why this is bad... 



k1 = kp XOR H(ka)
k2 = kp XOR H(kb)

H(ka) = k1 XOR kp
H(kb) = k2 XOR kp

H(ka) XOR H(kb) = k1 XOR k2

goal: get k_p or forge a key



requirements: 
1. forward-secure
2. DoS resistant
3. efficient 
4. mutual authentication
5. support re-keying 

