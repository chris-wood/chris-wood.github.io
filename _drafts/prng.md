---
layout: post
title: A Native PRNG
---

TLDR: Randomness and unpredictability are at the core of cryptography. All notions
of cryptographic security rely on the adversary's inability to guess a random
value. This includes block cipher keys, exponents used in the Diffie Hellman 
protocol, and prime factors used in cryptosystems like RSA. It's everywhere. 
In this post, I'll describe how I implemented the cryptographic PRNG for the 
PARC security library. 

# PRNGs and Pools

A cryptographic pseudorandom number generator (PRNG) is a primitive that, well,
generates random bits suitable for use in cryptographic protocols. The latter 
distinction is an important characteristic that differentiates cryptographic
PRNGs from other random number generators. It means that, given previous values
or outputs fro the PRNG, it should be impossible (read: only likely with extremely
negligible probability) for an adversary to predict what the future values will be
with any accuracy. This remains true even for PRNGs that output single bits at a
time. It must be the case that the adversary's probability of guessing the next
bit of a PRNG is negligibly larger than 1/2 (i.e., no better than a random guess).
To be a bit more formal, this means that a PRNG is able to stretch the length
of its input to some polynomially-sized output that is *computationally*
indistinguishable from a uniformly random bit string of the same length.
(The computational limitation means that it is not possible to build a distinguisher
that runs in polynomial time which is able to determine tell the PRNG output
apart from true randomness with probability non-negligibly better than 1/2. 
See [1] for more information about this notion.)

In practice, creating randomess of this quality is hard. It requires randomness with
a high amount of entropy. All modern operating systems that I'm aware of gather this
entropy from things that go in within and beneath the kernel, such as device I/O
and jitter. Some libraries like OpenSSL operate as userspace PRNGs. This is
problematic for a number of reasons:

1. Application-space PRNGs may share space across different applications. Sharing 
space is rarely a good idea, especially when this state is used to derive cryptographic
secrets.
2. There aren't great sources of entropy in the application space. The OpenSSL library,
for example, uses information like PIDs and time to accumulate entropy before mixing
into the pool. 

You should not be using these PRNGs; randomness provided by the kernel is "the right
answer," according to AGL [3]. Why? Well, the kernel can ensure that processes
don't share entropy space and has a better source of entropy from which to accumulate
random bits. If you're bored, just take a look at 
[the Linux source code](http://lxr.free-electrons.com/source/drivers/char/random.c)
to see all the different sources of entropy for the pool. It processes inputs
from timers, interrupts, and disk I/O, among other things. This code maintains a pool
of entropy. Bits can be added and extracted from this pool. Adding to the pool 
(via the sources above) increases the amount of entropy. Likewise, sampling from this 
pool decreases the amount of entropy. This has (used to have) important implications
on how the randomness is used, as we'll discuss in the following sections.

It's also worth mentioning that recent Intel chips have a new RDSEED instruction
that is able to provide "seed-grade entropy" [4]. The [Linux random code](http://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/tree/drivers/char/random.c#n946)
will attempt to use this instruction if available to add more entropy to the pool. 
This entropy is supposed to be in compliance with the guidelines outlined in 
SP 800-90A [5]. But even if it wasn't (i.e., even if it was not quality entropy),
it won't hurt adding to the a pool with other random sources.

# Randomness Exposure 

On *nix systems, this randomness is exposed via the /dev/random
and /dev/urandom character devices. As a matter of technicality, both /dev/random
and /dev/urandom use the exact same PRNG internally. On Linux, the /dev/random
device can block if there's insufficient entropy [6 CITE], which is an issue for
application stability and predictability. Conversely, the /dev/urandom device on
other modern platforms (FreeBSD and OS X, for example) provides the same randomness
as its counterpart by does so without blocking. 

Due to misleading documentation and false rumors, there has been confusion about
which of these two devices to use when generating entropy. Today, the definite
answer is to use /dev/urandom [2]. (Take it from djb, whose opinion I respect
greatly.) BoringSSL, the OpenSSL fork by Google, uses /dev/urandom directly [3].
Why? Because /dev/random blocks when there is insufficient entropy. This doesn't
mean that /dev/urandom provides any less amount of entropy. On the contrary,
/dev/urandom has more than enough entropy to seed any cryptographic uses. So
as a matter of usability, most folks just use /dev/random, and that's what we're
going to do. 

# A Simple PRNG

I wanted to keep the PRNG as simple and straight-forward to use as possibe. To that
end, its API consists of four core functions: two constructors (one without and one
with a seed), a function to return a random 32-bit integer, and a function to fill
a pre-allocated buffer with random bytes. I didn't add a function to re-seed the 
PRNG because, as discussed above, that's not necessary if we rely on /dev/urandom. 
The implementation of these functions is as you might expect. The default constructor 
opens a handle to the /dev/urandom device. The seed variant goes further and 
subsequently writes the seed to /dev/urandom before returning the PRNG. The actual
randomness extraction functions are trivial to use:

~~~
PARCSecureRandom *rng = parcSecureRandom_Create();
uint32_t randomWord = parcSecureRandom_Next(rng);

...

PARCBuffer *buffer = parcBuffer_Allocate(32); // 256 bits
ssize_t numBytes = parcSecureRandom_NextBytes(rng, buffer);
// numBytes == 32
~~~

The actual implementation [is available here](https://github.com/PARC/Libparc/blob/master/parc/security/parc_SecureRandom.c). (I may re-name the `parcSecureRandom_Next` function
to something that indicates the return type. I now recognize that this isn't
the best name.) Some quick profiling shows that it has satisfactory performance. 
On my laptop with a 2.8 GHz Intel Core i7 chip, it cranks out 32-bit integers at 
an average speed of 718ns, The following plot also shows the average time to
extract bytes into a buffer. There's a lot of variation, but you can see the
increasing trend. On average, the time hovers in the range of 3000ns and 8000ns. 

![`parcSecureRandom_NextBytes` profile results.](/images/prng.jp)


<!-- 
https://blog.cloudflare.com/ensuring-randomness-with-linuxs-random-number-generator/
http://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/tree/drivers/char/random.c#n946
http://www.mail-archive.com/cryptography@randombit.net/msg04763.html
http://www.2uo.de/myths-about-urandom/
http://sockpuppet.org/blog/2014/02/25/safely-generate-random-numbers/
-->

# References

- [1] katz and lindell
- [2] http://www.mail-archive.com/cryptography@randombit.net/msg04763.html
- [3] TODO: langley boringssl random citation
- [4] https://software.intel.com/en-us/blogs/2012/11/17/the-difference-between-rdrand-and-rdseed
- [5] http://csrc.nist.gov/publications/nistpubs/800-90A/SP800-90A.pdf
- [6] dev random blocking citation
