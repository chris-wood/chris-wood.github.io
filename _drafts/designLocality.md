---
layout: post
title: Design Locality
---

Coupling and cohesion are two properties that drive many software design decisions.
Coupling refers to the amount or degree of interdependencies between modules. High
coupling is foul because it means changes in one module may affect others. High
coupling is also indicative of incorrect abstractions. Cohesion refers to the number of
duties for which a particular module is responsible. A highly cohesive module is one
that does few tasks and does them well. It's never a good idea to spread a module too
thin. Therefore, high cohesion is best.

These properties are often considered separately but in unison. For instance, I was
taught to design software that exhibited low coupling and high cohesion. Although
they are both measured separately, they were (are) almost always the goal. Coupling
is trivial to quantify. Example methods are based on computing the number of module
dependencies via header inclusion, library imports, and function calls. Cohesion is
somewhat trickier to quantify. A rough estimate of can be gleaned from the relationship
between the functions of a module. For example, if all functions in a module perform
some type of logging and are all named "log_X," and the name of said function is
"Logger" (or something to that effect), it's clear that the module has good cohesive
properties. The purpose of the module can be succinctly described in a short sentence
(e.g., "this module handles logging") and the internals reflect that single
responsibility behavior [1]. It would be inappropriate, for example, to put a
function called "createFile" or "sendLogToServer" in this module.

What I often find lacking in my tool belt is a way to measure both simultaneously.
Or, rather, a principle that, if followed, would lead to designs with ideal coupling
and cohesion. Having worked on a fairly diverse set of projects and software systems,
I feel equipped to describe what I now call *design locality* as something to fill
this void. The idea of design locality is simple: a design *of a feature* exhibits
high locality if it is well-contained in a set of closely-related files and modules.
In other words, it does not have implementation dependencies that are located in or
spread out among other modules and the implementation logic is contained in
modules (and files) that are in close logical proximity.

Intuitively, poor locality means that a design is hard to comprehend because it
involves understanding a larger piece of the system. This leads to a sparse
representation of the problem where relevant details are scatted amongst other
pieces of code that are necessarily important. I consider the difference to be
between storing a sparse matrix as an actual two-dimensional data structure containing
many zeros (e.g., unnecessary details) or as a list of lists without any added fluff.
The difference is important. Leaner representations of the problem are easier
keep in memory at a time. This makes the design easier to understand and, as
a consequence, eases the development experience. Plainly put, designs with good
locality are those which can be easily understood without recursively digging
through unnecessary code.




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







Design locality is by no means a formal metric. It's a principle that I try to
use to guide my design decisions. As I mentioned at the beginning of this piece,
the intention is to use this to drive low coupling and high cohesion in designs.
One might then ask, "how do they relate to coupling and cohesion?". Consider it
this way. Narrow designs (i.e., those with good locality) necessitate minimal
coupling since they do not, by (my vague) definition, contain dependencies on
many other modules. The modules involved in the implementation of a design are
relatively small. Conversely, wide designs cross modules and include unrelated
code. As dependencies that may grow far and wide, high coupling is produced.
Let's now consider cohesion. As I've described it, a design with good locality
can be implemented as a single, massive module with many unrelated functions.
Moreover, the designs of many features can map to such a single module. This
may be indicative of a design with poor cohesion. However, it may also reflect the
fact that a module has a very high degree of cohesion and that it is being reused
appropriately. For example, many features require logging. The implementation of
these features will all depend on some logging module. That doesn't mean the logging
module has low cohesion (in fact, it's quite the contrary).

TODO: draw a picture of a design mapping to one module, many designs mapping to
a single module, and then many designs mapping to different modules

Clearly, the mapping of a design to its constituent modules must be more fine-grain.
It must map features to parts of modules. This mapping exposes the information we
need to identify where in the system are points of low cohesion.





how does it relate to coupling and cohesion? narrow designs, as described, necessitate
minimal coupling with maximal cohesion. It's a package deal and seems to be easy
to detect together (than measuring coupling and cohesion separately).

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


# References

- [1] http://mortoray.com/2015/04/29/cohesion-and-coupling-good-measures-of-quality/
