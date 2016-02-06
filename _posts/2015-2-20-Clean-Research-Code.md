---
layout: post
title: Clean research code
comments: true
---

As a student and part-time software engineer, I write a lot of code. Some of it lives ephemeraly until the "experiment" phase of a project is complete, and some of it has much more longevity. Does this mean that research code -- the short-lived product of a single project or paper -- deserves less tender loving care? Based on the overwhelming majority of unpopular open source projects that are born in academia, the answer appears to be, inarguably, yes.

Much of the research code I encounter serves one of two purposes: (1) to test some hypothesis, algorithm, or design, or (2) show the feasibility of a system by actually building it. In both cases, the underlying reason to write this code is usually to support the contents of some publication. Academic conferences have deadlines, and so code supporting conference submissions also has a deadline. Pressure in time and from advisors to finish code, especially when that pressure is placed on students who lack professional development experience, forces shortcuts and bad development practices, both of which produce smelly code.

In the end, people involved with the lifecycle of this code rarely care about its quality if the paper is published. The code is usually shipped only once -- in the publication. There are typically very few, if any, users of the code outside of its place origin. One might argue that this means the code is no longer living, and that non-living code does not require maintenance. In a way, that's correct. After the paper and corresponding code are accepted for publication, students move on to new endeavors that require new code.

Students with this mentality suffer in multiple dimensions. For starters, they perpetuate bad code. Secondly, the code often does not facilitate reuse. This is particularly troublesome if the student is working on a series of related problems which could benefit from shared code. One of the many facets of clean code is that it facilitates reuse in related or even different contexts. Thirdly, and perhaps most importantly, if the code needs to be revisited, maybe because someone raised a question about the results obtained from it or simply because they're interested in the design, the student is usually forced to re-learn the software from scratch. I ran into this situation myself when asked to describe a particular implementation detail regarding the software used for my M.S. thesis. It took longer than necessary to formulate an intelligent response for the inquirer.

In my opinion, although hitting publication deadlines is a crucial for those interested in a research career, research code quality should not suffer to make it happen. Take the time to build quality code. Even if the benefits aren't immediately apparent for the code being worked on, sound development practices make you a better programmer. In due time, they will resonate in your code, and you will prosper.
