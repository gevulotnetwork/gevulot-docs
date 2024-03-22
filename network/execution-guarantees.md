# Execution Guarantees

The Gevulot architecture is designed to enable performance equivalent to centralized systems while ensuring maximum liveness, availability, and execution guarantees, and minimizing the risk of censorship.

## Permissionless participation

Decentralization is a key priority, thus anyone can join Gevulot as a validator or prover node without needing permission. This approach also decreases the risk of censorship in the network.

## Censorship resistance

A highly decentralized network is one of the cornerstones to mitigate the risk of censorship, however, its true capacity to address censorship may be limited unless it is supported by the right prover selection algorithm. Auctions and pure proof-races may both be more susceptible to centralization through prover undercutting or resource-rich provers gaining dominance within the prover set. For this reason and to minimize the risk of censorship, Gevulot uses a Verifiable Random Function (VRF) for leader selection among the validators and workload distribution among the provers.

## Liveness guarantees

#### Prover capacity verification&#x20;

All provers must meet specific minimum hardware requirements and are subject to an initial capacity verification when they join the network. This information is used to allocate proving workloads to provers with the prerequisite hardware to complete a workload. All provers in the active prover set will also have to complete additional randomly assigned workloads to verify that they are continuously available and capable of completing proving workloads, failing which they will be removed from the prover set.

To learn more about capacity verification, please visit the  “[Global prover set](provers.md#global-prover-set)” section of the docs.

#### Fallback mechanism

A prover can choose to decline a workload if its capacity or bandwidth is constrained, however,  there is a limit to the number of workloads a prover can decline. If a workload is declined by the assigned prover, any prover in the active set of provers can contribute a proof.&#x20;

Similarly, if a prover does not decline the workload, but fails to generate a proof within the specified max compute time, the opportunity to generate a proof is opened to the entire prover set. This mechanism allows any prover to step in, produce a proof, and claim the associated reward.

This fallback mechanism ensures that no single point of failure compromises the system's efficiency. Following the inaction of a prover (no decline and no delivery), the non-responsive prover is removed from the set of active provers.

#### Customizable redundancy

Users can customize and set increased redundancy by creating multiple workload requests for the same inputs. In case of added redundancy (multiple single-prover workloads) each prover is rewarded proportional to their own performance.

## Optimized network performance

The Gevulot network is exclusively optimized for zero-knowledge proving and verification. By removing all blockchain features that don’t serve this purpose, Gevulot achieves centralized-equivalent performance.&#x20;

Blocks on Gevulot include transactions and proofs. There is no traditional smart contract state and re-execution, and the verification of computation is done through validating zk-proofs.

## Cost-efficient proving

Gevulot aims to dramatically decrease the costs associated with proving. All aspects of the architecture and network roll-out are designed with this in mind. Firstly, all Gevulot prover nodes are run on-prem and the economics are designed to make running prover nodes in a cloud or even via server rental unprofitable over the long term. We estimate that on-prem setups are around 50% cheaper than cloud within some assumptions for cost of energy and bandwidth.\
\
Secondly, all prover nodes are sized to be able to run multiple provers and verifiers in parallel. Amortizing the cost of larger machines over more workloads leads directly to better cost-efficiency. Finally, Gevulot minimizes all other network-related costs. The blockchain is extremely minimal, hosting no smart contract state and including a minimal amount of possible operations.\
\
All of these steps lead to significant cost-structure improvements when the network is at consistently near capacity. To ensure this is the case, the network architecture has been designed in a way which incentivizes the global prover set to grow when there is more demand than supply and shrink when there is more supply than demand. In other words, the network scales horizontally without increasing the cost structure.\
\
As demand grows, the network also begins to enjoy economies of scale, incentivizing larger-scale optimization efforts. The network essentially functions as a global search algorithm for the cheapest compute, energy, and bandwidth and passes those savings onto the user.

## Fair workload distribution

As described in the [“Global prover set”](provers.md#global-prover-set), workloads are assigned to prover nodes randomly.&#x20;

When distributing these workloads, the verifiable random function excludes any provers that were assigned workloads in the previous round to ensure a fair distribution of work. This selection process allows the prover of a given workload to be calculated asynchronously, allowing each active prover node to have an equal opportunity to be selected for new workloads, and avoiding any single prover to be overloaded.

\
\
