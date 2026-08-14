# 04 - Reputation, Incentive, and Verifiable Career Credentials (v2)

## 1. Problem: Gap Between Pseudonymous Reputation and Real-World Incentive

ZAP-Review provides long-term pseudonymous reputation via PRK_long that persists across conferences. Reviewers earn better assignments and eligibility for AC/PC roles.

Critical Gap (CV Problem): In academia, the main incentive for reviewing is to prove service for tenure: "Served as PC for ICLR 2027". How can a reviewer prove ownership of a high-reputation PRK without revealing which papers they reviewed?

## 2. Threat Model: Credential Transfer Attack (Fixed in v2)

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

Root cause: PRK issuance was not bound to real identity at registration.

## 3. Design Goals
1. Real-world verifiability
2. Paper anonymity
3. Non-transferability
4. Separation of powers: IA never learns reputation, KAA never learns real_id
5. Unlinkability
6. Recoverability

## 4. Protocol v2: Identity-Bound Anonymous Credentials

### 4.1 Primitives
- Pedersen Commitment: com_id = Commit(real_id, r)
- BBS+ Signature for selective disclosure
- Groth16 ZKP
- KDF: PRK_long = KDF(WebAuthn PRF || com_id)

### 4.2 Registration - Binding at Birth
Client-side during ORCID auth:
r = random()
com_id = pedersen_commit(Hash(X), r)
prf_out = webauthn_get_prf("ZAP-Review")
PRK_long = KDF(prf_out, com_id)

To IA: com_id + ZKP well-formed for X
To KAA: com_id + Commit(PRK_long)
IA stores: X <-> com_id
KAA stores: com_id <-> PRK_long commitment
Binding X --com_id-- PRK_long without single party knowing both ends.

### 4.3 Phase 1: KAA Issues ARC_v2
After conference:
ARC_v2 = BBS+_Sign(KAA_sk, {
  prk_commitment: Commit(PRK_long),
  id_commitment: com_id, // CRITICAL
  conference_id: "ICLR2027",
  reputation_bucket: "Top10%",
  role: "reviewer"
})

### 4.4 Phase 2: IA Issues RAC via ZKP_v2
Browser generates:
ZKP_v2 = Prove {
  public: com_id, KAA_pk
  private: X, r, PRK_long, ARC_v2
  Statement:
    1. com_id == Commit(X, r)
    2. BBS+_Verify(KAA_pk, ARC_v2, {Commit(PRK_long), com_id, ...})
    3. PRK_long == KDF(WebAuthn PRF, com_id)
}
IA verifies ZKP_v2, then issues:
RAC = BBS+_Sign(IA_sk, {
  real_id: X,
  achievement: "Top10% Reviewer at ICLR2027",
  conference_id: "ICLR2027",
  bucket: "Top10%",
  log_index: Trillian_index
})

### 4.5 Phase 3: Verification
Reviewer discloses {X, Top10% at ICLR2027} via BBS+ selective disclosure.
University verifies with IA_pk.

## 5. Security Analysis

| Attack | v1 | v2 Mitigation |
| Credential Transfer A->B | Vulnerable | Impossible. B cannot open Commit(A) as Commit(B) |
| Sybil | Possible | IA enforces 1 com_id per real_id via ORCID |
| IA Forges RAC | Possible | Threshold BLS + Transparency Log |
| KAA Learns Real ID | No | KAA only sees hiding com_id |

Proof Sketch: Transfer requires finding r' with Commit(A,r_A)=Commit(B,r'). Breaks Pedersen binding.

## 6. Key Management and UX - Mitigating Key Loss

Layer 1: WebAuthn Passkey PRF (Primary)
PRK_long = KDF(WebAuthn_PRF, com_id)
- Stored in Secure Enclave, synced via iCloud/Google
- No raw key export

Layer 2: Encrypted Cloud Backup
ciphertext = AES-GCM_Encrypt(passphrase, PRK_long)
Stored in KAA, KAA cannot decrypt.

Layer 3: Social Recovery
r Shamir-shared 2-of-3 to trusted peers.

UX: Click "Generate Career Proof" -> Touch ID -> 3 sec WASM ZKP -> Download PDF VC.

## 7. Governance: Mitigating IA Centralization

A: Threshold BLS: IA_sk is 2-of-3 BLS (ACM, IEEE CS, ICLR Foundation)
B: Transparency Log: Every RAC appends Hash(RAC) to Trillian Merkle tree. Public audit of counts.
C: Consortium Agreement: Annual audit.

## 8. Implementation
- BBS+: mattrglobal/bbs-signatures
- ZKP: Circom + SnarkJS Groth16
- WebAuthn: SimpleWebAuthn PRF extension
- Commitment: curve25519-dalek
- Log: Google Trillian / Sigstore Rekor

## 9. Evaluation Plan
1. Non-transferability: ProVerif model
2. Performance: ZKP time on M1 / iPhone / Pixel (<4 sec)
3. UX: 20 researchers SUS score
4. Governance: Simulate 1 malicious IA share

## 10. Future Work
- k-anonymous multi-conference proof
- RAC revocation list