## Reputation Algorithm

Input: review r
Evaluators: AC_score(1-5), Peer1_score(1-5), Peer2_score(1-5)
TrimmedMean = mean(sorted scores)[1:-1] if outlier >1.5 std

Noisy Reputation Update:
reputation[PRK_long] = 0.9*reputation[PRK_long] + 0.1*(TrimmedMean + Lap(1/epsilon))

epsilon = 1.0, sensitivity = 1 (score range 1-5 normalized to 0-1)

Promotion: reputation >4.2 and conferences >=3 => eligible for AC
Demotion: reputation <2.0 for 2 consecutive conferences => temp ban