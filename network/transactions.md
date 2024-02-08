# Transactions

Transaction types in Gevulot:

| Type     | Description                                   |
| -------- | --------------------------------------------- |
| Transfer | Transfer funds from one address to another    |
| Stake    | Stake funds to a prover or validator node     |
| Unstake  | Unstake funds from a prover or validator node |
| Deploy   | Deploy a prover and/or verifier program       |
| Run      | Run a proving workload                        |
| Cancel   | Cancel a proving workload                     |

**Deploying a Program**

Deployments in Gevulot are immutable and permanent by default. Deployment transactions must contain a pointer to the program binary (URL) and a hash commitment so that the downloaded program can be verified. Once a program is deployed, it becomes available in the next epoch.

**Running a Program**

The full lifecycle of running a prover program:

1. `Run` transaction is sent to the network specifying the information necessary to execute a proving workload as a Workflow.
2. The receiving node puts the transaction into the mempool for prover allocation and inclusion in the next block.
3. Prover nodes complete the workload and put the proof back into the mempool for verification.
4. Validator nodes verify the proof and vote on its correctness.
5. Once 2/3 of validator nodes have verified, the leader includes the proof in the next block.

A user can also cancel a workload, by submitting a `cancel` transaction. The cancellation will be heeded by the provers only when the transaction is finalized and propagated across the network, at which point they will cease running the program. When a proving workload is canceled, the user will pay for the cycles used until the cancellation is finalized.\
\
**Workflows**

Workflows represent the execution steps of a `run` transaction. Each execution step has a reference to a `Program`, its arguments and input data, which may be a downloadable file or the output from an earlier step. Each `Program` can run only once during the workflow.

Gevulot currently only supports workflows that contain steps for proof generation and its verification. Each of these steps results in a new transaction that references an earlier step. The resulting transactions can be represented as a transaction tree. Due to protocol requirements, there are also auxiliary transactions in the transaction tree to support secure execution of the workflow.

Example: Proof generation & verification workflow.

Transaction #1:

&#x20; \- Run:

&#x20;   \- Workflow:

&#x20;     \- Step #1: Generate proof with program 0xf0a3e81d

&#x20;       \- Program: 0xf0a3e81d

&#x20;       \- Arguments: \["--proof", "-o", "/workspace/proof.out", "-i", "input.dat"]

&#x20;       \- Program input:

&#x20;         \- Input file: URL: [https://example.com/foobar/proof/247791284/input.dat](https://example.com/foobar/proof/247791284/input.dat)

&#x20;     \- Step #2: Verify proof with program 0x30b1a8fc

&#x20;       \- Program: 0x30b1a8fc

&#x20;       \- Arguments: \["--verify", "-o", "/workspace/verification.out", "-i", "proof.out"]

&#x20;       \- Program input:

&#x20;         \- Output file: File "proof.out" from program 0xf0a3e81d

Transaction #2:

&#x20; \- Proof:

&#x20;   \- Parent: tx #1

&#x20;   \- Proof: "\<encrypted proof data>"

Transaction #3:

&#x20; \- ProofKey:

&#x20;   \- Parent: tx #2

&#x20;   \- Key: "\<decryption key for proof data>"

Transaction #4:

&#x20; \- Verification:

&#x20;   \- Parent: tx #2

&#x20;   \- Verification: "\<data from proof verification>"

\


\
