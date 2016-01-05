---
layout: post
title: Polymorphism in C
---

need: generic interface with different implementations -- we'd like to pass around different implementations
example: keystore (used by signers and verifiers)
implementations: certificate, Pkcs12 key store, etc

show the structure implementation template
show the actual implementation for keystores

describe how it's different from C++'s dispatch table (compile time binding vs runtime binding)

