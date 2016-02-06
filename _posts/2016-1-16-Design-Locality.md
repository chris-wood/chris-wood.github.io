---
layout: post
title: Design Locality
comments: true
---

Coupling and cohesion are two properties that drive many software design decisions.
Coupling refers to the amount or degree of interdependencies between modules. High
coupling is foul because it means changes in one module may affect others and
is also indicative of incorrect abstractions. Cohesion refers to the number of
duties for which a particular module is responsible. A highly cohesive module is one
that does few tasks and does them well. It's never a good idea to spread a module too
thin. Clearly, high cohesion is best.

These properties are often considered separately but assessed in unison. For instance, I was
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
behavior [1]. It would be inappropriate, for example, to put a function called
"createFile" or "sendLogToServer" in this module.

Experience is the only way to hone your skills and be able to identify code smells
indicative of poor designs. However, there are other ways to detect smells without
measuring these two metrics separately. Having worked on a fairly diverse set
of projects and software systems, I've observed correlations between well-designed
software (according to these two metrics) and easily observable properties. Software
that exhibits such a positive correlation has what I refer to as good *design locality*.
The idea of design locality is simple: a design *of a feature* exhibits
high locality if it is well-contained in a set of closely-related files and modules.
In other words, it does not have implementation dependencies that are located in or
spread out among other modules and the implementation logic is contained in
modules (and files) that are in close logical proximity.

Intuitively, poor locality means that a design is hard to comprehend because it
involves understanding a larger piece of the system. This difficult can manifest
itself when seemingly simple changes to a design or feature require one to understand
an unnecessarily large part of the system. Poor locality leads to a sparse
representation of the problem where relevant details are scatted amongst other
pieces of code that are necessarily important. A simple analogy to illustrate the difference
good and bad locality is the difference between storing a sparse matrix as an
actual two-dimensional data structure containing many zeros (e.g., unnecessary
details) or as a list of lists without any added fluff.
The difference is important. Leaner representations of the problem are easier
keep in memory at a time. This makes the design easier to understand and, as
a consequence, eases the development experience. Put simply, designs with good
locality are those which can be easily understood without excavating through
unnecessary code.

---

To give an example of how poor locality can manifest itself in code, I'll describe
an old piece of the PARC security library used in the CCNx project [2]. There once
were modules called _PublicKeySignerPkcs12Store_ and _SymmetricSignerFileStore_.
If you're not sure what these do from the name alone, you're not alone. Are they
key stores? Are they responsible for signing things? If so, what things do they sign?
Are symmetric keys considered to be digital signature keys? The answer to many
of these questions was "yes" and as a result the API for each of these modules was quite bloated.
They both conformed to a common interface which provided functions to obtain
a public key, private key, and certificate digest, extract DER encoded keys,
obtain a _hasher_ used for the signature algorithm (yes, I know...), get the
signature algorithm details (e.g., RSA, DSA, etc.), and sign a message (a buffer).
This once-specialized module mutated into a blob over time as the need for key-
and signature-related operations spread throughout the CCNx code base. This led to
dependencies in several different libraries for different purposes. For example,
the Libccnx code, which implements the CCNx network stack, uses it for computing
digital signatures on egress messages. Other parts of the code use it to build
key stores and extract public keys to create identities. Given this breadth, the
design is clearly not local. I set out to refactor this design when I couldn't
easily accomplish the simple task of creating a public key. I had to read
and understand unrelated modules and unit tests in order to do this (i.e.,
the Libccnx signing code).

To refactor these modules I divvied them up based on their apparent responsibilities.
Specifically, I split the modules into separate key stores, public and private key management
modules, and signing hierarchies. Each of these were given a succinct and concise interface that is used
to perform operations relevant to the module. For example, key stores only handle storing
and extracting keys. To create a public key I now only need to examine the public
key module. If I wanted to build a self-signed certificate, I look at the
certificate module. If I wanted to construct something to sign a message (buffer),
I look at the signer and its unit tests. There were longer any cyclical dependencies
between these modules; Key stores depend on certificates, signers depend on key stores,
etc. This is clearly a much more narrow design since a single feature (e.g.,
signing a message) is mapped to a well-defined and highly cohesive (set of) module(s).

To illustrate the reduction in locality, I measured the dependencies before and
after the refactoring. The reduction is measured with respect to the (PKCS12
variant of the) key store module and the signer. The dependencies are simple counts
of how many times the relevant modules were included for use when a key store was needed.

```bash
$ grep -i --include \*.h --include \*.c '/parc_PublicKeySignerPkcs12' -r * | wc -l
      57
```

```bash
$ grep -i --include \*.h --include \*.c '/parc_Pkcs12KeyStore' -r * | wc -l
      18
```

```bash
$ grep -i --include \*.h --include \*.c '/parc_PublicKeySigner' -r * | wc -l
      51
```

As you can see, only 18 out of 57 modules needed to use the _PublicKeySignerPkcs12Store_
module for PKCS12 key stores. The new design relocates this functionality to the
key store module and therefore improves the overall locality.

---

Design locality is by no means a formal metric. It's a principle that I try to
use to guide my design decisions. As I mentioned at the beginning of this piece,
the intention is to use this to drive low coupling and high cohesion in designs.
One might then ask, "how do they relate to coupling and cohesion?". Consider it
this way. Narrow designs (i.e., those with good locality) necessitate minimal
coupling since they do not, by (my informal) definition, contain dependencies on
many other modules. The modules involved in the implementation of a design are
relatively small. Conversely, wide designs cross modules and include unrelated
code. As dependencies that may grow far and wide, high coupling is produced.

Let's now consider cohesion. As I've described it, a design with good locality
can be implemented as a single, massive module with many unrelated functions.
Moreover, the designs of many features can map to such a module. This
may be indicative of a design with poor cohesion. However, it may also reflect the
fact that a module has a very high degree of cohesion and that it is being reused
appropriately. For example, many features require logging. The implementation of
these features will all depend on some logging module. That doesn't mean the logging
module has low cohesion (in fact, it's quite the contrary).

![Design to module mapping (i.e., module dependencies).](/images/posts/design_module_mapping.png)

Clearly, the mapping of a design to its constituent modules must be more fine-grain
to determine the cohesiveness of the module. It must map features to parts of modules.
This mapping exposes the information we need in order to identify where in the
system are points of low cohesion. To illustrate, consider the following slightly
modified mapping.

![Function-level module dependencies.](/images/posts/fine_grain_design_mapping.png)

The left-hand design has one module that contains three separate functions used
by three separate designs. This is clearly the wrong level of abstraction. Refactoring
the module to separate these functions into their own modules which are then
invoked by a parent module (or API) cleans up the dependencies and increases
the cohesion of the once-blob.

I do believe I've rambled enough for now, so I'll end here. In the future I'll
attempt to document the methods and software I use when measuring coupling and
cohesion in my own projects.

# References

- [1] http://mortoray.com/2015/04/29/cohesion-and-coupling-good-measures-of-quality/
- [2] http://blogs.parc.com/ccnx/
