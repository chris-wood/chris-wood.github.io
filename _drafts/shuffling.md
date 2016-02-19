---
layout: post
title: Public Shuffling
---

# Shuffling vs Re-Randomization

Consider the problem where we have $l$ parties with messages $m_i$ who wish to generate
the list $(m_{\pi(1)}, \dots, m_{\pi(l)})$, where $\pi$ is a permutations over the
integer range $[1,\dots,l]$. Suppose further that they wish to do this without
revealing $\pi$. Something that computes this is basically a mixnet -- it hides the
sender (provider) of each message $m_i$ by shuffling them around and about. What we
desire is for this mixing procedure to be truly public. Adida et al. [x] show that
any homomorphic encryption scheme can be used to build an inefficient shuffling schemes
by obfuscation. They also show how to construct efficient shuffling schemes based on
the BGN and Pallier encryption schemes. I'll give an overview of the one based on
Pallier.

[x] https://eprint.iacr.org/2005/394.pdf
