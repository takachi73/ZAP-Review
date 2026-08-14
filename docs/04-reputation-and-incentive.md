# 04 - Reputation, Incentive, and CV Problem with ZKP Verifiable Credentials

## Problem: Gap between Pseudonymous Reputation and Real-World Incentive

ZAP-Review's core value is long-term pseudonymous reputation (PRK_long) that persists across conferences. Reviewers earn better assignments and eligibility for AC/PC roles based on quality, not quantity, with no monetary reward.

**Critical Gap:** The primary motivation for reviewers in academia is to show service on their CV for tenure and promotion: "Served as PC for ICLR 2027". How can a reviewer prove that they own a high-reputation PRK without breaking anonymity and revealing which papers they reviewed?

If this is not solved, high-quality reviewers will not join ZAP-Review.

## Requirements

1.  Real-world verifiability: A university can verify "X was Top 10% reviewer at ICLR2027"
2.  Paper anonymity: University must NOT learn which papers X reviewed
3.  Separation of powers: IA must NOT learn reputation scores, KAA must NOT learn real identity
4.  Unlinkability across conferences: Proving achievement at Conf A must not link to PRK at Conf B

## Proposed Solution: Two-Stage Verifiable Credential with BBS+ and Blind Signatures

We introduce Accountable Career Credentials using W3C Verifiable Credentials (VC) + BBS+ Signatures for selective disclosure.

### Actors
- KAA: Knows reputation[PRK_long], does NOT know real identity
- IA: Knows real identity, does NOT know reputation or paper assignments
- Reviewer: Knows both, holds PRK_long secret key
- Verifier: University / Employer

### Protocol

**Phase 1: KAA issues Anonymous Reputation Credential (ARC)**

After conference ends, KAA computes for each PRK_long:
ARC = BBS+_Sign(KAA_sk, {
prk_long_commitment = Commit(PRK_long),
conference_id = "ICLR2027",
reputation_bucket = "Top10%", // not exact score
role = "reviewer"
})


ARC is sent to reviewer via PRK secure channel. KAA does not know real identity.

**Phase 2: IA issues Real-World Achievement Credential (RAC) via Blind Proof**

Reviewer generates ZKP:
Proof = ZKP{
I know sk such that:

Commit(PRK_long) opens to PRK_long corresponding to ARC
2. ARC is validly signed by KAA
3. My real identity is registered as X in IA
}

Reviewer sends Proof (not ARC itself) to IA. IA verifies Proof and blindly issues:
RAC = BBS+_Sign(IA_sk, {
real_id = X,
achievement = "Top10% Reviewer at ICLR2027",
timestamp,
// NO PRK, NO paper_ids, NO exact score
})


**Phase 3: Selective Disclosure to Verifier**

Reviewer presents RAC to university. Using BBS+ selective disclosure, reviewer can prove:

"I am X and I hold a valid RAC for Top10% at ICLR2027"

Without revealing: PRK_long, ARC, or any link to other conferences.

### Security Analysis

| Attack | Mitigation |
|---|---|
| IA learns which papers reviewer reviewed | IA only sees ZKP of ARC ownership, not paper_ids |
| KAA learns real identity | KAA only sees PRK_long, never real_id |
| University links reviewer across conferences | Each RAC is independent, BBS+ provides unlinkability |
| Collusion IA+KAA | Requires breaking ZKP soundness + BBS+ unforgeability. Still needs 3-of-5 BLS threshold for full deanonymization |

### Implementation

- Library: `docknetwork/crypto` or `mattrglobal/bbs-signatures` for BBS+
- ZKP: Circom + Groth16 for proof of BBS+ ownership
- VC Format: W3C VC v2.0 JSON-LD

### Open Issue

IA becomes VC issuer, increasing its trust. Mitigation: IA's signing key is itself threshold-shared (2-of-3 university registrars). IA can only sign if KAA's ARC proof is valid, enforced in code.

## Roadmap

- Phase 1: Define ARC/RAC schema
- Phase 2: Prototype with Dock BBS+
- Phase 3: User study on tenure committees' acceptance
