---
layout: post
title: Tuning Memory Hard Functions
---

XXX: intro

Typically, one chooses the MHF parameters such that the execution time is "within" reason
for a single execution. For example, the Argon2 paper [xXX] suggests one to do the following.

1. Detemrine the number of threads that can be used.
2. Determine the memory capacity.
3. Tune the number of rounds until the runtime is "within reason" for a single execution.

In a recent project I had to solve a different problem. Namely, given a reasonable runtime,
I needed to find the parameters such that execution met but did not exceed this runtime bound.
This turned out to be a bit of a challenge but also a good excuse to brush up on some optimization
algorithms from college.

# Optimization Algorithms

There is a plethora of algorithms available for this type of discrete optimization problem.
To me, it made the most sense to stick with graph-based search algorithms. Why? Try to picture
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
$$\lim_{x \to \infty, x \to \infty}f(x,y) = \infty$$.) Instead, we want to find the values of
$$x$$ and $$y$$ that maximizes $$f(x,y)$$ but does so under the constraint that $$f(x,y) \leq N$$
for some bound $$N$$. Thus, going back to our tree example, this essentially corresponds to
finding the node in the tree whose value is largest but not larger than $$N$$.

If $$f(x,y)$$ was easy to compute and the search space was small, we could simply construct the
entire tree by exhaustively generating all parameters, computing their values, and then searching
for the optimal node. In our case, however, let's assume that neither of these conditions are
true. The search space is potentially quite large and the function takes time to compute. So we'd
like to do better. This is where the optimization algorithm comes in.

I looked at three different types of algorithms: DFS, hill climbing, and branch & bound.
DFS is a standard algorithm and needs no clarification. Hill climbing is a related variant
that tries to cut down the search space. But it can get stuck in a local maxima since there
is no backtracking in place. The algorithm, modified to include our bound $$N$$, works as follows:

{% gist 1dade523510628698b5fa9fb0900b003 %}

As you can see, this algorithm traverses the search space in a greedy fashion. In our
tree, this algorithm would always choose the branch that results in the best overall value
that hasn't exceeded the bound.

XXX: draw tree in which this doesn't result in the maximal value, but a local maxima
