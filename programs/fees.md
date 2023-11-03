# Fees

Gevulot incorporates two types of fees. One is a standard small fee per transaction, the other a fixed fee per "cycle". A cycle in Gevulot is equal to one block and functions as an objective measure of a programs running time. In running a Gevulot program, the user decides how many cycles they want the prover program to run for and how many provers should do so simultaneously. The maximum fee for the user then is:

```
Prover Amount * Cycle Amount
```

If the user does not pay for enough cycles, the program will not complete and the nodes will return a fail. If the user pays for excessive cycles, the nodes will return the output as soon as the program completes and the user will only pay for the cycles it took for the fastest prover to complete the proof.

Note: CPU and GPU cycles will likely be priced differently.
