---
title: "Post-Quantum Cryptography Algorithms: ML-KEM, ML-DSA and SLH-DSA"
description: "Learn how post-quantum cryptography works, compare NIST PQC standards ML-KEM, ML-DSA and SLH-DSA, and plan a practical migration."
---

# Post-Quantum Cryptography Algorithms: A Guide to PQC

**Post-quantum cryptography (PQC)** uses cryptographic algorithms designed to remain secure against attacks by both classical and sufficiently capable quantum computers. PQC runs on conventional computers; it does not require a quantum network or quantum computer.

The first NIST PQC standards specify **ML-KEM** for establishing shared keys and **ML-DSA** and **SLH-DSA** for digital signatures. This guide explains their roles, differences, and practical implications for TLS, certificates, and PKI. Standards information is current as of November 22, 2026. [NIST: PQC standards](https://csrc.nist.gov/news/2024/postquantum-cryptography-fips-approved)

## Why is post-quantum cryptography needed?

A sufficiently powerful, fault-tolerant quantum computer could run Shor's algorithm to break the mathematical problems underlying RSA and elliptic-curve cryptography. That threatens commonly deployed public-key encryption, key agreement, and digital signatures. Such a machine is not assumed to be available today, and its arrival date is uncertain. [NIST: Post-Quantum Cryptography](https://csrc.nist.gov/projects/post-quantum-cryptography)

A concern is **harvest now, decrypt later**: an adversary can record encrypted traffic today and attempt to decrypt it in the future. Information that must remain confidential for many years may therefore need quantum-resistant key establishment before a large quantum computer exists. Digital signatures have different lifecycle considerations, including the need to verify long-lived signatures and protect signing keys.

PQC primarily replaces vulnerable **public-key** mechanisms. Symmetric encryption and hash functions are affected differently by quantum algorithms; they do not all become unusable, and parameter choices and applications still matter. PQC is not the same as quantum key distribution (QKD).

## NIST post-quantum algorithm overview

| Algorithm | Standard | Purpose | Main mathematical approach | Parameter sets |
| --- | --- | --- | --- | --- |
| **ML-KEM** | [FIPS 203](https://csrc.nist.gov/pubs/fips/203/final) | Establish a shared secret | Module lattices | ML-KEM-512, ML-KEM-768, ML-KEM-1024 |
| **ML-DSA** | [FIPS 204](https://csrc.nist.gov/pubs/fips/204/final) | Digital signatures | Module lattices | ML-DSA-44, ML-DSA-65, ML-DSA-87 |
| **SLH-DSA** | [FIPS 205](https://csrc.nist.gov/pubs/fips/205/final) | Digital signatures | Stateless hash-based signatures | SHA-2 and SHAKE variants across security and size/speed choices |

NIST published all three standards in August 2024. Their parameter-set names are identifiers; they should not be read as interchangeable security-bit counts. Security, performance, public-key size, signature size, and ecosystem support vary by parameter set. Consult the standards and deployment profile when selecting one.

## ML-KEM: post-quantum key establishment

ML-KEM (Module-Lattice-Based Key-Encapsulation Mechanism), derived from CRYSTALS-Kyber, establishes a shared secret across a public channel. It is a **key encapsulation mechanism (KEM)**, not a digital signature algorithm or a general-purpose file encryption format. [FIPS 203](https://csrc.nist.gov/pubs/fips/203/final)

A simplified exchange works as follows:

1. The recipient generates an ML-KEM public key and private key, and makes the public key available.
2. The sender encapsulates to the recipient's public key, producing a ciphertext and a shared secret.
3. The sender sends the ciphertext to the recipient.
4. The recipient decapsulates the ciphertext with the private key to obtain the same shared secret.
5. A protocol derives appropriate symmetric keys from that secret and uses authenticated encryption to protect data.

The public key must be authenticated by the surrounding protocol; ML-KEM alone does not establish who owns it. A secure protocol must also handle key derivation, algorithm negotiation, and failure behavior correctly. [NIST SP 800-227: Recommendations for KEMs](https://csrc.nist.gov/pubs/sp/800/227/final)

NIST IR 8547 recommends ML-KEM-768 as a default parameter set, while recognizing that other sets can be appropriate for particular requirements. This is NIST guidance, not a guarantee of compatibility with every product or policy. [NIST IR 8547](https://doi.org/10.6028/NIST.IR.8547)

## ML-DSA: lattice-based digital signatures

ML-DSA (Module-Lattice-Based Digital Signature Algorithm), derived from CRYSTALS-Dilithium, signs messages with a private key and verifies signatures with the corresponding public key. A successful verification provides evidence of integrity and of possession of the signing key; identity depends on how the public key is authenticated. [FIPS 204](https://csrc.nist.gov/pubs/fips/204/final)

ML-DSA is intended for signature use cases such as software signing and PKI, subject to the relevant profile and implementation support. The three standardized parameter sets offer different security and performance trade-offs. A digital signature does **not** encrypt the message or establish a shared secret.

## SLH-DSA: stateless hash-based digital signatures

SLH-DSA (Stateless Hash-Based Digital Signature Algorithm), derived from SPHINCS+, is a signature scheme based on hash-function security. Its different mathematical foundation provides an alternative to lattice-based signatures. It is called *stateless* because signing does not require maintaining the per-signature state required by stateful hash-based schemes. [FIPS 205](https://csrc.nist.gov/pubs/fips/205/final)

SLH-DSA has parameter sets using SHA-2 or SHAKE, with `s` variants favoring smaller signatures and `f` variants favoring faster signing. Signature sizes and signing performance can differ substantially from ML-DSA; benchmark the specific parameter set and workload before deployment.

## Other algorithms and standards in development

- **FALCON / FN-DSA:** NIST selected FALCON for standardization as a lattice-based signature scheme. FIPS 206 remains in development in NIST's published project status. Check the current standard before deployment. [NIST PQC standardization](https://csrc.nist.gov/Projects/Post-Quantum-Cryptography/Post-Quantum-Cryptography-Standardization)
- **HQC:** NIST selected this code-based KEM for standardization in 2025 as an alternative with a different mathematical foundation from ML-KEM. Selection is not the same as a final published FIPS standard. [NIST: HQC selection](https://www.nist.gov/news-events/news/2025/03/nist-selects-hqc-fifth-algorithm-post-quantum-encryption)

Additional signature candidates continue to be evaluated. Check NIST's current publications and applicable protocol profiles rather than assuming a candidate is standardized or interoperable.

## PQC, TLS, and X.509 certificates

TLS uses several cryptographic functions with different roles: key establishment protects the session's derived keys; certificate signatures authenticate the certificate chain; and handshake signatures authenticate the server or client. Replacing one role with a PQC algorithm does not automatically replace the others.

For example, using post-quantum key establishment in a TLS connection does not by itself mean that its X.509 certificates use post-quantum signatures. A complete migration must account for clients, servers, TLS libraries, certificate authorities, HSMs, certificate profiles, monitoring, and other relying parties. Whether a particular algorithm or combination is supported depends on the protocol standard and the products in use.

**Hybrid key establishment** combines a classical mechanism and a post-quantum mechanism in a specified construction. It can provide a transition path, but its security and interoperability depend on the actual protocol and implementation. Use an established, reviewed profile rather than inventing a combination. Hybrid key establishment and hybrid signatures solve different problems. [NIST IR 8547](https://doi.org/10.6028/NIST.IR.8547)

## How to plan a post-quantum migration

1. **Inventory cryptography.** Find RSA, Diffie–Hellman, ECDH, ECDSA, and other public-key uses across applications, certificates, protocols, HSMs, and suppliers. Record algorithm and parameter choices, dependencies, and certificate lifetimes.
2. **Prioritize long-lived data and high-impact systems.** Consider confidentiality lifetime, harvest-now-decrypt-later exposure, signed data retention, and the time needed to replace difficult-to-update systems.
3. **Identify dependencies and standards.** Check CA, operating system, browser, library, hardware, and protocol support, along with applicable regulatory and sector requirements.
4. **Test interoperability and performance.** Measure handshake sizes, bandwidth, memory, CPU, signature and key sizes, certificate chains, and operational limits with real workloads.
5. **Build crypto agility.** Support controlled algorithm and key rotation, inventory updates, monitoring, and rollback. Avoid hard-coding assumptions about key or signature sizes.
6. **Deploy in stages and monitor.** Use supported, reviewed implementations and protocol profiles. Test renewal, recovery, revocation, logging, and fallback behavior before wider rollout.

NIST's final [IR 8547 transition report](https://doi.org/10.6028/NIST.IR.8547) describes its approach to moving away from quantum-vulnerable algorithms. Its timelines apply within NIST standards and guidelines and should not be treated as a universal legal deadline for every organization. Applicable requirements can depend on jurisdiction, sector, and system classification.

## Frequently asked questions

### Is PQC the same as quantum cryptography?

No. PQC algorithms run on classical computers and are designed to resist quantum and classical attacks. Quantum cryptography, including QKD, uses quantum physical phenomena and has different infrastructure and security assumptions.

### Does PQC replace AES and SHA-2?

PQC standards primarily address public-key key establishment and digital signatures. AES and SHA-2 have different quantum security considerations; the right key size, hash function, and protocol still depend on the application and policy.

### Which PQC algorithm replaces RSA?

There is no single drop-in replacement for every RSA use. ML-KEM addresses key establishment, while ML-DSA and SLH-DSA address signatures. The applicable protocol must support the chosen algorithm and its encoding.

### Is ML-KEM encryption?

ML-KEM establishes a shared secret; it does not directly encrypt arbitrary application data. Protocols use the secret to derive symmetric keys for authenticated encryption.

### Are post-quantum algorithms guaranteed to be secure forever?

No. Their security depends on mathematical assumptions, correct implementations, secure key management, and continued cryptanalysis. Follow standards updates and plan for algorithm agility.

## Sources

- [NIST FIPS 203: ML-KEM](https://csrc.nist.gov/pubs/fips/203/final), [FIPS 204: ML-DSA](https://csrc.nist.gov/pubs/fips/204/final), and [FIPS 205: SLH-DSA](https://csrc.nist.gov/pubs/fips/205/final).
- [NIST SP 800-227: Recommendations for Key-Encapsulation Mechanisms](https://csrc.nist.gov/pubs/sp/800/227/final).
- [NIST IR 8547: Transition to Post-Quantum Cryptography Standards](https://doi.org/10.6028/NIST.IR.8547).
- [NIST Post-Quantum Cryptography Standardization](https://csrc.nist.gov/Projects/Post-Quantum-Cryptography/Post-Quantum-Cryptography-Standardization) and [HQC selection announcement](https://www.nist.gov/news-events/news/2025/03/nist-selects-hqc-fifth-algorithm-post-quantum-encryption).
