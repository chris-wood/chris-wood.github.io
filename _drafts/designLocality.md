---
layout: post
title: Design Locality
---

simple definition: a design is well-contained in set of closely-related files and modules, it does not have design dependencies that are located in other modules, and the logic of the design is contained in files that are in close proximity

why is it important: designs with poor locality are hard to understand all at once (more variables with unrelated or independent factors). good locality means that the variables are small or closely related to be well-understood by a single person. 

Example of a wide design: TODO
Example of a narrow design: TODO

how to measure it? module coupling and cohesion, dependencies, etc (maybe come up with a formula that depends on coupling and (inward and outward) dependency count)

Wide designs are hard to understand, hard to keep in your brain at a single point in time, etc

