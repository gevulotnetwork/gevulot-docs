# FAQ

**Why does Gevulot need to be a layer one?**

Being a layer one makes it possible to optimize the entire stack, resulting in dramatically better performance and cost efficiency. We view Gevulot as primarily competing with centralized provers and in that context traditional blockchain performance and cost structures are unacceptable.&#x20;

If Gevulot was implemented as an application on some other layer one, it would inherit the cost structure and performance characteristics of the underlying chain, which in turn would make it impossible to compete with centralized provers.\
\
Gevulot also cannot practically be implemented as a layer two, because it does not have a simple abstracted instruction set. In order for Gevulot to support things like multi-threading and GPU, which are a necessity for performant proving, the instruction set becomes practically unverifiable. \
\
Gevulot could in theory be implemented as some kind of optimistic layer one, where verification was not re-executed by default and, for example, was only done when a challenge was submitted. However, these types of structures necessarily have significantly slower finality and create additional complexity for the system. \
\
**How does Gevulot ensure that provers act honestly?**

Given that proving is not re-executed like state transitions in other blockchains, the network can't tell the difference between an honest prover trying and failing to compute a proof and a prover dishonestly saying they tried, while having actually not done so.\
\
However, it is possible for the network to ensure that provers both a) cannot make money by acting dishonestly and b) there is a cost to acting dishonestly. The specific implementation that achieves high prover honesty, while minimizing the associated cost overhead for the network is an optimization problem where the parameters are hard to predict in advance. Given this, we are starting with a relatively naive structure and iterating on the mechanisms as network dynamics begin to emerge in devnet, testnet and beyond.\
\
Currently, we are planning on utilizing the following mechanisms at the outset:

1. Provers can only make money by completing proofs. If a proof fails to be generated for a given workload for whatever reason, the provers do not get a network reward, nor do they receive cycle fees (these are burned).&#x20;
2. Provers are randomly allocated proof-of-work workloads which must be completed to remain in the active prover set. Completing these workloads does not yield any reward and so they function as a way to increase the cost of being dishonest.
3. Non-responsive provers get slashed. If a prover fails to respond with a declined, completed or failed message, their stake gets slashed.

Once network dynamics begin to emerge, we will explore further strategies such as:

1. Game-theoretic punishments. E.g. provers that fail to complete workloads when others succeed are dropped from the active prover set.
2. Time-outs. E.g. provers that decline a workload are ineligible for a workload for some amount of rounds.

Ultimately the objective is to find the optimal point where provers act honestly 99.99% of the time with an elegant fallback for the 0.01% case, but the "management" overhead for achieving this is as low as possible for the network.

**How do users know how many cycles to pay for?**

While it is impossible to perfectly predict how many cycles a proving workload with some given inputs will require, there are many things that can be done to make this easier for the user. It is worth noting that users only pay for the cycles expended until the first proof (or only proof if no redundancy is required) and the race conditions created for the provers incentivize them to complete proofs as quickly as possible. This dynamic allows users to set very large max cycles (similar to gas limit in Ethereum), without needing to worry about overpaying. \
\
This however does not solve the question of what max cycles to set. We are exploring a variety of strategies for this, such as:

1. Allowing the deployer to include a max cycles recommendation.
2. Tracking cycles expended for previous completed proofs using the same program and exposing these to the user.
3. Out-of-protocol runs with varied inputs to find a min-max cycle range for most runs of a given program.

**What happens if a prover or verifier program is faulty?**

Gevulot programs are arbitrary user-generated code. They are not explicitly constrained in any way beyond the constraints of the Nanos unikernel itself. Programs, however, must include some basic things to function properly such as:

1. Utilizing the gRPC service over the VSOCK for receiving inputs. We offer a helper library in Rust that takes care of this.
2. Producing a proof that can be verified by the associated verifier program.

To be clear, a user can deploy a program which does not do these things. It just won't work. \
\
A faulty prover can result in proofs not being produced when desired or produced proofs being unverifiable. A faulty verifier, on the other hand, can mean that proofs are not able to be verified by some or all validators. In this instance, provers can also not be rewarded and so are incentivized to decline workloads for that program. We are considering implementing an out-of-protocol flag in the node, which would be triggered if a valid proof was not verified after which the node would automatically decline workloads for that program.\
\
**Do you have a question you'd like to see answered here? Message us on** [**X**](https://twitter.com/gevulot\_network)**.**
