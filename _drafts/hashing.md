---
layout: post
title: CCN Network Name Equivalence Classes
---

CCN has one deeply disturbing problem: application data names are required in
the network layer to forward packets. Why is this a problem, you ask? Well, 
for starters, application names, being often-overloaded URIs, can leak a significant
amount of information about the data being requested. Far beyond what's leaked from
an IP packet address and port tuple. (I've discussed this in a [past post](LINK HERE)).

There are at least two ways to solve this problem:

1. Wrap application names with encryption and carry them to their destination via
unrelated network names (or locators). This is basically the TOR equivalent of CCN
and has been around for some time [1]. 

2. Obfuscate application names to flat network names that can be compared to 
other network names without leaking the unobfuscated form. 

The latter might not be very clear, so let me give an example. Assume you are
a consumer and wish to obtain the data named /foo/bar. Moreover, assume there
existed the following set of functions at your disposal:

- Hash(N): Obfuscate an appliation name N to a network name N'. Also, it should
be the case (with overwhelming) probability that Hash(N) != Hash(N). (This means
that the output of Hash is inherently randomized.)
- Compare(N1', N2'): Compare two network names for equality. Return a 1 if
the underlying application names are equal and a 0 otherwise. 

Moreover, it should be difficult to learn the application name N' from its
network name N'. 

I claim that you can build the same packet forwarding logic using these
two functions. (Albeit at significant performance costs.) But why
would one bother? For one thing, the application name is no longer revealed
to the network. This lets forwarders act on names without having any
encryption context and without using encapsulation as in case #1 above.

From a theoretical perspective, such functions allow you to build 
name *equivalence classes*. That is, sets of different representations
for the same name. We would like these equivalence classes to be formed
such that, given two distinct names N1 and N2, the probability
that Hash(N1) is in the equivalence class of Hash(N2) is overwhelmingly 
(negligibly) small. 

# Equivalence Class Construction

I'll now present a trivial construction for these name equivalence classes.
The core idea is based on randomized hashing of the name and a deterministic
comparison function. To begin, 
let $N = N_1,\dots,N_k$ be a name of $k$ components that we want to map
to an element of its equivalence class. Next, let $G$ be a finite cyclic
group in which the discrete log problem is hard, and let $g \in G$ be a 
generator of this group. Assume that there is some entity responsible
for comparing obfuscated names. This entity is given a secret key 
$k \overset{\$}{\gets} G$ and the corresponding public key $p = g^k$. 
Using these parameters, the hash and compare functions can be instantiated
as follows.

- Hash(N): Split $N$ into $k$ segments. For each segment $N_i$, compute
$N_i'$ as the hash of the first $i$ segments. (Any cryptographic hash function
will work here.) Then, sample a random $s \overset{\$}{\gets} G$. For each
transformed segment $N_i'$, compute the two values $\alpha_i = p^s$ and $\beta_i = g^{N_i' + s}$,
where $+$ is element addition, not concatenation. Return the list of
$\alpha, \beta$ pairs $(\alpha_1,\beta_1), \dots, (\alpha_k, \beta_k)$.

- Compare($N_1$, $N_2$): Return False if the names are of different lengths. 
Let each segment $i$ in these names be denoted as $(\alpha_{i,1},\beta_{i,1})$
and $(\alpha_{i,2},\beta_{i,2})$. Using these values, compute the following:

$$
\begin{align}
p_1 & = (\beta_{i,1}^k)\alpha_{i,2} \\
p_2 & = (\beta_{i,2}^k)\alpha_{i,1}
\end{align}
$$

If $p_1 = p_2$, then it must be the case that $N_1$ and $N_2$ are in the same
equivalence class, i.e., they are "hashed" versions of the same name.

- TODO: show the proof here (simple derivation)
- present code
- discuss complexity and its problems...

# References

- [1] ANDANA paper

