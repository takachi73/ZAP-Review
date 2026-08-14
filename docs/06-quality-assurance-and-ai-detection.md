# 06 - Quality Assurance, AI-Generated Review Mitigation, and DP Trade-off

## Problem 1: AI Detection Limitations and False Positives

Current approach: Penalty (reputation drop, temp BAN) for low-quality or AI-generated reviews with differential privacy epsilon=1.0.

**Gaps:**
1. No AI detector achieves high precision without false positives. Penalizing honest reviewers based on false positives destroys trust.
2. If IA and RP are separated, appeal and recovery process for wrongful penalty is extremely complex.
3. epsilon=1.0 adds too much noise: Laplace(1/epsilon) with std ~1.4 makes it impossible to distinguish good (4.5) vs average (3.5) reviewers.

## Proposed Solution A: AI Detection as Trigger, Not Verdict

**Pipeline:**
submitReview()
-> AI-likelihood score (RoBERTa + perplexity + citation hallucination check)
-> If score < 0.8: accept
-> If score >= 0.8: enqueue to Senior AC Review Queue (NOT auto penalty)


Senior AC (anonymized) does 2-min check:
- Does review cite specific line numbers / figures?
- Does review address paper's method, not generic knowledge?

Decision: 
- Pass: No penalty, flag removed
- Fail: Low-quality, apply staged penalty

**Staged Penalty:**
- 1st: Warning + reputation -0.1
- 2nd: reputation -0.5 + mandatory guideline re-read
- 3rd: 1-conference temp BAN

No single automated detector can directly reduce reputation.

## Proposed Solution B: Anonymous Appeal Channel

Because IA || KAA || RP separation makes recovery complex, we need appeal without deanonymization.

**Anonymous Appeal Protocol:**

1. Reviewer: `AppealToken = Sign(PRK_paper_sk, {paper_id, review_id, reason_hash})`
2. KAA verifies signature belongs to valid PRK_paper, forwards to 3 random Senior ACs (anonymized)
3. Senior ACs re-evaluate review blindly
4. If 2-of-3 overturn: KAA restores reputation and removes flag, IA is never involved

Audit: All appeals logged in Trillian, inclusion proof available to reviewer.

## Proposed Solution C: Differential Privacy Re-evaluation

**Analysis of epsilon=1.0:**

Reputation score domain [1,5] normalized to [0,1], sensitivity=1.
Noise = Lap(1/epsilon) = Lap(1). Std ~1.41. 
Signal difference between good vs average = 1.0. SNR < 1, incentive fails.

**New Proposal: Bucketized Disclosure, not Pure DP**

Reputation is already pseudonymous (PRK_long), not linked to real_id. Pure DP is overkill.

Instead:

1. KAA computes true reputation internally with no noise
2. For external display (RP leaderboard), map to 3 buckets:
      - Top 10% 
      - Top 30%
      - Average
3. No Laplace noise, just coarse-graining. k-anonymity within bucket >=20.

If DP is required for paper: Use epsilon = 4.0 to 8.0 and show utility-privacy trade-off graph. epsilon=1.0 should be documented as "too private to be useful".

**Evaluation Plan:**
- Simulate 1000 reviewers, true scores N(3.5,0.5)
- Plot ranking accuracy vs epsilon [0.5,1,2,4,8]
- Show that epsilon=1.0 -> Kendall tau <0.3, epsilon=4 -> tau >0.7

## Proposed Solution D: Quorum-based Identity Opening as Accountable Anonymity

"Identity can only be opened if quorum agrees" (Feldman VSS escrow) is the best compromise for cartel / severe misconduct.

Formalize as:
Accountable Anonymity = Anonymity by default + Accountability under threshold

Opening Procedure remains 5-step but add BLS threshold enforcement in code, not just policy.

## Summary

- AI detection = human-in-the-loop trigger
- Appeal = anonymous via KAA, no IA involvement
- DP = bucketization, not epsilon=1.0
- Opening = code-enforced BLS threshold

This makes quality assurance fair and restorable, which is essential for reviewer trust.
