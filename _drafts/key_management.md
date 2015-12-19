---
layout: post
title: Key Management for CCN
---

Key management is hard to get right. 

-- different types of keys, different scenarios (online vs offline), issues of authentication and trust, key lifetime, etc. 
-- focus on object encryption via public-key algorithms (not channel encryption that relies on key estbalishment protocols like CCNx-KE, whcih is like TLS)
-- key ideas: cryptographic identities, key generation, private key distribution (RSA case)
-- **not considering how to obtain decryption keys (different approachs a la CCN-AC and NDN NBAC)
