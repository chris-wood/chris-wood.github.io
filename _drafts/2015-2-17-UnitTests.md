---
layout: post
title: Unit Tests. Write Them. End of Story.
comments: true
---

I think somewhere along the line between classes at RIT and my current internship I lost sight of the value of unit tests. In school they seemed tedious, and in fact they are. I don’t think there’s an engineer out there who will claim to be passionate about writing unit tests. The problem in school was that we were simply writing unit tests, not using them for anything that seemed meaningful. Our class projects were isolated and short-lived — never meant to be used or maintained by others down the road. This is where academia falls short and time in industry shines. Experienced software engineers know that (well-written) unit tests serve many useful purposes, including:

Acting as a form of living, breathing documentation and code examples.
Letting a developer feeling out a class or module API so see what works well and what parts are absolutely horrible.
Making sure the damn code is correct!
From a “user’s” perspective, code without unit tests makes it the programmer’s job to figure out how a class or module is used through the documentation. And let’s face it, if there’s anything people enjoy less than writing unit tests, it’s writing documentation. Plus, even if documentation is present, there’s absolutely no guarantee that it’s up-to-date. Comments for highly volatile snippets of code become outdated very quickly, and often times they’re the last thing to fix after a change. As a developer that needs to figure out how to properly use a class to perform a particular function, there is nothing more irritating than playing the guessing game with documentation. It’s often better to just read the implementation and figure out what’s going on yourself. Now, there’s a time and place where understanding the internal implementation is important, but it’s certainly not when a developer is just looking to use an existing class or module with minimal effort.

From a developer’s perspective, unit tests enforce a test-driven development (TDD) style of engineering. If you aren’t familiar with TDD, then you’re doing something wrong. Spend some time learning it and applying it in practice. Unit tests are often the first working examples of an API; they enable a developer to see what works and what doesn’t to improve the quality of the internal implementation or external API. Driving the implementation with unit tests is a fundamental facet of TDD and one that cannot be overstated.

Finally, from the code’s perspective, unit tests serve to make sure the code is correct. What’s the point of spending hours in front of the laptop hacking away and writing bug-ridden code? Not only are you wasting your own time, but you’re probably wasting countless hours later down the road for users who try to determine which part of the code is broken. Users of your class should not have to deal with probabilistic correctness — your code should work as specificed. Unit tests help ensure the probability of correct code is 100%, or as close to that ideal as possible.

There are obviously other reasons to write unit tests, but clearly my rant should convince you that they’re a good idea. So, please, write them and keep them up to date. Have them run as part of your build system. Don’t check in code that doesn’t pass the unit tests. And lastly, always assume that the user’s of your code and unit tests is a psychopath that knows exactly where you live. If you’ve ever suffered due to poor unit tests or documentation, you know how the feeling…
