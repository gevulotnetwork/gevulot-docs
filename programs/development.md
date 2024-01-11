# Development

It is very easy to develop or modify a program to run in Gevulot as a prover or verifier. Almost any program that can be compiled as an ELF binary for the Linux x86\_64 architecture works, but in order to integrate the program with Gevulot and for it to be able to handle received workloads, it needs to work with the specified [gRPC protocol](https://github.com/gevulotnetwork/gevulot/blob/proto/crates/shim/proto/vm\_service.proto).

### Running environment

The program running environment in a Nanos unikernel is very similar to a normal Linux system, but it does include the following restrictions that originate from either Nanos or Gevulot:

* Not all syscalls are supported. See list of [Nanos supported syscalls.](https://github.com/nanovms/nanos/wiki/Supported-System-Calls)
* There is **no networking** for security reasons.\
  In on-prem environment there is not even a NIC.\
  In cloud environment there is strictly restricted networking between a Gevulot node and the program VM instance (for gRPC connection only).
* No **fork**() support. The whole running environment is single-process only. Multiple threads are supported normally.
* Root filesystem is read-only. Read-write enabled ephemeral volume is mounted in **/workspace**. This is replaced after every Task execution.

### Shim

To ease running of existing programs in Gevulot, there is a helper library called [**shim**](https://github.com/gevulotnetwork/gevulot/tree/proto/crates/shim) that provides some simple functionality to integrate the program with Gevulot.

Shim is written in Rust, but it has [C FFI bindings](https://github.com/gevulotnetwork/gevulot/tree/proto/crates/shim-ffi) to use in programs written in C/C++ or to allow creation of bindings to other programming languages.

### Rust

#### Needed changes for existing programs

To modify an existing program for use in Gevulot using the **shim** library, the following changes are needed:

#### 1. Provide callback function for running a task

Provide callback function for running a task. This could be e.g. a slightly modified existing main() function that parses program arguments from Task.args instead of std::env::args(). The signature for the callback function is the following:

```rust
impl Fn(&Task) -> Result<TaskResult, Box<dyn Error>>
```

#### 2. Create a new main()

Replace the old main() function with a new one that delegates control to **shim**:

```rust
fn main() -> Result<(), Box<dyn Error>> {
    gevulot_shim::run(my_run_task_callback)
}
```

#### Complete example

This is a complete example program in Rust, that can be run inside Gevulot.

```rust
use std::{error::Error, result::Result};

use gevulot_shim::{Task, TaskResult};

fn main() -> Result<(), Box<dyn Error>> {
    gevulot_shim::run(run_task)
}

// The main function that executes the prover program.
fn run_task(task: &Task) -> Result<TaskResult, Box<dyn Error>> {
    // Display program arguments we received. These could be used for
    // e.g. parsing CLI arguments with clap.
    println!("prover: task.args: {:?}", &task.args);

    // -----------------------------------------------------------------------
    // Here would be the control logic to run the prover with given arguments.
    // -----------------------------------------------------------------------

    // Write generated proof to a file.
    std::fs::write("/workspace/proof.dat", b"this is a proof.")?;

    // Return TaskResult with reference to the generated proof file.
    task.result(vec![], vec![String::from("/workspace/proof.dat")])
}
```

### C/C++

#### Gevulot shim

Since gevulot-shim itself is written in Rust, there is a separate **gevulot-shim-ffi** crate that provides C-bindings via FFI.

Compile the FFI crate to get **libgevulot\_shim\_ffi.so** that is then linked into the program binary.

#### Needed changes for existing programs

C/C++ programs are nearly identical with Rust programs regarding the changes needed for existing programs. There are couple small differences in the program flow due to differences in memory management.

#### 1. Include the header file for FFI bindings

[shim.h](https://github.com/gevulotnetwork/gevulot/blob/proto/crates/shim-ffi/shim.h) provides function definitions for **gevulot-shim-ffi**.

#### 2. Provide callback function for running a task

Provide callback function for running a task. This could be e.g. a slightly modified existing main() function that parses program arguments from Task->args instead of \*\*argv. The signature for the callback function is the following:

```c
void *(const struct Task*)
```

Here the return value of **void\*** denotes the TaskResult struct that is fully managed by the **shim** library.

#### 3. Create a new main()

Replace the old main() function with a new one that delegates the control to **shim**:

```c
int main() {
  run(my_run_task_callback);
  return 0;
}

```

#### Complete example

This is a complete example program in C, that can be run inside Gevulot.

```c
#include <stdio.h>

#include "shim.h"

void* compute(const struct Task* task) {
  printf("Received task with id: %s\n", task->id);

  printf("Args: \n");
  const char ** args = (const char**)task->args;
  while ((args != NULL) && (*args != NULL)) {
    printf("\t%s\n", *args);
    args++;
  }

  printf("Files: \n");
  const char **files = (const char**)task->files;
  while ((files != NULL) && (*files != NULL)) {
    printf("\t%s\n", *files);
    files++;
  }

  printf("Done with the task.\n");

  return new_task_result(NULL, 0);
}

int main() {
  printf("Starting example C program in Gevulot...\n");
  run(compute);
  printf("Example program finished. Terminating...\n");
  return 0;
}
```

#### Differences to Rust shim

Due to memory management there are a couple differences to the Rust version in **gevulot-shim-ffi** C/C++ interface:

* TaskResult object is created with **new\_task\_result(data, len)** function.
* Files that are communicated back to Gevulot node, are added with **add\_file\_to\_result(task\_result, file\_name)** function.
