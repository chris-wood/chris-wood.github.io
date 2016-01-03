---
layout: post
title: Design Locality
---

simple definition: a design is well-contained in set of closely-related files and modules, it does not have design dependencies that are located in other modules, and the logic of the design is contained in files that are in close proximity

why is it important: designs with poor locality are hard to understand all at once (more variables with unrelated or independent factors). good locality means that the variables are small or closely related to be well-understood by a single person.

EARLY EXAMPLE OF PARC CCNx CODE: CCNx validator code used to sign outgoing messages
Example of a wide design: CCNx validation code (Libccnx code for validating, Libparc code for security stuff, files with different modules, bad. had to understand the CCNx validator code to get keys and... ugh)
Example of a narrow design: Refactored design for the validator, keystore, and key relationship. (to understand and work with keys, just look at keys and keystores, to understand signers and validators, you must naturally know about keystores and signers.).

how to measure it? module coupling and cohesion, dependencies, etc (maybe come up with a formula that depends on coupling and (inward and outward) dependency count). signs that foreshadow it? rippling (non-isolated) changes.

how does it relate to coupling and cohesion? narrow designs, as described, necessitate
minimal coupling with maximal cohesion. It's a package deal and seems to be easy
to detect together (than measuring coupling and cohesion separately).

measure coupling: TODO (dependency count, but still difficult)
measure cohesion: TODO (hard! since this is qualitative)


%%% TODO: one way to compute this.
<!-- Afferent coupling: Number of responsibilities
Efferent coupling: Number of dependencies
Instability: Ratio of efferent coupling to total coupling (Afferent + Efferent).
Instability is supported in various code metric tools. -->

Wide designs are hard to understand, hard to keep in your brain at a single point in time, etc
