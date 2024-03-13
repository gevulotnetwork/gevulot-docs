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

The program files must be available from an HTTP URL so easiest way for this is to use e.g. Amazon S3 or similar for serving files.

### Deploy the prover & verifier programs as one deployment

One deployment always consists of prover & verifier. The individual programs can be reused in different deployments, but a single deployment must always contain both, a prover and a verifier, in order to guarantee a working system for the users.

To deploy programs, use a short but descriptive name for deployment so that the users can easily find them.

`gevulot-cli --jsonurl "http://api.devnet.gevulot.com:9944" --keyfile my-local-key.pki \`\
`deploy \`\
`--name "Simple Gevulot test prover & verifier" \`\
`--prover 4cf53954eac243e23e53fa135aed23ad6f5729fbda663ec3bd14a015bbfe15d8 \`\
`--proverimgurl 'https://storage.googleapis.com/gevulot-devnet-deployments/test-1/prover' \`\
`--verifier 4149857d28f3a376c18c0f4f5b0529dc3be6df6229123a2efd49f7e31f495a98 \`\
`--verifierimgurl 'https://storage.googleapis.com/gevulot-devnet-deployments/test-1/verifier'`
