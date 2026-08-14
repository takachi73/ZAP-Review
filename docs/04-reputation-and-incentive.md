# 04 - Reputation, Incentive, and Verifiable Career Credentials (v2.1 - Deterministic Recovery)

## 1. Problem: Gap Between Pseudonymous Reputation and Real-World Incentive

ZAP-Review provides long-term pseudonymous reputation via PRK_long that persists across conferences. Reviewers earn better assignments and eligibility for AC/PC roles.

Critical Gap (CV Problem): In academia, the main incentive for reviewing is to prove service for tenure: "Served as PC for ICLR 2027". How can a reviewer prove ownership of a high-reputation PRK without revealing which papers they reviewed?

## 2. Threat Model: Credential Transfer Attack (Fixed in v2.1)

### v1 Vulnerability
In v1 protocol, ZKP proves:
Proof_v1 = { I possess ARC AND My real identity is X }
But it does NOT prove:
Proof_required = { I possess ARC AND The true owner of ARC is X }

Attack Scenario:
1. Excellent reviewer A receives ARC Top10%
2. A transfers PRK_A + ARC to friend B
3. B generates ZKP using B's real identity and A's ARC
4. IA issues RAC: B is Top10% reviewer -> Fraud

Root cause: PRK issuance was not bound to real identity, and blinding factor r was random.

## 3. Design Goals
1. Real-world verifiability
2. Paper anonymity
3. Non-transferability
4. Separation of powers
5. Unlinkability
6. Recoverability with single Touch

## 4. Protocol v2.1: Identity-Bound Anonymous Credentials with Deterministic Recovery

### 4.1 Primitives
- Pedersen Commitment: com_id = g^{Hash(X)} * h^{r}
- BBS+ Signature
- Groth16 ZKP
- WebAuthn PRF
- HKDF-SHA256

### 4.2 Registration - Binding at Birth with Deterministic r (Core Improvement)

During ORCID auth (client-side):
prf_out = webauthn_get_prf("ZAP-Review:v2.1") // 32 bytes

r = HKDF-SHA256(ikm=prf_out, salt=0x00, info="ZAP-Review:v2.1:blinding_factor")
PRK_long = HKDF-SHA256(ikm=prf_out, salt=0x00, info="ZAP-Review:v2.1:prk_long")
com_id = PedersenCommit(Hash(X), r)

Why better than v2:
- No random r to lose: Touch ID -> same r reconstructed
- Stable com_id: Same X always same com_id -> Sybil prevention
- No encrypted cloud backup needed
- Domain separation: r and PRK_long independent

To IA: com_id + ZKP well-formed for X
To KAA: com_id + Commit(PRK_long)
IA stores: X <-> com_id
KAA stores: com_id <-> PRK_long commitment

### 4.3 Phase 1: KAA Issues ARC_v2.1
ARC_v2.1 = BBS+_Sign(KAA_sk, {
  prk_commitment: Commit(PRK_long),
  id_commitment: com_id,
  conference_id: "ICLR2027",
  reputation_bucket: "Top10%",
  role: "reviewer",
  issuance_epoch: 2027
})

### 4.4 Phase 2: IA Issues RAC via ZKP_v2.1

Browser flow:
1. Click "Generate Career Proof"
2. WebAuthn prompt: Touch ID
3. Browser reconstructs:
prf_out = webauthn_get_prf("ZAP-Review:v2.1")
r = HKDF(..., "blinding_factor")
PRK_long = HKDF(..., "prk_long")

4. Browser generates ZKP:
ZKP_v2.1 = Groth16_Prove {
  public: com_id, KAA_pk, IA_pk
  private: X, r, PRK_long, ARC_v2.1, prf_out
  Statement:
    1. com_id == Commit(Hash(X), r)
    2. r == HKDF(prf_out, "blinding_factor")
    3. PRK_long == HKDF(prf_out, "prk_long")
    4. BBS+_Verify(KAA_pk, ARC_v2.1, {Commit(PRK_long), com_id, ...}) == true
}

Reviewer sends ZKP_v2.1 (NOT ARC, NOT r) to IA.

IA verifies ZKP_v2.1, then issues:
RAC = BBS+_Sign(IA_sk_threshold, {
  real_id: X,
  achievement: "Top10% Reviewer at ICLR2027",
  conference_id: "ICLR2027",
  bucket: "Top10%",
  log_index: Trillian_index
})

### 4.5 Phase 3: Verification
Disclosed: { real_id = X, achievement = Top10% at ICLR2027 }
University verifies BBS+_Verify(IA_pk, RAC)

## 5. Security Analysis

| Attack | v1 | v2.1 Mitigation |
| Credential Transfer A->B | Vulnerable | Impossible. B cannot produce r' = HKDF(prf_B) that opens Commit(A) |
| Blinding Factor Loss | Fatal in v2 | Impossible in v2.1. r is deterministically recoverable |
| Sybil | Possible | 1 com_id per real_id via ORCID + stable com_id |
| IA Forges RAC | Possible | Threshold BLS + Transparency Log |
| KAA Learns Real ID | No | KAA only sees com_id |

Non-Transferability Proof Sketch: Transfer requires Commit(A,r_A)=Commit(B,r_B) where r_A=HKDF(prf_A). Breaks Pedersen binding OR HKDF collision OR WebAuthn PRF - infeasible.

Deterministic Recovery Proof Sketch: Same Passkey -> same prf_out -> same r and PRK_long -> same com_id -> ZKP verifies. No secret storage needed.

## 6. Key Management and UX - v2.1 Simplified

Previous v2 had 3 layers, v2.1 has 1 layer:

Single Layer: WebAuthn Passkey PRF Only
prf_out = WebAuthn PRF("ZAP-Review:v2.1")
r = HKDF(prf_out, "blinding")
PRK_long = HKDF(prf_out, "prk")

- Stored in Secure Enclave, synced via iCloud/Google
- User never sees raw key, no copy-paste
- No encrypted cloud backup needed

UX Flow:
1. Click "Generate Career Proof" -> Touch ID (0.5 sec)
2. WASM reconstructs r and PRK_long + ZKP (2-3 sec)
3. Download PDF VC + QR code

Edge Case: Passkey Loss = losing Google account. Acceptable. Optional Shamir 2-of-3.

## 7. Governance: Mitigating IA Centralization

Problem: IA is SPOF.

A: Threshold BLS: IA_sk is 2-of-3 BLS (ACM, IEEE CS, ICLR Foundation)
B: Transparency Log: Every RAC appends Hash(RAC) to Trillian Merkle tree
C: Governance Document: Annual audit

## 8. Implementation

- BBS+: mattrglobal/bbs-signatures
- ZKP: Circom deterministic_r_and_bbs_verify.circom (needs 2 HKDF sub-circuits ~30k constraints)
  Total ~1.8M constraints (vs 1.5M in v2)
- WebAuthn PRF: SimpleWebAuthn
- Commitment: curve25519-dalek
- Log: Google Trillian / Sigstore Rekor

Performance Target: <4 sec on M1 Mac, <6 sec on Pixel 8.

## 9. Evaluation Plan

1. Non-transferability: ProVerif model for deterministic r
2. Recovery: 20 users lose device, recover via iCloud, success rate
3. Performance: ZKP time with HKDF overhead
4. UX: SUS score for Touch ID flow vs passphrase backup
5. Governance: Simulate 1 malicious IA share

## 10. Future Work

- k-anonymous multi-conference proof
- RAC revocation list via BBS+ revocation
- Passkey-bound PRK as replacement for ORCID

## References

- BBS+ Signatures: Boneh et al. 2004
- Pedersen Commitments: Pedersen 1991
- WebAuthn PRF: W3C WebAuthn Level 3
- HKDF: RFC 5869
- Certificate Transparency: RFC 6962
- Deterministic Recovery: Idea from reviewer feedback (2026)