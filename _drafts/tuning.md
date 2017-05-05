---
layout: post
title: Tuning Memory Hard Functions
---

It's been a while since I wrote to this blog. That was an expected side effect of transitioning
into my post-PARC life at Apple. However, now that the dust seems to have settled, I'm trying
to get back into my routine. To that end, I'm going to pick back up on the topic of memory
hard functions (MHFs) that I discussed in the [last post](LINK). In particular, I'm going to
talk about parameter selection for MHFs.

Typically, one chooses the MHF parameters such that the execution time is "within" reason
for a single execution. For example, the Argon2 paper [1] suggests one to do the following.

1. Determine the number of threads that can be used.
2. Determine the memory capacity.
3. Tune the number of rounds until the runtime is "within reason" for a single execution.

In a recent project I had to solve a different problem. Namely, given a reasonable runtime,
I needed to find the parameters such that execution met *but did not exceed* this runtime bound.
This turned out to be a bit of a challenge but also a good excuse to brush up on some optimization
algorithms.

# Optimization Algorithms

There is an abundance of algorithms available for this type of discrete optimization problem.
To me, it made the most sense to stick with graph-based search algorithms. Why? Picture
the set of possible parameters as a tree with a root node containing default values for
these parameters. The value associated with a node is the output of our objective function
$$f(\cdot)$$ when computed on the node labels.
For example, suppose we had two parameters $$x$$ and $$y$$ to use when
optimizing $$f(x, y)$$. Moreover, let the default (minimum) values for $$x$$
and $$y$$ be $$1$$ and $$2$$, respectively. The root of our tree, or search space, will have
the tuple $$(1,2)$$. Also suppose that by increasing either $$x$$ or $$y$$ we see an
increase in $$f(x, y)$$. In other words, $$f(x,y)$$ is a monotonically increasing function.

Our goal is unlike most optimization algorithms. We are not trying to find the maximum value
of $$f(x,y)$$. (This would be impossible here since, as I've described it,
$$\lim_{x \to \infty, y \to \infty}f(x,y) = \infty$$.) Instead, we want to find the values of
$$x$$ and $$y$$ that maximizes $$f(x,y)$$ but does so under the constraint that $$f(x,y) \leq N$$
for some bound $$N$$. Thus, going back to our tree example, this essentially corresponds to
finding the node in the tree whose value is maximal but not larger than $$N$$.

If $$f(x,y)$$ was easy to compute and the search space was small, we could simply construct the
entire tree by exhaustively generating all parameters, computing their values, and then searching
for the optimal node. In our case, however, let's assume that neither of these conditions are
true. The search space is potentially quite large and the function takes time to compute. So we'd
like to do better. This is where the optimization algorithm comes into play.

I looked at three different types of graph-based algorithms: DFS, hill climbing, and branch & bound.
DFS is a standard algorithm and needs no clarification. Hill climbing is a related variant
that tries to cut down the search space. But it can get stuck in a local maxima since there
is no backtracking in place. The algorithm, modified to include our bound $$N$$, works as follows:

{% gist 1dade523510628698b5fa9fb0900b003 %}

As you can see, this algorithm traverses the search space in a greedy fashion. In our
tree, this algorithm would always choose the branch that results in the best overall value
that hasn't exceeded the bound. To see how this can result in a suboptimal node selection,
consider the following graph. The hill climb algorithm would choose the red path through
the tree since it optimizes the value of the objective function at each step. However,
with a bound of $N = 90$, the algorithm would terminate at the node with label $(2,4)$,
whose value is $80$. This is not the correct result, since the node with label $(3,2)$
yields a value of $89$, which is closer to $90$.

![Suboptimal hill climb instance](/images/hill_climb_tree.pdf)

The branch & bound (BNB) algorithm is similar to the hill climbing algorithm except that it
more intelligently trims the search space by only traversing down paths that could lead to
an optimal solution. The key to these algorithms is selection the bounding function, whose
role is to compute the optimal value of a given subproblem. Moreover, this computation
should be efficient, i.e., not require us to actually solve for the optimal value of the
subproblem. So if we are to use the BNB algorithm, then we need a bounding or estimation
function. Fortunately, MHFs (from the PHC [2]) are parameterized by their time and memory
requirements. Thus, in our context, the bounding function is a (linear) function of time parameter.
To build the bounding function $g(\cdot)$ for the MHF $f(\cdot)$, we just compute $$g(t) = tf(1)$$.
We'll use this in what follows.

# Tuning the MHF

It now seems that using the BNB algorithm is the best choice for tuning a MHF that is
parameterized by a time and memory parameter. (Of course, I'm no optimization expert. Far
from it, if that's not abundantly clear by now. So maybe there's a better way that I didn't
examine.)

XXX: code that does the tuning

# References

- [1] argon 2
- [2] PHC
