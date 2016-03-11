---
layout: post
title: Suppress Speculative Design
---

When you have time to kill, speculative design is a fun topic to get into with a
group of software engineers. Everyone is free to pitch their wildest ideas about
what a particular design should provide without any repercussions. There is no
filter or pushback from others in the group. Unfortunately, despite its entertainment value,
it's also extremely damaging (to progress) when an actual design is at stake.

Speculative design is feature creep with a different hat on. Everyone knows that
feature creep is a dangerous side effect of the software requirements elicitation
process. It's what happens when people stray from what is actually needed in a design to what
would be nice to have in their fantasy world. Now, don't get me wrong: creative
thinking like this drives progress on many fronts. But when it comes to the requirements
and design part of the SDLC, this speculation causes trouble for one simple reason:
not everyone lives in the same fantasy world. What might be important to you might
not be important to someone else. Moreover, disagreements about completely unnecessary features
often steal the spotlight from the core design issues that are really up for discussion.
And that lengthens the time it takes to reach consensus on a design to move an idea
to reality with code.

In my area of work, a solid design is usually a prerequisite for an implementation.
We don't start writing code until we have general consensus on the general design.
I'm not necessarily in favor of this process (especially in the context of research
code), but it's how my team operates. We also suffer from a great deal of speculative
design. We once spent *months* disagreeing about a particular design that led to
team members harboring resentment towards others. So not only did it take an extremely
long time to get from design to code, but our morale was hurt in the process.
What's most interesting about this particular case is that design was *not* the remedy
to our problem. When we got caught in the speculative design loop the only way out
seemed to be presenting something tangible that the team could discuss. That is,
code.

And that's exactly what I did (with the help of a colleague). He and I took what we
thought was the core design and feature set, implemented it, and presented it to
the group. Running code spoke orders of magnitude louder than fuzzy fantasy features
that might never be useful in certain circumstances. Running code broke the infinite
loop and got us on the path to progress again. Why? Because data speaks louder than ideas.

This entire experience taught me a few valuable lessons:

1. Avoid speculative design and feature creep at all costs. You may not truly understand
the pain they cause until they burn you.
2. If you get stuck in speculative design, seek something tangible to break out of the cycle.
In our case, code was the answer. In other cases, proofs, customer use cases, etc.
may be the answer.

So maybe falling victim to speculative design is useful... at least once.
