# Economics

## **Fees**&#x20;

#### **Transaction fee**

A transaction fee is applied to all types of transactions on ZkCloud. It is a small, standard fee that mainly serves the purpose of spam protection.

#### **Proving workload fee**

The proving workload fee consists of two components:&#x20;

* A small transaction fee.
* A compute fee.&#x20;

To submit a proving workload transaction to a specific prover program, the user needs to specify:

* Resource requirements for the prover workload.
* Maximum compute time.

The fee is calculated based on the above parameters. The amount is locked in the user’s balance until the workload is completed.

**Fee Calculation:**&#x20;

```
Tx Fee + (Compute Fee * Compute Time * Resource Requirement Multiplier)
```

Note: If the user-specified max compute time is too short, the program will not complete, the nodes will return a fail, and the fees will be burned.

#### **Customizable redundancy**

Users can customize and increase redundancy by creating multiple workload requests for the same inputs. In case of added redundancy (multiple single-prover workloads) each prover is rewarded in proportion to their performance.

#### **Verification fee**

No additional fee is applicable for the verification of proofs generated in ZkCloud.

## Rewards

#### **Validator rewards**

Validators are rewarded via a traditional block reward and a small transaction fee paid by the users for all types of transactions.

#### **Prover rewards**

After a workload is assigned to a prover, the network implements a fee structure that dynamically adjusts over time. As the prover works towards generating the required proof, the network deducts a small portion of the total fee, proportional to the elapsed time. This deduction is then returned to the user, effectively acting as a rebate on the initial fee.

After the proof is completed, the prover receives the remaining portion of the workload fee, calculated as follows:

```
Prover Reward = Workload Fee + Workload Subsidy
```

This approach incentivizes the provers to work as efficiently and quickly as possible to generate proofs. It applies consistently across all provers, whether they are part of the global or custom sets.

In the global prover set, the provers that complete the workload within max compute time and submit a valid proof earn an additional network reward. This reward is a fixed percentage of the user fee paid to the prover, minus the fee discount. The network ensures that both the user fees and network rewards are fairly distributed among provers, based on the speed of their proof generation.

#### **Verification rewards**

Provers are also incentivized to verify proofs (both valid or invalid) across the network. They receive a small reward for all proofs on which all of the verifying provers agree.



