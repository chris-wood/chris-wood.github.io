---
layout: post
title: IETF'95 Notes:
---

This week I'm in Buenos Aires for IETF'95. I'm going to use this post as 
my running notes from the meetings. Please excude the brevity.

# Sunday: ICNRG Interim Meeting

- Greg White and I took turns taking notes (see [here](https://www.ietf.org/proceedings/interim/2016/04/03/icnrg/minutes/minutes-interim-2016-icnrg-2)).
- Major highlights:
    - Stephen Farrell argued that confidentiality and authorization cannot be achieved under the following assumptions:
        - Untrusted caches.
        - No new cryptographic primitives.
        - No host-based networking.
        - Comment: I don't equate service-based networking with host-based networking. 
            - The former allows confidentiality and authorization without a paradox.
    - NDN folks pushed for more exploration of the "privacy design space" driven by applciation needs and requirements.
        - No consensus among the community about what's the right thing to do.
    - We need to revisit the term "session" in the context of CCNx-KE. It's an overloaded term that carries TLS-like baggage.
    - We shouldn't use the phrase "DTLS-like" to describe CCNx-KE. 
        - Perhaps the definitions we use in the draft are not sufficient.
    - Presented the MoveToken presentation that specifies how to build in Kerberos-like delegation in CCN.

# Monday: UTA, COSE

## UTA

- Talked about different use cases for TLS in SMTP servers.
- Action: read the drafts.

## COSE

- Discussed issues about the latest COSE message syntax
- Discussion about how algorithm identities (and agility) are handled
- Proposed a new key exchange mechanism using COBR
    - Parties exchange COSE(g^(secret)) and use standard DH to create a shared key
        - e.g., ECDH-ES HKDF-256, but no mechanism is specified in their draft
- Action: read the COSE draft.

# Tuesday: TLS, LURK, CURves, TWG

## TLS

- Updated authentication blocks based on Hugo's SIGMA (SIGn and MAc) class of protocols.
    - An authentication block is composed of a certificate (optional), CertificateVerify (signature, optional), and Finished (MAC, mandatory).
    - Signature is computed over hash of "handshake context" and certificate.
    - MAC is computed over the "handshake context", certificate, and signature.
- Introduction of signature algorithm negotiation structure (or enumeration) that defines each of the signature algorithm, curve, and hash type tuples together.
- Long-term secrets last about 24 hours. 
- Removed 0-RTT client authentication
- Removed 0-RTT DHE-based mode
- 0-RTT exporters (??)

## LURK

- Two approaches discussed so far:
    - Time-bound certificate/keyfile pairs
    - Interaction between terminating edge server and key server to generate an "identical" TLS transcript to the cient
- Uncertainty about client interoperability
    - One of the goals is to make this work without changing any clients
- Discussion about standardizing anything with LURK
    - Unclear about the scope of the problem statement (CDN is not the only use case)
- Concerns about creating a signing oracle (i.e., making the key server a signing oracle for any general purpose thing).
    - If the protocol is tailored for a specific use case then signing would occur for specific purpose

## CURves

- No notes takes.

## TWG

- Transport area open group
    - OS extensions for user-space transport stacks (https://www.ietf.org/proceedings/95/slides/slides-95-tsvarea-2.pdf)
    - Wrote their own framework that supports multiple transport stacks in user-space
    - Comes with a custom packet generator

# Wednesday: DPRIVE

## DPRIVE

- Different types of usage profiles
    - Strict, opportnistic, and no privacy
        - No privacy: cleartext DNS queries
        - Opportunistic: the use of cleartext as the baseline with use of encryption (and authentication) when possible
        - Strict: only encryption and authentication allowed (i.e., "authenticate or die")
            - Must be DNSSEC validated
    - Various attacker models:
        - Passive and active: only secure against an active attacker in the when authentication and encryption is used
            - Why would anything else be done?
        - Comment: get rid of opportunistic in favor of "strict and 'maybe private but we make no guarantees'" 
            - Opportunistic decision is misleading since it doesn't offer privacy in many cases (it falls back to cleartext)
    - 0-RTT problems:
        - client authentication can't be in the first round (not a stopper for DNS)
            - 0-RTT is basically session resumption
        - FS guarantees are different (less...)
        - No replay protection
            - queries can be replayed to figure out what's in the cache (by observing resolver output)
        - Additional client linkability
            - resolver can figure out that client has moved since session resumption occurred
            - adv can monitor session IDs and tie them together as a client moves

## MPTCP

- No notes taken.

# Thursday: ICNRG, SAG, KITTEN, TCPINC

## ICNRG

- Presented the specification updates and latest naming document.
    - Comment: the term "nameless objects" is confusing (but we don't actually use that in the documents).
    - We could benefit from a document that describes the semantics of various name types. 
        - The "policy" has always been that if a name type is in the naming document then it must have an associated document that outlines the protocol around that type (e.g., the Chunking and Versioning documents).
- Proposed moving towards a WG formation. 
- Discussed the topic of principles yet again.

## SAG

- Presented NSEC5 -- a new way to achieve authenticated denial of existence with DNS that does not allow zone enumeration.
    - Action: read the ePrint paper. 
- Discussed a general security architecture for IoT applications.

## KITTEN

- Moving away from password-based authentication and towards (a) PKI-based authentication and (b) PAKE-based key exchange.
- Not a lot of discussion but there was support for updating the initial authentication and key exchange mechanism.

## TCPINC

- tcpcrypt history and latest protocol reviewed.
- Current plan is to get the code into the Linux kernel.
    - Existing implementation is in userspace and requires some "hacks" to make proper use of it.
- An API for tcpcrypt presented.
    - Action: read the draft.
- The authors are looking for independent implementations to ensure the drafts are ready for publication as an RFC.

# Friday: CRFG

I had to leave early and wasn't able to attend the CRFG. Based on the mailing list
they talked about hash-based signatures and quantum resiliance -- two topics I
need to read more about.

# Summary

There's a lot of movement in the security and privacy area of the IETF. Many new protocols
are in flight (TLS 1.3 and tcpcrypt) and policies around new and existing protocols
are emerging. Particularly noteworthy endeavors are the DPRIVE usage profiles, 
tcpcrypt case for ubiquitious encryption, and the improvements to the DNS zone enumeration
problem with NSEC5. I need to spend time reading the other other papers in the next month
to prepare for the next IETF.
