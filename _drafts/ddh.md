---
layout: post
title: Decisional Diffie Hellman and Zp
---

As Dan Boneh put it, the Decisional Diffie Hellman (DDH) problem is a "gold mine" [1].
DDH is a fundamental building block to many cryptographic appliations, including the ElGamal
cryptosystem, efficient pseudorandom functions, and CCA-secure cryptosystems (see [1] for more details).
Those familar with the DDH are also likely familiar with the related Computational
Diffie Hellman (CDH) and Discrete Log (DL) problems. To understand these problems, 
let $G$ be a finite cyclic group with generator $g$. The DL problem states that,
given $g^x$ for some random $x \in G$, it is computationally intractable to discover $x$.
The CDH is a slightly harder problem formulated as follows. Given the tuple ($g, g^a, g^b)$
for some random $a,b \in G$, it is computationally intractible to compute $g^{ab}$. (The
Diffie Hellman part of its name should be apparent now.) As Boneh points out in [1],
relying on the CDH is not wise when, for example, generating shared secrects. The values
$g^a$ and $g^b$ leak a significant amount of information of the value $g^{ab}$ (e.g.,
up to 80% of the bits). The DDH is the hardest among these problems. In essence, it says
that there is no probabilistic polynomial time algorithm that can distinguish between
the $(g^a, g^b, g^{ab})$ and $(g^a, g^b, g^c)$. 

Consider the relationship between these problems. If one can solve the DL in $G$, 
then one can also solve the CDH by computing $a$ and $b$ from $g^a$ and $g^b$, respectively, 
and then computing $g^{ab}$. Moreover, if one can solve the CDH, then one can easily distinguish
between $g^{ab}$ and $g^c$ by computing $g^{ab}$. To my knowledge, there are no known algorithms
for solving the CDH using a solution to the DDH. Nor can one build a DL solver from a CDH solver. 
In plain English, this means that DDH is the *strongest* assumption of the three. If the DDH
is hard, then the the CDH and DL problems must also be hard. Conversely, if the DL is easy, then
the CDH and DDH are easy. 

This is all well known stuff. And not the reason I'm writing this week. Instead, I'd like to
focus just on the DDH problem. In particular, the DDH problem in a specific group. Namely,
$Z_p$, the group of integers modulo some prime $p$. This is a finite cyclig group in which
the DL problem is hard. However, I recently learned that the DDH is *trivial* in this
group. The reason relates to the law of quadratic reciprocity and the Legendre symbol, which
states if a given number is a quadratic residue. Formally, the Legendre symbol $(v,p)$ is
$1$ if $v$ is a quadratic residue in $Z_p$, $-1$ if it is a quadratic nonresidue, 
and $0$ otherwise (if it is congruent to $0$ in $Z_p$). This means that a generator $g$ of
$Z_p$ must be a *quadratic nonresidue*, since, by the property of generators whose orbit
is the group, there does not exist an integer $x$ such that $x^2 = g \mod p$. (If that were
the case then $g$ wouldn't be a generator.)

One more property of the Legendre symbol states that $(uv,p) = (u,p)(v,p)$, meaning that:

- The product of two quadratic residues or two quadratic nonresidues is a quadratic residue.
- The product of one quadratic residue and one quadratic nonresidue is a quadratic nonresidue.

This means that given the Legendre symbol of $g^a$ and $g^b$ in $Z_p$, it is easy to compute
$(g^{ab}, p)$. Now consider the possible values for $(g^{ab}, p)$. If $a$ is even, it must
be the case that $(g^a, p)$ is a quadratic residue. (This is because $(g, p) = -1)$,
$(g^a,p) = (g,p) \cdot (g,p)$ ($a$ times), and $(-1)^2 = 1.) Therefore, $(g^{ab},p) = 1$
if either $a$ or $b$ are event, and $-1$ otherwise. Conversely, $(g^c,p) = 1$ with probability
$(1/2)$ since $c$ is chosen at random in $Z_p$. Since the probability that $a$ or $b$ is
even is $0.75$, this gives an easy way to differentiate $g^{ab}$ from $g^c$ [2].

I verified this experiment by writing some (Sage) Python code to build this distinguisher.
The result is shown below. Run it using sage with `sage -python ddh-solver.py 128`, where 128 is
the number of bits in $p$. The distinguishing advantage is printed at the very end.
You should see the clear adversarial advantage. 

{% gist 0b75a91e6905995dbba7cc35afc9f36c %}

The question then remains: in what groups is the DDH assumed to be hard (as hard as a complete DL)?
One example is the subgroup of $k$ residues modulo a prime $p$, where $(p - 1) / k$ is also a large
prime. The reason ties back to our assessment above. When $k = 2$, the group of quadratic residues
modulo a safe prime will contain elements where for each one the Legendre symbol is $1$. In other words,
the above trick doesn't work. 

There are other groups as well. I need to do my homework and learn more about them before writing. 
So, until then...

# References

- [1] Boneh, Dan. "The decision diffie-hellman problem." Algorithmic number theory. Springer Berlin Heidelberg, 1998. 48-63.
- [2] http://security.hsr.ch/msevote/dh_assumptions.html


