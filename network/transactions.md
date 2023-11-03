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

Deployments in Gevulot are immutable and permanent by default. Deployment transactions need to contain a pointer to the program binary (URL) and a hash commitment so the downloaded program can be verified. When a program is deployed, it becomes usable at the next epoch.

**Running a Program**

The full lifecycle of running a prover program:

1. `Run` transaction is sent to network, which specifies program ID, input pointer + hash commitment, amount of provers and max cycles per prover.
2. Receiving node puts the transaction into the mempool for prover allocation and for inclusion in the next block.
3. Prover nodes complete workload and put the proof back into the mempool for verification
4. Validator nodes verify the proof and vote on its correctness
5. Once 2/3 of validator nodes have verified, the leader includes the proof in the next block

A user can also cancel a workload, by submitting a `cancel` transaction. The cancellation will only be heeded by the provers once the transaction has finality and has propagated across the network at which point they will cease running the program. Once a proving workload is cancelled the user will pay for the cycles used up until the cancellation transaction reached finality.

\
