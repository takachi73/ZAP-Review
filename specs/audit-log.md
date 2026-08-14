## Audit Log

All KAA ops: KeyGen, ShareDist, Rekey, OpenRequest

Log entry: {op_type, PRK_long_hash, timestamp, BLS_sigs_hash, merkle_leaf_hash}
Trillian: Append-only, inclusion proof API: /log/v1/get-proof-by-hash

Verifier: Anyone can fetch proof that their PRK issuance was logged