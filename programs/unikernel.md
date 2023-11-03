# Unikernel

Unikernels are very lightweight operating systems designed to run only a single process. Due to their simplicity, unikernels offer a compelling mix of performance and security, allowing Gevulot programs to match centralised prover implementations in speed, while ensuring effective sandboxing of the software.

Gevulot uses the [Nanos](https://nanos.org/) unikernel running in a [KVM](https://www.linux-kvm.org/page/Main\_Page) hypervisor, which provides the following features:

1. Multi-threading
2. GPU support
3. Language support
4. Efficient orchestration
5. Fast boot times

\
Note: Currently Nanos officially supports Nvidia 3090 and Nvidia 4090 in on-prem setups.
