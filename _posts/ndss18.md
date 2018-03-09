---
layout: post
title: DNS Privacy Workshop at NDSS 2018
---

# ipcipher: Encrypting IP Addresses, Bert Hubert (PowerDNS)
- TLDR: Encrypt IP addresses in place (in pcap files) from one IP address to another (FPE?), allow standard Unix tools to work 
- Links:
    - http://powerdns.org/ipcipher
    - https://medium.com/@bert.hubert/on-ip-address-encryption-security-analysis-with-respect-for-privacy-dabe1201b476
- dnswasher (from powerdns): replaces first customer IP address it sees with 0.0.0.1, and then 0.0.0.2, etc.
    - One-way function -- not ideal
- Design:
    - ipcrypt (for IPv4): [https://github.com/veorq/ipcrypt](https://github.com/veorq/ipcrypt)
        - Has not been thoroughly studied
        - Related key weaknesses (generates related output)
        - Use a password-based KDF, with a fixed salt, to derive a key for ipcrypt
    - Use AES (or any 128-bit block cipher) for IPv6
- Risks
    - Forced known plaintext attacks to learn ciphertext
    - IP checksums revealing original data
    - Failure to rotate encryption keys
    - Encrypting well known IP addresses
    - Encrypting only pseudonymizes clients -- nothing more
- Q: Should this be compared against FF1/FF3? (Latter have known weaknesses.)

# Enumerating Privacy Leaks in DNS Data Collected above the Recursive, Basileal Imana (USC), Aleksandra Korolova (USC), and John Heidemann (USC/ISI)
- Claim: data above recursive poses "minimal" privacy risk due to client aggregation
- Motivating question: what type of leaks occur despite aggregation?
- Two adversarial models considered: 
    - Passive recursive-to-authoritative eavesdropper, or authoritative server.
    - Active malicious client (stub)
- Leaks:
    - Trackable (or unique) names
    - IP addresses in QNAMEs: Reverse DNS queries, IP-based reputation system, customer provided equipment
    - Sensitive domain names, e.g., aa.org, gaycities.com, veggieboard.com. 
- Results:
    - 1.2% of data contain sensitive domain names (based on Alexa top million)
        - ~17 million queries/day
    - Tried to measure aggregation, but hard due to caching and NAT servers

# Trust relationships between users and private DNS resolvers, Daniel Kahn Gillmor (ACLU)
- Info: [https://dnsprivacy.org/jenkins/job/dnsprivacy-monitoring/](https://dnsprivacy.org/jenkins/job/dnsprivacy-monitoring/)
- Before: trust no one, with no privacy, confidentiality, or integrity guarantees
- After: trust relationship between stub and recursive
- Trust here means that we expect the recursive to behave honestly, correctly, and non maliciously
- Violations:
    - Bad data (no DNSSEC, what do we do?)
    - Build a dossier
    - Track across the Internet (due to long-term relationship between client and server)
    - Identify within a LAN (not possible with local resolvers before)
    - Submit data to 3rd parties
    - Correlation across users
    - Be unavailable when needed
    - Insufficient mixing (anonymity loves company)
- Client benefits:
    - Pad queries
    - Use DNSSEC
    - Avoid TLS session reuse
    - Verify server identities
    - Use Tor
- Trusted to trustworthy:
    - Spread queries across resolvers (?), chaff lookups (?), logging observable misbehavior
- Q: Should we have public verifier software for these resolvers? Uptime, filtering, session reuse, ECS/Client ID?

# Panel Session: DNS-over-HTTPS (DOH) – Privacy considerations, Panel Members: Paul Hoffman (ICANN), Ben Schwartz (Google), Daniel Kahn Gillmor (ACLU)
- DOTS runs in the OS, DOH runs in browsers, under controlled by application -- configuration will help or hinder adoption
    - DOH: likely be a full URL, not just domain name
    - DOTS: IP address or domain name
- DOTS configuration:
    - On-by-default?
    - Enter IP address or domain name
    - No DHCP option for DOTS resolver
- DOH: load balancing concerns -- are load balancing concerns delegated to the resolver?
- DNSSEC-validated response tagging attacks -- how can we prevent these?

# Quad9 Depoloyment of DNS-over-TLS, John Todd (PCH)
- TLDR: rolling out recursive support, and are in need of client support
- (general discussion of lessons learned and path forward)
- haproxy set up in front of recursives, will soon introduce native TLS support in recursives
- TCP failover and redirects do not happen commonly and are "cheap" to handle

# Panel Session: Deployment and Adoption Blockers, Panel Members: John Todd (PCH), Allison Mankin (Salesforce), Wes Hardaker (USC/ISI)
- Q: Can we illustrate some attack using DNS-only information that shows need for private DNS?
    - Need some way to drive client adoption and break chicken-and-egg problem, i.e., servers won't offer it until clients require it, and clients won't implement it until servers offer it

# DNS-over-TLS on Android, Ben Schwartz (Google), Erik Kline (Google)
- (History of DNS encryption, including DNSCurve and DNSCrypt)
- How do we convey privacy and security benefits to the user? (It's not a catch all solution.)
- Demo overview
    - Users enable private DNS and fall into opportunistic mode
        - No UI reflecting opportunistic mode
    - Users can choose Strict mode, bootstrapping through network-provided DNS provider
        - Failures in Strict mode manifest as network failures in the UI
- Implemented as libresolv qhook
- Group: "opportunistic" is an awful name

# Stubby – A DNS Privacy client: Current status and GUI, Sara Dickinson (Sinodun)
- (shaming iOS and celebrating Android, overview of Stubby)
- https://dnsdisco.com/iOS-dns-proxy-post.html

# Panel Session: DNS Privacy Usability, Panel Members: Mary Ellen Zurko (MIT), Emily Stark (Google), Ben Schwartz (Google),
- @estark37: Do not communicate positive DNS privacy or security indicators (think green lock problem) 

# Leakage via Certificate Transparency, Melinda Shore (Fastly)
- Interest in DNSSEC transparency
- (overview of CT, SCT deliverance, etc.)
- Domain names in certificates are privacy sensitive, and reference location, function, etc.
- DNSSEC chain logging not adopted by DPRIVE WG. New work on logging root DNSSEC records in progress from Wouters et al. -- claimed to be more scalable (?).
- Redaction in logs is critical for users who want to host private services behind HTTPS without revealing them publicly

# Analyzing and Mitigating Privacy with the DNS Root Service, Wes Hardaker (USC/ISI)
- (see slides for data -- too quick to document)

# DNS Client Privacy through CDNs, Christian Huitema (Private Octopus Inc.,)
- TLDR: Exploit DNS servies offered by CDNs.
    - Clients cache DNS responses that map to CDNs for resolution, use that CDN for future connections
    - How often do URLs require two queries back to back? For example, example.com -> fastly.com -> {AAAA, A}
- Not all CDN and edge services provide DNS services
    - Must fallback to network-provided DNS if CDN does not provide support
    
# Oblivious encryption applied to DNS, Nick Feamster (Princeton University)
- (Overview of Tor, onion services, and usability issues)
- Onion domain management techniques, sorted by popularity (of survey responses):
    - Bookmark in Tor browser
    - Save in local text file
    - Refer to trusted websites
    - No good solution (25% of respondents)
    - Use search engines
    - ...
- (see slides for more details)

