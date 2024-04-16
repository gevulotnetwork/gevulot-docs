---
description: Deployment guide for Gevulot programs.
---

# Prover/Verifier Deployment

### Prerequisites

1. Ensure you have `gevulot-cli` installed.\
   `cargo install --git https://github.com/gevulotnetwork/gevulot.git gevulot-cli`\

2. Ensure your local key has been [registered](https://docs.gevulot.com/gevulot-docs/devnet/key-registration).

### Calculate hash for your prover & verifier programs

When deploying provers/verifiers to Gevulot, they need to be served from an HTTP URL. In order to verify that the downloaded file is the correct one and has stayed intact, it must be accompanied with a checksum.

To compute the checksum of a file for Gevulot you can use the CLI tool:

`gevulot-cli calculate-hash --file <filepath>`

### Upload the programs to HTTP service

The program files must be available from an HTTP URL so the easiest way for this is to use e.g. Amazon S3 or similar for serving files.

### Deploy the prover & verifier programs as one deployment

One deployment always consists of prover & verifier. The individual programs can be reused in different deployments, but a single deployment must always contain both, a prover and a verifier, in order to guarantee a working system for the users.

To deploy programs, use a short but descriptive name for deployment so that the users can easily find them.

Individual programs should be named / tagged as well, as for example in prover's case that name is used in the devnet explorer to provide easier recognition for individual proofs.

```
gevulot-cli --jsonurl "http://api.devnet.gevulot.com:9944" --keyfile my-local-key.pki \
deploy \
--name "Simple Gevulot test prover & verifier" \
--prover 4cf53954eac243e23e53fa135aed23ad6f5729fbda663ec3bd14a015bbfe15d8 \
--provername '#testprover' \
--proverimgurl 'https://storage.googleapis.com/gevulot-devnet-deployments/test-1/prover' \
--verifier 4149857d28f3a376c18c0f4f5b0529dc3be6df6229123a2efd49f7e31f495a98 \
--verifiername '#testverifier' \
--verifierimgurl 'https://storage.googleapis.com/gevulot-devnet-deployments/test-1/verifier'
```

### Program resource requirements

As of writing, by default Gevulot node allocates 2vCPU and 2GB of RAM (0 GPUs) for a program. This can be changed by specifying resource requirements for the program. Each parameter can be specified alone and defaults are used as a base value that is then configured.

The unit for memory requirements is mebibytes.

**NOTE**: While the devnet nodes have certain minimum requirements, it is not recommended to try matching them with the program resource requirements as that can lead to a situation where due to normal system operation, there aren't enough resources available at any given time and therefore the program cannot ever run.

```
gevulot-cli --jsonurl "http://api.devnet.gevulot.com:9944"--keyfile my-local-key.pki \
deploy \
--name "Taiko demo prover & verifier" \
--prover ab8492be32194a090e52b1a85579dab3169f96922d59e5c4f3612be8c0aca3a3 \
--provername '#taiko' \
--proverimgurl 'https://gevulot.eu-central-1.linodeobjects.com/taiko_prover' \
--provercpus 32 \
--provermem 65536 \
--provergpus 0 \
--verifier 11d5d504f85e5e0ff40174a09659035ac96ba04ec1cf26fd7603c2f90c44b2fa \
--verifiername '#taiko' \
--verifierimgurl 'https://gevulot.eu-central-1.linodeobjects.com/taiko_verifier' \
--verifiercpus 4 \
--verifiermem 4096 \
--verifiergpus 0

```
