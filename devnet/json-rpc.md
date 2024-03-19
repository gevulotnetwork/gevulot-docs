# JSON-RPC

## Introduction

The Gevulot devnet provides a minimalist JSON-RPC API for end user interfacing. It allows for the creation and observation of transactions.

In order to access the Gevulot devnet JSON-RPC API, you must [register a key](https://docs.gevulot.com/gevulot-docs/devnet/key-registration).

Address to devnet API is `http://api.devnet.gevulot.com:9944`

_Note: Compute time for proving workloads is currently restricted to max 30 mins._

## Operations

### sendTransaction

`sendTransaction` operations allow users to submit a transaction for execution. Currently the devnet supports `Deploy` and `Run` transaction types.



### getTransaction

`getTransaction` operations allow users to fetch any transaction for a given hash. The return value contains all details of a transaction, excluding related file data.



### getTransactionTree

`getTransactionTree` returns a full transaction tree for a given hash that is part of an execution of a `Run` transaction.

`Run` transactions contain a workflow that is executed to generate a proof and then verify it. Each step results in a new transaction referring to the parent transaction. The result of a whole execution is a tree of transactions.



## Rust Client

The Gevulot node crate provides types for easily working with transactions and an RPC client to communicate with the servers.

### Construct client

Client construction is simple:

```rust
use gevulot_node::rpc_client::RpcClient;

// ...

let url = "http://api.devnet.gevulot.com:9944"
let client = RpcClient::new(url);
```

### Build a transaction

#### Deployment

This section assumes you have built program images for prover & verifier as described in the [Development](development.md) section. In order to deploy them, files must be available from an HTTP server and a BLAKE3 checksum must be calculated for them.

Given those details, a deploy transaction can be built as follows:

<pre class="language-rust"><code class="lang-rust"><strong>use gevulot_node::{
</strong>    types::{
        transaction::{Payload, ProgramMetadata},
        Hash, Transaction,
    },
};
use libsecp256k1::SecretKey;
use std::path::Path;
<strong>
</strong><strong>fn compute_file_hash(path: &#x26;Path) -> Hash {
</strong>        // First compute BLAKE3 hash for the program image file.
        let mut hasher = blake3::Hasher::new();
        let fd = std::fs::File::open(&#x26;img_file).expect("open");
        hasher.update_reader(fd).expect("checksum");
        hasher.finalize()
}


fn build_program_metadata(name: &#x26;str, img_path: &#x26;Path) -> ProgramMetadata {
        let img_file_name = img_path.file_name().unwrap().to_str().unwrap().to_string();
        
        // Then construct the program metadata struct.
        let mut prover_program_metadata = ProgramMetadata {
                name: "example-prover".to_string(),
                hash: Hash::default(),
                image_file_name,
                image_file_url: format!("http://my.example.com/images/{}", img_file_name),
                image_file_checksum: compute_file_hash(img_path),
        };

        // Compute the program hash.
        program_metadata.update_hash();
        
        program_metadata
}

fn construct_deployment_tx(private_key: &#x26;SecretKey, name: &#x26;str, prover_img_file: &#x26;Path, verifier_img_file: &#x26;Path) -> Transaction {
        // Construct the transaction with deployment payload.
        Transaction::new(Payload::Deployment{
            name: name.to_string(),
            prover: build_program_metadata(format!("{}-prover", name), prover_img_file),
            verifier: build_program_metadata(format!("{}-verifier", name), verifier_img_file),
        }, &#x26;private_key)
}
</code></pre>

#### Run transaction

When a prover and a verifier have been deployed, they can be used to generate and verify proofs.

Note that single workflow step maximum runtime is restricted to 30 minutes.&#x20;

`Run` transaction can be created in the following way:

```rust
use gevulot_node::{
    types::{
        transaction::{Payload, ProgramData, Workflow, WorkflowStep},
        Transaction,
    },
};

// Assume prover_hash & verifier_hash are variables to deployed prover and verifier.
let proving_step = WorkflowStep {
    program: *prover_hash,
    args: vec!["--prover-arg".to_string(), "arg value".to_string()],
    inputs: vec![ProgramData::Input {
        file_name: "proof_inputs.dat".to_string(),
        file_url: "http://rollup.example.com/proof/inputs/3212".to_string(),
        file_checksum: "886ace0f0ea0ddd53fca7f733586226b6ddbee7bcb769be71633d06e61ba36bc".to_string(),
    }],
};

let verifying_step = WorkflowStep {
    program: *verifier_hash,
    args: vec!["--nonce".to_string(), nonce.to_string()],
    inputs: vec![ProgramData::Output {
        source_program: *prover_hash,
        file_name: "/workspace/proof.dat".to_string(),
    }],
};

let tx = Transaction::new(
    Payload::Run {
        workflow: Workflow {
            steps: vec![proving_step, verifying_step],
        },
    },
    &private_key,
);
```

### Send transaction

Sending transactions is also straightforward:

```rust
client
    .send_transaction(&tx)
    .await
    .expect("send_transaction");
```

### Get transaction

Fetching known transaction:

```rust
let tx_hash = Hash::from("706c739440cd10ff7f8b20c4da722437ef42873607d66c320e1cef4d956f3512");
let tx = client
    .get_transaction(&tx_hash)
    .await
    .expect("get_transaction");

if tx.is_some() {
    dbg!(tx);
}
```

### Get transaction tree

Fetch a transaction tree and print each transaction:

```rust
fn print_tx_tree(tree: &TransactionTree, indentation: u16) {
    match tree {
        TransactionTree::Root{ children, hash } => {
            println!("Root: {hash}");
            children.iter().for_each(|x| print_tx_tree(&x, indentation+1));
        },
        TransactionTree::Node{ children, hash } => {
            println!("{}Node: {hash}", (0..indentation).map(|_| "\t").collect::<String>());
            children.iter().for_each(|x| print_tx_tree(&x, indentation+1));
        },
        TransactionTree::Leaf{ hash } => {
            println!("{}Leaf: {hash}", (0..indentation).map(|_| "\t").collect::<String>());
        },
    }
}

fn fetch_and_print_tx_tree() {
    let tx_hash = Hash::from("706c739440cd10ff7f8b20c4da722437ef42873607d66c320e1cef4d956f3512");
    let tx_tree = client
        .get_tx_tree(&tx_hash)
        .await
        .expect("get_tx_tree");
​
    print_tx_tree(&tx_tree, 0);
}
```

## Full E2E example

The Gevulot node repo contains test code for full e2e test of deploying a prover & verifier and then executing a `Run` transaction:

{% embed url="https://github.com/gevulotnetwork/gevulot/blob/main/crates/tests/e2e-tests/src/main.rs" %}
gevulot/crates/tests/e2e-tests/src/main.rs
{% endembed %}

The E2E example also contains embedded HTTP server for serving the program image files, when testing in local development environment.
