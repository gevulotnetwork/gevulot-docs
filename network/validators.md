# Validators

Validators are responsible for running verifier programs and coming to consensus on replicated state, such as balances, transfers, staking, prover set, prover rewards and proof system deployments. Gevulot avoids replicated state wherever possible, allowing for exceptional performance without introducing onerous hardware requirements.

**Verification**

All proofs are verified by 2/3 of network validators before they are included in a block by the leader. The verification threshold here simply constitutes finality from the network's perspective, but anyone can verify the proof for immediate use outside of the network.

**Validator Incentives**

Validators are rewarded via a traditional block reward and through a small transaction fee paid by the user. This transaction fee is distinct from the concept of cycle fees, which pay for prover compute time and its primary role is to prevent spam.

Validators are also incentivized to verify proofs (both valid or invalid). They receive a small reward for all proof which at least 2/3 of the validators agree on.\
