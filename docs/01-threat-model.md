## Enhanced Threat Model

### 1. Trust Domains
- TD1 (IA): Real identity DB, isolated VPC, no internet egress to RP
- TD2 (KAA): VSS shares, BLS threshold key, Trillian log
- TD3 (RP): PRK public keys only

### 2. Collusion Analysis
| Collusion | Impact before | Impact after VSS+BLS |
| IA+KAA | Full deanonymization | Requires 3-of-5 PC Chairs BLS signature to reconstruct |
| KAA custodian alone | Can leak share | Feldman VSS verification fails, share rejected |

### 3. STRIDE Detailed
- Spoofing: PRK = Ed25519 keypair, capability token = Macaroon with signature
- Tampering: All KAA ops -> Trillian Merkle Tree, inclusion proof verifiable