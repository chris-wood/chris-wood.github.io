---
layout: post
title: Sequence Numbers Are Considered Harmful?
---

% http://www.cs.ucr.edu/~zhiyunq/pub/sec16_TCP_pure_offpath.pdf

- review the use of sequence numbers (byte offsets in the stream and ACK offsets)
- describe the off-path attack
- core problem: easy to guess
- remedy: make them hard to guess with minimal state
- idea: OWF to advance sequence number counter

