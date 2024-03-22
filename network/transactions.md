# Proving Workloads

#### **Overview of running a prover program**

The lifecycle of proving workloads:

1. The user sends a `Run` transaction to the network specifying the information necessary to execute a proving workload.
2. The receiving node adds the transaction to the mempool for inclusion in the next block.
3. Once the transaction has been included in a block by the leader and finalized, it is randomly allocated to a prover node.
4. The prover node completes the workload and sends the proof back to the mempool for verification. (The proof can already be shared with the user if, for instance, verification and settlement are done on Ethereum.)
5. All provers participate in the verification of the proof and vote on its correctness.
6. Once 2/3 of the prover nodes have verified the proof, the leader includes the proof in the next block.
7. The proof reaches finality on Gevulot.

#### **Submitting a proving workload**

To submit a proving workload transaction to a given prover program, the user needs to specify:

* Resource requirements for the prover workload.
* Maximum compute time.

The network will calculate the fees based on the above parameters. To learn more about proving workload fees, please refer to the [“Economics”](fees.md) section.

#### **Block building and transaction inclusion**

At every block, a leader is selected from the active validator set. The leader is responsible for including the transactions and ordering them into blocks. Once the receiving node has added the proving workload transaction to the mempool, the leader includes it in a block. After consensus has been reached among the validators, and the workload transaction reaches finality, the proving workload is allocated to a randomly selected prover.

#### **Proof generation, verification, and finality**

The selected prover runs the prover program with the received inputs until it completes, and returns a proof. If the maximum compute time is not sufficient to complete the workload, the prover returns a fail.

Once the prover has generated the proof for the assigned workload, it is sent to the mempool for verification. All provers participate in verifying the proof and vote on its correctness. Once at least 2/3 of the prover nodes have verified the proof, the leader includes it in the next block, and the proof is finalized.&#x20;

The verification threshold constitutes finality from the network's perspective, but anyone can verify the proof for immediate use outside of the Gevulot network.

\
\


\
