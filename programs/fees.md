# Fees

**Proving workload fees**

Gevulot incorporates two types of fees. One is a standard small fee per transaction, the other a fixed fee per "cycle". A cycle in Gevulot is equal to one block and functions as an objective measure of a program's running time. In running a Gevulot program, the user decides how many cycles they want the prover program to run for and how many provers should do so simultaneously. The maximum fee for the user then is:

```
Prover Amount * Cycle Amount * Fee Per Cycle
```

If the user does not pay for enough cycles, the program will not complete and the nodes will return a fail. If the user pays for excessive cycles, the nodes will return the output as soon as the program completes and the user will only pay for the cycles it took for the fastest prover to complete the proof.

When a user broadcasts a workload transaction, the maximum fee is locked until the first proof is generated or cycles are exhausted, after which the realized fees are deducted from the locked amount.

Note: CPU and GPU cycles will be priced differently.

**Verification fee**

No verification fee applies for proofs generated through prover workloads in the Gevulot Network. However, if a user deploys a standalone verifier program to verify proofs generated outside the protocol, a small flat fee will be incurred.
