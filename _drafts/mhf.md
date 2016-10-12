---
layout: post
title: Tuning Memory Hard Functions
---

# Memory Hard Functions

Memory-Hard Functions (MHFs) are one of the hot topics in cryptography right now. 
If you're not familiar with the term, the gist is as follows. Given time $$T$$ and
space $$S$$, an optimal MHF is one where $$S \times T \in \Ohm(n^2)$$, where $$n$$ is the 
security parameter [1]. This means that it is computationally expensive to compute
the function in terms of both time *and* space. The goal of constructing MHFs is to 
increase the constant term for $$n^2$$ to as large as possible. Intuitively, this
makes the function expensive to compute in any computer. 

The most common use case for MHFs are as password hashing algorithms. To deter dictionary
attacks, passwords are (or should be) computed with an additional salt value. This
expands the search space for a single password exponentially by the length of the
salt. However, this is of primary concern for online dictionary attacks against these
functions. Offline dictionary attacks, on the other hand, can be carried out with tremendously
more resources and dedicated hardware. Depending on the type of password hashing algorithm
used, these attacks can be trivial or incredibly expensive. Standard
cryptographic hash functions are insufficient here since they can be 
effortlessly computed using special-purpose dedicated hardware. To given an example
quoted in [1], it takes approximately $$100,000\times$$ more energy to compute 
a single SHA-256 invocation on a general purpose CPU than on special purpose hardware
(e.g., an ASIC) [2]. While it is true that requiring more iterations of the hash 
function increases the cost of its computation, this is of negligible effect on
dedicated platforms. 

This is where MHFs come in. If increasing the time dimension does not do the trick, we
must look to the space dimension. Since MHFs require a large amount of *password independent*
state to compute, dedicated platforms are much more costly to implement. The attacker
must choose to use its resources for more "cores" (or hash execution engines) or more memory. 
And with a finite amount of resources available, any choice will lead to significantly
less computational power to hash passwords. Basically, the large memory requirement ensures 
that special-purpose chips can only contain a small number of cores. The attacker must
choose to dedicate his or her resources to more space or more computation time, not both. 
That's intuitively where the "hardness" comes from.

There are many MHFs floating around the cryptosphere right now. These include scrypt [3], 
Argon2 [4], Catena [5], and Balloon [1] (to name a few). This is in large part due to the recently
concluded Password Hashing Competition [6], which named Argon2 as the winner. I'll admit
I didn't follow the developments closely over the past couple of years, but I'm trying to
make due for lost time now.

# MHF Constructions

Standard MHFs take four inputs and produce a single out. The inputs include the password to
hash, a salt, and space and time parameters. The password and salt are what you might expect:
strings. The tuning parameters are more interesting. The space parameter $$s$$ determines how much
memory is to be consumed by the MHF. As [1] puts it, the MHF should be easy to compute with
$$s$$ blocks of memory but difficult to do so with anything less. The time parameter $$r$$ denotes
the number of rounds the function iterates before producing its output. Typically, this is tuned
to account for what space is available (or not) when computing the function. On systems without
a great deal of memory, $$r$$ is increased to maintain the memory hardness properties.

Most MHFs generally consist of three stages: expand, mix, and output. Their role is clear:
expand takes the inputs and blows it up into a large amount of memory, mix iterates over this
memory and diffuses the inputs throughout the entire state space, and output produces the final
digest. The mix step is where the meat of the algorithm works. And it's typically designed
such that it cannot be parallelized. On multi-core machines, there would be ample room to then
execute multiple invocations in simultaneously. This sort of defeats the purpose of the function's 
inherent computational hardness. (Especially on systems with *many* cores, such as GPUs.) Therefore,
to remedy this problem, many MHFs ([19, 42, 68, 69] from balloon) specify "parallel" versions. 
If $$F(p, s)$$ is the MHF with password $$p$$ and salt $$s$$, and $$M$$ cores are available, then
the parallel version $$F_M(p, s)$$ is:

$$
\begin{align}
F_M(p,s) = F(p, s || 1) \oplus F(p, s || 2) \oplus \dots \oplus F(p, s || M)
\end{align}
$$

Basically, $$M$$ different invocations are combined to produce the final result. 
This is exactly what the Balloon algorithm does, at least. It's important to
note that the salt is, well, salted for each of the $$M$$ invocations of $$F$$. 
Modifying the password would be silly for a number of reasons. First, at least
in the case of Balloon, the salt determines the memory access pattern. If this
is unchanged across cores, then it might be possible to precompute this pattern
and therefore speed up parallel implementations. By modifying the salt this
is effectively stopped. Second, we shouldn't be messing with the password --
the most sensitive input -- anyway. 

Now let's move on to some code. Here's the pseudocode description of Balloon. 
I'm focusing on it so much because I think it's simply elegant. 

{% gist 0e6f13cd3e144ba77554a3efd83f379c %}

It's amazingly simple. You can see that the amount of memory depends solely on $$s$$.
The expansion phase is intuitive: feed the password and salt into a cryptographic
hash function (necessary for one-wayness) to initialize the first block of memory. Then,
initialize the remaining blocks based on a monotonically increasing counter and the
value of the previous block. The mixing step is also very clear. There are $$st$$
iterations (which points to our earlier security definition where $$s \times t \in \Ohm(n^2)$$),
and during each iteration the following occurs. Every block $$B$$ is advanced based on the 
previous (iteration) block and the current value of $$B$$ (lines 18-19). Then, a random
selection of other blocks are mixed into $$B$. The selection of these other blocks depends
on the iteration count and salt. (This is why memory accesses are dependent on the salt.)
When all is said and done, the last block is chosen as the output. 

It's so simple in fact that I was able to whip it together in Go in about 30 minutes. Here's 
the program.

{% gist XXX %}

# Parameter Tuning

Typically, one chooses the MHF parameters such that the execution time is "within" reason 
for a single execution. For example, the Argon2 paper [xXX] suggests one to do the following.

1. Detemrine the number of threads that can be used.
2. Determine the memory capacity.
3. Tune the number of rounds until the runtime is "within reason" for a single execution. 

In a recent project I had to solve a different problem. Namely, given a reasonable runtime, 
I needed to find the parameters such that execution met but did not exceed this runtime bound. 
This turned out to be a bit of a challenge but also a good excuse to brush up on some optimization
algorithms from college. Here's what I did.

XXXX: describe what I did, show the code, and output, and blah





- what are memory hard functions
- why are they important
- where are they used
- common algorithms
- default parameters and tuning them -- how is it (or should it be) done?
- describe the optimization mechanisms used and how effective they are (or are not)

# References

- [1] [Balloon Hashing: A Memory-Hard Function Providing Provable Protection Against Sequential Attacks](https://eprint.iacr.org/2016/027.pdf).
- [2] [23] from ballon paper
