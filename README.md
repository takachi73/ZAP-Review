# ZAP-Review  
### Zero-trust Audited Pseudonymous Review  
*A next-generation architecture for secure, anonymous, and scalable academic peer review.*

---

## Overview

ZAP-Review is a novel peer-review framework designed to solve the **structural weaknesses** in  
current academic reviewing systems. Traditional review platforms (OpenReview, CMT, HotCRP,  
EasyChair) store **reviewer identity, assignments, and review content in the same trust domain**,  
making them vulnerable to deanonymization, insider threats, and administrative overreach.

The 2025 OpenReview identity leak demonstrated that anonymity cannot be guaranteed  
without a **fundamental redesign** of peer-review architecture.

ZAP-Review introduces a **zero-trust, cryptographically grounded, pseudonymous system** that  
separates identity, authority, and review actions at the architectural level.

---

## Key Principles

### 🔐 Zero-trust Architecture  
No single system, administrator, or database can reveal reviewer identities.

### 🕶 Pseudonymous Review Keys (PRKs)  
Reviewers act using cryptographic pseudonyms derived through secure key generation.  
Real identities are decoupled from review actions.

### 🧩 Three-layer Role Separation
ZAP-Review cleanly separates:

| Layer | Role | Access |
|------|------|--------|
| **Identity Authority (IA)** | Verifies real identities | Knows identities, not assignments |
| **Key & Audit Authority (KAA)** | Issues pseudonymous keys, stores secret-shared mapping | Knows partial identity mapping |
| **Review Platform (RP)** | Manages workflow, UI, assignment, discussion | Sees only pseudonyms |

### 🔏 Secret-shared Identity Recovery  
Identity recovery (for misconduct or legal reasons) requires quorum approval  
using **Shamir Secret Sharing**, preventing unilateral deanonymization.

### 🛡 Capability-based API  
APIs enforce minimum privilege:  
reviewers can only access their own assignments.  
Global enumeration of roles or users is cryptographically impossible.

### ⭐ Privacy-preserving Reviewer Reputation  
Long-term reviewer reputation is stored *per pseudonym*, not per real person,  
supporting sustainable incentives without risking identity exposure.

---

## Motivation

ZAP-Review addresses multiple long-standing challenges:

- Identity leakage risks (highlighted by the 2025 OpenReview incident)  
- Lack of cryptographic anonymity guarantees  
- Reviewer shortage in AI/ML fields  
- Structural bias enabled by admin visibility  
- No cross-conference reputation system  
- Rising threats from AI-generated low-quality reviews  
- Lack of architectural separation between identity and actions  

For a complete explanation, see:  
➡ **problem-statement.md**

---

## Architecture

ZAP-Review is designed around strict isolation of identity, key management, and review actions.

Full architectural details:  
➡ **architecture.md**

---

## Core Components

### Identity Authority (IA)
- Validates reviewer eligibility (e.g., ORCID, institutional SSO).  
- Maintains real identity data isolated from review processes.

### Key & Audit Authority (KAA)
- Generates PRKs.  
- Uses secret-sharing for identity mapping.  
- Maintains tamper-evident audit logs.

### Review Platform (RP)
- Handles submissions, assignments, discussions, reviews.  
- Operates exclusively on pseudonyms.  
- Enforces capability-based access.

---

## Threat Model

ZAP-Review protects against:

- Platform database leaks  
- Misconfigured or insecure APIs  
- Malicious admins or insider attacks  
- Metadata correlation  
- Reviewer harassment and retaliation  
- Credential enumeration or role scraping  
- Cross-conference deanonymization  
- AI-generated bulk low-quality reviews (via reputation design)

See full analysis:  
➡ **threat-model.md**

---

## API Design

ZAP-Review includes a capability-token–based API layer that ensures:

- reviewers only access assigned papers  
- no global listing APIs  
- no role enumeration  
- cryptographically enforced privilege boundaries  

API specification:  
➡ **api-design.md**

---

## Key Management

ZAP-Review introduces:

- Pseudonymous Review Keys (PRKs)  
- Per-paper subkeys (HKDF-derived)  
- Shamir secret-sharing for identity escrow  
- Key rotation and revocation processes  
- Quorum-based identity opening  

Details:  
➡ **key-management.md**

---

## Reviewer Incentives

ZAP-Review integrates a **privacy-preserving, cross-conference reputation system** to ensure:

- high-quality reviewing is rewarded  
- low-quality or AI-generated reviews are penalized  
- reputation accumulates over years  
- reputations remain pseudonymous but portable  
- incentive mechanisms avoid Gresham’s Law (“bad reviews driving out good reviews”)  

This solves a critical bottleneck in AI/ML reviewing:  
**sustainable, high-quality reviewer participation.**

An expanded incentives specification will be provided in a dedicated document.

---

## Repository Structure

---

## Contributing

Contributions are welcome from:

- researchers in AI/ML  
- cryptographers  
- security engineers  
- PC chairs and organizers  
- human–computer interaction specialists  
- open-source contributors  

Please open an issue or submit a pull request.

---

## License

MIT License (draft — subject to refinement based on community needs)

---

## Citation
Takahashi, H. (2025). ZAP-Review: Zero-trust Audited Pseudonymous Review Framework. GitHub repository. https://github.com/ZAP-Review
