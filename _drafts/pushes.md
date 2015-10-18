---
layout: post
title: Professional Pushes
---

Code reviews are a normal part of my day at PARC. No bug fix, enhancement, or feature is committed without
going through the proper channels. Since this is a fairly routine process, we sought to automate it
as much as possible. A big part of this automation included developing in-house static analysis tools
to measure the quality of all code changes, A subset of the metrics we compute include:

- Module coupling and cohesion: This metric is based on C namespace organization rules and the
degree to which we adhere to these rules. The names of files, functions, parameters, enumerations,
structs, and macros are all indicators of the amount of coupling (inter-dependence) and cohesion
(the degree to which a module does its single job--no more, and no less).
- Style: This metric is based on how far code changes deviate from our C coding style guidelines.
- Coverage: This metric refers to the amount of unit test coverage for code changes.
- Cyclomatic complexity: Loosely speaking, this metric seeks to quantify the nonlinear complexity
of our code. Larger numbers of branches (i.e., more spaghetti code) typically leads to larger
values.
- Vocabulary: Documentation is difficult enough to write and maintain throughout the lifecycle
of software. It is even more difficult to write documentation that is not overly difficult
for readers and users to parse. After all, why bother writing documentation if nobody finds
it useful? This metric quantifies the diversity of distinct vocabulary terms used to document
code changes.

The tools we wrote provide some more useful information, but these are the big ones. Before
requesting a code review, a developer shall use these tools to measure their changes. Any
metric that falls below an acceptable threshold (e.g., a unit test coverage percentage of
50%) must be corrected before beginning the review process. After all measures are within
acceptable margins, the review process begins. If all goes well there is a minimal amount of
churn between the developer and reviewer before the changes are accepted and committed.

There is certainly much left to be desired to make this process more formal, e.g., we could
be using Gerrit or Phabricator to help consolidate our bug tracking and code review systems
and processes. However, given our team size, we seem to have found a productive solution.
Still, though, what's missing from this equation is an element of professionalism. Reviewing
someone else's code is a very difficult task. To do it successfully you must understand
the purpose of a code change, the context in which it was made. A professional programmer may
(read: should) ask the developer to rationalize or explain design choices if they're not immediately
clear. However, it is unreasonable to expect the reviewer to understand the entire thought
process of the developer. This means that the cleanliness of the code under review is (for
obvious reasons) the responsibility of the developer.

This is where I want to bring the issue of code review and pre-commit quality checks back
to full circle. As a developer, it is not enough to be able to show that your code meets
some suitable measure. It is possible for "unclean" or "dirty" code to meet these objectives.
As a developer, you are ultimately responsible for producing clean code, and that includes,
at a minimum:

- Seeking simple solutions that can be easily understood by users of said code.
- Structuring your code in a readable and understandable fashion.
- Being able to describe exactly why your solution works. That is, don't just plug in
the first tidied up answer from SO. Understand both the problem and solution.
- Refactor, simplify, and refactor some more as needed.

These are just the first ones that come to mind for me. Any developer who takes his or her
craft seriously should invest in and read Bob Martin's Clean Code book for more tips,
suggestions, and hidden treasures.

At the end of the day, pushing production code requires more than achieving some basic quality
checks. It requires the code to be clean, and for the developer to understand exactly
*why* it is clean. Do not push code that does not satisfy these two conditions.
