---
layout: post
title: Block Ciphers and Random Access
---

TODO: start with cigital post about encrypted file, and how that was ECB (easily detected)
TODO: transititon to what other block cipher modes you might use and why? talk about different use cases (parallelizable, random access, etc)
TODO: give table of existing modes and their properties
TODO: focus on random access, talk about CTR (with expression) and then AES-GCM, which is based on them
TODO: bring up large problem (authenticating random access requires you to authenticate the entire file... what if we had authentication for blocks?)
TODO: write code to run the experiment (AES-GCM with intermeidate MACs) on a large file, client/server, random seeking in a file, and then test the throughput


