## KAA: Feldman VSS

1. KAA generates master_seed (256bit) and polynomial f(x) degree t-1=2
2. Commitment C_j = g^{a_j} for coefficients a_j
3. Share s_i = f(i), verifier checks g^{s_i} == product C_j^{i^j}
4. Reconstruction requires:
   - 3 BLS signatures from PC Chairs on "OPEN_ID request #123"
   - Then 3 shares combined via Lagrange interpolation

## PRK Lifecycle v2

master_seed -(HKDF)-> PRK_long = SHA256(master_seed) [cross-conference pseudonym]
PRK_long -(HKDF)-> PRK_conf_2027 = HKDF(PRK_long, "ICLR2027")
PRK_conf_2027 -(HKDF)-> PRK_paper = HKDF(PRK_conf, paper_id)