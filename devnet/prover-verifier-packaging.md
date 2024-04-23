# Prover/Verifier Packaging

Gevulot runs provers and verifiers in a virtual machine, under [Nanos](https://nanovms.com/) unikernel. Each program must be correctly packaged in order to work in Gevulot.

### Terminology

**Image** - See **Program Image**

**Manifest** - Manifest here refers to configuration file used to build VM image using Nanos.

**Nanos** - Nanos is the name of the unikernel that provides operating system for the program run in VM.

**Ops** - Ops is a tool that provides various functionalities when working with Nanos.

**Program Image** - Program image is the disk image that contains Nanos kernel and the program.

**VM Image** - See **Program Image**

### Prerequisites

* Install [Ops](https://ops.city/)
* [Gevulot integration](integration.md) has been implemented in the program
* The program has been compiled for Linux x86\_64 target

### Manifest

Nanos is a very lightweight, but full-featured unikernel, that provides the operating system facilities for the prover / verifier program. Due to its unikernel nature, it doesn't have any kinds of init systems that would automatically configure the system for running the user's program. Instead it requires a configuration file, called manifest, to know how the program should be run.

Most of the time the manifest is very simple for a program run in Gevulot, but it does provide wide range of functionality if needed. Certain elements however are mandatory in order to make the VM image work with Gevulot.

#### Example:

{% code title="prover.json" %}
```json
{
  "ManifestPassthrough": {
    "readonly_rootfs": "true"
  },
  "Env":{
    "RUST_BACKTRACE": "1",
    "RUST_LOG": "debug"
  },
  "Program":"prover",
  "Mounts": {
    "%1": "/workspace"
  }
}
```
{% endcode %}

Let's inspect that element by element:

* `ManifestPassthrough` element is a Nanos internal implementation detail that allows passing configuration values straight to kernel.
  * `radonly_rootfs` element informs Nanos kernel that the root filesystem is read-only.
* `Env` element is a map that contains environment variables that are passed to user space program - i.e. the prover / verifier program run under Nanos.
  * `RUST_BACKTRACE` environment variable is Rust specific configuration flag that allows detailed program stack trace in event of `panic().`
  * `RUST_LOG` environment variable can be used to configure `tracing` library's logging functionality. In this example it's set to `debug` level.
* `Program` element describes the name of the program binary that is added into the VM image and which is executed when the Nanos boots.
* `Mounts` element describes the workspace volume mount point. This must be always set as described here, in order to have functional runtime environment for the prover / verifier.

When writing a manifest for your prover / verifier program, the baseline should be always the manifest described above. Those Rust specific environment variables are not mandatory, but can be useful in some debugging scenarios as the Gevulot `shim` is written in Rust.

Most of the time the only modification needed is the `Program` binary name.

#### Detailed configuration options&#x20;

[Ops documentation](https://docs.ops.city/ops/configuration) provides more detailed information on its configuration options, in case the default manifest template is not enough.

### Packaging the program

Once the manifest has been written and the program compiled, the packaging is very straightforward:

```
ops build <program binary file> -c <manifest file>
```

### Complete example

Let's say you have a **my-prover** written in Rust. Once it's built, the resulting binary is in `./target/release/my_prover`.

The program doesn't require any special configuration so it's manifest will look like:

{% code title="my_prover.json" %}
```json
{
  "ManifestPassthrough": {
    "readonly_rootfs": "true"
  },
  "Env":{
    "RUST_BACKTRACE": "1",
    "RUST_LOG": "debug"
  },
  "Program":"my_prover",
  "Mounts": {
    "%1": "/workspace"
  }
}
```
{% endcode %}

The final program image can be built with:

```
ops build ./target/release/my_prover -c my_prover.json
```

Ops will print the resulting program image file location:

```
Bootable image file:/home/johndoe/.ops/images/my_prover
```

### Packaging for GPU-accelerated programs

GPU-accelerated programs require a bit more complex packaging. Documentation for those is coming soon.
