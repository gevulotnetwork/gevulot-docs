# Unikernel

Unikernels are very lightweight operating systems designed to run only a single process. Due to their simplicity, unikernels offer a compelling mix of performance and security, allowing Gevulot programs to match centralised prover implementations in speed, while ensuring effective sandboxing of the software.

Gevulot uses the [Nanos](https://nanos.org/) unikernel running in a [KVM](https://www.linux-kvm.org/page/Main\_Page) hypervisor, which provides the following features:

1. Multi-threading
2. GPU support
3. Language support
4. Efficient orchestration
5. Fast boot times

We provide instructions for how to run arbitrary provers in Nanos, along with examples for Ed25519, Marlin, Groth16, Filecoin SNARK & Starknet STARKs [here](https://github.com/gevulotnetwork/gevulot).

\
Note: Currently Nanos officially supports Nvidia 3090 and Nvidia 4090 in on-prem setups.
