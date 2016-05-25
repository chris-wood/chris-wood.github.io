---
layout: post
title: CCN Network Name Equivalence Classes
---

CCN has one deeply disturbing problem: application data names are required in
the network layer to forward packets. Why is this a problem, you ask? Well, 
for starters, application names, being often-overloaded URIs, can leak a significant
amount of information about the data being requested. Far beyond what's leaked from
an IP packet address and port tuple. (I've discussed this in a [past post](http://chris-wood.github.io/2016/03/04/Naming.html)).

There are at least two ways to solve this problem:

1. Wrap application names with encryption and carry them to their destination via
unrelated network names (or locators). This is basically the TOR [1] equivalent 
of CCN and has been around for some time [2]. 

2. Obfuscate application names to flat network names that can be compared to 
other network names without leaking the unobfuscated form. 

The latter might not be very clear, so let me give an example. Assume you are
a consumer and wish to obtain the data named /foo/bar. Moreover, assume there
existed the following set of functions at your disposal:

- Hash($N$): Obfuscate an appliation name $N$ to a network name $N$'. Also, it should
be the case (with overwhelming) probability that Hash($N$) != Hash($N$). (This means
that the output of Hash is inherently randomized.)
- Compare($N_1$', $N_2$'): Compare two network names for equality. Return a 1 if
the underlying application names are equal and a 0 otherwise. 

Moreover, it should be difficult to learn the application name $N$ from its
network name $N$'. 

I claim that you can build the same packet forwarding logic using these
two functions. (Albeit at significant performance costs.) But why
would one bother? For one thing, the application name is no longer revealed
to the network. This lets forwarders act on names without having any
encryption context and without using encapsulation as in case #1 above.

From a theoretical perspective, such functions allow you to build 
name *equivalence classes*. That is, sets of different representations
for the same name. We would like these equivalence classes to be formed
such that, given two distinct names $N_1$ and $N_2$, the probability
that Hash($N_1$) is in the equivalence class of Hash($N_2$) is overwhelmingly 
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
To prove this statement, let's do the actual computations here. I'll use
the notation $s_1$ and $s_2$ to refer to the distinct $s$ values for $N_1$
and $N_2$. Assuming that $N_{i,1}' = N_{i,2}'$ for each each segment $i=1,\dots,k$,
it follows that:

$$
\begin{align}
p_1 & = (\beta_{i,1}^k)\alpha_{i,2} \\
    & = (g^{N_{i,1}' + s_1)^k})p^{s_2} \\
    & = (g^{kN_{i,1}' + ks_1})g^{ks_2} \\
    & = g^{kN_{i,1}' + ks_1 + ks_2} \\
    & = (g^{kN_{i,1}' ks_2})g^{ks_1} \\
    & = (g^{N_{i,1}' + s_2}^k)p^{s_1} \\
    & = (\beta_{i,2}^k)\alpha_{i,1} \\
    & = p_2
\end{align}
$$

# Running Code

I implemented this basic scheme using the group of integers $Z$ modulo some
sufficiently large prime $p$. (We could use a better group in which the DL 
problem is hard, but $Z_p$ works to illustrate the idea.) The code proceeds
as follows. First, it fixes $p$ and then generates a generator $g$ and 
public and private key pair $k$ and $pk$ (denoted as $p$ in the previous
section). Then the secret to be obfsucated is sampled at random along with the
two random values $s_1$ and $s_2$ (denoted as $s$ and $r$ in the code). 
Finally, the program hashes both values and then compares them for equality. 
The result of this comparison is printed. 

The complete listing is below. You may run the code yourself to check its
correctness. 

{% gist aabeaf06e7bacdec752b %}

# Complexity Problems

While it's great that we now have *some* way to compare names without
revealing them, this entire scheme presents some challenges. For instance,
in vanilla CCN, we rely on names as given to perform matching and indexing 
into data structures such as the FIB, PIT, and CS. However, since the 
"verifier" (router) can only determine equality among a pair of
obfuscated names by invoking the compare function for each one, the time
to index these data structures is linear with the number of entries.

I'm currently working on a paper that will show this result formally. And 
the moral of the story is that wrapping names a la TOR is the best
we can do if we wish to hide information within packet headers. 

# References

- [1] TOR
- [2] ANDANA paper

