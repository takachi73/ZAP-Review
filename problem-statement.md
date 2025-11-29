# Problem Statement  
## Structural Vulnerabilities in Current Peer Review Systems  
### Why a Zero-trust, Audited, Pseudonymous Architecture Is Necessary

---

## 1. Background

The peer-review infrastructure used by modern academic conferences—particularly in AI/ML  
(e.g., NeurIPS, ICML, ICLR, ACL, CVPR)—is grounded in platform designs that originated  
more than a decade ago. These systems were built under assumptions that no longer hold:

- reviewer populations were small  
- anonymity risk was relatively low  
- adversarial behavior was not considered  
- administrators were implicitly trusted  
- cryptographic separation of identity and actions was infeasible  
- review loads were manageable

Today, the AI/ML ecosystem is radically different:

- submissions have grown exponentially  
- reviewer diversity is high and globally distributed  
- adversarial actors exist across industry and academia  
- social media amplifies harassment and retaliation risk  
- insider threats and API exposure are realistic attack vectors  

Yet the review infrastructure has **not evolved proportionally**.  
Current platforms share a foundational architectural flaw.

---

## 2. Core Structural Problem

In all major existing review systems (OpenReview, CMT, HotCRP, EasyChair),  
the following information exists within a **single trust domain**:

1. Real identities of reviewers  
2. Reviewer roles (reviewer / AC / SAC / PC)  
3. Paper–reviewer assignment tables  
4. Review text and numerical scores  
5. Internal discussion logs  
6. Administrative metadata and permissions

This means:

> If a single admin account, server, database, or API endpoint is compromised,  
> **the entire anonymous review process collapses.**

This is not hypothetical; it is an *architectural inevitability* under the current model.

---

## 3. The 2025 OpenReview Identity Leak  
### A Case Study Demonstrating Architectural Failure

In November 2025, a misconfigured API endpoint (`profiles/search?group=`) exposed  
the ability to enumerate:

- reviewer identities  
- conference groups  
- role assignments  

This allowed external actors to reconstruct:

- reviewer → paper mappings  
- AC and SAC identities  
- review network topology  

Although review text was not leaked, the event demonstrated that:

> **Existing systems cannot guarantee anonymity,  
> because identity, roles, and assignments coexist in one system.**

No amount of policy-writing or patching can eliminate this class of vulnerability.

---

## 4. Inherent Risks in Current Peer Review Platforms

### 4.1 Single-Point Identity Failure  
One leak or misconfiguration can deanonymize the entire reviewer population.

### 4.2 Malicious Administrators & Insider Threats  
Platform administrators have unrestricted access by design.  
There is no cryptographic barrier preventing identity disclosure.

### 4.3 API Enumeration Attacks  
Most systems include:

- group listing APIs  
- profile search functionality  
- role metadata endpoints  

which can expose identities via:

- indexing  
- scraping  
- misconfigured access control  
- caching leaks  

### 4.4 Correlation & Inference Attacks  
Even without explicit leaks, identities can be inferred from:

- affiliation metadata  
- email domains  
- assignment topology  
- timezone activity patterns  
- load balancing behavior  
- writing style across multiple reviews (stylometry)

### 4.5 Harassment, Retaliation, and Safety Risks  
Identity exposure—intentional or accidental—can lead to:

- targeted harassment  
- professional or legal threats  
- retaliation by authors or institutions  
- chilling effects on honest and critical reviewing  

These harms disproportionately affect junior researchers.

### 4.6 Structural Bias  
Internal visibility of identity or affiliation may influence:

- reviewer selection  
- paper decisions  
- AC assignments

### 4.7 Lack of Long-term Reviewer Incentives  
Current systems do not provide:

- sustainable recognition  
- cross-conference reputation  
- incentives for high-quality reviews  
- mechanisms to discourage low-quality or AI-generated reviews

This contributes to reviewer shortages.

---

## 5. Why Existing Platforms Cannot Be Fixed Through Patching

The fundamental issue is not bugs; it is **architectural coupling**.

Identity, authority, and assignments are:

- stored together  
- governed by a single system  
- accessible to the same administrators  
- reachable through unified APIs  

Even with:

- strict policies  
- monitoring  
- role auditing  
- access logs  

the architecture remains inherently insecure.

Three questions expose the flaw clearly:

### **(A)** Why does the platform need to know *who* the reviewer is?  
### **(B)** Why do administrators have global visibility?  
### **(C)** Why can a single database reconstruct reviewer → paper mappings?

Under zero-trust principles, the correct answers should be:

- It doesn’t.  
- They shouldn’t.  
- It must be cryptographically impossible.

Current systems cannot satisfy these without **architectural redesign**.

---

## 6. Requirements for a Next-generation Peer Review Architecture

A secure and trustworthy system must satisfy:

### ✔ Zero-trust identity separation  
No single actor or system can reconstruct identities.

### ✔ Cryptographic pseudonymous reviewer identities  
Reviewer actions must be unlinkable to their real identities.

### ✔ Distributed, quorum-based identity recovery  
Identity disclosure must require multi-party approval (e.g., 3-of-5).

### ✔ Capability-based access controls  
APIs must enforce minimum privilege and prevent global role enumeration.

### ✔ Cross-conference, privacy-preserving reputation  
High-quality reviewers must be recognized without exposing identity.

### ✔ Incentive systems that prevent “bad reviews driving out good reviews”  
AI-generated or low-effort reviews must be penalized, not rewarded.

### ✔ Scalability and resilience  
The system must handle modern AI/ML submission volumes.

---

## 7. Motivation for ZAP-Review

ZAP-Review (Zero-trust Audited Pseudonymous Review) emerges directly from  
these structural needs.

It provides:

- cryptographically generated pseudonymous identities (PRKs)  
- full separation of identity, authority, and review actions  
- secret-shared identity escrow  
- capability-token APIs  
- differential-privacy-based reputation  
- secure incentives for high-quality reviewing  
- protection against enumeration, insider threats, and AI-generated spam  

ZAP-Review is **not an incremental improvement**.  
It is a **new architectural foundation** for academic peer review.

---

## 8. Summary

Current peer-review systems are fundamentally insecure because they:

- store identity and review actions together  
- rely on trusted administrators  
- expose global metadata through APIs  
- provide no sustainable reviewer incentives  
- offer no cryptographic anonymity guarantees  
- cannot withstand modern attack surfaces  

A secure, scalable, trustable peer-review ecosystem requires  
a **zero-trust, audited, pseudonymous architecture**.

This defines the mission and necessity of **ZAP-Review**.

