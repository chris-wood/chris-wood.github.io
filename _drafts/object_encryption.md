---
layout: post
title: Channel and Object Encryption
---

- channel and object encryption is analogous to encrypting data in transit and at rest.

- cryptography is used to protect data communication, cite schnieier blog [1]

- in applied cryptograph (schnieier), he describes data storage as data that someone wishes
to communicate to themselves over time. so, in a way, it is data in transit. or,
as schnieier puts it data storage is a subset of data communication.

- Channel encryption keys are ephemeral (TLS, for example) and exist for the duration of the session, but data keys must exist
for the duration of the data. this is the common problem of key management. this
is no longer a cryptographic problem -- it's a problem of classic computer security
requiring access control and authorization to solve. it does not matter if data
is encrypted if an attacker can get access to the decryption key(s). In this sense,
why bother encrypting the data at all? (I'm not advocating this approach, just
pointing out one conclusion to be derived from this effect).

- In channel-based encryption the two endpoints control the keys. They are short
lived, the attack window (during which they can be stolen) is small, and exposure is
typically limited to two parties.
- In object encryption, it's not clear who controls the keys. Let's take S3 as an
example. AWS S3 buckets provide server-side encryption with server-generated and
managed keys and client-generated keys [3]. Regardless of who maintains the keys,
they must be secure from unauthorized access. The server and client are solving
the same problem.

- Amazon uses their KMI (key management infrastructure) to protect and manage these keys.
THe KMI is composed of a storage layer for plaintext keys and a management layer that
authorizes requests for these keys [4]. A HSM (hardware-security module) is a special
hardware component, typically comprised of a co-processor and taper-resistant protections,
for managing the lifecycle of cryptographic keys. That's a discussion for another time.

- what does all of this mean? the cost of object encryption is identical, if not greater,
than encrypting data at rest (due to scale). In TCP/IP applications where the the channel
moves arbitrary data from a server to a client (or vice versa). In this case, the benefit
of encrypting data at rest depends on the cost of losing that data (e.g., credit card or other PII information).
In CCN, where the server moves *specific data* to a client, the benefit of object
encryption is that, with proper key management facilities, it can subsume channel
encryption. but what does this buy? nothing, really, if we care about privacy,
since we'll have to use channel encryption anywhere.




# References
- [1] https://www.schneier.com/blog/archives/2010/06/data_at_rest_vs.html
- [2] B. Schneier. "Applied cryptography: protocols, algorithms, and source code in C." John Wiley & Sons, 2007.
- [3] http://docs.aws.amazon.com/AmazonS3/latest/dev/ServerSideEncryptionCustomerKeys.html
- [4] https://d0.awsstatic.com/whitepapers/aws-securing-data-at-rest-with-encryption.pdf


https://securityblog.redhat.com/2015/04/01/jose-json-object-signing-and-encryption/
http://tools.ietf.org/html/draft-ietf-jose-json-web-encryption-40
