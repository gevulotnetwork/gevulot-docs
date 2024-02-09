# Provers

Prover nodes complete proving workloads. Gevulot aggregates proving workloads across many different applications maximizing both hardware utilization and financial return for provers.

**Active Prover Set**

Provers must be in the active prover set to be allocated proving workloads. To join the active prover set, prover nodes must be staked and must provide information on their hardware capabilities. The completion of the proof of work workloads verifies that the hardware of the provers meets the necessary hardware requirements. This information is used to allocate proving workloads to provers with the prerequisite hardware to complete a workload. All provers in the active prover set will also have to complete additional randomly assigned proof of work workloads, which must be completed within the time constraints or the prover will be removed from the set. Leaving the active prover set always has a cooldown period before the stake will be unlocked.

**Proving**

Each proving workload is allocated to one or more prover nodes in the active prover set using a verifiable random function (VRF) in a deterministic manner. Before allocation, the system excludes any provers that were assigned workloads in the previous round to ensure a fair distribution of work and prevent overburdening of any single prover node. This selection process allows the prover for a given workload to be calculated asynchronously, allowing each active prover node to have an equal opportunity to be selected for new workloads.

The user pays for a certain amount of cycles per prover and the chosen provers either run the program until it completes, in which case they return the proof, or run it for the maximum time paid for cycles without completing it, in which case they return fail.

**Prover Failure**

A prover can choose to decline a workload if it's capacity or bandwidth is constrained. If a workload is declined by an allocated prover, any prover in the active set of provers can contribute a proof, but only the first of them will be rewarded and the reward schedule will not change. For example, a workload is allocated to 3 provers and 1 of them declines it. The reward for a single proof in this group then becomes open and any one prover can contribute a proof and will be paid as if this workload was allocated to them. However, once 3 proofs have been generated, two via allocation and one via the open market, subsequent proofs generated will not get a reward.&#x20;

To ensure the integrity of redundancy guarantees, if a prover does not decline the workload, but fails to generate a proof within the specified cycles - while all other selected provers fulfill their obligations - the opportunity to generate proof is opened to the entire prover set. This mechanism allows any prover to step in, produce the proof and claim the associated reward.&#x20;

**Single Prover**

In a single prover configuration, to foster a competitive environment and prevent delays, the reward for proof submission is designed to gradually decrease over time. If a single prover fails to decline the workload but also fails to submit a proof within the designated cycles, the task becomes available to the entire network within the same timeframe. This ensures that no single point of failure compromises the system's efficiency. Following such inaction, the non-responsive prover is removed from the pool of active provers.

**Copy Protection**

Once a prover generates a proof for a given workload, the proof is encrypted with a secret key. This key is then further encrypted by the public key of the user that broadcast the workload and a temporary prover key, leading to two encrypted versions of the proof. These encrypted proofs are then broadcast to the mempool. The end-user can then immediately pick up and decrypt the proof for use. Once the cycles have been exhausted or all the proofs have been generated and broadcast, the prover then broadcasts the temporary prover key so the validators can decrypt the proof and verify. \
\
This mechanism prevents provers from copying completed proofs from the mempool and broadcasting them as their own, while ensuring the end-user receives proofs as soon as they are finished. This mechanism does delay verification, but in practice this delay has a negligible effect on the speed of finality.&#x20;

**Prover Incentives**

There is a prover reward paid by the network to the provers that complete the workload in the given cycle time. This reward is tiered based on speed; the first node to return a proof receives a large portion of the reward, the second a smaller portion, and the third an even smaller portion. All nodes that complete the workload within the given cycle time receive at least part of the reward, in addition to their share of the fees paid by the user. If the program does not complete, the fees are burned.

Note: Proof of work workloads may also be rewarded but the reward will be considerably smaller than for end-user workloads.
