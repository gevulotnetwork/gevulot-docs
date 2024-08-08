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

GPU-accelerated programs require a bit more complex packaging. In order to control correct version of all different components in such a way that the resulting program runs correctly with the GPU acceleration, use following containerized build approach.

#### Containerized build

Use following `Containerfile` as a template for generating the program VM image.

{% code lineNumbers="true" %}
```docker
##############################################################################
# First build Gevulot (and especially the shim & shim-ffi)
FROM rust:bookworm AS build-gevulot

# Install dependencies for Gevulot
RUN apt-get update && apt-get install -y clang cmake curl git libssl-dev pkg-config protobuf-compiler

WORKDIR /build
RUN git clone https://github.com/gevulotnetwork/gevulot.git

WORKDIR /build/gevulot

# Comment this out if you want to build VM image for local run w/ `shim-executor`.
# To build VM image suitable for Devnet, leave following uncommented.
#RUN git checkout optional-file-based-task-to-shim

RUN cargo update -p time && cargo build --release

##############################################################################
# Switch to CUDA container image for building the GPU program
FROM nvidia/cuda:12.3.0-devel-ubuntu22.04

COPY --from=build-gevulot /build /build

RUN apt-get update && apt-get install -y curl git

WORKDIR /build

# Download static assets (Nanos klib, nVidia libraries, GPU firmware).
RUN curl -O https://storage.googleapis.com/gevulot-static-assets/nanos-0.1.51-gpu.tar.gz && tar xfz nanos-0.1.51-gpu.tar.gz --strip-components=1

# Install Ops
RUN curl https://ops.city/get.sh -sSfL | /bin/sh


##############################################################################
# <---------------------------------------------------------------------------
# Insert your program build step here.
# --------------------------------------------------------------------------->
##############################################################################

# e.g. following can be used to build the `deviceQuery` app from `cuda-samples` for Gevulot
#RUN git clone https://github.com/NVIDIA/cuda-samples.git
#ADD cuda-samples.patch .
#RUN cd cuda-samples && git checkout v12.2 && git apply ../cuda-samples.patch && cd Samples/1_Utilities/deviceQuery && make && cp /build/cuda-samples/bin/x86_64/linux/release/deviceQuery /build


##############################################################################
# Finally, build the VM image with GPU assets
##############################################################################

# Prepare directories for shared libraries
WORKDIR /build
RUN mkdir -p lib/x86_64-linux-gnu lib64 

# Move nVidia libraries to VM img's library path 
RUN mv lib*.so.* ./lib/x86_64-linux-gnu

# Copy driver version specific libraries to version-agnostic file
RUN cp /build/lib/x86_64-linux-gnu/libnvidia-ml.so.*.*.* /build/lib/x86_64-linux-gnu/libnvidia-ml.so.1
RUN cp /build/lib/x86_64-linux-gnu/libcuda.so.*.*.* /build/lib/x86_64-linux-gnu/libcuda.so.1

# Copy other dynamic libraries required for executing the program
RUN cp  /usr/lib64/ld-linux-x86-64.so.2 \
	/usr/lib/x86_64-linux-gnu/libc.so.6 \
	/usr/lib/x86_64-linux-gnu/libdl.so.2 \
	/usr/lib/x86_64-linux-gnu/libgcc_s.so.1 \
	/usr/lib/x86_64-linux-gnu/libm.so.6 \
	/usr/lib/x86_64-linux-gnu/libpthread.so.0 \
	/usr/lib/x86_64-linux-gnu/librt.so.1 \
	/build/gevulot/target/release/libgevulot_shim_ffi.so \
	lib/x86_64-linux-gnu

# Copy CUDA libraries
RUN cp /usr/local/cuda-12.*/targets/x86_64-linux/lib/libcudart.so.12.*.* lib/x86_64-linux-gnu && cp /build/lib/x86_64-linux-gnu/libcudart.so.12.*.* /build/lib/x86_64-linux-gnu/libcudart.so.12 && cp /build/lib/x86_64-linux-gnu/libcudart.so.12 /build/lib/x86_64-linux-gnu/libcudart.so

# Declare env variables so that Ops finds its dependencies
ENV OPS_DIR="/root/.ops"
ENV PATH="$PATH:/root/.ops/bin"
ENV LD_LIBRARY_PATH=/build/gevulot/target/release

##############################################################################
# Finally build the VM image
##############################################################################

# XXX: Add your VM image manifest
ADD deviceQuery.manifest .

# XXX: Configure your binary / VM image name and manifest name
RUN ops build deviceQuery --nanos-version 0.1.51 -c deviceQuery.manifest
```
{% endcode %}

If intention is to run the program VM using [`shim-executor`](https://github.com/gevulotnetwork/gevulot-shim) uncomment line **15**.

Lines **36-40** mark the position where the build steps of your program should be added. Ensure you `ADD` required artifacts and install necessary dependencies & tooling for the build. The container is based on Ubuntu 22.04.

To build the final resulting VM image, special manifest must be used so that `ops` includes the nVidia GPU driver and associated firmwares & libraries. Use following manifest as a base and modify it to your needs. Normally you only need to adjust the resource requirements and program binary name. Finally, fix line **87** and **90** with correct manifest file name.

```json
{
  "RebootOnExit": true,
  "RunConfig": {
    "CPUs": 2,
    "Memory": "2g",
    "GPUs": 1
  },
  "KlibDir": "/build",
  "Klibs": ["gpu_nvidia"],
  "Env":{
    "RUST_BACKTRACE": "1",
    "RUST_LOG": "trace"
  },
  "Program":"deviceQuery",
  "Dirs": ["lib", "nvidia"],
  "Files": ["/lib/x86_64-linux-gnu/libdl.so.2", "/lib/x86_64-linux-gnu/libpthread.so.0", "/lib/x86_64-linux-gnu/librt.so.1"],
  "Mounts": {
    "%1": "/workspace"
  }
}

```

Once the resulting program VM image has been built, use following to extract it from the container:

```sh
mkdir target
podman run --rm -it -v ./target:/target:z <CONTAINER ID> -- cp /root/.ops/images/<IMG NAME> /target
```

Target directory now contains the program VM image and it can be deployed to Gevulot.

#### Appendix: Example \`deviceQuery\` program

In order to generate a VM image from the `deviceQuery` program from [`cuda-samples`](https://github.com/NVIDIA/cuda-samples), uncomment the build section from the `Containerfile` above and use following patch that integrates the `shim` into the `deviceQuery` program, allowing it to be executed in Gevulot.

{% file src="../.gitbook/assets/cuda-samples.patch" %}

