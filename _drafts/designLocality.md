---
layout: post
title: Design Locality
---

Coupling and cohesion are two properties that drive many software design decisions. Coupling refers to the amount or degree of interdependencies between modules. High coupling is foul because it means changes in one module may affect others. High coupling is also indicative of incorrect abstractions. Cohesion refers to the number of duties for which a particular module is responsible. A highly cohesive module is one that does few tasks and does them well. It's never a good idea to spread a module too thin. Therefore, high cohesion is best. 

These properties are often considered separately but in unison. For instance, I was taught to design software that exhibited low coupling and high cohesion. Although they are both measured separately, they were (are) almost always the goal. Coupling is typically trivial to quantify. Example methods are based on computing the number of module dependencies via header inclusion, library imports, and function calls. Cohesion is somewhat trickier to quantify.

XXX


What I often found lacking in my toolbelt was a way to measure both simultaneously. Or, rather, a principle that, if followed, would lead to designs with ideal coupling and cohesion. Having worked on a fairly diverse set of projects and software systems, I feel equipped to formalize what I call *design locality*, as something to fill this void.







simple definition: a design is well-contained in set of closely-related files and modules, it does not have design dependencies that are located in other modules, and the logic of the design is contained in files that are in close proximity

why is it important: designs with poor locality are hard to understand all at once (more variables with unrelated or independent factors). good locality means that the variables are small or closely related to be well-understood by a single person.

EARLY EXAMPLE OF PARC CCNx CODE: CCNx validator code used to sign outgoing messages
Example of a wide design: CCNx validation code (Libccnx code for validating, Libparc code for security stuff, files with different modules, bad. had to understand the CCNx validator code to get keys and... ugh)

SymmetricSignerFileStore: signing, keystore managemet (certificate and public key digests)
PublicKeySignerPkcs12Store: signing (tagging), keystore manamgent, and more

this mixes: certificates, keystores, public and private keys, signing and verification
upper-layer utilities relied on these modules for separate purposes (creating key stores, performing signing, etc.) -- they did more than necessary
why is it wide? to understand the key store, you had to go through signers, veriifers, and more. this is too much. they were blobs

Example of a narrow design: Refactored design for the validator, keystore, and key relationship. (to understand and work with keys, just look at keys and keystores, to understand signers and validators, you must naturally know about keystores and signers.).

everything separated into appropriate modules: keystores (certificate families and factories), keys, and signers (which use keystores), and verifiers (which use keystores). There are no cyclical dependencies between modules: keystores depend on certificates, signers depend on keystores, etc. 
why is it narrow? a single feature maps to a small number of well-defined and highly cohesive modules

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

design locality for a set of features can give an indication of coupling: if many features share the same set of nodes then this is an indication of possibly poorly-defined abstractions or interfaces (give the example above)
TODO: come up with an equation based on set intersection of shared vertices and diameter of feature graph subsets


---


- pyramid of memory and the parallel idea with keeping software in a register or cache (actively working on it), memory (have to do some work to remember what was happening), and on disk (have to re-learn the code again) -> implications on design temporality


Wide designs are hard to understand, hard to keep in your brain at a single point in time, etc
