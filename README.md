# ZAP-Review  
### Zero-trust Audited Pseudonymous Review  
*A next-generation architecture for secure, anonymous, and scalable academic peer review.*

---
## What's New in v2.1 (Implemented)

This release moves from proposal to cryptographically enforced protocol.

**Core Breakthrough - Verifiable Career Credentials (v2.1):**
- **Non-transferable Reputation:** Credential Transfer Attack (name lending) mathematically eliminated via `com_id = Commit(Hash(X), r)` binding. ARC cannot be lent.
- **Deterministic Recovery:** Blinding factor `r` and `PRK_long` are no longer random. Both derived as `HKDF(WebAuthn PRF)` - single Touch ID reconstructs all secrets. No cloud backup needed (resolves key loss pitfall).
- **Separation of Powers Preserved:** IA never learns paper_ids or reputation, KAA never learns real_id. University verifies RAC via BBS+ selective disclosure.

**Enhanced Foundations (from Proposed -> Implemented):**
- Feldman VSS for verifiable escrow
- Threshold BLS (2-of-3: ACM/IEEE/ICLR) for quorum opening - prevents single IA forgery
- Macaroon capability tokens + PIR-ready API
- Differentially private reputation (epsilon=1.0)
- Temporal and stylometric deanonymization mitigation
- Trillian transparency log + ProVerif model (WIP)

## Roadmap
- Phase 1: Reference implementation of v2.1 circuit (deterministic_r_and_bbs_verify.circom) - Current
- Phase 2: Pilot at AI workshop <100 papers with Passkey flow evaluation
- Phase 3: Paper submission with formal ProVerif proof and de-anonymization eval

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
- **Non-transferable, verifiable reputation (v2.1)**  
- **Passkey-based deterministic recovery (v2.1)**  
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
- Stores `X <-> com_id` mapping (not PRK)
- Issues RAC via threshold BLS after verifying ZKP_v2.1
- Enforces 1 com_id per real_id (Sybil prevention via stable com_id)

## 4.2 Key & Audit Authority (KAA)
- Generates `PRK_long = HKDF(prf_out, "prk_long")`
- Stores `com_id <-> Commit(PRK_long)` (not real_id)
- Issues `ARC_v2.1 = BBS+_Sign(KAA_sk, {Commit(PRK_long), com_id, bucket})`
- Maintains Trillian transparency log

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

# 6. Pseudonymous Review Keys (PRKs) - v2.1 Update

### 6.1 Properties (Updated)
- Unique per reviewer, stable `com_id` per real identity (via deterministic `r`)
- `r = HKDF-SHA256(prf_out, "blinding_factor")` - recoverable via Touch ID only
- `PRK_long = HKDF-SHA256(prf_out, "prk_long")` - stored in Secure Enclave, synced via iCloud/Google
- Cryptographically bound to real identity at registration, but unlinkable to RP

### 6.2 PRK Lifecycle (Updated)
1. IA verifies ORCID, client generates `com_id = Commit(Hash(X), r)` deterministically
2. KAA generates PRK commitment
3. Reviewer interacts via PRK only
4. After conference: KAA issues ARC_v2.1
5. Reviewer: Touch ID -> reconstruct r, PRK_long -> generate ZKP_v2.1 -> IA issues RAC
6. PRK expires, but RAC persists as career credential
7. Reputation persists pseudonymously via com_id

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
# 10. Privacy-preserving Reviewer Reputation (v2.1 Detailed)

This section is now fully specified in `docs/04-reputation-and-incentive.md`.

**Protocol Summary:**
- **Phase 1 (KAA):** Issue ARC_v2.1 bound to `com_id`
- **Phase 2 (IA):** Verify `ZKP_v2.1 { com_id == Commit(X,r), r == HKDF(prf_out), PRK_long == HKDF(prf_out), BBS+_Verify(ARC)}` then issue RAC with threshold BLS
- **Phase 3 (University):** Verify RAC via `BBS+_Verify(IA_pk)`

**Security Guarantees:**
- Non-transferability: Breaking Pedersen binding required to lend credential
- No secret storage: Loss of `r` impossible in v2.1 (deterministic)
- Separation: IA learns no paper_ids, KAA learns no real_id
- Unlinkability: RACs across conferences unlinkable via BBS+

**UX:** Click "Generate Career Proof" -> Touch ID (0.5s) -> WASM ZKP (3s) -> PDF VC

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

### 10.4 Limitations (New - Required for Top Conference)
- ZKP cost: HKDF inside circuit replaced with Poseidon in implementation (~400 constraints vs 100k)
- WebAuthn PRF sync: Stable within Apple/Google ecosystem, cross-ecosystem fallback to encrypted backup
- IA collusion: 2-of-3 threshold prevents single compromise, Trillian provides post-audit detection

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
GitHub repository. https://github.com/takachi73/ZAP-Review
