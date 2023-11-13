# Provers

Prover nodes complete proving workloads. To join the active prover set, prover nodes must be staked and must provide information on their hardware capabilities. This information is used to allocate proving workloads to provers with the prerequisite hardware to complete a workload.

**Active Prover Set**

Provers must be in the active prover set to be allocated proving workloads. Joining the active prover set requires a stake and the completion of a proof of work workload or workloads to verify the provers hardware meets the hardware requirements. All provers in the active prover set will also have to complete additional randomly assigned proof of work workloads, which must be completed within the time constraints or the prover will be removed from the set. Leaving the active prover set always has a cooldown period before the stake will be unlocked.

**Proving**

Each proving workload is allocated to one or more prover nodes in the active prover set using a verifiable random function (VRF) in a deterministic manner, so that the prover for a given workload can be calculated asynchronously.

The user pays for a set amount of cycles per prover and the chosen provers run the program either until the program completes, in which case they return the output, or for the maximum paid for cycles without completing, in which case they return fail.

A prover can choose to decline a workload if they are at capacity or bandwidth constrained. If a workload is declined by at least one allocated prover, any prover in the active prover set can contribute a proof, but only the first of these will be rewarded and the reward schedule does not change. For example, a workload is allocated to 3 provers and 1 of them declines. The reward for a single proof in that group is then up for grabs and any prover can contribute a proof and be paid out as if they were allocated that workload. However, after 3 proofs have been generated, two via allocation and one via open market, any subsequent proofs generated will not get a reward.

**Copy Protection**

Once a prover generates a proof for a given workload, the proof is encrypted with a secret key. This key is then further encrypted by the public key of the user that broadcast the workload and a temporary prover key, leading to two encrypted versions of the proof. These encrypted proofs are then broadcast to the mempool. The end-user can then immediately pick up and decrypt the proof for use. Once the cycles have been exhausted or all the proofs have been generated and broadcast, the prover then broadcasts the temporary prover key so the validators can decrypt the proof and verify. \
\
This mechanism prevents provers from copying completed proofs from the mempool and broadcasting them as their own, while ensuring the end-user receives proofs as soon as they are finished. This mechanism does delay verification, but in practice this delay has a negligible effect on speed of finality.&#x20;

**Prover Incentives**

There is a prover reward paid by the network, which is paid to provers who complete the workload in the given cycle time. It is tiered based on speed, wherein the first node to return an output receives the majority of the reward, the second a smaller portion and the third a smaller portion still. All nodes who complete the workload in the given cycle time receive at least some portion of the reward, in addition to their portion of the fees paid by the user. If the program does not complete, the fees are burned.

Note: Proof of work workloads may also be rewarded but the reward will be considerably smaller than that for end-user workloads.
