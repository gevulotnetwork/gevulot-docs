# Starknet Stone Prover

This guide will walk you through the steps necessary to run an example workload on a Starkware Stone prover deployed on Gevulot.

### Generate the Inputs

We begin by generating the required input files. There are two main steps to this:

* compile a Cairo program
* run the program to generate the inputs.

We will be going through the same steps described in the Starkware documention [here](https://github.com/starkware-libs/stone-prover?tab=readme-ov-file#creating-and-verifying-a-proof-of-a-cairozero-program).

#### 1.1 Install Cairo

We will need two executables:

* `cairo-compile`
* `cairo-run`

Use this command to install version 0.12.0

```
pip install cairo-lang==0.12.0
```

#### 1.2 Compile a Cairo program

We will compile the fibonacci test program located in the `/e2e_test` folder

Go into that folder and compile the Cairo program there.

```
cd e2e_test
cairo-compile fibonacci.cairo --output fibonacci_compiled.json --proof_mode
```

A new file has been created: `fibonacci_compiled.json`.

#### 1.3 Run the program

Run the program in order to generate the required input files.

```
cairo-run \
    --program=fibonacci_compiled.json \
    --layout=small \
    --program_input=fibonacci_input.json \
    --air_public_input=fibonacci_public_input.json \
    --air_private_input=fibonacci_private_input.json \
    --trace_file=fibonacci_trace.json \
    --memory_file=fibonacci_memory.json \
    --print_output \
    --proof_mode
```

There are four new files:

```
fibonacci_input.json
fibonacci_public_input.json
fibonacci_memory.json
fibonacci_trace.json
```

We now have all of the inputs required for the Stone prover.

### Processing the inputs

#### 2.1 Upload the input files

All four input files must have a public url, so that Gevulot nodes may access them in a proof. One option is to upload the files to an S3 bucket.

It is important to make sure that the files have public read permission.

#### 2.2 Calculate the hashes

In addition to a url, each input file requires a hash value, which is a 256-bit value formatted as a 64-character hex string. That value may be computed with `gevulot-cli`, using the calculate-hash action and passing in the file path. Depending on where your `gevulot-cli` is, you may need to change the path.

```
./gevulot-cli calculate-hash --file  workspace/fibonacci_memory.json
The hash of the file is: 2fa6e451e18f82959c58363b274586964c133c71d1fae5116027c0b5de245f8c

./gevulot-cli calculate-hash --file  workspace/fibonacci_public_input.json
The hash of the file is: 82c352f2c1a5088c71aa87cb6c106c36b2eb56d207ef22d2d2c61a5e49abb521

./gevulot-cli calculate-hash --file  workspace/fibonacci_private_input.json
The hash of the file is: 3a26731cba5db571f740345cb2803687b41b329e4f033ee8bc1a7a26c508fae9

./gevulot-cli calculate-hash --file  workspace/fibonacci_trace.json
The hash of the file is: b345e7ca8ab9bba28fe1438b3cabc8e2a9b6029c68663326be5b59097197903d

```

#### 2.3 The configuration files

Two configuration files are passed in to the `cpu_air_prover`. In our implementation, default `prover_config_file` and `parameter_file` parameters point to files embedded in the prover unikernel. This is their content:

`cpu_air_params.json`:

```
{
    "field": "PrimeField0",
    "stark": {
        "fri": {
            "fri_step_list": [
                0,
                4,
                3
            ],
            "last_layer_degree_bound": 64,
            "n_queries": 18,
            "proof_of_work_bits": 24
        },
        "log_n_cosets": 4
    },
    "use_extension_field": false
}
```

`cpu_air_prover_config.json`:

```
{
    "cached_lde_config": {
        "store_full_lde": false,
        "use_fft_for_eval": false
    },
    "constraint_polynomial_task_size": 256,
    "n_out_of_memory_merkle_layers": 1,
    "table_prover_n_tasks_per_segment": 32
}
```

You may require (or choose) to employ different configuration files. In that case, treat them like normal input files, and change the parameters in the struct passed in to the prover (see section 3.1)

### Execute the proof

#### 3.1 The params structure for the proof

Use the json structure below as a template for creating the parameter inputs. You will observe:

* five arguments are passed in as key/value pairs
* two folders used
  * the `/gevulot` path is used for the default configuration files, embedded in the prover image.
  * the `/workspace` path is used for dynamic file instances
* four input files are passed in, with their respective urls and hash values, the latter associated with the `local_path` property.
* although the trace and memory files are not explicitly passed in as arguments, they must be listed as input files, as both files are referenced from the private inputs.
* the output proof from the prover is referenced as an input into the verifier.

```
[
    {
        "program": "ae713ce2635872d7dfe8607a5ebd129164954276d7cab131977bdc24c02f573d",
        "cmd_args": [
            {
                "name": "--out_file",
                "value": "/workspace/proof.json"
            },
            {
                "name": "--private_input_file",
                "value": "/workspace/fibonacci_private_input.json"
            },
            {
                "name": "--public_input_file",
                "value": "/workspace/fibonacci_public_input.json"
            },
            {
                "name": "--prover_config_file",
                "value": "/gevulot/cpu_air_prover_config.json"
            },
            {
                "name": "--parameter_file",
                "value": "/gevulot/cpu_air_params.json"
            }
        ],
        "inputs": [
            {
                "Input": {
                    "local_path": "2fa6e451e18f82959c58363b274586964c133c71d1fae5116027c0b5de245f8c",
                    "vm_path": "/workspace/fibonacci_memory.json",
                    "file_url": "https://gevulot.eu-central-1.linodeobjects.com/fibonacci_memory.json"
                }
            },
            {
                "Input": {
                    "local_path": "3a26731cba5db571f740345cb2803687b41b329e4f033ee8bc1a7a26c508fae9",
                    "vm_path": "/workspace/fibonacci_private_input.json",
                    "file_url": "https://gevulot.eu-central-1.linodeobjects.com/fibonacci_private_input.json"
                }
            },
            {
                "Input": {
                    "local_path": "82c352f2c1a5088c71aa87cb6c106c36b2eb56d207ef22d2d2c61a5e49abb521",
                    "vm_path": "/workspace/fibonacci_public_input.json",
                    "file_url": "https://gevulot.eu-central-1.linodeobjects.com/fibonacci_public_input.json"
                }
            },
            {
                "Input": {
                    "local_path": "b345e7ca8ab9bba28fe1438b3cabc8e2a9b6029c68663326be5b59097197903d",
                    "vm_path": "/workspace/fibonacci_trace.json",
                    "file_url": "https://gevulot.eu-central-1.linodeobjects.com/fibonacci_trace.json"
                }
            }
        ]
    },
    {
        "program": "140abc53852d49b3d940c1df462d5eec0fb6b55adaae49f8be07bce678378ac1",
        "cmd_args": [
            {
                "name": "--in_file",
                "value": "/workspace/proof.json"
            }
        ],
        "inputs": [
            {
                "Output": {
                    "source_program": "ae713ce2635872d7dfe8607a5ebd129164954276d7cab131977bdc24c02f573d",
                    "file_name": "/workspace/proof.json"
                }
            }
        ]
    }
]
```

Note: the program hashes are static, corresponding to the currently deployed instances of the prover and verifier.

```
Stone Prover hash: ae713ce2635872d7dfe8607a5ebd129164954276d7cab131977bdc24c02f573d
Stone Verifier hash: 140abc53852d49b3d940c1df462d5eec0fb6b55adaae49f8be07bce678378ac1
```

#### 3.2 Execute the proof

We then embed the json struct in a command line call to `gevulot-cli`, using the `exec` action.

```
./gevulot-cli --jsonurl http://api.devnet.gevulot.com:9944  exec --tasks '[{"program":"ae713ce2635872d7dfe8607a5ebd129164954276d7cab131977bdc24c02f573d","cmd_args":[{"name":"--out_file","value":"/workspace/proof.json"},{"name":"--private_input_file","value":"/workspace/fibonacci_private_input.json"},{"name":"--public_input_file","value":"/workspace/fibonacci_public_input.json"},{"name":"--prover_config_file","value":"/gevulot/cpu_air_prover_config.json"},{"name":"--parameter_file","value":"/gevulot/cpu_air_params.json"}],"inputs":[{"Input":{"local_path":"2fa6e451e18f82959c58363b274586964c133c71d1fae5116027c0b5de245f8c","vm_path":"/workspace/fibonacci_memory.json","file_url":"https://gevulot.eu-central-1.linodeobjects.com/fibonacci_memory.json"}},{"Input":{"local_path":"3a26731cba5db571f740345cb2803687b41b329e4f033ee8bc1a7a26c508fae9","vm_path":"/workspace/fibonacci_private_input.json","file_url":"https://gevulot.eu-central-1.linodeobjects.com/fibonacci_private_input.json"}},{"Input":{"local_path":"82c352f2c1a5088c71aa87cb6c106c36b2eb56d207ef22d2d2c61a5e49abb521","vm_path":"/workspace/fibonacci_public_input.json","file_url":"https://gevulot.eu-central-1.linodeobjects.com/fibonacci_public_input.json"}},{"Input":{"local_path":"b345e7ca8ab9bba28fe1438b3cabc8e2a9b6029c68663326be5b59097197903d","vm_path":"/workspace/fibonacci_trace.json","file_url":"https://gevulot.eu-central-1.linodeobjects.com/fibonacci_trace.json"}}]},{"program":"140abc53852d49b3d940c1df462d5eec0fb6b55adaae49f8be07bce678378ac1","cmd_args":[{"name":"--in_file","value":"/workspace/proof.json"}],"inputs":[{"Output":{"source_program":"ae713ce2635872d7dfe8607a5ebd129164954276d7cab131977bdc24c02f573d","file_name":"/workspace/proof.json"}}]}]'
```

You will get back a transaction hash:

```
Programs send to execution correctly. Tx hash:2c03326e4e64825a0bf4ac11dfb8c98ae5d7a2698805a7f31d6db660c987a9de
```

### Verify the results

Use the transaction hash to get the verifier output.

#### 4.1 View the transaction

First, you can query the transaction itself, by calling `gevulot-cli` with the `get-tx` action and the hash. It will return the parameters passed into the `exec` call, along with some additional metadata.

```
./gevulot-cli --jsonurl http://api.devnet.gevulot.com:9944 get-tx 2c03326e4e64825a0bf4ac11dfb8c98ae5d7a2698805a7f31d6db660c987a9de
```

#### 4.2 View the transaction tree

Now, print out the transaction tree using the `print-tx-tree` action. The verification results will be on the `Leafs` there.

```
./gevulot-cli --jsonurl http://api.devnet.gevulot.com:9944 print-tx-tree 2c03326e4e64825a0bf4ac11dfb8c98ae5d7a2698805a7f31d6db660c987a9de 

Root: 2c03326e4e64825a0bf4ac11dfb8c98ae5d7a2698805a7f31d6db660c987a9de
        Node: c495dd245a3f9dedcd884c932eca68494cd87def28204e33f97ddf22cd35f138
                Leaf: f2e76601ee8b7409d7ca0e2affde82cdf67da11f722b773e11d8046f731e96e9
```

#### 4.3 Examine a leaf

You may see several Leaf entries. Choose the first, and do a `get-tx` on it to print out the verifier results.

```
./gevulot-cli --jsonurl http://api.devnet.gevulot.com:9944 get-tx f4f981f594ead6b685cbd46c7c79baa5bdf7f7c4cd4248ec66d9dfd12d08959f

{"author":"0419f335c2b0a5f8b1bd4f5b079bd0756b21903f06e35bbea9f03614f598d8f356887af6df60a3d90981284cf4fafd2f288c5663238d627eaa21c91a8ee4088963","hash":"f4f981f594ead6b685cbd46c7c79baa5bdf7f7c4cd4248ec66d9dfd12d08959f","payload":{"Verification":{"parent":"c495dd245a3f9dedcd884c932eca68494cd87def28204e33f97ddf22cd35f138","verifier":"140abc53852d49b3d940c1df462d5eec0fb6b55adaae49f8be07bce678378ac1","verification":"eyJpc19zdWNjZXNzIjp0cnVlLCJtZXNzYWdlIjoiU3Rhcmt3YXJlIHZlcmlmaWVyIHJlc3VsdDogc3VjY2VzcyIsInByb29mX2ZpbGUiOiIvd29ya3NwYWNlL3Byb29mLmpzb24iLCJ0aW1lc3RhbXAiOjE3MTEzNTA2MDIwNzl9Cg==","files":[]}},"nonce":0,"signature":"1e7f99b1e5c37e3bafb750280e3d82b7ba64ad102fea9946c1ab93866f49dd6e5375007da6a018857ec0e896d01b1113aa7ce601b073a157110a3d6d069e3cff"}
```

#### 4.4 Parse the verification result

Under payload.Verification.verification, we see a string encoded as base64. Use `jq` plus `base64` to decode that at the command line. Now we can read the json text generated by the verifier.

```
./gevulot-cli --jsonurl http://api.devnet.gevulot.com:9944 get-tx f4f981f594ead6b685cbd46c7c79baa5bdf7f7c4cd4248ec66d9dfd12d08959f  | jq -r '.payload.Verification.verification' | base64 -d

{"is_success":true,"message":"Starkware verifier result: success","proof_file":"/workspace/proof.json","timestamp":1711350602079}
```

And you're done!\\
