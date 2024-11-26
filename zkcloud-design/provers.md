# Network Actors

## **Validators**

**Validator set**

Validators in Gevulot are responsible for ordering transactions and proofs into blocks and achieving consensus on replicated states, such as balances, transfers, staking, prover set, prover rewards, and program deployments. Gevulot strives to avoid replicated states whenever possible, enabling exceptional performance while minimizing hardware requirements for validators.

#### **Participation**

Participation in Gevulot’s validator set is permissionless, anyone can join the network. A Gevulot validator node requires modest hardware resources. Validators must deposit an amount of native tokens to participate in the network. The deposited stake will be locked for the entire duration of their active participation. Leaving the active validator set has a cooldown period before the stake can be unlocked.

#### **Leader selection and block-building**

At every block, a leader is selected from the active validator set through a Verifiable Random Function (VRF) in a deterministic manner. The leader is responsible for ordering transactions and proofs into blocks. Once the network reaches consensus, the block is finalized and becomes part of the canonical chain.

#### **Consensus mechanism**

To achieve consistent, high-performance, and secure state replication, Gevulot employs CometBFT as its consensus engine, which is based on the Tendermint consensus algorithm.

## Provers

The Gevulot network features two distinct sets of provers designed to facilitate proof generation and verification. The versatility of these sets allows for a wide range of use cases, empowering users to choose or integrate external software that best suits their operational needs. This approach eliminates the dependency on a universal, one-size-fits-all software solution across the network, thereby enhancing flexibility and user autonomy.&#x20;

### **Global prover set**

The Global Prover Set serves as the foundational layer within the network, where provers are initially integrated and assigned proving workloads. Participation in this primary prover set is governed by two key criteria to ensure security and equitable distribution of tasks:

* **Prover staking:** Provers must stake an amount of native tokens to be eligible to participate in the set. This requirement serves as a security measure and ensures commitment to the network. The deposited stake will be locked for the entire duration of their active participation.&#x20;
* **Proof of Workload completion:** Upon joining the network prover nodes need to complete designated proof of workload tasks. Successfully completing these tasks demonstrates that the prover's hardware meets the network's minimum requirements, thereby qualifying them for workload allocation. \
  In addition, all provers in the active prover set that have not completed a proving workload for more than 3 hours (assuming a network utilization of 70% or more) will also have to complete additional randomly assigned Proof of Workload tasks to verify that they are available and capable of generating proofs, failing which they will be removed from the prover set.

A cooldown period is instituted for provers exiting the global prover set, during which their stakes remain locked to ensure network stability.

#### **Workload allocation**

Workload allocation within both the global and custom prover sets utilizes a Verifiable Random Function (VRF) for fair and deterministic distribution. The system ensures diversity in workload assignment by excluding provers that participated in the previous round, thereby preventing overload and maintaining a balanced distribution of tasks.

Every workload is allocated to one prover through the VRF. All provers need to accept or decline their assigned workloads, failing which they will be removed from the prover set after a timeout of 3 blocks.

Provers can decline workloads when faced with capacity or bandwidth limitations. However, there is a limit to the number of workloads a prover can decline: at most one workload a week, or four workloads a month. Exceeding this limit will also result in being removed from the prover set.&#x20;

If the selected prover does not decline or accept the workload, the network will automatically select another prover randomly and reallocate the workload accordingly within the same allocation round. This applies to both the initial allocation of a workload and the fallback scenario.

#### **Fallback mechanism**

If a prover fails to complete the workload within the maximum compute time, the fallback mechanism is activated, and the workload is allocated to one additional prover selected randomly from the global prover set. This ensures that network redundancy and efficiency are not compromised. The same economic structure applies to the fallback allocation as to the initial one. (For more details on rewards, visit the [Economics](fees.md) section.)

If the fallback prover successfully generates the proof within the max compute time:&#x20;

1. The requester gets back 50% of the fees as compensation for the delay in proving.
2. The fallback prover receives regular rewards.&#x20;
3. The initial prover gets kicked out of the prover set.&#x20;

However, if the fallback prover also fails to generate the proof within the allocated time, all fees will be burned.&#x20;

### **Custom prover sets**

Nested within the global prover set, custom prover sets enable a modular approach for data storage and allow the integration of distinct external software for various use cases. In short, they allow a deployer to define external software that prover nodes need to run alongside the Gevulot node to join the prover set.

Custom prover sets significantly increase the system's flexibility and efficiency through several key benefits:

* Reduces the complexity of maintaining extensive data repositories across the network.
* Encourages a rich ecosystem of specialized provers, fostering innovation and diversity.
* Supports voluntary participation, enabling the network to evolve dynamically in response to user demands without enforcing a uniform architecture.

#### **External software communication**

Messaging between external programs and prover programs over a network involves creating a virtual, isolated network within a node, using virtual network interfaces like TUN/TAP, to ensure secure and confined interactions. This setup allows for flexible configurations, where each program, whether a prover, verifier, or external software, communicates over this isolated network, employing various protocols such as custom protocols, gRPC, or JSON-RPC based on performance and integration needs. Service discovery within this isolated network could leverage mechanisms like mDNS or configuration details passed through the task definition, ensuring programs know where and how to connect. This approach offers node operators the flexibility to maintain strict isolation for security and integrity while providing the ability to integrate a wide range of external software and communication protocols.

**Custom prover set initialization**

Custom prover sets are programmed and verified through an initialization prover node. When creating a custom set, the user defines a trusted node with a unique ID and initializes the network state, which other nodes validate upon joining. If the initialization node goes offline, other online nodes can assume its role, ensuring continuous validation and network integrity.

#### Prover participation

To join any custom prover set, provers must be part of the global prover set. Participation in custom prover sets is optional, but the criteria for joining are similar to that of the global set. Custom prover sets require proof of workload completion specific to the specialized software features. This proof of workload is generated by the deployer of the prover program and verifies the provers' capability in processing the custom sample inputs for proof generation.

#### **Workload allocation**

When a workload transaction is initiated on a prover program requiring external software, only those prover nodes that explicitly opt in to participate in the corresponding custom prover set will be included in the Verifiable Random Function (VRF) process. The network maintains transparency in participation decisions, keeping all users informed about the number of active provers in the custom prover sets.

#### Proof pricing

Within custom prover sets, proof pricing is set by the deployer of the prover program and remains consistent across all prover nodes in the set. This ensures uniform pricing within a custom prover set, maintaining the fairness and reliability of the randomness produced by the VRF.&#x20;

However, users have the option to deploy distinct provers with unique pricing where participation can be restricted to specific nodes. This provides the opportunity to develop more flexible pricing strategies that cater to diverse needs and circumstances.

To prevent potential disparities, such as proprietary provers offering proofs at no cost or at rates lower than those of open-source counterparts, a standardized minimum cost is enforced across the network. This prevents any single entity from skewing the pricing dynamics, ensuring equitable and sustainable operation within the network.

\
