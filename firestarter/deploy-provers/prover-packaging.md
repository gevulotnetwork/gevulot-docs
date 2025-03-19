# Prover Packaging

Prover packaging on Firestarter works by converting container image into a Linux-based VM image. Any program that can run in a container, can run on Firestarter.

## Prerequisites

* Linux or MacOS as a host machine
* Install `gvltctl` from [releases](https://github.com/gevulotnetwork/gvltctl/releases/latest)
* Install following programs from your distributions package manager:
  * **Ubuntu:**
    * `git build-essential libncurses-dev gawk flex bison openssl libssl-dev dkms libelf-dev libudev-dev libpci-dev libiberty-dev autoconf llvm bc ca-certificates podman`
  * **Fedora:**
    * gcc
    * make
    * libtool
    * objtool
    * flex
    * bison
    * kernel-devel
    * git
    * podman

## Runtime environment

The VM image contains the Linux kernel, Firestarter-specific init, and the container image file tree that is passed for the `gvltctl build` command.

All the input files are mounted read-only under `/mnt/gevulot/input` which is the only allowed location for files passed in `Task` `inputContexts`.

All the output files are mounted read-write under `/mnt/gevulot/output` which is the only writable location for files that are specified in `Task` `outputContexts` .

## Package CPU-only prover

The first step is to ensure that the prover either has a pre-built container image or a working Containerfile/Dockerfile that can be used to build the container image.

### Container image

Once the prover works in the container, then it can be packaged into a VM image:\
`gvltctl build --container containers-storage:localhost/my_prover:latest -o prover.img`

That command uses container image build with _podman_ locally and packages it into a VM image.

**NOTE:** This command will build Linux kernel from sources, which may take significant amount of time on small machines.

### Containerfile

Building a VM image from a Containerfile works similarly:

`gvltctl build --containerfile Containerfile -o prover.img`

## Package GPU accelerated prover

**NOTE:** For now, only nVidia GPUs are supported.

The GPU-accelerated prover packaging is nearly the same as CPU-only, but there are a couple of additional requirements:

* The source container must have all nVidia runtime libraries present. This is easiest to achieve by using the official `docker.io/nvidia/cuda` as the base image. As of writing, the latest runtime tag is [`12.6.2-runtime-ubuntu24.04`](https://gitlab.com/nvidia/container-images/cuda/blob/master/dist/12.6.2/ubuntu2404/runtime/Dockerfile)
* `--nvidia-drivers` argument specified for `gvltctl build` - this will build nVidia GPU drivers to the VM image, matching the installed kernel

Building the VM image is simple:

`gvltctl build --container containers-storage:localhost/my_gpu_prover:latest --nvidia-drivers -o gpu_prover.img`

## Advanced options for VM image

The `gvltctl build` provides flexible options for customizing the VM image if advanced tuning is needed for some reason. This is not advisable, however, unless you really know what you are doing.

For further information: Refer to `gvltctl build --help` and respective source code for [gvltctl](https://github.com/gevulotnetwork/gvltctl) and [mia](https://github.com/gevulotnetwork/mia).





