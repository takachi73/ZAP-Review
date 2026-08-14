Macaroon Example:
token = HMAC(root_key, PRK_paper_pub) + caveats
caveats = [paper_id=1234, action=joinDiscussion, exp=2027-03-01T00:00Z]

RP verifies with OPA policy:
allow if token.signature valid and caveats satisfied and not in revocation list