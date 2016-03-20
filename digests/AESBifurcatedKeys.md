---
layout: digest
title: AES-GCM using two independent keys
date: 2016-3-18
categories: digest
---

## Context

AES-GCM is the most popular AEAD algorithm to provide ciphertext confidentiality
and integrity. It uses a single key to encrypt the content and generate the 
corresponding MAC. This prevents parties between a sender and receiver from
verifying the authenticity of the data without revealing the encryption key 
and therefore subjecting the data to inadvertant or malicious modification.

## Purpose

Specify a mode of operation for AES-GCM that uses a separate encryption and 
authentication key. This would allow only end-points in the protocol to view
the plaintext data while delegating authentication check privileges to intermediate
nodes, e.g., network nodes forwarding packets protected by AES-GCM. Since this
algorithm is employed in protocols such as IPSec and TLS, this has significant
value. 

## Main Points

The bifurcated key approach works as follows:

1. Define two separate keys: Ke and Ka (encryption and authentication).
2. Define the AES-GCM GHASH function using Ka. Specifically, compute the H value
as the encryption of 0^128 using Ka -- not Ke. 
3. Define the plaintext encryption function using Ke (as in the standard AES-GCM). 
Specifically, counter values are encrypted with Ke and then XOR'd with the plaintext
to obtain the ciphertext.

The authors do not define a protocol to derive Ke and Ka as it is out of scope.

## Takeaways

Standard AES-GCM uses a single key for encryption and authentication checks. 
This draft proposes the use of two *independent* keys for encryption and 
authentication without compromising either security property. This allows 
authentication checks to be delegated to nodes that do not possess the 
encryption key, e.g., network forwarders and middleboxes.

