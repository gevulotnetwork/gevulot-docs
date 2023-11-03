# Overview

Gevulot programs come in two varieties: provers and verifiers. A verifier program can be deployed standalone, while a prover program must always be deployed with an accompanying verifier program (referred to jointly as a proof system). There are no hard constraints on what gevulot programs need to contain, besides that the prover program must output a proof that can be verified by the verifier program for the system to be useful.

Both types of programs can be written in a variety of languages such as Rust, C, C++, etc, and are compiled into [unikernel](https://en.wikipedia.org/wiki/Unikernel) images. Most open source prover implementations can be compiled into unikernel images with minimal or no modification. Examples for Marlin, ED25519 and Filecoin SNARK provers can be found [here](https://github.com/gevulotnetwork/gevulot/tree/main/prover).

Supported languages: C, C++, Go, Java, Node.js, Python, Rust, Ruby and PHP.
