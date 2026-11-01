---
title: ML-KEM and Hybrid Cipher Suites for Messaging Layer Security
abbrev: MLS Cipher Suites with ML-KEM
docname: draft-ietf-mls-pq-ciphersuites-latest
ipr: trust200902
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
workgroup: MLS
area: sec
category: std
keyword:
 - ML-KEM
 - Kyber
 - post-quantum
 - post quantum KEM
 - KEM
 - PQ/T
 - hybrid
 - hybrid KEM
 - Key Encapsulation Mechanism

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
    organization:
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
These cipher suites are constructed using the new Module-Lattice-Based Key-Encapsulation Mechanism (ML-KEM), optionally in combination with traditional elliptic curve KEMs, together with appropriate authenticated encryption, hash, and signature algorithms.

--- middle

# Introduction

The potential availability of a cryptographically-relevant quantum computer has caused concern that well-funded adversaries could overturn long-held assumptions about the security assurances of classical Key Encapsulation Mechanisms (KEMs) and classical cryptographic signatures, which are fundamental to modern security protocols, including the MLS protocol {{!RFC9420}}.

Of particular concern are "harvest now, decrypt later" attacks, by which an attacker could collect encrypted traffic now, before a quantum computer exists, and later use a quantum computer to break the confidentiality protections on the collected traffic.

In response to these concerns, the cryptographic community has defined "post-quantum" algorithms, which are designed to be resilient to attacks by quantum computers.
Symmetric algorithms can be made post-quantum secure simply by using longer keys and hashes.
For asymmetric operations such as KEMs and signatures, entirely new algorithms are needed.

In this document, we define cipher suites that use the post-quantum secure Module-Lattice-Based Key-Encapsulation Mechanism (ML-KEM) {{MLKEM}} together with appropriate symmetric algorithms, and either traditional or Module-Lattice-Based Digital Signature Algorithm (ML-DSA) {{MLDSA}} post-quantum signature algorithms.
The traditional signature cipher suites address the risk of "harvest now, decrypt later" attacks, while not taking on the additional cost of post-quantum signatures.
The cipher suites with post-quantum signatures use either a pure post-quantum KEM or a PQ/T hybrid KEM.

# Post-quantum KEM Cipher Suites

Following the pattern of base MLS, we define several variations, to allow for users that prefer to only use NIST-approved cryptography, users that prefer a higher security level, and users that prefer a PQ/traditional hybrid KEM over pure ML-KEM:

* ML-KEM-768 + X25519 (128-bit security, Non-NIST, PQ/T hybrid KEM)
* ML-KEM-768 + P-256 (128-bit security, NIST, PQ/T hybrid KEM)
* ML-KEM-1024 + P-384 (192-bit security, NIST, PQ/T hybrid KEM)
* ML-KEM-768 (128-bit security, Non-NIST, pure PQ KEM)
* ML-KEM-768 (128-bit security, NIST, pure PQ KEM)
* ML-KEM-1024 (192-bit security, NIST, pure PQ KEM)
* ML-KEM-768 + X25519 (128-bit security, Non-NIST, PQ/T hybrid KEM + PQ signature)
* ML-KEM-768 (192-bit security, NIST, pure PQ)
* ML-KEM-1024 (256-bit security, NIST, pure PQ)

The first two entries are each registered twice: once with AES128 GCM and SHA-256, and once with AES256 GCM and SHA-384.

Some parts of the community wish to support the 128-bit security level with the same Authenticated Encryption with Associated Data (AEAD) {{!RFC5116}} algorithms and hash function as used in the default cipher suite registered in {{!RFC9420}} (AES128 GCM {{GCM}} and HMAC {{!RFC2104}} with SHA-256 {{SHS}}), while other parts of the community would like to follow recent recommendations to transition immediately to AES256 GCM {{GCM}} and HMAC {{!RFC2104}} with SHA-384 {{SHS}}.

For the cipher suites at the 192-bit or 256-bit security levels, we use AES256 GCM {{GCM}} as the AEAD algorithm, and HMAC {{!RFC2104}} with SHA-384 {{SHS}} as the hash function.

ML-KEM uses the SHAKE and SHA-3 algorithms {{FIPS202}} internally (see Section 4.1 of {{MLKEM}}).
Likewise the three hybrid KEMs {{!I-D.irtf-cfrg-concrete-hybrid-kems}} used in this document use SHAKE as a Pseudo-Random Generator (PRG) and SHA-3 as a Key Derivation Function (KDF) inside the hybrid combiner.

MLS extensively uses the two-stage KDF functions `Extract` and `Expand` defined in {{Section 4.2 of !I-D.ietf-hpke-hpke}} to derive keys in the ratchet tree {{Section 7.4 of RFC9420}}, the key schedule {{Section 8 of RFC9420}}, and the secret tree {{Section 9 of RFC9420}}.
In particular, MLS uses the `Extract` function to incorporate commit secrets and pre-shared keys into the key schedule.
As {{!I-D.ietf-hpke-pq}} does not define either function for "single-stage" KDFs like SHAKE (see {{Section 5 of !I-D.ietf-hpke-pq}}), the cipher suites defined here use the HMAC-based KDF (HKDF) algorithms HKDF-SHA384 and HKDF-SHA256 for the KDF functions used inside MLS.

Finally, one of the cipher suites at the 128-bit security level uses the same hybrid KEM as the first cipher suite, the ChaCha20-Poly1305 {{!RFC8439}} AEAD algorithm, HMAC with SHA-384, and the pure PQ signature algorithm ML-DSA-44.
ChaCha20-Poly1305 was chosen for acceptable performance when implemented entirely in software.

For the PQ/T hybrid KEMs and the pure ML-KEM HPKE integration, we use the KEMs defined in {{!I-D.ietf-hpke-pq}}.
The signature schemes for ML-DSA-44, ML-DSA-65, and ML-DSA-87 {{MLDSA}} are defined in {{!I-D.ietf-tls-mldsa}}.


# IANA Considerations

## MLS Cipher Suites

This document requests that IANA add the following entries to the "MLS Cipher Suites" registry, replacing "XXXX" with the RFC number assigned to this document:

| Value  | Name                                            | Rec | Reference |
|:=======|:================================================|:====|:========|
| 0x0010 | MLS_128_MLKEM768X25519_AES128GCM_SHA256_Ed25519 |  Y  | RFCXXXX |
| 0x0011 | MLS_128_MLKEM768X25519_AES256GCM_SHA384_Ed25519 |  Y  | RFCXXXX |
| 0x0012 | MLS_128_MLKEM768P256_AES128GCM_SHA256_P256      |  Y  | RFCXXXX |
| 0x0013 | MLS_128_MLKEM768P256_AES256GCM_SHA384_P256      |  Y  | RFCXXXX |
| 0x0014 | MLS_192_MLKEM1024P384_AES256GCM_SHA384_P384     |  Y  | RFCXXXX |
| 0x0015 | MLS_128_MLKEM768_AES256GCM_SHA384_Ed25519       |  Y  | RFCXXXX |
| 0x0016 | MLS_128_MLKEM768_AES256GCM_SHA384_P256          |  Y  | RFCXXXX |
| 0x0017 | MLS_192_MLKEM1024_AES256GCM_SHA384_P384         |  Y  | RFCXXXX |
| 0x0020 | MLS_128_MLKEM768X25519_CHACHA20POLY1305_SHA384_MLDSA44 | Y | RFCXXXX |
| 0x0021 | MLS_192_MLKEM768_AES256GCM_SHA384_MLDSA65       |  Y  | RFCXXXX |
| 0x0022 | MLS_256_MLKEM1024_AES256GCM_SHA384_MLDSA87      |  Y  | RFCXXXX |


The mapping of cipher suites to HPKE primitives {{!I-D.ietf-hpke-hpke}}, HMAC hash functions, and TLS signature schemes {{!I-D.ietf-tls-rfc8446bis}} is as follows:

| Value  | KEM     | KDF    | AEAD   | Hash   | Signature              |
|:=======|:========|:=======|:=======|:=======|:=======================|
| 0x0010 | 0x647a  | 0x0001 | 0x0001 | SHA256 | ed25519                |
| 0x0011 | 0x647a  | 0x0002 | 0x0002 | SHA384 | ed25519                |
| 0x0012 | 0x0050  | 0x0001 | 0x0001 | SHA256 | ecdsa_secp256r1_sha256 |
| 0x0013 | 0x0050  | 0x0002 | 0x0002 | SHA384 | ecdsa_secp256r1_sha256 |
| 0x0014 | 0x0051  | 0x0002 | 0x0002 | SHA384 | ecdsa_secp384r1_sha384 |
| 0x0015 | 0x0041  | 0x0002 | 0x0002 | SHA384 | ed25519                |
| 0x0016 | 0x0041  | 0x0002 | 0x0002 | SHA384 | ecdsa_secp256r1_sha256 |
| 0x0017 | 0x0042  | 0x0002 | 0x0002 | SHA384 | ecdsa_secp384r1_sha384 |
| 0x0020 | 0x647a  | 0x0002 | 0x0003 | SHA384 | mldsa44                |
| 0x0021 | 0x0041  | 0x0002 | 0x0002 | SHA384 | mldsa65                |
| 0x0022 | 0x0042  | 0x0002 | 0x0002 | SHA384 | mldsa87                |

The hash used for the MLS transcript hash is the one referenced in the cipher suite name. "SHA256" and "SHA384" refer to the SHA-256 and SHA-384 functions defined in {{SHS}}.

# Security Considerations

The first eight cipher suites defined in this document combine a post-quantum (or PQ/T hybrid) KEM with a traditional signature algorithm. As such, they are designed to provide confidentiality against quantum and classical attacks, but provide authenticity against classical attacks only.  Thus, these cipher suites do not provide full post-quantum security, only post-quantum confidentiality.
That post-quantum confidentiality holds only against passive attackers, such as those mounting "harvest now, decrypt later" attacks.
An attacker able to forge the traditional signatures, or the credentials that bind the signature keys to identities, can inject KeyPackages or group operations and thereby compromise the confidentiality of later epochs.

The last three cipher suites also use post-quantum signature algorithms.

For security considerations related to the KEMs used in this document, please see the documents that define those KEMs: {{!I-D.ietf-hpke-pq}}, {{!I-D.irtf-cfrg-hybrid-kems}}, and {{!I-D.irtf-cfrg-concrete-hybrid-kems}}.
For security considerations related to the post-quantum signature algorithms used in this document, please see {{!I-D.ietf-tls-mldsa}} and {{!RFC9881}}.

--- back

# Acknowledgments
{:numbered="false"}

Mark Atwood provided useful corrections and clarifications.

# Acknowledgments
{:numbered="false"}

This work would not be possible without the hard work of the CFRG Hybrid KEM design team: Aron Wussler, Bas Westerbaan, Deirdre Connolly, Mike Ounsworth, Nick Sullivan, and Stephen Farrell.
Thanks also to Joël Alwen, Marta Mularczyk, and Britta Hale.
