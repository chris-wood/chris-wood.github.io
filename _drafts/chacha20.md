---
layout: post
title: ChaCha20 in Haskell
---



- goal for project: implement ChaCha20 stream cipher to in haskell (learn the algorith and get more experience with haskell)
- trivial parts: algorithm to function breakdown -- going from encryption to individual pieces of the QR function was easy. treating the alg as the composition of many smaller functions was straightforward
- sort of hard parts: data processing, type converting (string to word8/word32), mild: little endian swap and type convert (mild because there didn't seem to be a trivial function to do it for me)
- desires: replace existing tests with HUnit tests (postponed to future)

