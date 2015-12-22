---
layout: post
title: Key Management for CCN
---

Key management is hard to get right. 

-- different types of keys, different scenarios (online vs offline), issues of authentication and trust, key lifetime, etc.
-- focus on object encryption via public-key algorithms (not channel encryption that relies on key establishment protocols like CCNx-KE, which is like TLS)
-- key ideas: cryptographic identities, key generation, private key distribution (RSA case)
-- **not considering how to obtain decryption keys (different approaches a la CCN-AC and NDN NBAC)


https://speakerdeck.com/bdpayne/key-management-in-aws-how-netflix-secures-sensitive-data-without-its-own-data-center
https://d0.awsstatic.com/whitepapers/AWS_Securing_Data_at_Rest_with_Encryption.pdf
https://d0.awsstatic.com/whitepapers/compliance/AWS_Security_at_Scale_Logging_in_AWS_Whitepaper.pdf
https://d0.awsstatic.com/whitepapers/Security/Intro_to_AWS_Security.pdf
https://d0.awsstatic.com/whitepapers/architecting-for-genomic-data-security-and-compliance-in-aws-executive-overview.pdf
https://d0.awsstatic.com/whitepapers/Security/AWS%20Security%20Whitepaper.pdf
https://d0.awsstatic.com/whitepapers/aws-security-best-practices.pdf
