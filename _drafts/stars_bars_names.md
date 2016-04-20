---
layout: post
title: Stars, Bars, and Names
---

TLDR: I learned about the stars and bars technique from combinatorics and used it to count
the number of unique names that are possible in CCNx. 

# Introduction
 
In one of my ongoing research projects we needed to compute the number of unique CCNx names. 
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
3. Compute the total number of all names generated. 

The number of unique byte strings of length $B$ is simply XXX.
For step 2, given a random string represented as a list $L$, it is easy to determine all possible
values of $k$ -- it is one less than the length of $L$. All that remains is to compute the number
of possible partitions given the list $L$ and integer $k$. The following Python function
does this accurately. 

~~~
def num_partitions(l, k):
    if k == 1:
        return 1
    else:
        total = 0
        for i in range(1, len(l)):
            total += num_partitions(l[i:], k - 1)
    return total
~~~

Given this approach, we can compute the total number of CCNx names with the following bit of code.

~~~
def num_partitions(l, k):
    if k == 1:
        return 1
    else:
        total = 0
        for i in range(1, len(l)):
            total += num_partitions(l[i:], k - 1)
        return total

B = 8 # 8 bytes for the fixed name length
total = 0 # accumulator for the number of names
for b in range(B):
    total += 1 # for the name without any partition (a single segment name)

    # Compute the number of unique partitions for all possible values of k
    l = [0] * b
    partition_total = 0
    for k in range(b):
        partition_total += num_partitions(l, k)

    # Multiply the partition total by the number of possible names of length b
    num_unique = 2 ** (8 * b)
    total += (partition_total * num_unique)

    print "For b=%d, partition_total=%d" % (b, partition_total)
    print "For b=%d, num_unique=%d" % (b, num_unique)

print "Total: %d" % (total)
~~~

# Stars and Bars

While the above technique works, it's far from ideal. I would prefer a closed-form solution to
the expression rather than something messy like the above. Thankfully, I was pointed to the 
stars and bars technique -- a useful combinatorics result that helps in counting many things. 
Using the stars and bars method, it can be shown that the number ways to insert separators into
$k - 1$ of the $n - 1$ gaps in a list $L$ is 

$$
\begin{align}
\binom {n - 1} {k - 1}
\end{align}
$$

Using this fact, we can replace the above computation with the following simple expression.

$$
\begin{align}
\sum_{b=1}^B 2^{8b}\left(\sum_{k=1}^{b - 1} \binom {b - 1} {k - 1}\right) + 1
\end{align*}
$$

TODO: computation to make sure it matches.
