# Proving Workloads

#### **Overview of running a prover program**

The lifecycle of proving workloads:

1. The user sends a `Run` transaction to the network specifying the information necessary to execute a proving workload.
2. The receiving node adds the transaction to the mempool for inclusion in the next block.
3. Once the transaction has been included in a block by the leader and finalized, it is randomly allocated to a prover node.
4. The prover node completes the workload and sends the proof back to the mempool for verification. (The proof can already be shared with the user if, for instance, verification and settlement are done on Ethereum.)
5. A certain number of randomly selected provers participate in the verification of the proof and vote on its correctness.
6. Once all the prover nodes have verified the proof, the leader includes the proof in the next block.
7. The proof reaches finality on Gevulot.

#### **Submitting a proving workload**

To submit a proving workload transaction to a given prover program, the user needs to specify:

* Resource requirements for the prover workload.
* Maximum compute time.

The network will calculate the fees based on the above parameters. There is a minimum computation time limit based on hardware requirements and proof size. To learn more about proving workload fees, please refer to the [“Economics”](fees.md) section.&#x20;

#### **Block building and transaction inclusion**

At every block, a leader is selected from the active validator set. The leader is responsible for including the transactions and ordering them into blocks. Once the receiving node has added the proving workload transaction to the mempool, the leader includes it in a block. After consensus has been reached among the validators, and the workload transaction reaches finality, the proving workload is allocated to a randomly selected prover.

#### **Proof generation, verification, and finality**

The selected prover runs the prover program with the received inputs until it completes, and returns a proof. If the maximum compute time is not sufficient to complete the workload, the prover returns a fail.

Once the prover has generated the proof for the assigned workload, it is sent to the mempool for verification. A proof is verified by a group of provers, with at least 10 participating in the verification. These provers are selected randomly using a Verifiable Random Function (VRF), which ensures the randomness and fairness of the selection process.

The total number of verifier nodes, which are specific provers tasked with the verification of proofs, is set to be 1 percent of the entire set of provers. This means that as the number of provers increases, the number of verifier nodes also increases proportionally. However, regardless of the total number of provers, there are always at least 10 verifier nodes involved in the process.

To maintain impartiality, the prover who computes the proof is excluded from the selected group of verifiers. Once all verifiers approve the proof, the leader includes it in the next block, and the proof is finalized.&#x20;

The verification threshold constitutes finality from the network's perspective, but anyone can verify the proof for immediate use outside of the Gevulot network.

\
\


\
