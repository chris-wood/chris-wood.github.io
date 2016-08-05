---
layout: post
title: The Kolmogorov-Smirnov Test and RNGs
---

During a recent Web crawl, I came across the topic of pseudorandom number generators (PRGs). I've talked
about ways in which they're implemented [in the past](http://chris-wood.github.io/2016/05/13/Native-PRNG.html). There are plenty
of deterministic PRGs that stretch a small amount of randomness with sufficiently
high entropy into a longer stream of bytes with comparable entropy. Nowadays, Bernstein's ChaCha20
seems to be the most widely used algorithm. You might ask yourself why ChaCha20 is so prevalent. One
reason is that it's been thoroughly vetted as a secure stream cipher. And from a stream cipher, one
can build a PRG. The process is simple: the key stream that's generated which would originally
be XORed with some plaintext message is used as the PRG byte stream output. Clearly, the cipher is
only secure if this byte stream is indistinguishable from random. So if the cipher is secure then we have a cryptographically secure PRG (CSPRG). The only caveat is that the initial state must be secret;
otherwise someone could easily determine future output of the PRG. When used in the Linux kernel, this
state is the result of entropy collected from the environment. So it is effectively secret.

![A stream cipher as a CSPRG.](/images/stream_cipher_prg.png)

For the sake of argument, assume we have a (CS)PRG $$G$$ that we want to use. Moreover, even if we have a security proof, we do not trust it. (Bear with me...) Instead, we only have faith in randomness that we can test. But how do we assess the randomness of $$G$$? Long ago (over two decades), George Marsaglia
proposed a suite of statistical tests -- called the DieHard tests [1] -- aimed at
measuring the effectiveness of random number generators.
It included tests such as the "birthday spacings," "overlapping permutations,"
"minimum distance," and so on. While the Diehard
tests are no longer suitable for assessing the randomness properties of PRGs like $$G$$, it's a good place to start.
(Modern test suites include DieHarder [2], TestU01 [3], and the NIST suite [4].)

Suppose the first test we want to run is the "minimum distance" test [5]. This works as follows.
Let $$T = 100$$ be the number of trials we conduct for this test. For each trial, pick $$n=8000$$
random points in a square with sides of length $$10000$$. Then, find the minimum distance $$d$$
between each pair of points. The Python code to do this is below.

{% gist efc1465a2bad29acb21491db5633b0bf %}

If the points are truly i.i.d. variables drawn from a uniform distribution, as we would expect for
a random number generator, then $$d^2$$ should be exponentially distributed with a mean of $$0.995$$.

So how do we actually test that the distribution of $$d^2$$ is exponential with the given mean? This is
where the Kolmogorov-Smirnov (KS) test comes in handy. The KS test tests that the distribution
of some set of data matches a specific distribution. (In this case, the exponential distribution.)
The test itself is rather elegant. It basically works as follows. Let $$F_k$$ be the empirical distribution
function of the input data set, and let $$F_0$$ be the CDF of the known distribution. $$F_k$$ is computed
iterating over each element $$x$$ in the data set and computing the number of other elements that are less than
or equal to $$x$$. This frequency is then divided by the total number of elements in the set. (Its relation
to the CDF should be clear, then.) Once this is obtained, the KS statistic $$D_n$$ is computed as [6]

$$
\begin{align}
D_n = \sup_{x}|F_k(x) - F_0(x)|
\end{align}
$$

That is, $$D_n$$ equals the maximal difference between any two outputs of the $$F_k$$ and $$F_0$$ functions.
If the distributions are identical then $$D_n$$ converges to 0 as $$n$$ approaches infinity. So, given
a finite set, these distributions are equated by comparing $$D_n$$ against a table of acceptable values.
A confidence level $$\alpha$$ is also used to determine the error margin for this test. The larger the
value of $$\alpha$$, the smaller the value of $$D_n$$ that is supported (in the long run of $$n$$). Or,
put another way, as our confidence level increases, the acceptable difference between the empirical distribution
function and the known CDF decreases.

I recognize that the KS test is implemented in most major statistical software tools. but let's look at
how we might implement it if we had to do so from scratch. The code is actually somewhat simple. We begin
by computing the empirical distribution function $$F_k$$. This works by iterating over every unique sample
$$x$$ and counting the number of elements that are less than $$x$$.

{% gist 57d243c66e8d1a7d0159054df3df758a %}

Now we need to compute the KS statistic given the two distributions $$F_k$$ and $$F_0$$. This
is done by finding the maximum difference between the two distributions. Simple enough.

{% gist 8867990e01afa35c858f603b366c11d2 %}

The last step is to actually perform the test given some confidence level $$\alpha$$. I just hard-coded the KS test
table into the code and compare the KS statistic against this value with $$\alpha$$. If the distributions are close,
i.e., if the statistic is less than the corresponding value in the table, the test returns true. Otherwise, it
returns false.

{% gist 20a907c634aaeec09db1590a6416b97d %}


So now we can finally get back to the question at hand: does the minimum distance from the MDT follow an exponential
distribution? To check this, I created the exponential CDF and ran it through the KS test with the minimum distance
test code. As we would expect, the result was positive.

{% gist c74dd061376445d4344dfa99389417ee %}

I'd like to explore other randomness tests in the future. But for now, this was a nice
way to get started. Recently there was a paper published entitled, "PCG: A Family of Simple Fast Space-Efficient Statistically Good
Algorithms for Random Number Generation" [7]. The accompanying website [8] has
a lot of great information about related random number generators. I hope to read
through this paper soon to catch up with the state of the art.

# References

- [1] [https://en.wikipedia.org/wiki/Diehard_tests](https://en.wikipedia.org/wiki/Diehard_tests)
- [2] [https://www.phy.duke.edu/~rgb/General/dieharder.php](https://www.phy.duke.edu/~rgb/General/dieharder.php)
- [3] [http://www.pages.drexel.edu/~bdm25/testu01.pdf](http://www.pages.drexel.edu/~bdm25/testu01.pdf)
- [4] [http://csrc.nist.gov/groups/ST/toolkit/rng/index.html](http://csrc.nist.gov/groups/ST/toolkit/rng/index.html)
- [5] [http://www.cs.hku.hk/cisc/projects/va/details/mindis.html](http://www.cs.hku.hk/cisc/projects/va/details/mindis.html)
- [6] [https://onlinecourses.science.psu.edu/stat414/node/322](https://onlinecourses.science.psu.edu/stat414/node/322)
- [7] [http://www.pcg-random.org/pdf/toms-oneill-pcg-family-v1.02.pdf](http://www.pcg-random.org/pdf/toms-oneill-pcg-family-v1.02.pdf)
- [8] [http://www.pcg-random.org/](http://www.pcg-random.org/)
