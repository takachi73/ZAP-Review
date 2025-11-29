# ZAP-Review  
### Zero-trust Audited Pseudonymous Review  
*A next-generation architecture for secure, anonymous, and scalable academic peer review.*

---

# 1. Problem Statement  
## Structural Vulnerabilities in Current Peer Review Systems

Modern peer-review platforms (OpenReview, CMT, HotCRP, EasyChair) store:

- reviewer identity  
- role data  
- paper assignments  
- review content  
- discussion threads  

in the **same trust domain**.

This creates a systemic vulnerability:

> If one admin, one server, one API endpoint, or one database snapshot leaks,  
> **reviewer anonymity collapses.**

### The 2025 OpenReview Identity Leak  
A misconfigured API made it possible to enumerate reviewer identities and groups,  
exposing reviewer → paper linkages and internal role structure.

This was not merely a bug — it was a structural design flaw:

> **Identity, roles, and assignments must never be stored together.**

---

# 2. ZAP-Review Overview  
A fundamentally different peer-review architecture built on:

- **Zero-trust principles**  
- **Cryptographic pseudonymity**  
- **Secret-shared identity recovery**  
- **Capability-based API restrictions**  
- **Privacy-preserving reviewer reputation**  
- **Incentives that promote high-quality reviewing**  
- **Resistance to AI-generated low-quality reviews**

ZAP-Review is **not a patch**, but a **reconstruction of trust boundaries**.

---

# 3. Architectural Goals

ZAP-Review aims to:

1. Prevent deanonymization even under partial system compromise  
2. Separate identity, key management, and review actions  
3. Remove global admin visibility  
4. Provide secure and auditable identity escrow  
5. Offer long-term incentives for high-quality reviewing  
6. Scale to the rapidly growing AI/ML ecosystem  
7. Ensure integrity against AI-generated or low-effort reviews  

---

# 4. System Architecture

ZAP-Review consists of **three independent layers**:

## 4.1 Identity Authority (IA)
- Verifies real identity (ORCID, institutional SSO)  
- Confirms reviewer eligibility  
- Stores real identities isolated from all other data  

## 4.2 Key & Audit Authority (KAA)
- Generates pseudonymous review keys (PRKs)  
- Stores identity–pseudonym linkage via **Shamir Secret Sharing**  
- Maintains tamper-evident audit logs  
- Handles key rotation, revocation, and quorum-based identity recovery  

## 4.3 Review Platform (RP)
- Manages submissions, assignments, discussions, and reviews  
- Operates entirely on PRKs (pseudonyms)  
- Enforces capability-token based permissioning  

---

# 5. Architecture Diagrams

## 5.1 High-level System Flow

```
[ Identity Authority (IA) ]  -- verifies real identity
            |
            | pseudonymization request
            v
[ Key & Audit Authority (KAA) ]  -- secret-shared identity escrow
            |
            | pseudonymous review key (PRK)
            v
[ Review Platform (RP) ]  -- pseudonym-only workflow
            |
            | capability tokens
            v
    Review Actions (submit, discuss, score)
```

## 5.2 Identity Separation Boundary

```
        Real Identity Zone (IA)
        ------------------------
        Name, affiliation, ORCID
                 ||
                 || Shamir-shared identity fragments
                 \/
------------------------  <-- Trust Boundary
        Pseudonym Zone (RP)
        PRK-A12F, PRK-09XZ
```

---

# 6. Pseudonymous Review Keys (PRKs)

### 6.1 Properties
- Unique per reviewer per conference  
- Derived subkeys per paper (HKDF-based)  
- Cryptographically unlinkable to real identity  
- Renewable and revocable  

### 6.2 PRK Lifecycle

1. IA approves reviewer  
2. KAA generates PRK  
3. Reviewer interacts via PRK only  
4. Identity recovery (rare, quorum-based)  
5. PRK expires at end of conference  
6. Reputation persists pseudonymously  

---

# 7. Capability-based API Design

ZAP-Review invalidates entire classes of attacks (enumeration, role listing, scraping).

### 7.1 Rules

- No global “list reviewers” API  
- No “list ACs” API  
- No “list assignments” API  
- No platform-wide role visibility

### 7.2 Capability Tokens

Each PRK receives tokens allowing:

- listAssignedPapers()  
- submitReview(paperID)  
- joinDiscussion(paperID)  

Nothing else.

### 7.3 Role Isolation

PC members receive tokens restricted to:

- assigned tracks  
- authorized discussion channels  

No global visibility.

---

# 8. Threat Model (STRIDE + ATT&CK)

## 8.1 STRIDE Mapping

| Threat | Mitigation |
|--------|-------------|
| **Spoofing** | PRKs, signed tokens |
| **Tampering** | Append-only audit logs |
| **Repudiation** | Audit trails tied to PRK, not identity |
| **Information Disclosure** | Identity/assignment separation |
| **Denial of Service** | Token-limited flows |
| **Elevation of Privilege** | Capability-based controls |

## 8.2 MITRE ATT&CK Considerations

ZAP-Review protects against:

- Credential access (T1552)  
- Data staged for exfiltration (T1074)  
- Privilege escalation (T1068)  
- Insider attacks (T1098)  
- Enumeration (T1087)  
- API abuse (T1190)  

---

# 9. Key Management

## 9.1 Shamir Secret Sharing

Identity recovery requires a quorum, e.g.:

- 3-of-5 PC chairs  
- 2-of-3 KAA custodians  

No single admin can deanonymize reviewers.

## 9.2 Key Rotation

- PRKs are rotated each conference  
- Subkeys rotated per paper  
- Revocation tokens allow invalidating compromised PRKs

## 9.3 Identity Opening Procedure

Used only for severe misconduct:

1. Incident report  
2. Multi-party approval (quorum)  
3. KAA reconstructs identity  
4. IA validates  
5. RP notified  

---

# 10. Privacy-preserving Reviewer Reputation

### 10.1 Reputation is:
- long-term  
- pseudonymous  
- cross-conference  
- based on quality *not quantity*  
- aggregated via multiple evaluators  
- protected via differential privacy  

### 10.2 Penalties for Low-quality or AI-generated Reviews
- PRK reputation drops  
- Reduced assignment likelihood  
- Temporary review bans  

### 10.3 Rewards
- Better assignments  
- Eligibility for AC/PC roles  
- Recognized pseudonymous contributions  

---

# 11. Incentive Design Principles

1. Reward *quality*, not quantity  
2. Multi-evaluator scoring to avoid bias  
3. Full anonymity preservation  
4. Pseudonym carry-over across conferences  
5. AI-generated review detection + penalties  
6. No monetary incentives (avoid corruption)  
7. Long-term sustainability and fairness  

---

# 12. Repository Structure

```
/README.md  <-- unified specification
```

---

# 13. Contributing

We welcome contributions from:

- AI/ML researchers  
- Cryptography experts  
- Security engineers  
- Peer-review system designers  
- Program Chairs and Senior Area Chairs  
- Open-source contributors  

Open an issue or submit a pull request.

---

# 14. License

MIT License (draft)

---

# 15. Citation

```
Takahashi, H. (2025). ZAP-Review: Zero-trust Audited Pseudonymous Review Framework.
GitHub repository. https://github.com/ZAP-Review
