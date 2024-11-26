# Prover Packaging

Prover packaging on Gevulot Firestarter works by converting container image into a Linux-based VM image. Any program that can run in a container, can run on Gevulot Firestarter.

## Prerequisites

* Linux as a host machine
* `sudo` rights (needed for disk image loopback mounts)
* Install `gvltctl` from [releases](https://github.com/gevulotnetwork/gvltctl/releases)
* Install following programs from your distributions package manager:
  * build-essentials
    * gcc
    * make
    * libtool
    *
  * git
  * podman
  * skopeo
  * syslinux
  * extlinux

## Runtime environment

The VM image contains Linux kernel, Gevulot specific init and the container image file tree that is passed for the `gvltctl build` command.

All the input files are mounted read-only under `/mnt/gevulot/input` which is the only allowed location for files passed in `Task` `inputContexts`.

All the output files are mounted read-write under `/mnt/gevulot/output` which is the only writable location for files that are specified in `Task` `outputContexts` .

## Package CPU-only prover

First step is to ensure that the prover either has pre-built container image or a working Containerfile/Dockerfile that can be used to build the container image.

### Container image

Once the prover works in the container, then it can be packaged into a VM image:\
`gvltctl build --container containers-storage:localhost/my_prover:latest -s 250M -o prover.img`

That command uses container image build with _podman_ locally and packages it into a VM image with 250MB disk.

### Containerfile

Building a VM image from a Containerfile works similarly:

`gvltctl build --containerfile Containerfile -s 250M -o prover.img`

## Package GPU accelerated prover

**NOTE:** For now, only nVidia GPUs are supported.

The GPU accelerated prover packaging is nearly the same as CPU-only, but there are couple additional requirements:

* The source container must have all nVidia runtime libraries present. This is easiest to achieve by using official `docker.io/nvidia/cuda` as base image. As or writing, the latest runtime tag is [`12.6.2-runtime-ubuntu24.04`](https://gitlab.com/nvidia/container-images/cuda/blob/master/dist/12.6.2/ubuntu2404/runtime/Dockerfile)
* `--nvidia-drivers` argument specified for `gvltctl build` - this will build nVidia GPU drivers to the VM image, matching the installed kernel

Building the VM image is simply:

`gvltctl build --container containers-storage:localhost/my_gpu_prover:latest --nvidia-drivers -s 2G -o gpu_prover.img`

fo

## Advanced options for VM image

The `gvltctl build` provides flexible options for customizing the VM image if advanced tuning is needed for some reason. This is not advisable however, unless you really know what you are doing.

For further information: Refer to `gvltctl build --help` and respective source code for [gvltctl](https://github.com/gevulotnetwork/gvltctl) and [mia](https://github.com/gevulotnetwork/mia).





