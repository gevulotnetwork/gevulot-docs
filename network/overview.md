# Overview

The Gevulot network stands out with its innovative dual-node architecture, specifically designed for efficient proof generation and verification. The two primary node types are provers and validators. On a high level, users broadcast workloads, validators process transactions, provers complete proving workloads, and validators verify proofs and order them into blocks.

The unique feature of Gevulot is that it allows for centralized equivalent proving performance, while providing the high liveness and availability guarantees common in decentralized networks. It does this by allowing users to specify their desired amount of redundancy and leveraging the efficient verifiability of zero-knowledge proofs. Gevulot also leverages maximum parallelization of computation at both the node and network levels. This parallel processing capability significantly boosts the network's efficiency and scalability. It allows nodes to run multiple provers and verifiers simultaneously, so the network as a whole can handle various provers working on distinct proofs.

Proofs are generated through a user-configurable subset of prover nodes, which are then verified by a predefined subset of validators. The network's design promises several advantages, including liveness guarantees, higher censorship resistance, lower and predictable fees, and efficient workload distribution among multiple provers.

Note: In the future, we anticipate the addition of new node types, such as full nodes that do not validate transactions but only verify proofs and re-execute replicated state transitions.

