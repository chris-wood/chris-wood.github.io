---
layout: post
title: TCP Initial Sequence Numbers in Linux
---

In [my last post](http://chris-wood.github.io/2016/09/23/TCP-Sequence-Prediction.html) I talked about TCP sequence number prediction attacks.
To recap, these are possible when an off-path adversary is able to correctly guess
the sequence number and connection tuple of an ongoing connection. The natural remedy
to this problem is to make the sequence numbers unpredictable for *every* packet
sent back and forth. I discussed this in my last post. However, the current remedy
is to only make the initial sequence number (ISN) unpredictable. The general approach
specified in RFC 6528 [1] is to set the ISN to be the output of a *keyed* PRF computed
over the connection tuple and some quickly evolving timer. The PRF key is generated
once for the system, but the "salt", or counter, changes over time
in a predictable fashion. In RFC 6528, the counter is supposed to advance every 4 microseconds.
(That's an important part of this scheme and how its implemented.)

In theory, this scheme is fine. A PRF $$F_k$$ is indistinguishable from truly random function $$f_r$$,
which have the property that, for a given input $$x$$, $$f_r(x)$$ is an element chosen uniformly
at random from the range. Thus, without knowledge of the key $$k$$, let alone the counter, an off-path 
adversary cannot compute the PRF and predict the ISN. 

But theory often differs from practice. Case in point: the Linux implementation uses MD5 with the 
secret-suffix method as its PRF. There's a wealth of information available to show that this
scheme is in fact *not* a secure MAC -- the primary reason being that MD5 is not collision 
resistant -- but is it okay as a PRF? Do the poor collision resistance properties of MD5 compromise
the security of the PRF? Take a look at the following ISN generation code extracted from version
4.8 of the Linux kernel [2].

{% gist d0a4a61c2679ed8712547ae7d248f18c %}

The `seq_scale` function is shown below.

{% gist e486d0e2e44f0b7563101daaa29d5b39 %}

This is responsible for adding the timer value (M in RFC 6528) to the output of the PRF.
In actuality, I see little value added by the timer. We should assume it can easily be
predicted since it's based on the actual clock time. Therefore, the security of
this scheme reduces to the PRF. And in practice, as I mentioned in the last post, 
MD5 is used in a particularly problematic way. 

The Linux implementation uses MD5 as a PRF with the so-called "secret-suffix" construction [3].
This scheme works as follows. Given a collision resistant function $$H$$, we build a PRF
with key $$k$$ as $$H(m || k)$$, where $$m$$ is the input. We'll denote this construction 
as a function $$H'$$. This is not safe if $$H$$ is not
collision resistant (which MD5 is not!). Specifically, a simple PRF attack (distinguisher)
can be carried out by (1) obtaining $$H'(x_0)$$ and then (2) finding another message $$x_1$$ such that
$$H(x_0) = H(x_1)$$. By the properties of $$H$$, which is typically a function of the Merkle
Damgard variety, the output of $$H'$$ on $$x_0$$ will then be identical to the output of
$$H'$$ on $$x_1$$. Attacks of this variety were formally studied in [4]. I don't do them 
much justice here.

Since MD5 is not collision resistant, it falls prey to this attack. (However, I'm not sure
how easy it is to find collisions in the small message space used in the implementation.
I'll defer that to a future experiment and writeup.) Let $$x_0$$ be a connection tuple
(i.e., the prefix of the MD5 PRF) from which an ISN is computed. Let $$x_1$$ be a different,
yet valid, connection tuple that collides with $$x_0$$. If connections for $$x_0$$ and 
$$x_1$$ are started in the same counter epoch, then the ISNs will be the same. (If they're
started outside of the same counter epoch, they could just as well be predicted.)
And there's the start of a prediction attack. 

What can we do to fix this? Well, get rid of MD5 once and for all. The authors of RFC 6528 
claim that MD5 is the fastest of the available functions suitable for this PRF. But that's
false. SipHash [5] is a cryptographic PRF designed for short inputs. Quoting from their 
main page, SipHash was designed to

> target applications include network traffic authentication and defense against hash-flooding DoS attacks. 

Plus, djb is one of the designers, so you know it's good. (That's one of the IETF security
adage :) Lastly, it performs *much* better than MD5. How good? To test this, I extract the 
MD5 PRF code directly from the Linux kernel, removed lines that would induce context switches
when run in usermode (e.g., getting the current time), and compared it to the *reference*
implementation of SipHash. On average, SipHash finishes in half the time as MD5. 
(You can check the code out and run it for yourself [here](https://github.com/chris-wood/secure-sequence-perf).) This
is shown nicely in the SipHash paper [6] on page 13. According to their data and the 
type of input for the ISN generation (256 bits, or 32 bytes, assuming the secret key is 
included in this input), MD5 takes roughly 600 cycles compared to just less than 200 cycles
on an AMD FX-8150 CPU. That confirms my observation.

So what's stopping SipHash from being used here? Several things. First, MD5 is recommended
as part of the RFC, so a change to the Linux implementation would require a change to
this particular RFC (or a new one to obselete the old one). Second, some people might
be dubious about the severity of this problem. (Aren't sequence number prediction attacks
a thing of the past?) To them I say, attacks always get better, they never get worse. 
Using SipHash instead of MD5 erases this problem. And it performs better, too, which is
something the Linux community seems to care about.

In the next couple of weeks, time pending, I'm going to get my hands dirty preparing a 
patch for the kernel to address this issue. Maybe I can convince someone else this is 
important in the process. 

# References

- [1] [https://tools.ietf.org/html/rfc6528](https://tools.ietf.org/html/rfc6528)
- [2] [Linux Kernel 4.8 Browser](https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/log/?id=refs/tags/v4.8)
- [3] [https://cr.yp.to/bib/1999/preneel.pdf](https://cr.yp.to/bib/1999/preneel.pdf)
- [4] [http://people.scs.carleton.ca/~paulv/papers/Crypto95.pdf](http://people.scs.carleton.ca/~paulv/papers/Crypto95.pdf)
- [5] [https://131002.net/siphash/](https://131002.net/siphash/)
