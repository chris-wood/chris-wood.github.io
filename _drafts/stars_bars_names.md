---
layout: post
title: Stars, Bars, and Names
---

TLDR: I learned about the stars and bars technique from combinatorics and used it to get an
estimate of the number of unique names that are possible in CCNx. In short, it's an astronomically
huge number. A fixed packet length therefore imposes no practical restriction on the number of possible
names that can be formed. 

# Introduction
 
In one of my ongoing research projects we needed to compute a rough estimate of the 
number of unique CCNx names.
On the wire, a name is TLV representing a URI with an arbitrarily sized list of segments. Each 
segment is yet another TLV. In most cases, the type of a segment is T_NAMESEGMENT, which means
that the value is just a generic name segment (i.e., a flat, opaque string). Other segment
types include T_IPID (interest payload ID), T_CHUNK (chunk number), and T_VERSION (version number).

The name is nested several layers deep within a CCNx packet. The diagram below shows the TLV
packet format up to the name field. 

~~~
                     1                   2                   3
   0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +---------------+---------------+---------------+---------------+
   |    Version    |  PacketType   |         PacketLength          |
   +---------------+---------------+---------------+---------------+
   |           PacketType specific fields          | HeaderLength  |
   +---------------+---------------+---------------+---------------+
   / Optional Hop-by-hop header TLVs                               /
   +---------------+---------------+---------------+---------------+
   |         MessageType           |         MessageLength         |
   +---------------+---------------+---------------+---------------+
   | Name TLV       (Type = T_NAME)                                |
   +---------------+---------------+---------------+---------------+
   /                   REST OF THE PACKET HERE                     /
   +---------------+---------------+---------------+---------------+
~~~

Beyond the Name TLV, there are two other important fields in this packet snippet:
PacketLength and HeaderLength. The PacketLength is the length of the entire packet
(including the header), bounded by 64KB, and the HeaderLength is the length of the
header up to the Message TLV (i.e., the MessageType and MessageLength fields). 
This means that the name has a fixed length. Thus, we are at least able to compute 
the total number of unique names. But how?

# First Attempt

Assume we know the fixed byte length $B$ for our name. One way to compute the number of unique
names is to do the following:

1. For each byte length $b = 1$ to $B$ generate all possible random strings of length b. 
2. For each random string, compute the number of ways to partition the string into 
non-empty ordered sets. For example, let $L = [e_1, e_2, \dots, e_b]$ be one random string
of length $b$. Moreover, let $k \geq 2$ be the number of partitions we seek for this list.
One valid partition for $k = 2$ would be $L_1 = [e_1]$ and $L_2 = [e_2,\dots,e_b]$. We would 
need to consider all possible values of $k$ from $1$ to $b - 1$. 
3. Compute the total number of all names generated from steps 1 and 2.

The number of unique byte strings of length $b$ is simply $2^{8b}$.
For step 2, given a random string represented as a list $L$, there are $b - 1$ possible
values for $k$. All that remains is to compute the number of possible partitions given 
the list $L$ and integer $k$. The following Python function does this accurately. 

{% gist 9202048b72ce8e2188d2112b71ec8af8 %}

Given this approach, we can compute the total number of CCNx names with the following bit of code.

{% gist f37b1061ed974d76d8ee18e6ce726117 %}

# Stars and Bars

While the above technique works, I find it messy. I would prefer a less verbose alternative to
the computation rather than this algorithmic description. During my search for a cleaner 
way to compute this value I was pointed to the stars and bars technique [1] -- 
a useful combinatorics result that helps in counting many things. 
Using the stars and bars method, it can be shown that the number ways to insert $k - 1$ 
separators into the $n - 1$ gaps in a list $L$ is $\binom {n - 1} {k - 1}$.
This is identical to finding the number of ways to split a list $L$ of $n$ elements
$k$ partitions. So using this fact we can replace the above computation with the following 
simple expression.

$$
\begin{align}
\sum_{b=1}^B 2^{8b}\left(1 + \sum_{k=1}^{b - 1} \binom {b - 1} {k - 1}\right)
\end{align}
$$

The following program implements this computation in addition to the one previously listed.
You can run it to check that the outputs are the same. Bear in mind, though, that the number
of names is *huge* due to the ability to allow for arbitrarily many segments
in a name. To give you an estimate, if we consider names with an upper bound of 
8 bytes, there are roughly $2.3 \times 10^{21}$ different names. Now imagine that number
when you consider that names are really capped at just under 64KB in length. 
I think it's safe to say that we won't suffer from namespace saturation any time soon.

{% gist 26216ecc97bc624b08bded6d31db4253 %}
