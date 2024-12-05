# Tasks

A [Task](https://github.com/gevulotnetwork/gevulot-rs/blob/main/proto/gevulot/gevulot/task.proto#L9-L61) is the basic building block for a program execution on Firestarter. It contains the URL or CID of the program image, the command to execute in VM and the program arguments. It also has fields for defining the input and output files for the program.

Task specific input files are specified as [inputContexts](https://github.com/gevulotnetwork/gevulot-rs/blob/main/proto/gevulot/gevulot/util.proto#L12-L15) where the `source` field contains either a URL or a Firestarter private IPFS network CID.\
\
Input context `target` specifies the file path in the program VM. Its prefix **must** always be `/mnt/gevulot/input`

Task specific output files are specified as [outputContexts](https://github.com/gevulotnetwork/gevulot-rs/blob/main/proto/gevulot/gevulot/util.proto#L17-L20) where the `source` specifies the file path in the program VM. Its prefix **must** always be `/mnt/gevulot/output`. `retentionPeriod` value species how long the file is kept live in the Gevulot network. Unit of `retentionPeriod` is seconds. In general `900` (15 minutes) is a good default.

## Example

Here is a Hello World example task that reads a text input file from an HTTP URL, prints it and writes the system's CPU & RAM information into the output file.

```yaml
kind: Task
version: v0
metadata:
  name: 'Hello I/O task'
  description: 'Task that reads a text file from HTTP, prints it, and writes an output file.'
  tags:
    - hello-world
  labels:
    - key: 'content-type'
      value: 'text/plain'
spec:
  image: 'https://storage.googleapis.com/gevulot-static-assets/hello.img'
  command: ['/app/hello']
  args: ['-input', '/mnt/gevulot/input/input.txt','-output','/mnt/gevulot/output/output.txt']
  env:
    - name: 'DEBUG'
      value: 'true'
  inputContexts:
    - source: 'https://storage.googleapis.com/gevulot-static-assets/input.txt'
      target: '/mnt/gevulot/input/input.txt'
  outputContexts:
    - source: '/mnt/gevulot/output/output.txt'
      retentionPeriod: 3600
  storeStdout: true
  storeStderr: true
  resources:
    cpus: 1
    gpus: 0
    memory: 512
    time: 120

```

The task can be submitted using `gvltctl.` Install the latest `gvltctl` tool from [releases](https://github.com/gevulotnetwork/gvltctl/releases/latest).&#x20;

`gvltctl task create -e "$GEVULOT_ENDPOINT" -n "$GEVULOT_MNEMONIC" -f task.yaml`

The `gvltctl` will then submit a transaction that allocates a dedicated worker for the task on-chain. The output looks roughly as follows:

```
Create task from file task.yaml
message: Task created successfully
status: success
task_id: e870966891918f11f192ccbc383d65a2a39ce5a3e62a82955cb12a06f7972831
```

You can use `gvltctl task get` to fetch the latest version of a task (with the [status](https://github.com/gevulotnetwork/gevulot-rs/blob/main/proto/gevulot/gevulot/task.proto#L31-L61) field):

```
$ gvltctl task get e870966891918f11f192ccbc383d65a2a39ce5a3e62a82955cb12a06f7972831
kind: Task
version: v0
metadata:
  id: e870966891918f11f192ccbc383d65a2a39ce5a3e62a82955cb12a06f7972831
  name: ''
  creator: gvlt1e9knatcs7a9jkj2cf94reldkn2tyqt0m00uqqq
  description: ''
  tags: []
  labels: []
  workflowRef: null
spec:
  image: https://storage.googleapis.com/gevulot-static-assets/hello.img
  command:
  - /app/hello
  args:
  - -input
  - /mnt/gevulot/input/input.txt
  - -output
  - /mnt/gevulot/output/output.txt
  env:
  - name: DEBUG
    value: 'true'
  inputContexts:
  - source: https://storage.googleapis.com/gevulot-static-assets/input.txt
    target: /mnt/gevulot/input/input.txt
  outputContexts:
  - source: /mnt/gevulot/output/output.txt
    retentionPeriod: 3600
  resources:
    cpus: 1
    gpus: 0
    memory: 512
    time: 120
  storeStdout: true
  storeStderr: true
status:
  state: Done
  createdAt: 333108
  startedAt: 333109
  completedAt: 333112
  assignedWorkers:
  - c19f979d53b7e25efb534379905bb47ee52b3917e5672ce775d8571cce832bc3
  - 82658c0a23a2ef0874596695c667f18da3eddc9c0f6d568062f9f46e85479df7
  activeWorker: c19f979d53b7e25efb534379905bb47ee52b3917e5672ce775d8571cce832bc3
  exitCode: 0
  outputContexts:
  - QmfF7B77SF1ehLrohzphWj8YtpXLJ4biGFqJjW6sjPTUNe
  stdout: QmReATRWt5Cya4sCVUXGr9xiuta3StEg5R8WA14ojKNCpm
  stderr: QmbFMke1KXqnYyBBWxB74N4c5SBnJMVAiMNRcGu6x1AwQH
  error: ''

```

In the example above, the task was already executed.

### Task outputs

Taske always have `stdout` and `stderr` output stored in the Firestarter private IPFS network, if the respective `storeStdout` and `storeStderr` are set to `true`. If the task has `outputContexts` specified, the corresponding CIDs can be found from the `status` part as well. The resulting CIDs in `status.outputContexts` are in the same order as specified in the `spec.outputContexts`.

In the above example, the output file produced by the program can be accessed through a read-only IPFS gateway at `https://data.firestarter.gevulot.com/ipfs`

For example above output file can be fetched with:\
`​curl https://data.firestarter.gevulot.com/ipfs/QmfF7B77SF1ehLrohzphWj8YtpXLJ4biGFqJjW6sjPTUNe`

Similarly the stdout and stderr can be fetched with:\
`curl https://data.firestarter.gevulot.com/ipfs/QmReATRWt5Cya4sCVUXGr9xiuta3StEg5R8WA14ojKNCpm`\
and\
`curl https://data.firestarter.gevulot.com/ipfs/QmbFMke1KXqnYyBBWxB74N4c5SBnJMVAiMNRcGu6x1AwQH`





