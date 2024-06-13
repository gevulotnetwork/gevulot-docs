# Taiko Prover

This guide will walk you through the steps necessary to run the Taiko prover on Gevulot. You can watch the demo video for a visual demonstration.

{% embed url="https://www.youtube.com/watch?v=rxAm-jyvRhU" fullWidth="false" %}

### The Witness

We will first need a witness for the proof. The steps here:

1. download a random witness
2. rename it to a unique name
3. upload to http server

#### 1.1 Download a random witness

There are 58 Taiko witnesses of various sizes located here, numbered 46800-46857:

```
https://gevulot-test.eu-central-1.linodeobjects.com/witness-46800.json
```

Choose one at random and download it.

If you'd like a genuinely random number, [this link](https://www.random.org/integers/?num=1\&min=46800\&max=46857\&col=5\&base=10\&format=html\&rnd=new) will generate a number in the given range. Just plug the value into the url above and download the file.

#### 1.2 Rename the file

Next, rename the file to something unique. It is important that the entire proof request be a unique string. This may be done via unique input files, or accomplished with a nonce (not used here).

For this example, we have downloaded `witness-46807.json` and renamed it here:

```
~/Downloads/mynewwitness-46807.json
```

#### 1.3 Calculate the hash

For this step, you will need to have built `gevulot-cli` and have it in your path. Call it with the `calculate-hash` action, and pass in the witness file.

```
gevulot-cli calculate-hash --file  ~/Downloads/mynewwitness-46807.json
The hash of the file is: 77263b62caae635536c7fefe99e249c55dec5625fecdb550cf83c26108e3f03a
```

#### 1.4 Upload to a public http server

Lastly, you will need a public url for for the file. An S3 bucket would be one option. Just make sure the file has public read permissions.

### Execute the proof

#### 2.1 The parameters structure for the proof

Use the json structure below as a template for creating the parameter inputs. You will observe:

* three arguments are passed in as key/value pairs
* two folders are used
  * the `/gevulot` path is used for static files embedded in the prover image. In this case, there is one, a 512MB proof parameters file, with degree k = 22.
  * the `/workspace` path is used for dynamic file instances
* one input file is passed in, namely our witness. Edit the url and hash values, the latter associated with the `local_path` property.
* the output proof from the prover is referenced as an input into the verifier.

In sum, you will have to edit witness file data in four places:

* the argument list, for the `-w` parameter
* the `vm-path` property
* the `file_url` property
* the `local_path` property, which holds the file hash string

```
[
    {
        "program": "0a9bc2d6c1578f23ebf4823f68ac8f6becfc95b893d59b11810b4aa4407a9fce",
        "cmd_args": [
            {
                "name": "-k",
                "value": "/gevulot/kzg_bn254_22.srs"
            },
            {
                "name": "-p",
                "value": "/workspace/proof.json"
            },
            {
                "name": "-w",
                "value": "/workspace/mynewwitness-46807.json"
            }
        ],
        "inputs": [
            {
                "Input": {
                    "local_path": "77263b62caae635536c7fefe99e249c55dec5625fecdb550cf83c26108e3f03a",
                    "vm_path": "/workspace/mynewwitness-46807.json",
                    "file_url": "https://gevulot-test.eu-central-1.linodeobjects.com/mynewwitness-46807.json"
                }
            }
        ]
    },
    {
        "program": "60d747ff7e5282da8223ebb4f756059fc2fe30cd14479eebca945bb5ce3401ff",
        "cmd_args": [
            {
                "name": "-p",
                "value": "/workspace/proof.json"
            }
        ],
        "inputs": [
            {
                "Output": {
                    "source_program": "0a9bc2d6c1578f23ebf4823f68ac8f6becfc95b893d59b11810b4aa4407a9fce",
                    "file_name": "/workspace/proof.json"
                }
            }
        ]
    }
]
```

Note: the program hashes are static, corresponding to the currently deployed instances of the Taiko prover and verifier.

```
Taiko Prover hash: 0a9bc2d6c1578f23ebf4823f68ac8f6becfc95b893d59b11810b4aa4407a9fce
Taiko Verifier hash: 60d747ff7e5282da8223ebb4f756059fc2fe30cd14479eebca945bb5ce3401ff
```

#### 3.2 Execute the proof

We then embed the json struct (with whitespace removed) in a command line call to `gevulot-cli`, using the `exec` action.

```
gevulot-cli --jsonurl http://api.devnet.gevulot.com:9944  exec --tasks '[{"program":"0a9bc2d6c1578f23ebf4823f68ac8f6becfc95b893d59b11810b4aa4407a9fce","cmd_args":[{"name":"-k","value":"/gevulot/kzg_bn254_22.srs"},{"name":"-p","value":"/workspace/proof.json"},{"name":"-w","value":"/workspace/witness-417428.json"}],"inputs":[{"Input":{"local_path":"90bd4068d10eec71080d95bf3df85622936172342b614c1c413c24495461f750","vm_path":"/workspace/witness-417428.json","file_url":"https://storage.googleapis.com/gevulot-devnet-deployments/test-3/witness-417428.json"}}]},{"program":"60d747ff7e5282da8223ebb4f756059fc2fe30cd14479eebca945bb5ce3401ff","cmd_args":[{"name":"-p","value":"/workspace/proof.json"}],"inputs":[{"Output":{"source_program":"0a9bc2d6c1578f23ebf4823f68ac8f6becfc95b893d59b11810b4aa4407a9fce","file_name":"/workspace/proof.json"}}]}]'
```

You will get back a transaction hash, e.g:

```
Programs send to execution correctly. Tx hash:7bdd606c3ba5c5c34e32a598dba8ecd8b0964918a29508fc33e286fce8e5e092
```

### Verify the results

Use the transaction hash to get the verifier output.

#### 3.1 View the transaction

First, you can query the transaction itself, by calling `gevulot-cli` with the `get-tx` action and the hash. It will return the parameters passed into the `exec` call, along with some additional metadata.

```
gevulot-cli --jsonurl http://api.devnet.gevulot.com:9944 get-tx 7bdd606c3ba5c5c34e32a598dba8ecd8b0964918a29508fc33e286fce8e5e092
```

#### 3.2 View the transaction tree

Now, print out the transaction tree using the `print-tx-tree` action and the txhash you got back from the `exec` call. The tree will eventually contain leaves, from which you can read the verifier results.

```
gevulot-cli --jsonurl http://api.devnet.gevulot.com:9944 print-tx-tree 7bdd606c3ba5c5c34e32a598dba8ecd8b0964918a29508fc33e286fce8e5e092 

```

In the case of The Taiko prover, it can take 6-7 minutes for a proof to complete. Until then, no tree will be found.

```
An error while fetching transaction tree: RPC response error:not found: no root tx found for e462679e6d3345d76005ee42301e01065db688e080c10adb2648ad645c77d1a5
```

When the proof completes, you will see a tree like this

```
Root: e462679e6d3345d76005ee42301e01065db688e080c10adb2648ad645c77d1a5
        Node: 53ee422a8122b7ffd721ad59464ad114215dbcce061aeb91b293ca462df64cc3
                Leaf: 267a086104427813a2474bd9aef65a721719f2d7dec24a673e3bdce6982490ad
                Leaf: dd6ec406e8ea156e8b9a16dfee38e1ba39e15c22ffaad3586dd746510a8c186f
                Leaf: 8f655cfc476dc57182dbc0fd21e7d51bc9d8720bdcb4326dde2ae7ebe1bde758
```

#### 3.3 Examine a leaf

You may see several `Leaf` entries. Choose the first, and do a `get-tx` on it to print out the verifier results.

```
gevulot-cli --jsonurl http://api.devnet.gevulot.com:9944 get-tx a57fb861befe658b106aefa7d5ca951957f1130eed7343d5535c26ac9bab65c1

{"author":"0419f335c2b0a5f8b1bd4f5b079bd0756b21903f06e35bbea9f03614f598d8f356887af6df60a3d90981284cf4fafd2f288c5663238d627eaa21c91a8ee4088963","hash":"a57fb861befe658b106aefa7d5ca951957f1130eed7343d5535c26ac9bab65c1","payload":{"Verification":{"parent":"5f59ae0d0e78c09f900ca1a0c6d1f81aec4cc636cc5b1c9f333dacad99b426ea","verifier":"60d747ff7e5282da8223ebb4f756059fc2fe30cd14479eebca945bb5ce3401ff","verification":"eyJpc19zdWNjZXNzIjp0cnVlLCJtZXNzYWdlIjoiVGFpa28gdmVyaWZpZXIgcmVzdWx0OiBzdWNjZXNzIiwicHJvb2ZfZmlsZSI6Ii93b3Jrc3BhY2UvcHJvb2YuanNvbiIsInRpbWVzdGFtcCI6MTcxODI2MjI4ODgyOX0=","files":[]}},"nonce":0,"signature":"1b3ad666e26195c2c8886b4553cc66875f2562248b81f6223a285f78b836c9fa02b371057cc3b0d4c047185e0baee5204a6c8e876c38fa60ecfbe59beb84a2c7"}
```

#### 3.4 Parse the verification result

Under `payload.Verification.verification`, we see a string encoded as base64. Use `jq` plus `base64` to decode that at the command line. Now we can read the json text generated by the verifier.

```
gevulot-cli --jsonurl http://api.devnet.gevulot.com:9944 get-tx a57fb861befe658b106aefa7d5ca951957f1130eed7343d5535c26ac9bab65c1  | jq -r '.payload.Verification.verification' | base64 -d

{"is_success":true,"message":"Taiko verifier result: success","proof_file":"/workspace/proof.json","timestamp":1718262288829}
```

And you're done!\
