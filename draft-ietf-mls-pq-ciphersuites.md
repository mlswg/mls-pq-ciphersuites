---
title: ML-KEM and Hybrid Cipher Suites for Messaging Layer Security
abbrev: MLS Cipher Suites with ML-KEM
docname: draft-ietf-mls-pq-ciphersuites-latest
ipr: trust200902
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"area: art
workgroup: MLS
area: sec
category: info
keyword:
 - ML-KEM
 - Kyber
 - post-quantum
 - post quantum KEM
 - KEM
 - PQ/T
 - hybrid
 - hybrid KEM
 - Key Exchange Mechanism

stand_alone: yes
pi: [toc, sortrefs, symrefs]

venue:
  group: MLS
  type: Working Group
  mail: mls@ietf.org
  arch: https://mailarchive.ietf.org/arch/browse/mls/
  github: mlswg/mls-pq-ciphersuites/
  latest: https://mlswg.github.io/mls-pq-ciphersuites/#go.draft-ietf-mls-pq-ciphersuites.html

author:
 -  ins: R. Mahy
    name: Rohan Mahy
    organization:
    email: rohan.ietf@gmail.com
 -  ins: R.L. Barnes
    name: Richard L. Barnes
    organization: Cisco
    email: rlb@ipv.sx


normative:
  SHS: DOI.10.6028/NIST.FIPS.180-4
  MLKEM: DOI.10.6028/NIST.FIPS.203
  MLDSA: DOI.10.6028/NIST.FIPS.204
  FIPS202: DOI.10.6028/NIST.FIPS.202
  GCM: DOI.10.6028/NIST.SP.800-38D

informative:

--- abstract

This document registers new cipher suites for Messaging Layer Security (MLS) based on "post-quantum" algorithms, which are intended to be resilient to attack by quantum computers.
These cipher suites are constructed using the new Module-Lattice Key Encapsulation Mechanism (ML-KEM), optionally in combination with traditional elliptic curve KEMs, together with appropriate authenticated encryption, hash, and signature algorithms.

--- middle

# Introduction

The potential availability of a cryptographically-relevant quantum computer has caused concern that well-funded adversaries could overturn long-held assumptions about the security assurances of classical Key Exchange Mechanisms (KEMs) and classical cryptographic signatures, which are fundamental to modern security protocols, including the MLS protocol {{!RFC9420}}.

Of particular concern are "harvest now, decrypt later" attacks, by which an attacker could collect encrypted traffic now, before a quantum computer exists, and later use a quantum computer to break the confidentiality protections on the collected traffic.

In response to these concerns, the cryptographic community has defined "post-quantum" algorithms, which are designed to be resilient to attacks by quantum computers.
Symmetric algorithms can be made post-quantum secure simply by using longer keys and hashes.
For asymmetric operations such as KEMs and signatures, entirely new algorithms are needed.

In this document, we define ciphersuites that use the post-quantum secure Module-Lattice-Based KEM (ML-KEM) {{MLKEM}} together with appropriate symmetric algorithms, and either traditional or Module-Lattice-Based Digital Signature Algorithm (ML-DSA) {{ML-DSA}} post-quantum signature algorithms.
The traditional signature cipher suites address the risk of "harvest now, decrypt later" attacks, while not taking on the additional cost of post-quantum signatures.
The cipher suites with post-quantum signatures use only post-quantum KEMs.

Following the pattern of base MLS, we define several variations, to allow for users that prefer to only use NIST-approved cryptography, users that prefer a higher security level, and users that prefer a PQ/traditional hybrid KEM over pure ML-KEM:

* ML-KEM-768 + X25519 (128-bit security, Non-NIST, PQ/T hybrid)
* ML-KEM-768 + P-256 (128-bit security, NIST, PQ/T hybrid)
* ML-KEM-1024 + P-384 (192-bit security, NIST, PQ/T hybrid)
* ML-KEM-768 (128-bit security, NIST, pure PQ KEM)
* ML-KEM-1024 (192-bit security, NIST, pure PQ KEM)
* ML-KEM-768 (192-bit security, NIST, pure PQ)
* ML-KEM-1024 (256-bit security, NIST, pure PQ)

For all the cipher suites defined in this document, we use AES256 GCM {{GCM}} as the Authenticated Encryption with Authenticated Data (AEAD) {{!RFC5116}} algorithm;
HMAC {{!RFC2104}} with SHA-384 {{SHS}} as the hash function;
and SHAKE256 (Section 3.2 of {{FIPS202}}) as the Key Derivation Function (KDF).

For the PQ/T hybrid KEMs and the pure ML-KEM HPKE integration, we use the KEMs defined in {{!I-D.ietf-hpke-pq}}.
The signature schemes for ML-DSA-65 and ML-DSA-87 {{MLDSA}} are defined in {{!I-D.ietf-tls-mldsa}}.

# IANA Considerations

## MLS Cipher Suites

This document requests that IANA add the following entries to the "MLS Cipher Suites" registry, replacing "XXXX" with the RFC number assigned to this document:

| Value | Name                                              | Rec | Reference |
|:======|:====================================================|:====|:========|
| TBD1 | MLS_128_QSF-X25519-MLKEM768_AES256GCM_SHA384_Ed25519 |  Y  | RFCXXXX |
| TBD2 | MLS_128_QSF-P256-MLKEM768_AES256GCM_SHA384_P256      |  Y  | RFCXXXX |
| TBD3 | MLS_192_QSF-P384-MLKEM1024_AES256GCM_SHA384_P384     |  Y  | RFCXXXX |
| TBD4 | MLS_128_ML-KEM-768_AES256GCM_SHA384_P256             |  Y  | RFCXXXX |
| TBD5 | MLS_192_ML-KEM-1024_AES256GCM_SHA384_P384            |  Y  | RFCXXXX |
| TBD6 | MLS_192_ML-KEM-768_AES256GCM_SHA384_MLDSA65          |  Y  | RFCXXXX |
| TBD7 | MLS_256_ML-KEM-1024_AES256GCM_SHA512_MLDSA87         |  Y  | RFCXXXX |


The mapping of cipher suites to HPKE primitives {{!I-D.ietf-hpke-hpke}}, HMAC hash functions, and TLS signature schemes {{!RFC8446}} is as follows:

| Value  | KEM     | KDF    | AEAD   | Hash   | Signature              |
|:=======|:========|:=======|:=======|:=======|:=======================|
| 0xTBD1 | 0x647a  | 0x0011 | 0x0002 | SHA384 | ed25519                |
| 0xTBD2 | 0x0050  | 0x0011 | 0x0002 | SHA384 | ecdsa_secp256r1_sha256 |
| 0xTBD3 | 0x0051  | 0x0011 | 0x0002 | SHA384 | ecdsa_secp384r1_sha384 |
| 0xTBD4 | 0x0041  | 0x0011 | 0x0002 | SHA384 | ecdsa_secp256r1_sha256 |
| 0xTBD5 | 0x0042  | 0x0011 | 0x0002 | SHA384 | ecdsa_secp384r1_sha384 |
| 0xTBD6 | 0x0041  | 0x0011 | 0x0002 | SHA384 | mldsa65                |
| 0xTBD7 | 0x0042  | 0x0011 | 0x0002 | SHA384 | mldsa87                |

The hash used for the MLS transcript hash is the one referenced in the cipher suite name. "SHA384" refers to the SHA-384 functions defined in [SHS].

# Security Considerations

The first five ciphersuites defined in this document combine a post-quantum (or PQ/T hybrid) KEM with a traditional signature algorithm. As such, they are designed to provide confidentiality against quantum and classical attacks, but provide authenticity against classical attacks only.  Thus, these cipher suites do not provide full post-quantum security, only post-quantum confidentiality.

The last two cipher suites also use post-quantum signature algorithms.

For security considerations related to the KEMs used in this document, please see the documents that define those KEMs {{!I-D.ietf-hpke-pq}} and {{?I-D.irtf-cfrg-hybrid-kems}}.
For security considerations related to the post-quantum signature algorithms used in this document, please see {{!I-D.ietf-tls-mldsa}} and {{?I-D.ietf-lamps-dilithium-certificates}}.

--- back

# Acknowledgments
{:numbered="false"}

This work would not be possible without the hard work of the CFRG Hybrid KEM design team: Aron Wussler, Bas Westerbaan, Deirdre Connolly, Mike Ounsworth, Nick Sullivan, and Stephen Farrell.
Thanks also to Joël Alwen, Marta Mularczyk, and Britta Hale.
